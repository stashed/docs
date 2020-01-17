---
title: Hooks Examples | Stash
menu:
  docs_{{ .version }}:
    identifier: configuring-hooks
    name: Configuring Hooks
    parent: hooks
    weight: 40
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Configuring Different Types of Hooks

## HTTPGet

**Structure :**

```yaml
preBackup:
  httpGet:
    path: /demo
    host: my-service.mynamespace.svc
    port: 8080
    scheme: HTTP
    httpHeaders:  
```

**Examples :**

## HTTPPost

**Structure :**

**Examples :**

## TCPSocket

**Structure :**

**Examples :**

## Exec

**Structure :**

**Examples :**
