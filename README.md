# Service Catalog Demo

This repo provides a walkthrough of the Service Catalog using only Helm charts.
Before proceeding to Demo Instructions, make sure you perform the Setup Instructions below.

## Demo Instructions

### 1. Manual S3 Consumer

```console
helm install manual-s3-consumer --namespace=steward --name=manual-s3-consumer \
    --set AccessKey=${STEWARD_ACCESS_KEY},SecretKey=${STEWARD_SECRET_KEY},BucketName=gabrtv-bucket
```

### 2. Automatic S3 Consumer

```console
helm install s3-consumer --namespace=steward --name=s3-consumer
```

### 3. Install the Minio Provider

```console
helm install minio-provider --namespace=steward --name=minio-provider \
 --set minio.endpoint=minio-minio-svc.minio.svc.cluster.local:9000,minio.access_key_id=${STEWARD_ACCESS_KEY},minio.secret_access_key=${STEWARD_SECRET_KEY}
```

### 4. Consumer Switch to Minio

```console
helm install minio-consumer --namespace=steward --name=minio-consumer
```

## Setup Instructions

Install a Kubernetes cluster and make sure Helm is ready (i.e. `helm init`).

### Cleanup

```console
aws s3 rm s3://gabrtv-bucket/ --recursive
aws s3 ls | grep cf- | awk {'print $3'} | xargs -t -I{} aws s3 rb s3://{} --force
helm delete --purge svc-catalog
helm delete --purge s3-provider
helm delete --purge manual-s3-consumer
helm delete --purge s3-consumer
helm delete --purge minio-provider
helm delete --purge minio-consumer
kubectl delete ns steward
```

### Install the Service Catalog

```console
helm install svc-catalog \
    --namespace=steward --name=svc-catalog \
    --set registry=quay.io/kubernetes-service-catalog,version=v0.0.1-RC1-3-gbb3a42a,storageType=tpr,debug=true,insecure=true,imagePullPolicy=IfNotPresent,globalNamespace=steward
```

### Install the S3 Provider

```console
helm install s3-provider --namespace=steward --name=s3-provider \
    --set AdminAwsAccessKeyId=${STEWARD_ACCESS_KEY},AdminAwsSecretAccessKey=${STEWARD_SECRET_KEY}
```

### Install Distributed Minio

```
helm install kubernetes-charts/minio --name=minio --namespace=minio \
 --set mode=distributed,accessKey=${STEWARD_ACCESS_KEY},secretKey=${STEWARD_SECRET_KEY}

mc config host add minio http://40.68.249.80:9000 ${STEWARD_ACCESS_KEY} ${STEWARD_SECRET_KEY} S3v4
mc ls minio
```