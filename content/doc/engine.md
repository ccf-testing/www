+++
title = "Testing a Program"
toc = true
type = "docs"
weight = 14

[menu.doc]
weight = 4
parent = "setup"
+++

In this section, we will exhaustively explore a simple program by using concolic execution and (distributed) hybrid concolic fuzzing.

## Building a Target

Before we can start testing a program, we need a suitable candidate for exploration. Here, we create a new file `crash_check.c` and add the following code:

```c
#include <unistd.h>
#include <sys/types.h>
#include <fcntl.h>
#include <string.h>
#include <stdlib.h>

int main(void) {
    char inp[6];
    inp[5] = '\0';

    if (read(STDIN_FILENO, &inp, 5) != 5) {
        return 1;
    }

    if (strcmp(inp, "crash") == 0) {
        abort();
    }
    
    return 0;
}
```
This program reads 5 bytes of input from `stdin` and writes them to a pre-allocated buffer. It then checks whether the given input is equal to `"crash"` and, if true, crashes by calling `abort()`.

We then enter our workspace shell and compile the program as described in a [prior section]({{< relref "hello-world" >}} "Hello World").

## Configuring a Worker
Before we can start exploring our target program, we have to configure the worker that performs the exploration. The following command: 

```console
$ worker --create-config-file
```

creates a new config file `worker_config.toml` in the current directory. The file contains various settings for changing the behavior of the worker but for now, the only interesting setting is `concex_target` which should point to the executable of the program under test. For our newly created target program `crash_check`:

```toml
concex_target = "./crash_check"
```

## Running the Worker
Now that we have configured the worker, we can start the exploration of the program by running:

```console
$ worker
```

The worker should now run for a few seconds and then inform us that it has finished exploring our program under test and terminate. By default, the output of the worker is stored in `worker_out` in the same directory that the worker was started from. To find the inputs that the worker has generated, we use

```console
$ cd worker_out/sync_dir/concolic/queue
```

Here, we see multiple files of the form `id:XXXXX` that denote inputs that took different paths through the program. One of these inputs should have an additional `crash` in its name, denoting that this input crashed the program under test. If we inspect the contents of this file, we see that it starts with `"crash"` followed by random bytes which crashes our test program.

## Adding a Fuzzer
In addition to concolic execution, we can use fuzzing to explore the program under test. The engine supports [AFL](https://github.com/google/AFL) as its fuzzer. To use fuzzing, we first have to compile the program with the instrumentation of the fuzzer:

```console
$ <PATH TO AFL>/afl-clang -o crash_check_fuzzer crash_test.c
```

Then we have to change three settings in the `worker_config.toml`:

```toml
fuzzer = <path to AFL>/afl-fuzz
num_fuzzers = 1
fuzzer_target = "./crash_check_fuzzer"
```

This tells the worker where it can find the executable of the fuzzer, how many fuzzer instances should run in parallel and which executable is instrumented for use with the fuzzer.

Afterward, we can start the worker:

```console
$ worker
```

After the worker has terminated, `worker_out/sync_dir` contains an additional folder which contains the output of the fuzzer.

## Distributed Hybrid Concolic Fuzzing
When testing larger software it might be advantageous to use multiple worker instances on multiple machines. For simplicity, we will start two worker instances on the same machine but the procedure is the same over a network.

---

:warning: Depending on your network settings, the default addresses used by the workers and the master might have to be changed.

---


First, open `worker_config.toml` and set

```toml
local = false
```

to instruct the worker to connect to a master.

Then copy `worker_config.toml` to `worker2_config.toml` and set

```toml
ip = "127.0.0.1:8885"
output_dir = "worker2_out"
```

to create a configuration file for the second worker that does not conflict with the first worker.


Next, we have to create a configuration for the central coordinator that synchronizes the exploration of both workers. Inside a workspace shell type

```console
master --create-config-file
```

which creates a file named `master_config.toml`.

In the config file set

```toml
fuzzer_target = "./crash_check_fuzzer"
concex_target = "./crash_check"
update_interval = "1"
```

Changing `fuzzer_target` and `concex_target` is necessary because in a distributed setting, the target binaries are sent from the master to all workers. Changing the update interval to one second is not necessary but it allows use to see the workers communicating with the master more quickly.

Now open three terminals and enter the workspace shell in each one. In the first terminal start the first worker

```console
$ worker --config-file ./worker_config.toml
```

and the second worker in the second terminal

```console
$ worker --config-file ./worker2_config.toml
```

Since we set `local = false`, both workers will not start exploring the program but, instead, try to connect to the master. 
In the third terminal, start the master by typing

```console
$ master
```

The master should now output that two workers have connected and the workers begin exploring the program.

In addition to the terminal output, the master also provides a web interface to control the workers and to check the status of the exploration. By default, the web interface is reachable under the address `127.0.0.1:8888`.

Once all workers report that 0 candidates are left for exploration, the master can be shut down, either via the web interface, or by pressing `CTRL-C` inside the terminal. When the master shuts down, it automatically shuts down all connected workers.

After the exploration, `master_out` contains a folder with all crashes that were found and a JSON file that contains all received status updates from the workers.

## Conclusion
In this section we explored a simple program and found a crashing input by using concolic execution and fuzzing. Furthermore, we ran multiple worker instances in parallel by using a central master instance.

<!---
To do so, start three terminal instances and enter the workspace shell in each terminal instance. Navigate to the directory that contains the `worker_config.toml` that we used before and make two copies named `worker1_config.toml` and `worker2_config.toml`.

The interesting settings are:
```toml
master_ip = "127.0.0.1:8887"
ip = "127.0.0.1:8886"
output_dir = "worker_out"
local = "true"
```

test

The last step before exploring the program is to inform the concolic executor from where the symbolic input comes, in our case `stdin`. The worker created a second file `format.json` that is used to configure the symbolic input.

Finally, we need a way to tell our concolic executor that we expect to read symbolic input from `stdin`. This is done by providing a JSON file that contains the expected format for the executor. Thus, we create another file `format.json` with the following content:
```json
{
    "rootfs": "./rootfs",
    "files": {},
    "vars": {},
    "argv": ["crash_check"],
    "stdin": "{10}"
}
```
Currently, the only important line for us is `"stdin": "{10}"`. This denotes that we want to use 10 bytes of symbolic input that should be read from `stdin`.

## Concolic Execution
-->

