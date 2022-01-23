---
layout: page
title: Oracle Cloud Free Tier
description: Oracle Cloud Free Tier
keywords: Oracle Cloud Free Tier
permalink: /clouds/oracle/free-tier/
---

# Oracle Cloud Free Tier

<br/>

### [Oracle Cloud Free Tier](/clouds/oracle/free-tier/info/)

<br/>

**Инсталляция OCI в ubuntu**

```
$ bash -c "$(curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)"
```

<br/>

```
// Чтобы shell мог выполнять команды oci
$ exec -l $SHELL
```

<br/>

```
oci --version
3.4.2
```

<br/>

**Configure the OCI CLI**
https://www.youtube.com/watch?v=x2iWGXIa-rQ

<br/>

```
$ oci setup config
```

<br/>

```
$ ls -la ~/.oci/
```

<br/>

Oracle Cloud Web UI -> Profile -> User-Settigns -> API-Keys -> Add -> Choose Public Key File -> Add

<br/>

```
$ oci iam user list
```

<br/>
<br/>
<br/>
<br/>
<br/>
<br/>

<br/>

### Create cloud services for the Linux VM

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

$ export COMPARTMENT_NAME=testcompartment

$ oci iam compartment create \
    --name ${COMPARTMENT_NAME} \
    --description "test compartment for linux" \
    --compartment-id ${TENANCY_COMPARTMENT_ID}
```

<br/>

```
// compartment-id
$ oci iam compartment list \
  | jq -r -c '.data[] ["id"]'
```

<br/>

```
$ export COMPARTMENT_ID=результат
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
$ export VCN_ID=
```

<br/>

```
$ oci network vcn list \
    --compartment-id ${COMPARTMENT_ID} \
  | jq -r -c '.data[] ["id"]'
```

<br/>

```
$ export DEFAULT_SECURITY_LIST_ID=
```

<br/>

```
$ oci network vcn list \
    --compartment-id ${COMPARTMENT_ID} \
  | jq -r -c '.data[] ["default-security-list-id"]'
```

<br/>

```
$ export DEFAULT_ROUTE_TABLE_ID=
```

<br/>

```
$ oci network vcn list \
    --compartment-id ${COMPARTMENT_ID} \
  | jq -r -c '.data[] ["default-route-table-id"]'
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

$ oci network subnet create \
    --vcn-id ${VCN_ID} \
    --compartment-id ${COMPARTMENT_ID} \
    --availability-domain ${AVAILABILITY_DOMAIN} \
    --display-name ${SUBNET_DISPLAY_NAME} \
    --dns-label ${SUBNET_DNS_LABEL} \
    --cidr-block "10.0.0.0/24" \
    --security-list-ids '["<security_list_id>"]'
```

<br/>

```
$ export SUBNET_ID=ocid1.subnet.oc1.eu-frankfurt-1.aaaaaaaarqrgq3slrfd4k6d6nb2fudjpp3uwogxrzru3tidl6oi2mlnxbwqa

$ export ROUTE_TABLE_ID=ocid1.routetable.oc1.eu-frankfurt-1.aaaaaaaawhijpni6r35f26rshruartz2svegn47afxu4z4ntvy6qrj4treuq
```

<br/>

### Craete Network Internet Gateway

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
$ export GATEWAY_ID=ocid1.internetgateway.oc1.eu-frankfurt-1.aaaaaaaajungy23o6xt7bpdsrdvg3aexegk2dgowthv3ojm476mpvhbaheba
```

<br/>

### Adding Route Rules to Route Table

<br/>

```
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
$ export IMAGE_ID=ocid1.image.oc1.eu-frankfurt-1.aaaaaaaadqrjjiunkzkf62ggllx56s3p5775gonlifl74d4ri3bykztb4bha
```

<br/>

```
$ ssh-keygen -t rsa
```

<br/>

```
$ oci compute instance launch \
    --availability-domain ${AVAILABILITY_DOMAIN} \
    --compartment-id ${COMPARTMENT_ID} \
    --shape "VM.Standard.E2.1.Micro" \
    --display-name ${DISPLAY_NAME} \
    --image-id ${IMAGE_ID} \
    --ssh-authorized-keys-file "/home/username/.ssh/id_rsa.pub" \
    --subnet-id ${SUBNET_ID}
```

<br/>

```
$ export INSTANCE_ID=ocid1.instance.oc1.eu-frankfurt-1.antheljtljudzkacezkneuv5a5k5gftp6hgbepdmwlaz3snq4fh73c65kuiq
```

<br/>

```
$ oci compute instance list-vnics \
    --instance-id ${INSTANCE_ID}
```

<br/>

```
$ export PUBLIC_IP=<PUBLIC_IP>
```

<br/>

```
$ ssh ubuntu@${PUBLIC_IP}



// Если не ubuntu
$ ssh opc@${PUBLIC_IP}
```

<br/>

## Добавить диск

<br/>

```
// Create block volume
$ oci bv volume create \
    --availability-domain ${AVAILABILITY_DOMAIN} \
    --compartment-id ${COMPARTMENT_ID} \
    --size-in-mbs 51200
    --display-name VOLUME_DISPLAY_NAME
```

<br/>

```
$ export VOLUME_ID=ocid1.volume.oc1.eu-frankfurt-1.abtheljtdfi26t4mjt3rv5z2bmeo4enkgm2givrvm4oiqs4ys6u6umj4jsmq
```

<br/>

```
// Check status of the block volume
$ oci bv volume get \
    --volume-id ${VOLUME_ID}
```

<br/>

```
// Add block volume to instance
$ oci compute volume-attachment attach \
    --instance-id ${INSTANCE_ID} \
    --type iscsi \
    --volume-id ${VOLUME_ID}
```

<br/>

```
$ oci compute volume-attachment list \
    --instance-id ${INSTANCE_ID}
```

<br/>

```
$ export ATTACHMENT_ID=ocid1.volumeattachment.oc1.eu-frankfurt-1.antheljtljudzkacz7yiszxkj27qo6bslw3q7pwehhb2oobpa474ycj4phva
```

```
$ oci compute volume-attachment get \
    --volume-attachment-id ${ATTACHMENT_ID}
```

<br/>

**См. Подробнее:**

https://git.ir/pluralsight-provisioning-virtual-machines-on-oracle-compute-cloud/

<br/>

### Доступ к облаку Oracle по http

```
$ python -m http.server 8000
```
