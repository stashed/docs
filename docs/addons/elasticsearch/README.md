---
title: Elasticsearch Addon Overview | Stash
description: Elasticsearch Addon Overview | Stash
menu:
  docs_{{ .version }}:
    identifier: stash-elasticsearch-readme
    name: Readme
    parent: stash-elasticsearch
    weight: -1
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
url: /docs/{{ .version }}/addons/elasticsearch/
aliases:
  - /docs/{{ .version }}/addons/elasticsearch/README/
---

# Stash Elasticsearch Addon

Stash 0.9.0+ supports extending its functionality through addons. Stash Elasticsearch addon enables Stash to backup and restore Elasticsearch databases.

This guide will give you an overview of which Elasticsearch versions are supported and how the docs are organized.

## Supported Elasticsearch Versions

Stash supports backup and restore of the following Elasticsearch versions:

- [5.6](/docs/addons/elasticsearch/guides/5.6/elasticsearch.md)
- [5.6.4](/docs/addons/elasticsearch/guides/5.6.4/elasticsearch.md)
- [6.2](/docs/addons/elasticsearch/guides/6.2/elasticsearch.md)
- [6.2.4](/docs/addons/elasticsearch/guides/6.2.4/elasticsearch.md)
- [6.3](/docs/addons/elasticsearch/guides/6.3/elasticsearch.md)
- [6.3.0](/docs/addons/elasticsearch/guides/6.3.0/elasticsearch.md)
- [6.4](/docs/addons/elasticsearch/guides/6.4/elasticsearch.md)
- [6.4.0](/docs/addons/elasticsearch/guides/6.4.0/elasticsearch.md)
- [6.5](/docs/addons/elasticsearch/guides/6.5/elasticsearch.md)
- [6.5.3](/docs/addons/elasticsearch/guides/6.5.3/elasticsearch.md)
- [6.8](/docs/addons/elasticsearch/guides/6.8/elasticsearch.md)
- [6.8.0](/docs/addons/elasticsearch/guides/6.8.0/elasticsearch.md)
- [7.2](/docs/addons/elasticsearch/guides/7.2/elasticsearch.md)
- [7.2.0](/docs/addons/elasticsearch/guides/7.2.0/elasticsearch.md)
- [7.3](/docs/addons/elasticsearch/guides/7.3/elasticsearch.md)
- [7.3.2](/docs/addons/elasticsearch/guides/7.3.2/elasticsearch.md)

>Version **M.M** actually represents the latest patch of **M.M.P** series. For example, version **7.3** actually represents **7.3.2** as it is the latest supported patch of **7.3** series. Now, if **7.3.3** is released then **7.3** will represents **7.3.3**.

## Documentation Overview

Stash Elasticsearch documentations are organized as below:

- [How does it works?](/docs/addons/elasticsearch/overview.md) gives an overview of how backup and restore process for Elasticsearch database works in Stash.
- [Setup](/docs/addons/elasticsearch/setup/install.md) shows how to install and uninstall Elasticsearch addon for Stash.
- [Guides](/docs/addons/elasticsearch/guides/6.5/elasticsearch.md) contains step by step guides to backup and restore different versions of Elasticsearch databases.
