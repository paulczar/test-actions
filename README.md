# Google Cloud Platform Operator

This is a demo operator for the [Google Cloud Platform](https://cloud.google.com) which simplifies requesting google cloud resources in the form on Kubernetes Manifests.

The goal of the Operator is to provide a bare minimum set of Kubernetes CRDs to enable provisioning GCP services. To enable this the manifest Specs are generally a 1:1 mapping to the GCP API objects.

Currently supports creating and destroying the following GCP Services:

* Addresses
* Images
* Firewalls
* Networks
* Subnetworks
* Forwarding Rules
* Target Pools
* Instances



## Add new gcp compute service

```
API=compute.google.golang.org/v1 KIND=Image
operator-sdk add api --api-version=$API --kind=$KIND
operator-sdk add controller --api-version=$API --kind=$KIND
```

## Annotations

You can set the following Annotations:

| Annotation | Description |
| ---------- | ----------- |
| `compute.gce/project-id` | Sets the GCP Project ID if different to that used in operator service account |


## Example Usage

Create a namespace to run the operator in:

```bash
kubectl create namespace gcp-operator
```

Create a secret containing your GCP account credentials:

```bash
kubectl -n gcp-operator create secret \
    generic gcp-operator \
  --from-file=google.json=/path/to/credentials.json
```

If using GKE you need to ensure your user has the cluster admin role binding:

```bash
kubectl create clusterrolebinding cluster-admin-binding \
    --clusterrole=cluster-admin --user=<your gcp account email address>
clusterrolebinding.rbac.authorization.k8s.io/cluster-admin-binding created
```

Deploy the GCP Operator:

```bash
kubectl -n gcp-operator apply -f deploy/service_account.yaml
kubectl -n gcp-operator apply -f deploy/role.yaml
kubectl -n gcp-operator apply -f deploy/role_binding.yaml
kubectl -n gcp-operator apply -f deploy/operator.yaml
```

Deploy the CRDs:
```bash
kubectl apply -f deploy/crds/compute_v1_address_crd.yaml
kubectl apply -f deploy/crds/compute_v1_firewall_crd.yaml
kubectl apply -f deploy/crds/compute_v1_forwardingrule_crd.yaml
kubectl apply -f deploy/crds/compute_v1_image_crd.yaml
kubectl apply -f deploy/crds/compute_v1_instance_crd.yaml
kubectl apply -f deploy/crds/compute_v1_network_crd.yaml
kubectl apply -f deploy/crds/compute_v1_subnetwork_crd.yaml
kubectl apply -f deploy/crds/compute_v1_targetpool_crd.yaml
```

### Create GCP Address

Edit `deploy/examples/address.yaml` replacing the project ID placeholders with your GCP project.

Once the GCP Operator is deployed you can create a GCP instance:

```bash
kubectl -n gcp-operator apply -f deploy/examples/address.yaml
```

After a few minutes check to see if the new instance exists:

```bash
gcloud compute addresses list
NAME                   REGION       ADDRESS          STATUS
example                us-central1  35.226.61.203    RESERVED

```

### Create GCP Instance

Once the GCP Operator is deployed you can create a GCP instance:

> Note: you'll need to edit this file first

```bash
kubectl -n gcp-operator apply -f deploy/examples/instance.yaml
```

After a few minutes check to see if the new instance exists:

```bash
gcloud compute instances list
NAME                                     ZONE           MACHINE_TYPE               PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
test                                     us-central1-a  custom (2 vCPU, 4.00 GiB)               10.128.0.2                   RUNNING
```

Cleanup:

```
kubectl delete -f deploy
kubectl delete -f deploy/crds
kubectl delete -f deploy/examples

```
