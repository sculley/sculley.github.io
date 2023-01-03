---
layout: post
title:  "Listing all the Kubernetes RBAC resources/sub-resources"
date:   2023-01-01 15:26:33 +0000
categories: posts
---

When working with Kubernetes RBAC, it can be difficult to know what resources and sub-resources are available. This is especially true when you are trying to create a role or cluster role that grants access to specific resources and sub-resources for applications and users. In this post, I will show you how to list all the Kubernetes RBAC resources/sub-resources using a script that I wrote in bash. This script uses the Kubernetes API to get the list of resources/sub-resources and then uses the `jq` command to parse the JSON output and print out the list of resources/sub-resources in a human-readable format using the `column` command.

## Requirements

To run this script, you will need the following:

* A Kubernetes cluster
* A Kubernetes API server URL
* A client key
* A client cert
* A CA cert
* jq (https://stedolan.github.io/jq/)

To get the client key, client cert, and CA cert, you can use the following commands (assuming you are using the default context and have a kubeconfig file):

```shell
kubectl config view --raw -o json | jq -r '.users[0].user."client-certificate-data"' | base64 -d > client.crt
kubectl config view --raw -o json | jq -r '.users[0].user."client-key-data"' | base64 -d > client.key
kubectl config view --raw -o json | jq -r '.clusters[0].cluster."certificate-authority-data"' | base64 -d > ca.crt
```

## Usage

Copy the following script to a file and run it with the required arguments

```shell
#!/usr/bin/env bash
# list_rbac_resources.sh - list all the kubernetes rbac resources/sub-resources
# Requires jq (https://stedolan.github.io/jq/)
# Usage: ./list_rbac_resources.sh <kubernetes api server url> <client key> <client cert> <ca cert>

# Generate a UUID for the tmp output file
UUID=$(uuidgen)

# Get the list of APIs
APIS=$(curl --key $2 --cert $3 --cacert $4 -s $1/apis | jq -r '[.groups | .[].name] | join(" ")')

# Add header to tmp output file
echo "API Resource Verb Namespaced Kind" >> /tmp/list_rbac_resources_${UUID}

# Get the list of resources/sub-resources from the core API
curl --key $2 --cert $3 --cacert $4 -s $1/api/v1 | jq -r --arg api "$api" '.resources | .[] | "\($api) \(.name) \(.verbs | join(",")) \(.namespaced) \(.kind)"' >> /tmp/list_rbac_resources_${UUID}

# Get the list of resources/sub-resources from the other APIs
for api in $APIS; do
    version=$(curl --key $2 --cert $3 --cacert $4 -s $1/apis/$api | jq -r '.preferredVersion.version')
    curl --key $2 --cert $3 --cacert $4 -s $1/apis/$api/$version | jq -r --arg api "$api" '.resources | .[]? | "\($api) \(.name) \(.verbs | join(",")) \(.namespaced) \(.kind)"' >> /tmp/list_rbac_resources_${UUID}
done

# Print the list of resources/sub-resources using the column command
column -t /tmp/list_rbac_resources_${UUID}

# Remove the tmp output file
rm -rf /tmp/list_rbac_resources_${UUID}
```

## Example

```shell
$ ./list_rbac_resources.sh https://172.31.123.141:6443 client.key client.crt ca.crt
```

<br>

![list rbac resources output](https://raw.githubusercontent.com/sculley/sculley.github.io/main/img/list_rbac_resources_output.png)