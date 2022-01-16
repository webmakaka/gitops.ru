---
layout: page
title: Oracle Clouds
description: Oracle Clouds
keywords: devops, clouds, Oracle Clouds
permalink: /devops/clouds/oracle/
---

# Oracle Clouds

```
https://www.youtube.com/watch?v=nFlFswEpwnA

Networking -> Virtualc Cloud Networks -> Create VCN

<br/>

С помощью мастера

<br/>

Name: Marley-VCN

CIDR BLOCK: 10.0.0.0/16

<br/>

Default Security List for Marley-VCN

Ingress Rules

10.0.0.0/16 80

<br/>

Compute -> Instances -> Create Instance

marley-instance

ubuntu 20

<br/>

    $ ssh ubuntu@<public-ip>

<br/>

Запускаю приложение на 80 порту

<br/>

Networking -> Load Balancers -> Create Load Balancer

Marley-LB

Public

Virtual Cloud Network -> Marley-VCN

Subnet -> Public Subnet Marley

Next

Weighted Round Robin

Add Backends

Protocol TCP

Next

Listener Name: marley-listener_lb

HTTP

Create

<br/>

VCN -> Default Security List for Marley-VCN

0.0.0.0/0

80

```

<br/>

```

<br/>

### НИЧЕГО НЕ ЗАРАБОТАЛО!!!

Запускаю консоль в панели облаков Oracle

<br/>

### Set up to use the API

    $ oci setup config

<br/>

<Tenancy OCID> взять в левом верхнем углу

Administration -> Tenancy Details

    $ cd ~/.oci/

    $ cat oci_api_key_public.pem

<br/>

Profile -> User Serttings -> API Keys -> Add API Key -> Paste Public Key

<br/>

### Create cloud services for the Linux VM

    $ export TENANCY_OCID=

    $ oci iam compartment create \
        --name testcompartment \
        -c ${TENANCY_OCID} \
        --description "test compartment for linux"

<br/>

## Network setup for Linux VM

    $ export COMPARTMENT_ID=ocid1.compartment.oc1..aaaaaaaaeg3oy2tmo6dfr3v3n4u7gyjqp4bbksddbiasrx7wzzwmu6pynjra

    $ export DISPLAY_NAME="VCN Linux"
    $ export DNS_LABEL="VCNLINUX"

<br/>

    $ oci network vcn create \
        --compartment-id ${COMPARTMENT_ID} \
        --display-name ${DISPLAY_NAME} \
        --dns-label ${DNS_LABEL} \
        --cidr-block "10.0.0.0/24"

<br/>

    $ export VCN_ID=ocid1.vcn.oc1.eu-frankfurt-1.amaaaaaaljudzkaamc37ptz6iin2wa5u4nmgqz3ley22752tu5lkvuhwbvea

    $ export DEFAULT_SECURITY_LIST_ID=ocid1.securitylist.oc1.eu-frankfurt-1.aaaaaaaa7ql5dxwecflkulajs2wutvc7bsfvuie3y55xckwjrh6ktsflrfpa

    $ export DEFAULT_ROUTE_TABLE_ID=ocid1.routetable.oc1.eu-frankfurt-1.aaaaaaaauii3mw3rssclsm7dllz3pjivg7yekmnfrfxrjlqbsrmonzjav4ia

<br/>

    $ oci iam availability-domain list \
        -c ${COMPARTMENT_ID}

<br/>

    $ export AVAILABILITY_DOMAIN=QBXU:EU-FRANKFURT-1-AD-1

<br/>

    $ export DISPLAY_NAME=subnetlinuxtest
    $ export DNS_LABEL=subnetlinuxtest

    $ oci network subnet create \
        --vcn-id ${VCN_ID} \
        -c ${COMPARTMENT_ID} \
        --availability-domain ${AVAILABILITY_DOMAIN} \
        --display-name ${DISPLAY_NAME} \
        --dns-label ${DNS_LABEL} \
        --cidr-block "10.0.0.0/24" \
        --security-list-ids '["ocid1.securitylist.oc1.eu-frankfurt-1.aaaaaaaa7ql5dxwecflkulajs2wutvc7bsfvuie3y55xckwjrh6ktsflrfpa"]'

<br/>

    $ export SUBNET_ID=ocid1.subnet.oc1.eu-frankfurt-1.aaaaaaaarqrgq3slrfd4k6d6nb2fudjpp3uwogxrzru3tidl6oi2mlnxbwqa

<br/>

### Craete Network Internet Gateway

<br/>

    $ export DISPLAY_NAME=LinuxGateWay
    $ export VCN_ID=LinuxGateWay

<br/>

    $ oci network internet-gateway create \
        -c ${COMPARTMENT_ID} \
        --is-enabled true \
        --vcn-id ${VCN_ID} \
        --display-name ${DISPLAY_NAME}

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

    $ oci compute image list \
        -c ${COMPARTMENT_ID}

<br/>

    $ export AVAILABILITY_DOMAIN=QBXU:EU-FRANKFURT-1-AD-3

<br/>

    $ oci compute shape list \
        -c ${COMPARTMENT_ID} \
        --availability-domain ${AVAILABILITY_DOMAIN}

<br/>

    $ ssh-keygen -t rsa

<br/>

    $ oci compute instance launch \
        --availability-domain ${AVAILABILITY_DOMAIN} \
        -c ${COMPARTMENT_ID} \
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
