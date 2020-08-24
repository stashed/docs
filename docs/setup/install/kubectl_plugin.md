---
title: Install Stash kubectl Plugin
description: Installation guide for Stash kubectl Plugin
menu:
  docs_{{ .version }}:
    identifier: install-stash-kubectl-plugin
    name: Stash kubectl Plugin
    parent: installation-guide
    weight: 30
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: setup
---

## Install Stash kubectl Plugin

Stash provides a CLI to use as kubectl plugin quickly manipulating Stash objects. You can download the pre-build binaries from [stashed/cli](https://github.com/stashed/cli/releases) releases and put it into one of your installation directory denoted by `$PATH` variable.

Here is a simple Linux command to install the latest 64-bit Linux binary directly into your `/usr/local/bin` directory:

```bash
# Linux amd 64-bit
wget -O kubectl-stash https://github.com/stashed/cli/releases/download/{{< param "info.cli" >}}/kubectl-stash-linux-amd64 \
  && chmod +x kubectl-stash \
  && sudo mv kubectl-stash /usr/local/bin/
```

If you prefer to install kubectl Stash cli from source code, make sure that your go development environment has been setup properly. Then, just run:

```bash
go get github.com/stashed/cli/...
```

>Please note that this will install kubectl stash cli from master branch which might include breaking and/or undocumented changes.
