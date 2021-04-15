
+++
title = "Hello World"
toc = true
type = "docs"
weight = 13

[menu.doc]
weight = 3
parent = "setup"
+++

In this section we will compile a simple program for concolic execution and run it.

## Entering a workspace shell

First we have to make all the executables built by the workspace available in our shell.
We can do so by entering our workspace directory and running:

```console
$ ./ws shell
```

In this shell we can now use all the tools that the workspace built.
To check if everything works, we can try to run of the executables, for example:

```console
$ musl-clang --help
```

should output a help message.
`musl-clang` is the compiler that can build programs with concolic instrumentation.
We will next use it to build a simple program.

## Building a simple program

Create a new file `hello-world.c` and paste the following code into it:

```c
#include <stdio.h>

int main(void) {
  printf("Hello World!\n");
  return 0;
}
```

Currently compilation and linking for concolically instrumented binaries have to be performed in two steps.
Therefore we have to run:

```console
$ musl-clang --instrument -c hello-world.c -o hello-world.o
$ musl-clang -o hello-world hello-world.o
```

Which will create the binary `hello-world`, which is ready for concolic execution.

## Running the example program

A binary compiled for concolic execution cannot be executed as-is, as it expects a specific environment during execution.
Therefore, simply running `./hello-world` will output an error message instead of the text we expected.

Instead, we can run our binary in a basic concolic environment using `env_tool`:

```console
$ env_tool run-concolic ./hello-world
```

Which will now print the expected "Hello World!" to our terminal.
`env_tool run-concolic` is a basic way to execute a binary that was compiled for concolic execution without running the full concolic execution itself.

## Conclusion

In this section we compiled and executed a simple program for concolic execution, but did not yet run the full concolic execution engine.