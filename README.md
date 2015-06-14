### tmux-cluster

tmux-cluster is a wrapper script for [tmux](http://tmux.sourceforge.net/) that aims to be fully compatible with [clusterssh](https://github.com/duncs/clusterssh) clusters configs.

tmux-cluster is written in pure shell and only depends on tmux and standard unix utils (grep, sed, awk, cut, etc.).

#### Usage

```
Usage: tmc [OPTIONS] [CLUSTERNAME]

Options:
    -h              Display help message and quit
    -d              Dump space-separated list of hosts in CLUSTERNAME.
                        Conflicts with -t option.
    -t              Dump generated list of tmux commands.
                        Conflicts with -d option.
    -c CLUSTERLINE  Create custom cluster
                        CLUSTERLINE is treated as a clusters config line.
                        This option conflicts with passing CLUSTERNAME.
                        The name used for the cluster must not exist in the
                            clusters config.
    -x EXCLUDES     Space-separated list of hosts to exclude from cluster.
                        The hosts in EXCLUDES will not be connected to.
```

tmux-cluster will look for your clusterssh clusters config at `$HOME/.clusterssh/clusters`. See the [configuration](#configuration) section for more info on the configuration syntax.

A new session will be created with the name `cluster-CLUSTERNAME`. All of the hosts specified by `CLUSTERNAME` will be in a single window with a tiled layout, and the panes will be synchronized.

You can run tmux-cluster either from outside or inside an attached tmux instance. In either case, tmux-cluster will automatically attach your terminal to the newly created cluster session.

Note that if the `-c` option is used, then passing `CLUSTERNAME` is invalid; the `CLUSTERNAME` used is the first element of `CLUSTERLINE`, since `CLUSTERLINE` is treated exactly as a clusters config line.

#### Installation

There are two ways to install tmux-cluster. You can do so manually or through the [tmux plugin manager](https://github.com/tmux-plugins/tpm).

##### Manual installation

# Clone this repository: `git clone https://github.com/davidscholberg/tmux-cluster`
# Optionally copy or symlink the `tmc` script into a directory in your `$PATH`.

##### Installation via tmux plugin manager

# Ensure that [tmux plugin manager](https://github.com/tmux-plugins/tpm) is installed.
# Add `davidscholberg/tmux-cluster` to your list of tmux plugins.
# Optionally configure a tmux keybinding for tmux-cluster using the `@tmux_cluster_prompt_key` option in your `~/.tmux/conf` file.
  * If this option is not specified, `C` is the default.
# To use this plugin, open a tmux-cluster prompt with `prefix + C` (or whatever you specified in `@tmux_cluster_prompt_key`), and type the name of the cluster you want to launch.
# Optionally copy or symlink the `tmc` script into a directory in your `$PATH` if you want to be able to run `tmc` on the command line.

#### Why tmux-cluster?

Question: Why does tmux-cluster exist? There are boatloads of clusterssh tmux wrappers out there already.

Answer: [Configuration](#configuration), [session handling](#session-handling), [error reporting](#error-reporting), and [performance](#performance).

##### Configuration

When one googles "tmux clusterssh", most of the results are for scripts that purport to be clusterssh-like, but they don't actually support clusterssh clusters files.

Clusterssh clusters files have a simple but powerful configuration format that allows one to easily build up hierarchical groupings of servers. Being able to define a cluster and then use that cluster in the definition of other clusters is a huge benefit that many clusterssh-like tmux wrappers lack.

Here is an example clusterssh clusters file:

```
# first cluster, consisting of host1, host2, and host3
cluster1 host1 host2 host3

# second cluster, consisting of all hosts from first cluster as well as host4 and host5
cluster2 cluster1 host4 host5
```

[lowens/tmux-cssh](https://github.com/lowens/tmux-cssh) seems to support clusterssh clusters files, but (as of this writing) it doesn't seem to support using cluster names inside other cluster definitions for building cluster hierarchies.

[dennishafemann/tmux-cssh](https://github.com/dennishafemann/tmux-cssh) does support building cluster hierarchies, but it uses a different configuration format that is not compatible with clusterssh clusters files and is not as straightforward.

##### Session handling

tmux-cluster uses the name of the specified cluster to build the tmux session name, so that you can have multiple simultaneous tmux-cluster sessions open. The name of the session is of the form `cluster-<name>`, where `<name>` is the name of the specified cluster. Because each session name is preceded with `cluster-`, it's easy to see at a glance which tmux-cluster sessions are open since they'll be grouped together alphabetically.

To give a concrete example, lets say you have the following clusters file:


```
cluster1 host1 host2 host3
cluster2 cluster1 host4 host5
```

If you run `tmc cluster1`, then the name of the session that tmux-cluster creates is `cluster-cluster1`. Running `tmc cluster2` creates the session name `cluster-cluster2`.

If you attempt to open a cluster for which there is already an open tmux session, then tmux-cluster will quit gracefully, informing you that the session name is already taken.

##### Error reporting

tmux-cluster will alert you of every single host that failed to connect by holding open the panes that contain the failed connections until you press enter. This way, you can easily see which hosts failed instead of trying to figure it out manually by comparing open panes with the list of hosts that are supposed to be in the cluster.

##### Performance

tmux-cluster should perform faster than most of the clusterssh tmux wrappers out there because of how tmux-cluster passes tmux commands to tmux. Most other clusterssh tmux wrappers make individual calls to tmux for every single tmux command that needs to be run. tmux-cluster, on the other hand, makes a list of native tmux commands first, places this list of commands into a temporary file, and then passes these commands to tmux with a single call to tmux's `source-file` command. This makes tmux-cluster very fast, even if you have much more than a handful of hosts in a particular cluster.

#### TODO

* Add usage examples to README.
* Add ability to resolve cluster names in the EXCLUDES list.
* Add command line option to start a local login shell after ssh exits.
* Add command line option to exclude current host from cluster.
* Add command line option to specify unique suffix to append to session name to allow multiple sessions of the same cluster. Could default to appending 4 digit number.
* Add ability to specify multiple clusters on command line, each one creating a different session.
* Add command line option to specify a different pane layout to use instead of the default `tiled` layout.
* Add command line option to disable pane synchronization.
* Add command line option to specify path to clusterssh config.
* Make tmux-cluster match clusterssh's behavior of using the "default" cluster if no cluster is specified.
* Add command line option to specify path to custom ssh config.
