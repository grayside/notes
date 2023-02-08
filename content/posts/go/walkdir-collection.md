---
title: "Collecting Data while walkng a filesystem with Go"
date: "2023-02-06T09:18:54-08:00"
author: "grayside"
authorTwitter: "" #do not include @
cover: ""
tags: ["til", "golang", "lexical-scope", "filesystem"]
keywords: ["", ""]
description: "Using `fs.WalkDir` to collect data."
showFullContent: false
color: "" # ["orange", "blue", "red", "green", "pink"]
---

There are several ways to use [`fs.WalkDir`](https://pkg.go.dev/io/fs#WalkDir)
to collect data.

## Basic `fs.WalkDir`

Here's a relatively simple WalkDir to print all paths that start with "magic/".

```go
func ImportantBusinessFunc(fsys fs.FS) (error) {
    err := fs.WalkDir(fsys, ".", func(path string, dir fs.DirEntry, err error) error {
        if err != nil {
            return err
        }
        
        if !strings.HasPrefix(path, "magic/") {
			return nil
		}

		fmt.Println(path)

		return nil
    })
}
```

This doesn't help because I want to collect those paths, not just print them out.

## Add Collector Slice

Assemble data by instantiating a collector that we can add to as we walk.

(WalkDir is not concurrent.)

```go
func ImportantBusinessFunc(fsys fs.FS) ([]string, error) {
    s := []string{}
    err := fs.WalkDir(fsys, ".", func(path string, dir fs.DirEntry, err error) error {
        if err != nil {
            return err
        }
        
        if !strings.HasPrefix(path, "magic/") {
			return nil
		}

		s = append(s, path)

		return nil
    })

    return s, err
}
```

The function passed as a parameter to `fs.WalkDir` is a closure, and lexical scoping
means variables defined in ImportantBusinessFunc are available inside contained closures.

## Pointer to a Collection

Suppose we want the file walking to be more reusable?

```go
func ImportantBusinessFunc(fsys fs.FS) ([]string, error) {
    s := []string{}
    err := _importantBusiness(fsys, &s, "magic/")

    return s, err
}

func _importantBusiness(fsys fs.FS, collector *[]string, needle string) error {
    return fs.WalkDir(fsys, ".", func(path string, dir fs.DirEntry, err error) error {
        if err != nil {
            return err
        }
        
        if !strings.HasPrefix(path, needle) {
			return nil
		}

		collector = append(collector, path)

		return nil
    })
}
```
