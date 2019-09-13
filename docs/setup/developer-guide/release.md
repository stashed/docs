---
title: Release | Stash
description: Stash Release
menu:
  product_stash_{{ .version }}:
    identifier: release
    name: Release
    parent: developer-guide
    weight: 15
product_name: stash
menu_name: product_stash_{{ .version }}
section_menu_id: setup
---
# Release Process

The following steps must be done from a Linux x64 bit machine.

- Do a global replacement of tags so that docs point to the next release.
- Push changes to the `release-x` branch and apply new tag.
- Push all the changes to remote repo.
- Build and push stash docker image:
```console
$ cd ~/go/src/stash.appscode.dev/stash
./hack/docker/setup.sh; env APPSCODE_ENV=prod ./hack/docker/setup.sh release
```

- Now, update the release notes in Github. See previous release notes to get an idea what to include there.
