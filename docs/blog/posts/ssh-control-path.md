---
draft: false
authors:
  - benkirk
date: 2024-01-09
categories:
  - Example
  - Tips
  - Post
---

# Streamlining two-factor authentication with `ssh`

Do you tire of responding to Duo pushes every single time you log in to NCAR's HPC resources?  While two-factor authentication (2FA) is a hard requirement for user access, there are techniques for "sharing" authentication, allowing multiple `ssh` or `scp` processes to reuse a pre-established connection.  This technique uses the `ControlPath`  and `ControlMaster` functionality built in to `ssh` and requires some straightforward local configuration.

<!-- more -->

---
`ControlPath` and `ControlMaster` are `ssh` client options that work together to a socket file which allow an initial `ssh` connection to a host be reused and optionally persist after the initial session has disconnected. In practice this means the first connection to a host will be authenticated normally, but subsequent connections will simply reuse the initial connection without requiring additional authentication.



## Local configuration
Nothing special is required on the HPC systems to enable this functionality, however you will need to perform some local configuration of your `ssh` client.  In the example below I'm using `OpenSSH` on my Macbook laptop, and all that is required is editing the `config` file in my `~/.ssh` directory:

```pre title="~/.ssh/config" linenums="1"  hl_lines="7-12"
host casper
     HostName casper.ucar.edu

host derecho
     HostName derecho.hpc.ucar.edu

host *
    ControlMaster auto
    ControlPath ~/.ssh/controlpath-%r@%h:%p
    ControlPersist 12h
    ServerAliveInterval 120s
    ServerAliveCountMax 5
```

The first two sections are simply convenience: They define aliases for the host names `casper` and `derecho` to their fully-qualified domain names.  The idea then is I can `ssh casper` without needing to use the full host and domain name.

The final section (`host *`) contains the specific configuration of interest:

- `ControlPath ~/.ssh/controlpath-%r@%h:%p` specifies the path to a control socket used for connection sharing.  This makes use of some special tokens:
     - `%r` : username on the remote system,
     - `%h` : the remote system host name, and
     - `%p` : the remote port.

     This allows for a unique `ControlPath` file to be placed in our `~/.ssh/` directory for each unique user/host/port combination.

