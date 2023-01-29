+++
title = "Fighting the Forward Slashes with Go's MapFS"
date = "2023-01-28T11:34:34-08:00"
# author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["til", "golang", "testing", "bug"]
keywords = ["golang", "testing"]
description = "Avoiding Go testing/fstest.MapFS crashes on absolute paths."
showFullContent = false
hideComments = false
color = "orange" #["orange", "blue", "red", "green", "pink"]
framed = true
+++

Today I'm recounting my adventures in using MapFS to write unit tests for code
that walks filesystems defined using io/fs.FS. MapFS is easy to start with, but
there is a rough edge if you try to define absolute paths in your map.

I assumed I should use absolute paths. When my tests started to segfault, it was
very confusing.

I found [go/issues/34591](https://github.com/golang/go/issues/34591) which seems
to describe the same problem, but is not about the fstest package implementation.

## Solution 1: Ubiquitous Warning

To make sure I don't forget this problem, preface each MapFS instantiation
with this comment:

```
// WARNING: Paths that start with leading slashes give stack overflows.
// Closest issue I've tracked down: https://github.com/golang/go/issues/34591
```

I really appreciate well explained code, but it's better for orienting new
developers than reminding yourself of a problem: I will get used to not reading
this comment, and that habit will probably stick with me longer than my recollection
of the problem.

## Solution 2: Validate the Map

You could implement a constructor or validation function like
`validateMapFS(fstest.MapFS) error`, but this doesn't seem that much better than
the comment and requires just as much remembering to take an extra step.

Since a MapFS is a `map`, this might look like:

```go
func validateMapFS(m fstest.MapFS) error {
    for k, _ := range m {
        if k[0] == '/' {
            return errors.New("do not use absolute paths in MapFS, bad key " + k)
        }
    }

    return nil
}
```

## Solution 3: Test Case Utility

If I need to remember to do something, maybe the answer is to make it valuable
in more than one way. My goal in using MapFS is not using MapFS. My goal is to
define test cases for code that only cares about filesystem contents.

So what if I go back to writing my tests with clean table tests and use a helper
to create the MapFS my code expects to receive?

```go
package testhelper

import (
	"io/fs"
	"testing"
	"testing/fstest"
)

type TestCase struct {
	path    string
	data    string
	want    string
}

func NewMapFSFromTestCases(t *testing.T, testCases []TestCase) fs.FS {
	t.Helper()
	fsys := make(fstest.MapFS, len(testCases))

	for _, tc := range testCases {
		if len(tc.path) == 0 {
			t.Fatalf("no path specified for want %q", tc.want)
			return nil
		}

		if string(tc.path[0]) == "/" {
			t.Fatal("Invalid fstest.MapFS path: cannot start with leading slash. This is not documented as part of fstest.MapFS. See https://github.com/golang/go/issues/34591")
			return nil
		}

		fsys[tc.path] = &fstest.MapFile{
			Data: []byte(tc.data),
		}
	}

	return fsys
}
```

In my test code, I define my test cases, call this function to generate the
filesystem input for the function under test, then proceed with testing.

## Nest Steps

- [ ] File a bug

## Background References

* **Packages:** [testing/fstest](https://pkg.go.dev/testing/fstest)
* **Useful Article:** [Walking with filesystems: Go's new fs.FS interface](https://bitfieldconsulting.com/golang/filesystems)
