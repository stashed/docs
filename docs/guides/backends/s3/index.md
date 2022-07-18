---
title: AWS S3/Minio/Rook Backend | Stash
description: Configure Stash to use AWS S3/Minio/Rook as Backend.
menu:
  docs_{{ .version }}:
    identifier: backend-s3
    name: AWS S3/Minio/Rook
    parent: backend
    weight: 30
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# AWS S3

Stash supports AWS S3 or S3 compatible storage services like [Minio](https://minio.io/) servers, [Rook Object Store](https://rook.io/docs/rook/v0.9/ceph-object.html), [DigitalOceans Space](https://www.digitalocean.com/products/spaces/) as a backend. This tutorial will show you how to use this backend.

In order to use S3 or S3 compatible storage service as backend, you have to create a `Secret` and a `Repository` object pointing to the desired bucket.

>If the bucket does not exist yet, Stash will create it automatically in the default region (`us-east-1`) during the first backup. In this case, you have to make sure that the bucket name is unique across all S3 buckets. Currently, it is not possible for Stash to create bucket in different region. You have to create the bucket in your desired region before using it in Stash.

#### Create Storage Secret

To configure storage secret for this backend, following secret keys are needed:

| Key                     | Type       | Description                                                                                                                                                            |
| ----------------------- | ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `RESTIC_PASSWORD`       | `Required` | Password that will be used to encrypt the backup snapshots.                                                                                                            |
| `AWS_ACCESS_KEY_ID`     | `Required` | AWS / Minio / Rook / DigitalOcean Spaces access key ID                                                                                                                 |
| `AWS_SECRET_ACCESS_KEY` | `Required` | AWS / Minio / Rook / DigitalOcean Spaces secret access key                                                                                                             |
| `CA_CERT_DATA`          | `optional` | CA certificate used by storage backend. This can be used to pass the root certificate that has been used to sign the server certificate of a TLS secured Minio server. |

Create storage secret as below,

```bash
$ echo -n 'changeit' > RESTIC_PASSWORD
$ echo -n '<your-aws-access-key-id-here>' > AWS_ACCESS_KEY_ID
$ echo -n '<your-aws-secret-access-key-here>' > AWS_SECRET_ACCESS_KEY
$ kubectl create secret generic -n demo s3-secret \
    --from-file=./RESTIC_PASSWORD \
    --from-file=./AWS_ACCESS_KEY_ID \
    --from-file=./AWS_SECRET_ACCESS_KEY
secret/s3-secret created
```

For TLS secured Minio Server, create secret as below,

```bash
$ echo -n 'changeit' > RESTIC_PASSWORD
$ echo -n '<your-minio-access-key-id-here>' > AWS_ACCESS_KEY_ID
$ echo -n '<your-minio-secret-access-key-here>' > AWS_SECRET_ACCESS_KEY
$ cat ./directory/of/root/certificate/ca.crt > CA_CERT_DATA
$ kubectl create secret generic -n demo minio-secret \
    --from-file=./RESTIC_PASSWORD \
    --from-file=./AWS_ACCESS_KEY_ID \
    --from-file=./AWS_SECRET_ACCESS_KEY \
    --from-file=./CA_CERT_DATA
secret/minio-secret created
```

{{< notice type="warning" message="If you are using a Minio backend, make sure that you are using `AWS_ACCESS_KEY_ID` instead of `MINIO_ACCESS_KEY` and `AWS_SECRET_ACCESS_KEY` instead of `MINIO_SECRET_KEY`." >}}

### Create Repository

Now, you have to create a `Repository` crd. You have to provide the storage secret that we have created earlier in `spec.backend.storageSecretName` field.

Following parameters are available for `S3` backend.

| Parameter     | Type       | Description                                                                                                                                                                                                                                                                                                                      |
| ------------- | ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `s3.endpoint` | `Required` | For S3, use `s3.amazonaws.com`. If your bucket is in a different location, S3 server (s3.amazonaws.com) will redirect Stash to the correct endpoint. For DigitalOCean, use `nyc3.digitaloceanspaces.com` etc. depending on your bucket region. For S3-compatible other storage services like Minio / Rook use URL of the server. |
| `s3.bucket`   | `Required` | Name of Bucket. If the bucket does not exist yet it will be created in the default location (`us-east-1` for S3). It is not possible at the moment for Stash to create a new bucket in a different location, so you need to create it using a different program.                                                                 |
| `s3.region`   | `Optional` | Specify the region of your bucket.                                                                                                                                                                                                                                                                                               |
| `s3.prefix`   | `Optional` | Path prefix inside the bucket where the backed up data will be stored.                                                                                                                                                                                                                                                           |

Below, the YAML of a sample `Repository` crd that uses an `S3` bucket as a backend.

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: s3-repo
  namespace: demo
spec:
  backend:
    s3:
      endpoint: s3.amazonaws.com # use server URL for s3 compatible other storage service
      bucket: stash-demo
      region: us-west-1
      prefix: /backup/demo/deployment/stash-demo
    storageSecretName: s3-secret
```

For S3 compatible Minio and other storage services, specify the endpoint with connection scheme (`http`, or `https`),

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: minio-repo
  namespace: demo
spec:
  backend:
    s3:
      endpoint: https://my-minio-service.minio-namespace.svc
      bucket: stash-demo
      prefix: /backup/demo/deployment/stash-demo
    storageSecretName: s3-secret
```

Create the `s3-repo` Repository we have shown above using the following command,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/backends/s3/examples/s3.yaml
repository/s3-repo created
```

Now, we are ready to use this backend to backup our desired data using Stash.

## Next Steps

- Learn how to use Stash to backup workloads data from [here](/docs/guides/workloads/overview.md).
- Learn how to use Stash to backup databases from [here](/docs/guides/addons/overview.md).
- Learn how to use Stash to backup stand-alone PVC from [here](/docs/guides/volumes/overview.md).
