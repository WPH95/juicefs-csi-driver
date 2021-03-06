# Basic

This example shows how a basic example to use JuiceFS in Kubernetes pod.

## Prerequisite

Create JuiceFS filesystem using [juicefs format](https://github.com/juicedata/juicefs/#format-a-volume). Take Amazon S3 as an example:

```
./juicefs format --storage=s3 --bucket=https://<BUCKET>.s3.<REGION>.amazonaws.com \
    --access-key=<ACCESS_KEY> --secret-key=<SECRET_KEY> \
    redis://[:<PASSWORD>]@<HOST>:6379[/<DB>] <NAME>
```

Replace fields enclosed by `<>` with your own environment variables. The fields enclosed `[]` is optional which related your deployment environment.

This step can also executed by CSI driver in Kubernetes, but you should make sure:

1. The `access-key`, `secret-key` pair has `GET`, `PUT`, `DELETE` permission for the object bucket
2. The redis db is clean and the password (if provided) is right

Then create secret for access in Kubernetes.

```sh
kubectl -n default create secret generic juicefs-secret \
    --from-literal=name=<NAME>
    --from-literal=metaurl=redis://[:<PASSWORD>]@<HOST>:6379[/<DB>] \
    --from-literal=storage=s3
    --from-literal=bucket=https://<BUCKET>.s3.<REGION>.amazonaws.com
    --from-literal=access-key=<ACCESS_KEY> \
    --from-literal=secret-key=<SECRET_KEY>


```

## Apply the Example

Create storage class, persistence volume claim (PVC) and sample pod

```sh
>> kubectl apply -f basic.yaml
```

The persisten volume will be dynamically provisioned as a directory in the JuiceFS filesystem configured in storage class.

## Check JuiceFS filesystem is used

After all objects are created, verify that a 10 Pi PV is created:

```sh
kubectl get pv
```

Verify the pod is running:

```sh
kubectl get pods
```

Verify that data is written onto JuiceFS filesystem:

```sh
kubectl exec -ti juicefs-app -- tail -f /data/out.txt
```

Verify the directory created as PV in JuiceFS filesystem by mounting it in a host:

```
juicefs mount -d redis://[:<PASSWORD>]@<HOST>:6379[/<DB>] /jfs
```

