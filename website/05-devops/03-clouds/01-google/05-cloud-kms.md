---
layout: page
title: Cloud KMS
description: Cloud KMS
keywords: Cloud KMS
permalink: /devops/clouds/google/cloud-kms/
---

# Cloud KMS

Cloud KMS is a cryptographic key management service on GCP. Before using KMS you need to enable it in your project.

    $ gcloud services enable cloudkms.googleapis.com

<br/>

    $ KEYRING_NAME=test CRYPTOKEY_NAME=qwiklab
    $ gcloud kms keyrings create $KEYRING_NAME --location global

    $ gcloud kms keys create $CRYPTOKEY_NAME --location global \
          --keyring $KEYRING_NAME \
          --purpose encryption

<br/>

Navigation menu > IAM & Admin > Cryptogrphic keys

<br/>

### Encrypt Your Data

    $ PLAINTEXT=$(cat 1. | base64 -w0)

    $ curl -v "https://cloudkms.googleapis.com/v1/projects/$DEVSHELL_PROJECT_ID/locations/global/keyRings/$KEYRING_NAME/cryptoKeys/$CRYPTOKEY_NAME:encrypt" \
    -d "{\"plaintext\":\"$PLAINTEXT\"}" \
    -H "Authorization:Bearer $(gcloud auth application-default print-access-token)"\
    -H "Content-Type: application/json"

<br/>

    $ curl -v "https://cloudkms.googleapis.com/v1/projects/$DEVSHELL_PROJECT_ID/locations/global/keyRings/$KEYRING_NAME/cryptoKeys/$CRYPTOKEY_NAME:encrypt" \
      -d "{\"plaintext\":\"$PLAINTEXT\"}" \
      -H "Authorization:Bearer $(gcloud auth application-default print-access-token)"\
      -H "Content-Type:application/json" \
    | jq .ciphertext -r > 1.encrypted

<br/>

    $ curl -v "https://cloudkms.googleapis.com/v1/projects/$DEVSHELL_PROJECT_ID/locations/global/keyRings/$KEYRING_NAME/cryptoKeys/$CRYPTOKEY_NAME:decrypt" \
    -d "{\"ciphertext\":\"$(cat 1.encrypted)\"}" \
    -H "Authorization:Bearer $(gcloud auth application-default print-access-token)"\
    -H "Content-Type:application/json" \

| jq .plaintext -r | base64 -d

<br/>

### Configure IAM Permissions

    $ USER_EMAIL=$(gcloud auth list --limit=1 2>/dev/null | grep '@' | awk '{print $2}')

    $ gcloud kms keyrings add-iam-policy-binding $KEYRING_NAME \
    --location global \
    --member user:$USER_EMAIL \
    --role roles/cloudkms.admin

    $ gcloud kms keyrings add-iam-policy-binding $KEYRING_NAME \
    --location global \
    --member user:$USER_EMAIL \
    --role roles/cloudkms.cryptoKeyEncrypterDecrypter
