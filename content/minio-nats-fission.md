+++
title = "Live-Reload in Fission: Instant feedback on your Serverless Functions"
date = "2020-06-05T19:45:01+05:30"
author = "Vishal Biyani"
description = "Live reloading of functions for faster feedback!"
categories = ["Minio", "S3", "NATS"]
type = "post"
+++

## Introduction

## Install Minio Server


https://docs.min.io/docs/deploy-minio-on-kubernetes.html
$ helm install minio --set accessKey=admin,secretKey=password helm/minio --version 5.0.26

## Install Minio Client

https://docs.min.io/docs/minio-client-quickstart-guide

$ mc config host add minfission http://127.0.0.1:9000 admin password


```
$ mc admin config get minfission
KEYS:
region                label the location of the server
cache                 add caching storage tier
compression           enable server side compression of objects
etcd                  federate multiple clusters for IAM and Bucket DNS
identity_openid       enable OpenID SSO support
identity_ldap         enable LDAP SSO support
policy_opa            enable external OPA for policy enforcement
kms_vault             enable external HashiCorp Vault key management service
kms_kes               enable external MinIO key encryption service
api                   manage global HTTP API call specific features, such as throttling, authentication types, etc.
logger_webhook        send server logs to webhook endpoints
audit_webhook         send audit logs to webhook endpoints
notify_webhook        publish bucket notifications to webhook endpoints
notify_amqp           publish bucket notifications to AMQP endpoints
notify_kafka          publish bucket notifications to Kafka endpoints
notify_mqtt           publish bucket notifications to MQTT endpoints
notify_nats           publish bucket notifications to NATS endpoints
notify_nsq            publish bucket notifications to NSQ endpoints
notify_mysql          publish bucket notifications to MySQL databases
notify_postgres       publish bucket notifications to Postgres databases
notify_elasticsearch  publish bucket notifications to Elasticsearch endpoints
notify_redis          publish bucket notifications to Redis datastores
```

$ mc admin config get  minfission/ notify_nats
notify_nats enable=off address= subject= username= password= token= tls=off tls_skip_verify=off cert_authority= client_cert= client_key= ping_interval=0 streaming=off streaming_async=off streaming_max_pub_acks_in_flight=0 streaming_cluster_id= queue_dir= queue_limit=0

$ mc admin config set minfission notify_nats:1 password="defaultFissionAuthToken" streaming_max_pub_acks_in_flight="10" subject="test" address="nats-streaming:4222"  token="defaultFissionAuthToken" username="yourusername" ping_interval="0" queue_limit="0" tls="off" streaming_async="on" streaming_cluster_id="fissionMQTrigger" streaming_enable="on"

## In Minio

$  mc mb minfission/natsdata
Bucket created successfully `minfission/natsdata`.

$  mc event add minfission/natsdata arn:minio:sqs::1:nats --suffix .txt
Successfully added arn:minio:sqs::1:nats

$ mc event list minfission/natsdata
arn:minio:sqs::1:nats   s3:ObjectCreated:*,s3:ObjectRemoved:*,s3:ObjectAccessed:*   Filter: suffix=".txt"
