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

In this guide, we are going to discuss how to configure different types of hooks. Here, we will give some examples of different configurations.

## HTTPGet

`httpGet` hook allows sending an HTTP GET request to an HTTP server before and after the backup/restore process. The following configurable fields are available for `httpGet` hook.

| Field         | Type       | Usage                                                                                                                                                                                                                                                                                    |
| ------------- | ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `host`        | `Optional` | Specify the name of the host where the GET request will be sent.  If you don't specify this field, Stash will use the hook executor Pod IP as host.                                                                                                                                      |
| `port`        | `Required` | Specify the port where the HTTP server is listening. You can specify a numeric port or the name of the port in case of the HTTP server is running in the same pod where the hook will be executed. If you specify the port name, you must specify the `containerName` field of the hook. |
| `path`        | `Required` | Specify the path to access on the HTTP server.                                                                                                                                                                                                                                           |
| `scheme`      | `Required` | Specify the scheme to use to connect with the host.                                                                                                                                                                                                                                      |
| `httpHeaders` | `Optional` | Specify a list of custom headers to set in the request.                                                                                                                                                                                                                                  |

**Examples:**

- Hook with `host` and `port` are specified:

```yaml
preBackup:
  httpGet:
    host: my-host.demo.svc
    port: 8080
    path: "/"
    scheme: HTTP
```

- Hook to extract `host` and `port` from the hook executor pod:

```yaml
preBackup:
  httpGet:
    port: my-port
    path: "/"
    scheme: HTTP
  containerName: my-container
```

- Hook for sending custom header in the HTTP request:

```yaml
preBackup:
  httpGet:
    host: my-host.demo.svc
    port: 8080
    path: "/"
    scheme: HTTP
    httpHeaders:
    - name: "User-Agent"
      value: "stash/0.9.x"
```

## HTTPPost

`httpPost` hook allows sending an HTTP POST request to an HTTP server before and after the backup/restore process. The following configurable fields are available for `httpPost` hook.

| Field         | Type       | Usage                                                                                                                                                                                                                                                                             |
| ------------- | ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `host`        | `Optional` | Specify the name of the host where the POST request will be sent. If you don't specify this field, Stash will use the hook executor Pod IP as host.                                                                                                                               |
| `port`        | `Required` | Specify the port where the HTTP server is listening. You can specify a numeric port or the name of the port if your HTTP server is running in the same pod where the hook will be executed. If you specify the port name, you must specify the `containerName` field of the hook. |
| `path`        | `Required` | Specify the path to access on the HTTP server.                                                                                                                                                                                                                                    |
| `scheme`      | `Required` | Specify the scheme to use to connect with the host.                                                                                                                                                                                                                               |
| `httpHeaders` | `Optional` | Specify a list of custom headers to set in the request.                                                                                                                                                                                                                           |
| `body`        | `Optional` | Specify the data to send in the request body.                                                                                                                                                                                                                                     |
| `form`        | `Optional` | Specify the data to send as URL encoded form with the request.                                                                                                                                                                                                                    |

**Examples:**

- Send JSON data in the request body:

```yaml
preBackup:
  httpPost:
    host: my-service.mynamespace.svc
    path: /demo
    port: 8080
    scheme: HTTP
    httpHeaders:
    - name: Content-Type
      value: application/json
    body: '{
            "name": "john doe",
            "age": "28"
           }'
```

- Send URL encoded form in the request:

```yaml
preBackup:
  httpPost:
    host: my-service.mynamespace.svc
    path: /demo
    port: 8080
    scheme: HTTP
    form:
    - key: "name"
      values: ["john doe"]
    - key: "age"
      values: ["28"]
```

## TCPSocket

`tcpSocket` hook allows running a check against a TCP port to verify whether it is open or not. The following configurable fields are available for `tcpSocket` hook.

| Field  | Type       | Usage                                                                                                                                                                                                                                                |
| ------ | ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `host` | `Optional` | Specify the name of the host where the server is running. If you don't specify this field, Stash will use the hook executor Pod IP as host.                                                                                                          |
| `port` | `Required` | Specify the port to check. You can specify a numeric port or the name of the port when your server is running in the same pod where the hook will be executed. If you specify the port name, you must specify the `containerName` field of the hook. |

**Examples:**

- Hook with `host` and `port` field specified.

```yaml
preBackup:
  tcpSocket:
    host: my-host.demo.svc
    port: 8080
```

- Hook to extract `host` and `port` from the hook executor pod.

```yaml
preBackup:
  tcpSocket:
    port: my-port
  containerName: my-container
```

## Exec

`exec` hook allows running commands inside a container before or after the backup/restore process. The following configurable fields are available for `exec` hook.

| Field     | Type       | Usage                                  |
| --------- | ---------- | -------------------------------------- |
| `command` | `Required` | Specify a list of commands to execute. |

> If you don't specify `containerName` for `exec` hook, the command will be executed into the 0th container of the respective pod.

**Examples:**

- Hook to remove old corrupted data before restore:

```yaml
preRestore:
  exec:
    command: ["/bin/sh","-c","rm -rf /corrupted/data/directory"]
  containerName: my-container
```

- Hook to make a MySQL database read-only before backup:

```yaml
preBackup:
  exec:
    command:
      - /bin/sh
      - -c
      - mysql -u root --password=$MYSQL_ROOT_PASSWORD -e "SET GLOBAL super_read_only = ON;"
  containerName: mysql
```

- Hook to make a MySQL database writable after backup:

```yaml
postBackup:
  exec:
    command:
      - /bin/sh
      - -c
      - mysql -u root --password=$MYSQL_ROOT_PASSWORD -e "SET GLOBAL super_read_only = OFF;"
  containerName: mysql
```

- Hook to remove corrupted MySQL database before restoring:

```yaml
preRestore:
  exec:
    command:
      - /bin/sh
      - -c
      - mysql -u root --password=$MYSQL_ROOT_PASSWORD -e "DROP DATABASE companyRecord;"
  containerName: mysql
```

- Hook to apply some migration after restoring a MySQL database:

```yaml
postRestore:
  exec:
    command:
      - /bin/sh
      - -c
      - mysql -u root --password=$MYSQL_ROOT_PASSWORD -e "RENAME TABLE companyRecord.employee TO companyRecord.salaryRecord;"
  containerName: mysql
```

If the above hooks do not cover your use cases or you have any questions regarding the hooks, please feel free to open an issue [here](https://github.com/stashed/stash/issues).
