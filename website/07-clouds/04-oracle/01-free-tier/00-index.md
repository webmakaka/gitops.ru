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

Новый терминал

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

```
$ oci iam compartment list
```

<br/>

```
$ export ROOT_COMPARTMENT_ID=

$ echo ${ROOT_COMPARTMENT_ID}=
$ export COMPARTMENT_NAME=testcompartment

$ oci iam compartment create \
    --name ${COMPARTMENT_NAME} \
    --description "test compartment for linux" \
    --compartment-id ${ROOT_COMPARTMENT_ID}
```

<br/>

```
// compartment-id
$ oci iam compartment list \
  | jq -r -c '.data[] ["compartment-id"]'
```

```
// compartment-id
$ oci iam compartment list \
  | jq -r -c '.data[] ["id"]'
```

<br/>

```
$ COMPARTMENT_ID = результат
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

    $ export SUBNET_DISPLAY_NAME=subnetlinuxtest
    $ export SUBNET_DNS_LABEL=subnetlinuxtest

    $ oci network subnet create \
        --vcn-id ${VCN_ID} \
        --compartment-id ${COMPARTMENT_ID} \
        --availability-domain ${AVAILABILITY_DOMAIN} \
        --display-name ${SUBNET_DISPLAY_NAME} \
        --dns-label ${SUBNET_DNS_LABEL} \
        --cidr-block "10.0.0.0/24" \
        --security-list-ids '["ocid1.securitylist.oc1.eu-frankfurt-1.aaaaaaaa7ql5dxwecflkulajs2wutvc7bsfvuie3y55xckwjrh6ktsflrfpa"]'

<br/>

    $ export SUBNET_ID=ocid1.subnet.oc1.eu-frankfurt-1.aaaaaaaarqrgq3slrfd4k6d6nb2fudjpp3uwogxrzru3tidl6oi2mlnxbwqa

<br/>

### Craete Network Internet Gateway

<br/>

    $ export INTERNET_GATEWAY_DISPLAY_NAME=LinuxGateWay
    $ export INTERNET_GATEWAY_VCN_ID=LinuxGateWay

<br/>

    $ oci network internet-gateway create \
        --compartment-id ${COMPARTMENT_ID} \
        --is-enabled true \
        --vcn-id ${INTERNET_GATEWAY_VCN_ID} \
        --display-name ${INTERNET_GATEWAY_DISPLAY_NAME}

<br/>

    $ export GATEWAY_ID=ocid1.internetgateway.oc1.eu-frankfurt-1.aaaaaaaajungy23o6xt7bpdsrdvg3aexegk2dgowthv3ojm476mpvhbaheba

<br/>

### Adding Route Rules to Route Table

<br/>

    $ oci network route-table update \
        --rt-id ${DEFAULT_ROUTE_TABLE_ID} \
        --route-rules '[
            {"cidrBlock" : "0.0.0.0/0",
            "networkEntityId" : "ocid1.internetgateway.oc1.eu-frankfurt-1.aaaaaaaajungy23o6xt7bpdsrdvg3aexegk2dgowthv3ojm476mpvhbaheba"}
        ]'

<br/>

## Set up and connect to the Linux VM

<br/>

    $ export DISPLAY_NAME="LinuxVMTest"
    $ export IMAGE_ID=ocid1.image.oc1.eu-frankfurt-1.aaaaaaaazixyfjdjd7vsnzsucbnvabadypmijraftu7t6jn5hroxgh35jhuq

<br/>

```
// Получить список образов операционных систем
$ oci compute image list --all \
    --compartment-id ${COMPARTMENT_ID}  \
| jq -r -c '.data[] ["display-name"]'
```

<br/>

    $ ssh-keygen -t rsa

<br/>

    $ oci compute instance launch \
        --availability-domain ${AVAILABILITY_DOMAIN} \
        --compartment-id ${COMPARTMENT_ID} \
        --shape "VM.Standard.E2.1.Micro" \
        --display-name ${DISPLAY_NAME} \
        --image-id ${IMAGE_ID} \
        --ssh-authorized-keys-file "/home/username/.ssh/id_rsa.pub" \
        --subnet-id ${SUBNET_ID}

<br/>

    $ export INSTANCE_ID=ocid1.instance.oc1.eu-frankfurt-1.antheljtljudzkacvzoqzlcx3nkgraya434v4npmtnl7mdtabtylxpa4wcaa

<br/>

    $ oci compute instance list-vnics \
        --instance-id ${INSTANCE_ID}

<br/>

<br/>

**См. Подробнее:**

https://git.ir/pluralsight-provisioning-virtual-machines-on-oracle-compute-cloud/

```

```
