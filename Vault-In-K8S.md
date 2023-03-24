# Vault in Kubernetes

## Install vault in kubernetes

```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo list
helm repo update
helm install vault hashicorp/vault --set "server.dev.enabled=true"
```

```
kubectl get pods
```
We can see two pods related to vault
- vault-0
- vault-agent-injector-xxxxxx

Lets check what are the properties of the vault-0 pod
```
kubectl describe pod vault-0
```
Check logs of the vault-0 pod and search for token
```
kubectl logs vault-0
```
Difficult to find right we can use this
```
kubectl logs vault-0 | grep "Root Token"
```

Now lets do a simple port forwarding
```
kubectl port-forward vault-0 8200:8200
```

```
export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN="root"
vault auth list
```

## Create Secret

```bash
vault kv put secret/endgame mysecret=123456
vault kv get secret/endgame
```



## Create Policy
> Authorization

```bash
vault policy write endgame-policy ../resources/endgame-policy.hcl
```



## Enable Kubernetes Authentication on Vault
Vault <-> K8S (Api server)

We have seen userpass and token auth in Demo 1 this auth basically allows us to bind k8s service accounts to vault plocies so that the pods tied to the service account will have access to secrets which is defined in the policies.


```bash
vault auth enable kubernetes
vault auth list
```



We first tell Vault where to find  the kubernetes control pane endpoint
to verify the serviceaccount. Pods belonging to the service account will have a token (we have already enabled automountServiceAccountToken) this token will be forwarded to the k8s control pane asking to verify whther this is really the serviceaccount which it says it is.


## Allow Vault to communicate to K8S
Vault -> K8S (Api server)
```
kubectl cluster-info
```

take the cluster endpoint and add it to the below command

```
vault write auth/kubernetes/config kubernetes_host=https://{cluster_endpoint}:6443
```

Once we assign the k8s endpoint to vault, we create a k8s role (not to be confused with the cluster role in K8s) Here role is the attaching a vault policy to the serviceaccount in K8s

## Create Service Account and associate to a policy
> Authentication

```bash
kubectl apply -f ./resources/service-account.yml
```


```bash
vault write auth/kubernetes/role/endgame \
    bound_service_account_names=endgame \
    bound_service_account_namespaces=default \
    policies=endgame-policy \
    ttl=1h
```

## Run deployment
> two containers will be running
> How sidecar injected? - Mutation webhook -> Hijack
> Injector will keep looking into pods that it is interested in it and mutate the pods (figure out using annotiation)
    > Inject
    > What to inject
    > How to inject
    > Who (Permission role)
    > use this service accoint - authentication

```bash
kubectl apply -f ../resources/dobby.yml
kubectl describe pod <dobby-deploy>
```
See that there are 2 containers in a single pod
> vault init container - make sure auth and authentication to vault server
> endgame  - main container
> vault-agent - talk to vault and get data

## Check secret inside pod
Watch the secret
```bash
kubectl exec -it <POD> bash
watch cat vault/secrets/endgame.json
```

## change values and see if pod can sync new data
Please do this in a new terminal.
```bash
vault kv put secret/endgame mysecret=test
vault kv get secret/endgame
```
Check the results of the watch command
