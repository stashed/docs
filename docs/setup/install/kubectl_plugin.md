---
title: Install Stash kubectl Plugin
description: Installation guide for Stash kubectl Plugin
menu:
  docs_{{ .version }}:
    identifier: install-stash-kubectl-plugin
    name: Stash kubectl Plugin
    parent: installation-guide
    weight: 10
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: setup
---

## Install Stash kubectl Plugin

Stash provides a CLI using kubectl plugin to work with the stash Objects quickly. Download pre-build binaries from [stashed/cli Githhub release]() and put the binary to some directory in your `PATH`. To install linux 64-bit you can run the following commands:

```console
# Linux amd 64-bit
wget -O kubectl-stash https://github.com/stashed/cli/releases/download/{{< param "info.cli" >}}/kubectl-stash-linux-amd64 \
  && chmod +x kubectl-stash \
  && sudo mv kubectl-stash /usr/local/bin/
```

If you prefer to install kubectl Stash cli from source code, you will need to set up a GO development environment following [these instructions](https://golang.org/doc/code.html). Then, install the CLI using `go get` from source code.

```console
go get github.com/stashed/cli/...
```

>Please note that this will install kubectl stash cli from master branch which might include breaking and/or undocumented changes.
