---
title: Elasticsearch Addon Overview | Stash
description: Elasticsearch Addon Overview | Stash
menu:
  product_stash_{{ .version }}:
    identifier: stash-elasticsearch-readme
    name: Readme
    parent: stash-elasticsearch
    weight: -1
product_name: stash
menu_name: product_stash_{{ .version }}
section_menu_id: stash-addons
url: /products/stash/{{ .version }}/addons/elasticsearch/
aliases:
- /products/stash/{{ .version }}/addons/elasticsearch/README/
---

# Stash Elasticsearch Addon

Stash 0.9.0+ supports extending its functionality through addon mechanism. Stash Elasticsearch addon enables Stash to backup and restore Elasticsearch database.

This guide will give you an overview of which Elasticsearch versions are supported and how the docs are organized.

## Supported Elasticsearch Version

Stash supports backup and restore of the following Elasticsearch versions:

- [6.5](/docs/addons/elasticsearch/guides/6.5/elasticsearch.md)

## Documentation Overview

Stash Elasticsearch documentations are organized as below:

- [How does it works?](/docs/addons/elasticsearch/overview.md) gives an overview how backup and restore process for Elasticsearch database works in Stash.
- [Setup](/docs/addons/elasticsearch/setup/install.md) shows how to install and uninstall Elasticsearch addon for Stash.
- [Guides](/docs/addons/elasticsearch/guides/6.5/elasticsearch.md) contains step by step guides for backup and restore different versions of Elasticsearch databases under their respective version sub-category.
