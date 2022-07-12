---
layout: page
title: Запустить в облаке Oracle виртуальную машину
description: Запустить в облаке Oracle виртуальную машину
keywords: Запустить в облаке Oracle виртуальную машину
permalink: /tools/clouds/oracle/free-tier/compute-instance/
---

<br/>

# Запустить в облаке Oracle виртуальную машину

<br/>

```
$ export TENANCY_COMPARTMENT_ID=$(
    oci iam compartment list \
    --all \
    --compartment-id-in-subtree true \
    --access-level ACCESSIBLE \
    --include-root \
    --raw-output \
    --query "data[?contains(\"id\",'tenancy')].id | [0]"
)

$ echo ${TENANCY_COMPARTMENT_ID}

$ export COMPARTMENT_NAME=test-compartment

$ oci iam compartment create \
    --name ${COMPARTMENT_NAME} \
    --description "Test compartment for linux" \
    --compartment-id ${TENANCY_COMPARTMENT_ID}
```

<br/>

```
// compartment-id
$ oci iam compartment list
```

<br/>

```
// delete compartment-id
// $ oci iam compartment delete --force \
    --compartment-id ocid1.compartment.oc1..aaaaaaaaxary2whptgduzl3m3uqu6p2ci4wpwvty7ow2qrh6xejvv5sb4xaa
```

<!-- <br/>

```
// compartment-id
$ oci iam compartment list \
  | jq -r -c '.data[] ["id"]'
``` -->

<br/>

```
$ export COMPARTMENT_ID=указать ранее созданный compartment
$ echo ${COMPARTMENT_ID}
```

<br/>

## Network setup for Linux VM

```
$ export VCN_DISPLAY_NAME="VCN_LINUX_DISPLAY_NAME"
$ export VCN_DNS_LABEL="VCNDNS"
```

<br/>

```
$ oci network vcn create \
    --compartment-id ${COMPARTMENT_ID} \
    --display-name ${VCN_DISPLAY_NAME} \
    --dns-label ${VCN_DNS_LABEL} \
    --cidr-block "10.0.0.0/24"
```

<br/>

```
// $ oci network vcn delete -y \
//     --vcn-id <VCN_ID>
```

<br/>

```
$ oci network vcn list \
    --compartment-id ${COMPARTMENT_ID}
```

<br/>

```
$ export VCN_ID=$(
    oci network vcn list \
    --compartment-id ${COMPARTMENT_ID} \
    | jq -r -c '.data[] ["id"]'
)
```

<br/>

```
$ echo ${VCN_ID}
```

<br/>

```
$ export DEFAULT_SECURITY_LIST_ID=$(
    oci network vcn list \
    --compartment-id ${COMPARTMENT_ID} \
    | jq -r -c '.data[] ["default-security-list-id"]'
)
```

<br/>

```
$ echo ${DEFAULT_SECURITY_LIST_ID}
```

<br/>

```
$ export DEFAULT_ROUTE_TABLE_ID=$(
    oci network vcn list \
    --compartment-id ${COMPARTMENT_ID} \
    | jq -r -c '.data[] ["default-route-table-id"]'
)
```

<br/>

```
$ echo ${DEFAULT_ROUTE_TABLE_ID}
```

<br/>

```
$ oci iam availability-domain list \
  --compartment-id ${COMPARTMENT_ID}  \
  | jq -r -c '.data[] ["name"]'
```

<br/>

```
QBXU:EU-FRANKFURT-1-AD-1
QBXU:EU-FRANKFURT-1-AD-2
QBXU:EU-FRANKFURT-1-AD-3
```

<br/>

```
$ export AVAILABILITY_DOMAIN=QBXU:EU-FRANKFURT-1-AD-3
```

<br/>

```
$ oci compute shape list \
    --compartment-id ${COMPARTMENT_ID} \
    --availability-domain ${AVAILABILITY_DOMAIN}  \
    | jq -r -c '.data[] ["shape"]'
```

<br/>

```
BM.Standard.A1.160
VM.Standard.A1.Flex
VM.Standard.E2.1.Micro
```

<br/>

VM.Standard.E2.1.Micro дают бесплатно только в QBXU:EU-FRANKFURT-1-AD-3

<br/>

```
$ export SUBNET_DISPLAY_NAME=subnetlinuxtest
$ export SUBNET_DNS_LABEL=subnetlinuxtest

$ echo ${DEFAULT_SECURITY_LIST_ID}


// Прописать руками <default_secrity_list_id>
$ oci network subnet create \
    --vcn-id ${VCN_ID} \
    --compartment-id ${COMPARTMENT_ID} \
    --availability-domain ${AVAILABILITY_DOMAIN} \
    --display-name ${SUBNET_DISPLAY_NAME} \
    --dns-label ${SUBNET_DNS_LABEL} \
    --cidr-block "10.0.0.0/24" \
    --security-list-ids '["<default_secrity_list_id>"]'
```

