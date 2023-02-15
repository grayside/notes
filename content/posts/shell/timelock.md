---
title: "Timelock"
date: "2023-02-10T11:45:44-08:00"
author: "grayside"
authorTwitter: "" #do not include @
cover: ""
tags: ["til", "shell", "sentinel", "productivity"]
keywords: ["", ""]
description: "Run a script no more than every 24 hours."
showFullContent: false
color: "blue" # ["orange", "blue", "red", "green", "pink"]
---

tl;dr: Script to prevent another script from running more often than every 24 hours.

Some types of local developer environment operations should happen on a regular
basis but don't fit into cron. Things like updating [homebrew dependencies](https://brew.sh/).

Some reasons things may not be a good fit for cron:

* You want to see what will/has happened in detail when it happens
* You don't want to remember how to investigate when cron operations fail

Today's solution? Run these tasks automatically after something else I know I will do,
such as loading my `.bash_profile` when I open a new terminal session.

## Timelock (Original)

My timelock script wraps is shell executable argument in checks to prevent them
from being run more often than every 24 hours. Time is configurable by the
`TIMELOCK_DELAY_SECONDS` environment variable.

```bash
#!/usr/bin/env bash
set -eu

##
# timelock.sh
#
# Run supplied command no more than every 24 hours.
#
# Usage:
#  timelock.sh ~/bin/brew-install.sh
#  VERBOSE=1 timelock.sh ~/bin/brew-install.sh

TIMELOCK=${TIMELOCK_DELAY_SECONDS:-86400}

# The operation being run is the first argument sent to the timelock.
operation=$(basename $1)
store="$HOME/.config/$(whoami)-bin/$operation.last-run"
mkdir -p $(dirname "$store")

last_run=$(cat "$store" 2>/dev/null || echo 0)
now=$(date +%s)
elapsed=$((now-last_run))

if (( "$elapsed" < "$TIMELOCK" )); then
  lock_duration=$(eval "echo $(gdate -ud "@$TIMELOCK" +'$((%s/3600/24)) days %H hours %M minutes %S seconds')")
  last_run_date=$(gdate -u -d @"$last_run")

  [[ $(type -t vlog) == function ]] && vlog "timelocked (${operation}): Waiting until ${lock_duration} hours after ${last_run_date} before running"
  exit 0
fi

if (( "$last_run" > 0 )); then
  elapsed_duration=$(eval "echo $(gdate -ud "@$elapsed" +'$((%s/3600/24)) days %H hours %M minutes %S seconds')") 
  echo "timelocked (${operation}): Running script (${elapsed_duration} hours since last run)"
else
  echo "timelocked (${operation}): Running script for the first time"
fi

"$@"

# Update latest run time.
echo "$now" > "$store"
```

`vlog` is a custom function in my `.bashrc`. It looks like this:

```bash
vlog() {
  test -z "$VERBOSE" || echo "[v] $@"
}

# Make vlog available in my scripts.
export -f vlog
```

## What it looks like

```txt
$> timelock.sh echo "hello"
timelocked (echo): Running script (0 days 00 hours 18 minutes 48 seconds hours since last run)
hello

$> VERBOSE=1 timelock.sh echo "hello"
[v] timelocked (echo): Waiting until 1 days 00 hours 00 minutes 00 seconds hours after Tue Feb 14 05:04:15 UTC 2023 before running
```

## Limitations, Constraints, & Considerations

1. The "[sentinel file](https://logicgrimoire.wordpress.com/2015/05/05/the-sentinel-file-pattern-3/)"
   groups commands together by the first argument.

   ```bash
   operation=$(basename $1)
   store="$HOME/.config/$(whoami)-bin/$operation.last-run"
   ```

   I use this to wrap custom scripts kept in a single directory, so I'm not
   concerned with multiple scripts with the same filename or commands with
   different arguments.

2. If the timelock is active, the default is to do nothing and report nothing.
   The custom verbose logging function (vlog) allows simple debug output if
   I don't trust the silence. (Or would, except it seems broken).

3. We're not using `exec` to run the command passed to timelock, so timelock
   variables will be available in the called script. On the other hand, when
   successful the script passes control back to timelock which can update the
   sentinel.

4. I'm writing the date to the file because it's mildly less cryptic, but at
   scale better to save the redundant storage:

   * Create or update the sentinel: `touch "$store"`
   * Read the sentinel: `date -r "$store" +%s`
