# vault-init

The `vault-init` service automates the process of [initializing](https://www.vaultproject.io/docs/commands/operator/init.html) and [unsealing](https://www.vaultproject.io/docs/concepts/seal.html#unsealing) HashiCorp Vault instances running on [Google Cloud Platform](https://cloud.google.com).

After `vault-init` initializes a Vault server it stores master keys and root tokens, encrypted using [Google Cloud KMS](https://cloud.google.com/kms), to a user defined [Google Cloud Storage](https://cloud.google.com/storage) bucket.

## Usage

The `vault-init` service is designed to be run alongside a Vault server and
communicate over local host.

You can download the code and compile the binary with Go. Alternatively, a
Docker container is available via the Docker Hub:

```text
$ docker pull sethvargo/vault-init
```

To use this as part of a Kubernetes Vault Deployment:

```yaml
containers:
- name: vault-init
  image: registry.hub.docker.com/sethvargo/vault-init:0.1.1
  imagePullPolicy: Always
  env:
  - name: GCS_BUCKET_NAME
    value: my-gcs-bucket
  - name: KMS_KEY_ID
    value: my-kms-key-id
```

## Configuration

The vault-init service supports the following environment variables for configuration:

* `CHECK_INTERVAL` - The time in seconds between Vault health checks. (300)
* `GCS_BUCKET_NAME` - The Google Cloud Storage Bucket where the vault master key and root token is stored.
* `KMS_KEY_ID` - The Google Cloud KMS key ID used to encrypt and decrypt the vault master key and root token.

### Example Values

```
CHECK_INTERVAL="300"
GCS_BUCKET_NAME="vault-storage"
KMS_KEY_ID="projects/my-project/locations/global/keyRings/my-keyring/cryptoKeys/key"
```

### IAM &amp; Permissions

The `vault-init` service uses the official Google Cloud Golang SDK. This means
it supports the common ways of [providing credentials to GCP][cloud-creds].

To use this service, the service account must have the following minimum
scope(s):

```text
https://www.googleapis.com/auth/cloudkms
https://www.googleapis.com/auth/devstorage.read_write
```

Additionally, the service account must have the following minimum role(s):

```text
roles/cloudkms.cryptoKeyEncrypterDecrypter
roles/storage.objectAdmin OR roles/storage.legacyBucketWriter
```

For more information on service accounts, please see the
[Google Cloud Service Accounts documentation][service-accounts].

[cloud-creds]: https://cloud.google.com/docs/authentication/production#providing_credentials_to_your_application
[service-accounts]: https://cloud.google.com/compute/docs/access/service-accounts
