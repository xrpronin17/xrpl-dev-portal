---
html: commandline-usage.html
parent: http-websocket-apis.html
blurb: Commandline usage options for the rippled server.
curated_anchors:
  - name: Available Modes
    anchor: "#available-modes"
  - name: Daemon Mode Options
    anchor: "#daemon-mode-options"
  - name: Stand-Alone Mode Options
    anchor: "#stand-alone-mode-options"
  - name: Client Mode Options
    anchor: "#client-mode-options"
  - name: Unit Tests
    anchor: "#unit-tests"
labels:
  - Core Server
---
# rippled Commandline Usage Reference

The <span class="code-snippet">rippled</span> executable usually runs as a daemon that powers the XRP Ledger, although it can also run in other modes. This page describes all the options you can pass to <span class="code-snippet">rippled</span> when running it from the command line.

## Available Modes

- **Daemon Mode** - The default. Connect to the XRP Ledger to process transactions and build a ledger database.
- **Stand-Alone Mode** - Use the <span class="code-snippet">-a</span> or <span class="code-snippet">--standalone</span> option. Like daemon mode, except it does not connect to other servers. You can use this mode to test transaction processing or other features.
- **Client Mode** - Specify an API method name to connect to another <span class="code-snippet">rippled</span> server as a JSON-RPC client, then exit. You can use this to look up server status and ledger data if the executable is already running in another process.
- **Other Usage** - Each of the following commands causes the <span class="code-snippet">rippled</span> executable to print some information, then exit:
    - **Help** - Use <span class="code-snippet">-h</span> or <span class="code-snippet">--help</span> to print a usage statement.
    - **Unit Tests** - Use <span class="code-snippet">-u</span> or <span class="code-snippet">--unittest</span> to run unit tests and print a summary of results. This can be helpful to confirm that you have compiled <span class="code-snippet">rippled</span> successfully.
    - **Version statement** - Use <span class="code-snippet">--version</span> to have <span class="code-snippet">rippled</span> print its version number, then exit.

## Generic Options

These options apply to most modes:

| Option          | Description                                                |
|:----------------|:-----------------------------------------------------------|
| <span class="code-snippet">--conf {FILE}</span> | Use <span class="code-snippet">{FILE}</span> as the config file instead of looking for config files in the default locations. If not specified, <span class="code-snippet">rippled</span> first checks the local working directory for a <span class="code-snippet">rippled.cfg</span> file. On Linux, if that file is not found, <span class="code-snippet">rippled</span> next checks for <span class="code-snippet">$XDG_CONFIG_HOME/ripple/ripple.cfg</span>. (Typically, <span class="code-snippet">$XDG_CONFIG_HOME</span> maps to <span class="code-snippet">$HOME/.config</span>.) |

### Verbosity Options

The following generic options affect the amount of information written to standard output and log files:

| Option      | Short Version | Description                                    |
|:------------|:--------------|:-----------------------------------------------|
| <span class="code-snippet">--debug</span>   |               | **DEPRECATED** Enables trace-level debugging (alias for <span class="code-snippet">--verbose</span>). Use the [log_level method][] instead. |
| <span class="code-snippet">--silent</span>  |               | Don't write logs to standard out and standard error during startup. Recommended when starting <span class="code-snippet">rippled</span> as a systemd unit to reduce redundant logging. |
| <span class="code-snippet">--verbose</span> | <span class="code-snippet">-v</span>          | **DEPRECATED** Enables trace-level debugging. Use the [log_level method][] instead. |



## Daemon Mode Options

```bash
rippled [OPTIONS]
```

