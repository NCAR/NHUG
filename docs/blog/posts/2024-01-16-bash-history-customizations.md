---
draft: false
authors:
  - benkirk
date: 2024-01-16
categories:
  - Example
  - Tips
  - Post
---

# Unique `bash` `history` on different HPC resources

NCAR's HPC resource share a common home files system, which greatly simplifies working across systems.  One slight annoyance I noted with this setup, however, is a common `bash` history across all machines.  While the commands I usually run on *Casper* and *Derecho* are very similar, they are not identical.  Fortunately, `bash` allows its `history` behavior to be customized easily, and with just a little effort we can get HPC resource-specific `history` implementations.

<!-- more -->

---

## Customizing your `bash` history behavior

By default, `bash` stores its `history` in the file `~/.bash_history`, so shells on different machines will share the same history file.  It's really easy to change this behavior with just a little shell customization.

The following snippet is lifted from my `~/.profile`, where I do all my [shell customization](https://ncar-hpc-docs.readthedocs.io/en/latest/environment-and-software/user-environment/customizing/):

### Unique `HISTFILE` per NCAR resource type

```bash title="~/.profile" linenums="1"
# increase the number of lines stored in our history file
# (https://www.gnu.org/software/bash/manual/html_node/Bash-History-Facilities.html)
export HISTSIZE=1024

# customize history behavior
# (https://www.bashsupport.com/bash/variables/bash/histcontrol/)
export HISTCONTROL=ignorespace:erasedups

# Machine-specific history file
case "x${NCAR_HOST}" in
    "x") ;;
    *) export HISTFILE=$HOME/.bash_history.${NCAR_HOST}; ;;
esac
```

The first two `#!bash export` statements modify the default behavior of the `history` built-in command, as described in the references.

The `#!bash case` switch statement on lines 10-13 examines the value of the `NCAR_HOST` environment variable and (if set) uses it to create a unique `HISTFILE`.  `NCAR_HOST` is set by all our `ncarenv` modules, so will exist on *Derecho* and *Casper* login and compute nodes.

With the configuration above, all my `casper` and `derecho` commands stay put and each machine has its own distinct `history`!

---

### Bonus: search behavior customization
`bash` used the [GNU readline](https://tiswww.case.edu/php/chet/readline/rltop.html) library for interacting with the user, which allows for a lot of behavior customization.  One feature I found that I like to use under `bash` is:

- `history-search-forward ()`

    Search forward through the history for the string of characters between the start of the current line and the point. The search string must match at the beginning of a history line.

- `history-search-backward ()`

    Search backward through the history for the string of characters between the start of the current line and the point. The search string must match at the beginning of a history line.

which can be enabled via:

```bash title="~/.profile"
# substring search history
bind '"\e[A"':history-search-backward
bind '"\e[B"':history-search-forward
```

What this means is, say I type `$ qs` at a terminal prompt and then hit the "up" arrow, my shell will cycle through my history for any commands beginning with `qs*`, e.g. `qsub`, `qstat`.

For other options (and there are many...), see [this page](https://www.gnu.org/software/bash/manual/html_node/Commands-For-History.html).

<!--  LocalWords:  benkirk HPC NCAR's Casper Derecho HISTFILE NCAR hl linenums HISTSIZE HISTCONTROL ignorespace esac erasedups ncarenv casper derecho readline
 -->