- `ControlMaster auto` enables "opportunistic `ssh` multiplexing," meaning `ssh` will attempt to use an existing established connection if possible, and establish a new one if required.
- `ControlPersist 12h` specifies that the controlling connection should remain open in the background (waiting for future client connections) after the initial client connection has been closed.  **Without this option, closing a controlling `ssh` session will abruptly terminate any other active, shared authentication connections.**
- The final two lines, `ServerAliveInterval 120s` and `ServerAliveCountMax 5`, are useful for maintaining `ssh` connections.

    When our client is idle, eventually we will be disconnected from the server.  These settings send very minimal traffic at intervals even when we are not actively using `ssh`, triggering the server to respond.  If the server does not respond after `ServerAliveCountMax` steps, our client will finally give up and disconnect.

    See [here](https://www.baeldung.com/linux/ssh-keep-alive) for additional discussion.

Your `~/.ssh/config` file supports many, many more options.  See [here](https://man.openbsd.org/ssh_config) for additional details.

## Demonstration

To see how these pieces work together, consider the following examples:

!!! example "Connecting to `casper` through a new `ControlPath` & examining the mechanics of the process"
    ```console linenums="1"
    ssh-client(1)$ ls ~/.ssh/control*
    ls: /Users/benkirk/.ssh/control*: No such file or directory

    ssh-client(2)$ ssh casper uptime
    (benkirk@casper.hpc.ucar.edu) ncar-two-factor:
     07:53:13  up 22 days 12:30,  89 users,  load average: 7.74, 7.87, 8.09

    ssh-client(3)$ file ~/.ssh/control*
    /Users/benkirk/.ssh/controlpath-benkirk@casper.hpc.ucar.edu:22: socket

    ssh-client(4)$ ps aux | grep "ssh: " | grep -v grep | awk '{print $11 " " $12 " " $13}'
    ssh: /Users/benkirk/.ssh/controlpath-benkirk@casper.hpc.ucar.edu:22 [mux]

    ssh-client(5)$ ssh casper uptime
     07:53:34  up 22 days 12:31,  87 users,  load average: 7.66, 7.85, 8.08
    ```

    ---

    **Detailed Discussion**

      1. For demonstration purposes, we begin on a quiet client with no existing `ControlPath` instances (line 1).
      2. On line 4 we start a new `ssh` session to `casper` to execute a remote command (`uptime`). You can see from the `ncar-two-factor:` prompt we are required to two-factor authenticate with Duo, as usual.

         Also, note that we were able to reference the short host name `casper` since our `~/.ssh/config` has this aliased to `casper.hpc.ucar.edu`.
      3. Lines 8-9 show that `ssh` has now automatically created the `~/.ssh/controlpath-benkirk@casper.hpc.ucar.edu:22` socket for us, as specified by the `ControlPath` configuration.
      4. Lines 11-12 demonstrate the impact of the `ControlPersist` option.  Even though we are not currently using `ssh` (our connection in step 2 has terminated), we still have an `ssh` process active in the background referencing our `ControlPath` file.  This process keeps the connection active and ready to be reused by other processes, up to the `ControlPersist` timeout.
      5. In line 14 we repeat the `ssh casper uptime` command, and no 2FA is required!


The same process applies when adding a new connection to *Derecho* as seen in the following example.

!!! example "Additionally connecting to `derecho` through another `ControlPath`"
    ```console linenums="1"
    ssh-client(6)$ ssh derecho "uname -a"
    Access to and use of this UCAR computer system is limited to authorized use by
    UCAR Policies 1-7 and 3-6 and all applicable federal laws, executive orders,
    policies and directives. UCAR computer systems are subject to monitoring at all
    times to ensure proper functioning of equipment and systems including security
    devices, to prevent unauthorized use and violations of statutes and security
    regulations, to deter criminal activity, and for other similar purposes.

    Users should be aware that information placed in the system is subject to
    monitoring and is not subject to any expectation of privacy. Unauthorized use
    or abuse will be dealt with according to UCAR Policy, up to and including
    criminal or civil penalties as warranted.

    By logging in, you are agreeing to these terms.

    (benkirk@derecho.hpc.ucar.edu) ncar-two-factor:
    Linux derecho4 5.14.21-150400.24.18-default #1 SMP PREEMPT_DYNAMIC Thu Aug 4 14:17:48 UTC 2022 (e9f7bfc) x86_64 x86_64 x86_64 GNU/Linux

    ssh-client(7)$ file ~/.ssh/control*
    /Users/benkirk/.ssh/controlpath-benkirk@casper.hpc.ucar.edu:22:  socket
    /Users/benkirk/.ssh/controlpath-benkirk@derecho.hpc.ucar.edu:22: socket

    ssh-client(8)$ ps aux | grep "ssh: " | grep -v grep | awk '{print $11 " " $12 " " $13}'
    ssh: /Users/benkirk/.ssh/controlpath-benkirk@derecho.hpc.ucar.edu:22 [mux]
    ssh: /Users/benkirk/.ssh/controlpath-benkirk@casper.hpc.ucar.edu:22 [mux]

    ssh-client(9)$ ssh derecho "uname -a"
    Linux derecho4 5.14.21-150400.24.18-default #1 SMP PREEMPT_DYNAMIC Thu Aug 4 14:17:48 UTC 2022 (e9f7bfc) x86_64 x86_64 x86_64 GNU/Linux
    ```

    ---

    **Detailed Discussion**

      1. In lines 1-16 we similarly start a new `ssh` session to `derecho` to execute a remote command (`uname -a`), using the short host name alias.
      2. Lines 19-25 demonstrate that we now have a `ControlPath` and persistent connection to *Derecho* as well.
      3. Lines 27-28 show that subsequent `ssh` connections do not require 2FA. Success!!


In the examples above we have used `ssh` to run a remote command, but the same functionality applies with terminal sessions, and extends to `scp` and `sftp` as well.

**Happy `ssh`'ing!!**

<!--  LocalWords:  derecho ControlMaster ControlPath ControlPersist ServerAliveInterval ServerAliveCountMax casper linenums uptime ncar -->
