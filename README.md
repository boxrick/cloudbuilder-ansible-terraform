# cloudbuilder-ansible-terraform

Abandoned since the the current use case is a little simple. But kept for future reference. Working cloud builder with encrypted files and various build steps for Terraform Ansible and Gcloud

### Create bucket to store encrypted keys in
```gsutil mb gs://rick-test-cloudbuider-bucket/ -p GCPPROJECT -l EU```
### Create a service account with permissions for cloud builder
```
Permissions tested with:

Cloud Build Service Account
Cloud KMS CryptoKey Decrypter
Compute Instance Admin (beta)
Compute OS Admin Login
Service Account User
```
### Copy ssh key and service account key to current directory
```cp FOLDERHERE/id_rsa ./```

```cp FOLDERHERE/service_account.json ./```

### Create kms symmetric key and use this to encrypt required values
```
gcloud kms keyrings create rick-cloudbuild-test --location=global --project GCPPROJECT

gcloud kms encrypt --plaintext-file=./id_rsa --ciphertext-file=./id_rsa.enc --location=global --keyring=rick-cloudbuild-test --key=github-key --project GCPPROJECT

gcloud kms encrypt --plaintext-file=./service_account.json --ciphertext-file=./service_account.json.enc --location=global --keyring=rick-cloudbuild-test --key=github-key --project GCPPROJECT
```

### Copy encrypted values to bucket
```gsutil cp ./service_account.json.enc ./id_rsa.enc gs://rick-test-cloudbuider-bucket/```

### Build docker image and push to GCR
docker build -t ansible-terraform-gcloud .
docker tag ansible-terraform-gcloud eu.gcr.io/GCPPROJECT/ansible-terraform-gcloud
docker push eu.gcr.io/GCPPROJECT/ansible-terraform-gcloud

### Run gcloud build
gcloud builds submit . --config=cloudbuild.yaml --project=GCPPROJECT
