---
layout: page
title: Binary Authorization
description: Binary Authorization
keywords: Binary Authorization
permalink: /devops/clouds/google/gke/qwiklabs/kubernetes-best-practices-security/binary-authorization/
---

# [GSP479] Binary Authorization

<br/>

Делаю:  
08.06.2019

https://www.qwiklabs.com/focuses/5154?parent=catalog

Binary Authorization is a GCP managed service that works closely with GKE to enforce deploy-time security controls to ensure that only trusted container images are deployed. With Binary Authorization you can whitelist container registries, require images to be signed by trusted authorities, and centrally enforce those policies. By enforcing this policy, you can gain tighter control over your container environment by ensuring only approved and/or verified images are integrated into the build-and-release process.

<br/>

### Architecture

The Binary Authorization and Container Analysis APIs are based upon the open source projects Grafeas and Kritis.

-   Grafeas defines an API spec for managing metadata about software resources, such as container images, Virtual Machine (VM) images, JAR files, and scripts. You can use Grafeas to define and aggregate information about your project’s components.

-   Kritis defines an API for ensuring a deployment is prevented unless the artifact (container image) is conformant to central policy and optionally has the necessary attestations present.
    In a simplified container deployment pipeline such as this:

<br/>

    $ gcloud config set compute/region us-central1
    $ gcloud config set compute/zone us-central1-a

    $ git clone https://github.com/GoogleCloudPlatform/gke-binary-auth-demo.git
    $ cd gke-binary-auth-demo
    $ ./create.sh -c my-cluster-1

<br/>

The script will:

1. Enable the necessary APIs in your project. Specifically, container, containerregistry, containeranalysis, and binaryauthorization.
2. Create a new Kubernetes Engine cluster in your default ZONE, VPC and network.
3. Retrieve your cluster credentials to enable kubectl usage.

<br/>

    $ ./validate.sh -c my-cluster-1

<br/>

### Using Binary Authorization

Security > Binary Authorization > Configure Policy.

Configure Policy

-   Change your project default rule to Disallow all images
-   Click down arrow in the Rules section, then click Add Rule.
-   In the Add cluster-specific rule field, enter your location and cluster name in the form location.clustername. e.g. us-central1-a.my-cluster-1 which corresponds to the zone us-central1-a and the cluster name my-cluster-1.
-   Select the default rule of Allow all images for your cluster.
-   Click Add.
-   Click Save Policy.

<br/>

### Creating a Private GCR Image

    $ docker pull gcr.io/google-containers/nginx:latest
    $ gcloud auth configure-docker
    $ PROJECT_ID="$(gcloud config get-value project)"
    $ docker tag gcr.io/google-containers/nginx "gcr.io/${PROJECT_ID}/nginx:latest"
    $ docker push "gcr.io/${PROJECT_ID}/nginx:latest"
    $ gcloud container images list-tags "gcr.io/${PROJECT_ID}/nginx"

<br/>

```
$ cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: "gcr.io/${PROJECT_ID}/nginx:latest"
    ports:
    - containerPort: 80
EOF
```

<br/>

    $ kubectl get pods
    $ kubectl delete pod nginx

Next, prove that the Binary Authorization policy can block undesired images from running in the cluster.

<br/>

On the Binary Authorization page, click Edit Policy

Expand the Rules dropdown, click on the three vertical dots to the right of your Cluster rule, and click edit. Then, select Disallow all images, click Submit.

Finally, click Save Policy to apply those changes.

Important: Wait at least 30 seconds before proceeding to allow the policy to take effect.

Выполняем тот же скрипт, но теперь получаем ошибку.

<br/>

### Смотрим логи

Navigation menu > Logging --> Advanced Filter

    resource.type="k8s_cluster" protoPayload.status.message="PERMISSION_DENIED"

<br/>

    $ echo "gcr.io/${PROJECT_ID}/nginx*"
    gcr.io/qwiklabs-gcp-4d48b14a6965df4e/nginx*

Теперь этот вывод нужно добавить в policy.

<br/>

Images exempt from deployment rules

Add

Выполняем тот же скрипт. Все запустилось.

    $ kubectl delete pod nginx

<br/>

### Теперь еще и с подписью

    $ ATTESTOR="manually-verified"
    $ ATTESTOR_NAME="Manual Attestor"
    $ ATTESTOR_EMAIL="$(gcloud config get-value core/account)" # This uses your current user/email

<br/>

    $ NOTE_ID="Human-Attestor-Note"
    $ NOTE_DESC="Human Attestation Note Demo"

<br/>

    $ NOTE_PAYLOAD_PATH="note_payload.json"
    $ IAM_REQUEST_JSON="iam_request.json"

<br/>

// Create the ATTESTATION note payload:

```
$ cat > ${NOTE_PAYLOAD_PATH} << EOF
{
  "name": "projects/${PROJECT_ID}/notes/${NOTE_ID}",
  "attestation_authority": {
    "hint": {
      "human_readable_name": "${NOTE_DESC}"
    }
  }
}
EOF

```

<br/>

// Submit the ATTESTATION note to the Container Analysis API

    $ curl -X POST \
        -H "Content-Type: application/json" \
        -H "Authorization: Bearer $(gcloud auth print-access-token)"  \
        --data-binary @${NOTE_PAYLOAD_PATH}  \
        "https://containeranalysis.googleapis.com/v1beta1/projects/${PROJECT_ID}/notes/?noteId=${NOTE_ID}"

