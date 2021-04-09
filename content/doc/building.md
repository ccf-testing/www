+++
title = "Building"
toc = false
type = "docs"
weight = 12

[menu.doc]
weight = 2
parent = "setup"
+++

Once the prerequisites have been installed, we can fetch and build the project.
We assume that you have access to a git repository from which you can pull the workspace.

```console
$ git clone <git repo link> workspace
$ cd workspace
$ ./ws build
```

Running `./ws build` can take some time when first run, as it will fetch all sources and build them, which includes, e.g., building a version of LLVM.
Subsequent usages of `./ws build` will run incrementally, only recompiling what needs to be recompiled dependening on changes that were made (if any).
Also see the `README.md` file in the repository, it gives additional usage instructions for the `./ws` command.