<br/>

```
$ oci network subnet list  \
    --compartment-id ${COMPARTMENT_ID}
```

<br/>

```
$ export SUBNET_ID=$(
    oci network subnet list \
    --compartment-id ${COMPARTMENT_ID} \
    | jq -r -c '.data[] ["id"]'
)
```

<br/>

```
$ export ROUTE_TABLE_ID=$(
    oci network subnet list \
    --compartment-id ${COMPARTMENT_ID} \
    | jq -r -c '.data[] ["route-table-id"]'
)
```

<br/>

```
$ echo ${SUBNET_ID}

$ echo ${ROUTE_TABLE_ID}
```

<br/>

### Create Network Internet Gateway

<br/>

```
$ export INTERNET_GATEWAY_DISPLAY_NAME=LinuxGateWay
```

<br/>

```
$ oci network internet-gateway create \
    --compartment-id ${COMPARTMENT_ID} \
    --is-enabled true \
    --vcn-id ${VCN_ID} \
    --display-name ${INTERNET_GATEWAY_DISPLAY_NAME}
```

<br/>

```
$ export GATEWAY_ID=$(
    oci network internet-gateway list \
    --compartment-id ${COMPARTMENT_ID} \
    | jq -r -c '.data[] ["id"]'
)
```

<br/>

```
$ echo ${GATEWAY_ID}
```

<br/>

### Adding Route Rules to Route Table

<br/>

```
// Прописать руками <internet_gateway_id>
$ oci network route-table update \
    --rt-id ${ROUTE_TABLE_ID} \
    --route-rules '[
        {"cidrBlock" : "0.0.0.0/0",
        "networkEntityId" : "<internet_gateway_id>"}
    ]'
```

<br/>

## Set up and connect to the Linux VM

<br/>

```
// Получить список образов операционных систем
$ oci compute image list --all \
    --compartment-id ${COMPARTMENT_ID}  \
    | jq -r -c '.data[] ["display-name"]'
```

<br/>

```
$ export DISPLAY_NAME="LinuxVMTest"

// Выбрал Ubuntu 20
$ export IMAGE_ID=ocid1.image.oc1.eu-frankfurt-1.aaaaaaaadqrjjiunkzkf62ggllx56s3p5775gonlifl74d4ri3bykztb4bha
```

<br/>

```
$ ssh-keygen -t rsa
```

<br/>

```
// Заменить путь до публичного ключа
$ oci compute instance launch \
    --availability-domain ${AVAILABILITY_DOMAIN} \
    --compartment-id ${COMPARTMENT_ID} \
    --shape "VM.Standard.E2.1.Micro" \
    --display-name ${DISPLAY_NAME} \
    --image-id ${IMAGE_ID} \
    --ssh-authorized-keys-file ${HOME}/.ssh/id_rsa.pub \
    --subnet-id ${SUBNET_ID}
```

<br/>

```
$ oci compute instance list \
    --compartment-id ${COMPARTMENT_ID}
```

<br/>

```
$ export INSTANCE_ID=$(
    oci compute instance list \
    --compartment-id ${COMPARTMENT_ID} \
    | jq -r -c '.data[] ["id"]'
)
```

<br/>

```
// DELETE
// $ oci compute instance terminate \
//    --instance-id ${INSTANCE_ID}
```

<br/>

```
$ echo ${INSTANCE_ID}
```

<br/>

```
$ oci compute instance list-vnics \
    --instance-id ${INSTANCE_ID}
```

<br/>

```
$ export PUBLIC_IP=$(
    oci compute instance list-vnics \
    --instance-id ${INSTANCE_ID} \
    | jq -r -c '.data[] ["public-ip"]'
)
```

<br/>

```
$ echo ${PUBLIC_IP}
```

<br/>

```
$ ssh ubuntu@${PUBLIC_IP}


// Если не ubuntu
$ ssh opc@${PUBLIC_IP}
```

<br/>

```
$ sudo apt update -y && sudo apt upgrade -y
```

<br/>

**См. Подробнее:**

https://git.ir/pluralsight-provisioning-virtual-machines-on-oracle-compute-cloud/

<br/>

```
$ oci limits quota list --compartment-id ocid1.compartment.oc1..aaaaaaaaoujbluer6x6sjjtmzzpvkq4reidzyuolihmfsbie5b6tvp35crpq
```