<br/>

    $ curl -H "Authorization: Bearer $(gcloud auth print-access-token)"  \
        "https://containeranalysis.googleapis.com/v1beta1/projects/${PROJECT_ID}/notes/${NOTE_ID}"

<br/>

### Creating a PGP Signing Key

Create a new PGP key and export the public PGP key.

    $ PGP_PUB_KEY="generated-key.pgp"

<br/>

    $ sudo apt-get install rng-tools
    $ sudo rngd -r /dev/urandom
    $ gpg --quick-generate-key --yes ${ATTESTOR_EMAIL}

Extract the public PGP key:

    $ gpg --armor --export "${ATTESTOR_EMAIL}" > ${PGP_PUB_KEY}

Create the Attestor in the Binary Authorization API:

    $ gcloud --project="${PROJECT_ID}" \
        beta container binauthz attestors create "${ATTESTOR}" \
        --attestation-authority-note="${NOTE_ID}" \
        --attestation-authority-note-project="${PROJECT_ID}"

<br/>

Add the PGP Key to the Attestor:

    $ gcloud --project="${PROJECT_ID}" \
        beta container binauthz attestors public-keys add \
        --attestor="${ATTESTOR}" \
        --public-key-file="${PGP_PUB_KEY}"

<br/>

    $ gcloud --project="${PROJECT_ID}" \
        beta container binauthz attestors list

<br/>

### "Signing" a Container Image

The preceeding steps only need to be performed once. From this point on, this step is the only step that needs repeating for every new container image.

    $ GENERATED_PAYLOAD="generated_payload.json"
    $ GENERATED_SIGNATURE="generated_signature.pgp"

Get the PGP fingerprint:

    $ PGP_FINGERPRINT="$(gpg --list-keys ${ATTESTOR_EMAIL} | head -2 | tail -1 | awk '{print $1}')"

Obtain the SHA256 Digest of the container image:

    $ IMAGE_PATH="gcr.io/${PROJECT_ID}/nginx"

    $ IMAGE_DIGEST="$(gcloud container images list-tags --format='get(digest)' $IMAGE_PATH | head -1)"

Create a JSON-formatted signature payload:

    $ gcloud beta container binauthz create-signature-payload \
    --artifact-url="${IMAGE_PATH}@${IMAGE_DIGEST}" > ${GENERATED_PAYLOAD}

View the generated signature payload:

    $ cat "${GENERATED_PAYLOAD}"

"Sign" the payload with the PGP key:

    $ gpg --local-user "${ATTESTOR_EMAIL}" \
        --armor \
        --output ${GENERATED_SIGNATURE} \
        --sign ${GENERATED_PAYLOAD}

View the generated signature (PGP message):

    $ cat "${GENERATED_SIGNATURE}"

Create the attestation:

    $ gcloud beta container binauthz attestations create \
        --artifact-url="${IMAGE_PATH}@${IMAGE_DIGEST}" \
        --attestor="projects/${PROJECT_ID}/attestors/${ATTESTOR}" \
        --signature-file=${GENERATED_SIGNATURE} \
        --pgp-key-fingerprint="${PGP_FINGERPRINT}"

View the newly created attestation:

    $ gcloud beta container binauthz attestations list \
        --attestor="projects/${PROJECT_ID}/attestors/${ATTESTOR}"

<br/>

### Running an Image with Attestation Enforcement Enabled

The next step is to change the Binary Authorization policy to enforce that attestation is to be present on all images that do not match the whitelist pattern(s).

To change the policy to require attestation, run the following and then copy the full path/name of the attestation authority:

    $ echo "projects/${PROJECT_ID}/attestors/${ATTESTOR}"
    projects/qwiklabs-gcp-4d48b14a6965df4e/attestors/manually-verified

<br/>

Binary Authorization policy --> edit policy

Allow only images that have been approved by all of the following attestors instead of Disallow all images in the pop-up window.:

Click the three dots by your cluster name to Edit your cluster rules.

<br/>

Next, click on Add Attestors followed by Add attestor by resource ID. Enter the contents of your copy/paste buffer in the format of projects/${PROJECT_ID}/attestors/${ATTESTOR}, then click Add 1 Attestor.

Add by attestor resource ID.

Submit, and finally Save Policy.

<br/>

The default policy should still show Disallow all images, but the cluster-specific rule should be requiring attestation.

Now, obtain the most recent SHA256 Digest of the signed image from the previous steps:

    $ IMAGE_PATH="gcr.io/${PROJECT_ID}/nginx"

    $ IMAGE_DIGEST="$(gcloud container images list-tags --format='get(digest)' $IMAGE_PATH | head -1)"

<br/>

```
$ cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: "${IMAGE_PATH}@${IMAGE_DIGEST}"
    ports:
    - containerPort: 80
EOF

```

<br/>

### Tear Down

    $ ./delete.sh -c my-cluster-1

    $ gcloud container images delete "${IMAGE_PATH}@${IMAGE_DIGEST}" --force-delete-tags

    $ gcloud --project="${PROJECT_ID}" \
    beta container binauthz attestors delete "${ATTESTOR}"

    $ curl -X DELETE \
    -H "Authorization: Bearer $(gcloud auth print-access-token)" \
    "https://containeranalysis.googleapis.com/v1beta1/projects/${PROJECT_ID}/notes/${NOTE_ID}"
