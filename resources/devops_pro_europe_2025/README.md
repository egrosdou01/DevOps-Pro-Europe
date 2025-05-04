# DevOps Pro Europe 2025

## Introduction
The repository is dedicated to the DevOps Pro Europe Conference presentation of [Sveltos](https://github.com/projectsveltos).

## How to

### Assumptions
- We assume a Kubernetes **management cluster** is already in place with **Sveltos** installed. Sveltos installation details can be found [here](https://projectsveltos.github.io/sveltos/getting_started/install/install/)
- We assume [FluxCD](https://fluxcd.io/) is already installed in the **management cluster**
- We assume the Kubernetes **managed clusters** are in place with **no CNI**

### Prerequisites
- Install `kubectl` and `sveltosctl`
- Obtain the `kubeconfig` of the clusters

**Note**: Download the `sveltosctl` [here](https://github.com/projectsveltos/sveltosctl/releases).

### Management Cluster - Create Required Namespaces
```bash
$ kubectl apply -f resources/namespaces/ns_test.yaml,resources/namespaces/ns_prod.yaml
```

### Management Cluster - Register Clusters with Sveltos

```bash
$ ./sveltosctl register cluster \
--namespace=prod --cluster=test01 \
--kubeconfig=/path/to/kubeconfig --labels=env=prod

$ ./sveltosctl register cluster \
--namespace=test --cluster=prod01 \
--kubeconfig=/path/to/kubeconfig --labels=env=test
```

#### Validate Registration

```bash
$ kubectl get sveltosclusters -A --show-labels
```

### Management Cluster - Deploy Cilium

```bash
$ kubectl apply -f cluster_resources/clusterprofile_cilium_tpl.yaml
```

#### Validate Deployment

```bash
$ ./sveltosctl show addons
```

### Management Cluster - Deploy Kyverno and Policies

```bash
$ kubectl apply -f cluster_resources/clusterprofile_kyverno_tpl.yaml
```

#### Validate Deployment

```bash
$ ./sveltosctl show addons
```

**Note**: Feel free to update the Helm chart versions mentioned in the manifest files.

### Management Cluster - Sveltos Dashboard

The Sveltos Dashboard installation details are found [here](https://projectsveltos.github.io/sveltos/getting_started/optional/dashboard/).

```bash
$ export KUBECONFIG=/path/to/management cluster kubeconfig/
$ echo "Deploy the Sveltos Dashboard"
$ kubectl apply -f https://raw.githubusercontent.com/projectsveltos/sveltos/main/manifest/dashboard-manifest.yaml
$ kubectl get pods,svc -n projectsveltos | grep -i 'dashboard'
```

```bash
$ echo "Expose the Sveltos Dashboard via a LoadBalancer Service"
$ kubectl patch svc dashboard -n projectsveltos -p '{\"spec\":{\"type\":\"LoadBalancer\"}}'
```

```bash
$ echo "Create a valid user"
$ kubectl create sa platform-admin
$ kubectl create clusterrolebinding platform-admin-access --clusterrole cluster-admin --serviceaccount default:platform-admin

$ echo "Create a token valid for 1 hour"
$ kubectl create token platform-admin --duration=1h
$ kubectl get svc dashboard -n projectsveltos -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```

### Access Resources

#### Hubble UI
The access to the Cilium Hubble UI, we can either use `kubectl port-forward` or update the `hubble-ui` service to `NodePort` or `LoadBalancer`.

For more details about the Hubble UI, check out the [link](https://docs.cilium.io/en/v1.16/observability/hubble/hubble-ui/index.html).

#### Sveltos Dashboard UI
Choose your favourite browser and open the `LoadBalancer` IP address generated in a previous step. Login using the generated token for the `serviceaccount` `platform-admin`.