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

```pre title="~/.ssh/config" linenums="1"
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

---

Your `~/.ssh/config` file supports many, many more options.  See [here](https://man.openbsd.org/ssh_config) for additional details.

## Demonstration

<!--
### Utility shell functions
```pre
#########################################
# ssh
#########################################
casper_ssh_control_agent()
{
    rm -f /tmp/ssh-controlpath-*casper*
    echo "Starting an ssh control path connection for casper"
    ssh -YMNf benkirk@casper.ucar.edu
}
derecho_ssh_control_agent()
{
    rm -f /tmp/ssh-controlpath-*derecho*
    echo "Starting an ssh control path connection for derecho"
    ssh -YMNf benkirk@derecho.hpc.ucar.edu
}
ncar_ssh_control_agents()
{
    casper_ssh_control_agent
    derecho_ssh_control_agent
}
```
-->

<!--  LocalWords:  derecho
 -->
