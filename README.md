# home-ops

## Setting up

### Initialize Environment

To initialize the environment, run the following command:

```shell
task init
```

To clean the repository and start off fresh, execute the following command:

```shell
task setup:cleanup
```

### Configure SOPS

To create the SOPS key, run:

```shell
task sops:setup
```

Replace `<insert your key here>` with your newly created public key in the `.sops.yaml` file at the root of this repository.

Create the `cluster-secrets.sops.yaml` file in `./cluster/flux/vars/` with the following content:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: cluster-secrets
  namespace: flux-system
stringData:
  SOME_SECRET: SOME_VALUE
```

Create the `github-deploy-key.sops.yaml` file in `./cluster/bootstrap/flux/` with the following content:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: github-deploy-key
  namespace: flux-system
type: Opaque
data:
  identity: YOUR_BASE64_ENCODED_PRIVATE_KEY_USED_TO_PULL_SOURCE_CODE
```

The base64 encoded private key can be generated with the following command:

```shell
base64 -w 0 <private_key_path>
```

Now, encrypt all your `*.sops.yaml` files to avoid leaking any sensitive information when pushing to your Git repository:

```shell
task sops:encrypt-all
```

### Adjust Machine Configuration

Edit `./cluster/bootstrap/talos/configs/talconfig.yaml` and ensure it matches your machines.

Generate per-machine Talos configs:

```shell
task talos:generate
```

## Deploy Cluster

Apply machine configurations to the machines:

```shell
task talos:setup cluster=<your cluster name>
```

Fetch the kubeconfig file:

```shell
task talos:fetch-kubeconfig cluster=<your cluster name>
```

List all pods in the cluster:

```shell
export KUBECONFIG=./kubeconfig

$ kubectl get pods -A
NAMESPACE     NAME                                 READY   STATUS    RESTARTS   AGE
kube-system   coredns-68d75fd545-8glfv             1/1     Running   0          80s
kube-system   coredns-68d75fd545-kbb9x             1/1     Running   0          80s
kube-system   kube-apiserver-control-01            1/1     Running   0          12s
kube-system   kube-controller-manager-control-01   0/1     Pending   0          5s
kube-system   kube-flannel-6f2vv                   1/1     Running   0          80s
kube-system   kube-flannel-cdtk7                   1/1     Running   0          76s
kube-system   kube-flannel-mpvqk                   1/1     Running   0          79s
kube-system   kube-proxy-ddb9d                     1/1     Running   0          80s
kube-system   kube-proxy-sp6k9                     1/1     Running   0          79s
kube-system   kube-proxy-tkc4f                     1/1     Running   0          76s
kube-system   kube-scheduler-control-01            0/1     Pending   0          4s
```

Adjust Github repository link in `cluster/flux/config/cluster.yaml` to match your repository.

**Important:** commit & push changes to git repository before deploying Flux.

Deploy Flux:

```shell
task flux:setup cluster=<your cluster name>
```