Daemon mode is the default mode of operation for <span class="code-snippet">rippled</span>. In addition to the [Generic Options](#generic-options), you can provide any of the following:

| Option              | Description                                            |
|:--------------------|:-------------------------------------------------------|
| <span class="code-snippet">--fg</span>              | Run the daemon as a single process in the foreground. Otherwise, <span class="code-snippet">rippled</span> forks a second process for the daemon while the first process runs as a monitor. |
| <span class="code-snippet">--import</span>          | Before fully starting, import ledger data from another <span class="code-snippet">rippled</span> server's ledger store. Requires a valid <span class="code-snippet">[import_db]</span> stanza in the config file. |
| <span class="code-snippet">--net</span>             | **DEPRECATED** Intended for debugging: do not build a local ledger until one can be obtained from the network. |
| <span class="code-snippet">--newnodeid</span>       | Generate a random node identity for the server. |
| <span class="code-snippet">--nodeid {VALUE}</span>  | Specify a node identity. <span class="code-snippet">{VALUE}</span> can also be a parameter associated with the container or hardware running the server, such as <span class="code-snippet">$HOSTNAME</span>. |
| <span class="code-snippet">--nodetoshard</span>     | Before fully starting, copy any complete [history shards](history-sharding.html) from the ledger store into the shard store, up to the shard store's configured maximum disk space. Uses large amounts of CPU and I/O. Caution: this command copies data (instead of moving it), so you must have enough disk space to store the data in both the shard store and the ledger store. <!--{# Task for writing a tutorial to use this: DOC-1639 #}--> |
| <span class="code-snippet">--quorum {QUORUM}</span> | This option is intended for starting [test networks](parallel-networks.html). Override the minimum quorum for validation by requiring an agreement of <span class="code-snippet">{QUORUM}</span> trusted validators. By default, the quorum for validation is automatically set to a safe number of trusted validators based on how many there are. If some validators are not online, this option can allow progress with a lower than normal quorum. **Warning:** If you set the quorum manually, it may be too low to prevent your server from diverging from the rest of the network. Only use this option if you have a deep understanding of consensus and have a need to use a non-standard configuration. |

The following option has been removed: <span class="code-snippet">--validateShards</span>. [Removed in: rippled 1.7.0][]

## Stand-Alone Mode Options

```bash
rippled --standalone [OPTIONS]
rippled -a [OPTIONS]
```
Run in [stand-alone mode](rippled-server-modes.html). In this mode, <span class="code-snippet">rippled</span> does not connect to the network or perform consensus. (Otherwise, <span class="code-snippet">rippled</span> runs in daemon mode.)

### Initial Ledger Options

The following options determine which ledger to load first when starting up. These options are intended for replaying historical ledgers or starting test networks.

| Option                | Description                                          |
|:----------------------|:-----------------------------------------------------|
| <span class="code-snippet">--ledger {LEDGER}</span>   | Load the ledger version identified by <span class="code-snippet">{LEDGER}</span> (either a ledger hash or a ledger index) as the initial ledger. The specified ledger version must be in the server's ledger store. |
| <span class="code-snippet">--ledgerfile {FILE}</span> | Load the ledger version from the specified <span class="code-snippet">{FILE}</span>, which must contain a complete ledger in JSON format. For an example of such a file, see the provided [<span class="code-snippet">ledger-file.json</span>]({{target.github_forkurl}}/blob/{{target.github_branch}}/content/_api-examples/rippled-cli/ledger-file.json). |
| <span class="code-snippet">--load</span>              | **DEPRECATED** Intended for debugging. Only load the initial ledger from the ledger store on disk. |
| <span class="code-snippet">--replay</span>            | Intended for debugging. Use with <span class="code-snippet">--ledger</span> to replay a ledger close. Your server must have the ledger in question and its direct ancestor already in the ledger store. Using the previous ledger as a base, the server processes all the transactions in the specified ledger, resulting in a re-creation of the specified ledger. With a debugger, you can add breakpoints to analyze specific transaction processing logic. |
| <span class="code-snippet">--start</span>             | Intended for debugging. Start with a new genesis ledger that has all known amendments (except those the server is configured to vote against) enabled. This makes the functionality of those amendments available right away, instead of needing to wait two weeks for the [Amendment Process](amendments.html). |
| <span class="code-snippet">--valid</span>             | **DEPRECATED** Intended for debugging. Consider the initial ledger a valid network ledger even before fully syncing with the network. |

## Client Mode Options

```bash
rippled [OPTIONS] -- {COMMAND} {COMMAND_PARAMETERS}
```

In client mode, the <span class="code-snippet">rippled</span> executable acts as a client to another <span class="code-snippet">rippled</span> service. (The service may be the same executable running in a separate process locally, or it could be a <span class="code-snippet">rippled</span> server on another server.)

To run in client mode, provide the [commandline syntax](request-formatting.html#commandline-format) for one of the [<span class="code-snippet">rippled</span> API](http-websocket-apis.html) methods.

Besides the individual commands, client mode accepts the [Generic Options](#generic-options) and the following options:

| Option                  | Description                                        |
|:------------------------|:---------------------------------------------------|
| <span class="code-snippet">--rpc</span>                 | Explicitly specify that the server should run in client mode. Not required. |
| <span class="code-snippet">--rpc_ip {IP_ADDRESS}</span> | Connect to the <span class="code-snippet">rippled</span> server at the specified IP Address, optionally including a port number. |
| <span class="code-snippet">--rpc_port {PORT}</span>     | **DEPRECATED** Connect to the <span class="code-snippet">rippled</span> server on the specified port. Specify the port alongside the IP address using <span class="code-snippet">--rpc_ip</span> instead. |

**Tip:** Some arguments accept negative numbers as values. To ensure that arguments to API commands are not interpreted as options instead, pass the <span class="code-snippet">--</span> argument before the command name.

Example usage (get account transaction history from the earliest available to latest available ledger versions):

```bash
rippled -- account_tx r9cZA1mLK5R5Am25ArfXFmqgNwjZgnfk59 -1 -1
```


## Unit Tests

```bash
rippled --unittest [OPTIONS]
rippled -u [OPTIONS]
```

Unit testing runs tests built into the <span class="code-snippet">rippled</span> source code to confirm that the executable performs as expected. After running unit tests, the process displays a summary of results and exits. Unit tests cover functionality such as built-in data types and transaction processing routines.

If unit testing reports a failure, that generally indicates one of the following:

- A problem occurred when compiling <span class="code-snippet">rippled</span> and it is not functioning as intended
- The source code for <span class="code-snippet">rippled</span> contains a bug
- A unit test has a bug or has not been updated to account for new behavior

While running unit tests, you can specify the [Generic Options](#generic-options) and any of the following options:

| Option                             | Short Version | Description             |
|:-----------------------------------|:--------------|:------------------------|
| <span class="code-snippet">--unittest-ipv6</span>                  |               | Use [IPv6](https://en.wikipedia.org/wiki/IPv6) to connect to the local server when running unit tests. If not provided, unit tests use IPv4 instead. [New in: rippled 1.1.0][] |
| <span class="code-snippet">--unittest-jobs {NUMBER_OF_JOBS}</span> |               | Use the specified number of processes to run unit tests. This can finish running tests faster on multi-core systems. The <span class="code-snippet">{NUMBER_OF_JOBS}</span> should be a positive integer indicating the number of processes to use. |
| <span class="code-snippet">--unittest-log</span>                   |               | Allow unit tests to write to logs even if <span class="code-snippet">--quiet</span> is specified. (No effect otherwise.) |
| <span class="code-snippet">--quiet</span>                          | <span class="code-snippet">-q</span>          | Print fewer diagnostic messages when running unit tests. |


### Specific Unit Tests

```bash
rippled --unittest={TEST_OR_PACKAGE_NAME}
```

By default, <span class="code-snippet">rippled</span> runs all unit tests except ones that are classified as "manual". You can run an individual test by specifying its name, or run a subset of tests by specifying a package name.

Tests are grouped into a hierarchy of packages separated by <span class="code-snippet">.</span> characters and ending in the test case name.

#### Printing Unit Tests

```bash
rippled --unittest=print
```

The <span class="code-snippet">print</span> unit test is a special case that prints a list of available tests with their packages.

#### Manual Unit Tests

Certain unit tests are classified as "manual" because they take a long time to complete. These tests are marked with <span class="code-snippet">|M|</span> in the output of the <span class="code-snippet">print</span> unit test. Manual tests do not run by default when you run all unit tests or a package of unit tests. You can run manual tests individually by specifying the name of the test. For example:

<span class="code-snippet"></span>`bash
$ ./rippled --unittest=ripple.tx.OversizeMeta
ripple.tx.OversizeMeta
Longest suite times:
   60.9s ripple.tx.OversizeMeta
60.9s, 1 suite, 1 case, 9016 tests total, 0 failures
<span class="code-snippet"></span>`

#### Providing Arguments to Unit Tests

Certain manual unit tests accept an argument. You can provide the argument with the following option:

| Option                  | Description                                        |
|:------------------------|:---------------------------------------------------|
| <span class="code-snippet">--unittest-arg {ARG}</span>  | Provide the argument <span class="code-snippet">{ARG}</span> to the unit test(s) currently being run. Each unit test that accepts arguments defines its own argument format.  |


<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}
{% include '_snippets/tx-type-links.md' %}
{% include '_snippets/rippled_versions.md' %}
