# Documentation

## Cluster deployment
---

To start a fresh MicroK8s cluster, please follow [the guide here](https://medium.com/@emilfabrice/deploy-microk8s-kubernetes-on-ubuntu-with-ansible-844f22e154a0)


Additionally, as at the time of writing installation of `metallb` on MicroK8S [does not work via Ansible](https://github.com/istvano/ansible_role_microk8s/issues/35), it should be done manually on the controll node:

```bash
microk8s enable metallb:10.0.2.200-10.0.2.220
```
*metallb will allow exposing services using load balancers, eliminating the need of port forwarding!*

## Add sample Helm application
---

Follow steps [here](https://artifacthub.io/packages/helm/bitnami/nginx) with some adjustments:

```bash
# install the bitnami chart repo
helm repo add bitnami https://charts.bitnami.com/bitnami

# (optional) verify the repo is present
helm repo ls
#NAME    URL
#bitnami https://charts.bitnami.com/bitnami

# install the application
helm install webserver bitnami/nginx
```

As per the instructions displayed on the screen, to access the deployed webserver, you should execute:

```bash
    export SERVICE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].port}" services webserver-nginx)
    export SERVICE_IP=$(kubectl get svc --namespace default webserver-nginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    echo "http://${SERVICE_IP}:${SERVICE_PORT}"
    ## http://10.0.2.200:80
```

At this point the webserver should be accessible at the above ip/port combination.

## Helm with values
---

The previous example was deploying Nginx with the default values. Let's see how can we deploy a different application with specified values. In this case WordPress.

So steps are:

- update the `wordpress-values.yml`

- install this helm chart

```bash
 helm install wordpress bitnami/wordpress --values=helm/wordpress-values.yml
```

- Find the required values for the connection
```bash

# get the connection details
 export SERVICE_IP=$(kubectl get svc --namespace default wordpress --include "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
   echo "WordPress URL: http://$SERVICE_IP/"
   echo "WordPress Admin URL: http://$SERVICE_IP/admin"

# if above fails, just do
kubectl get service wordpress
## NAME        TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
## wordpress   LoadBalancer   10.152.183.65   10.0.2.201    80:31311/TCP,443:32060/TCP   3m31s
# and use the extenal IP returned for connection, so for example: http://10.0.2.201/admin in this case

# get the password with
echo Password: $(kubectl get secret --namespace default wordpress -o jsonpath="{.data.wordpress-password}" | base64 -d)

```

## A more complex Helm deployment:Hashicorp - Vault
---

[refernce](https://developer.hashicorp.com/vault/tutorials/kubernetes/kubernetes-raft-deployment-guide)

```bash

# create namespace
kubectl create namespace vault

# add Helm repo
helm repo add hashicorp https://helm.releases.hashicorp.com

# install Vault
helm install vault hashicorp/vault --namespace vault --values=helm/vault-values.yml

# initialise vault (ref: https://github.com/hashicorp/vault-helm/issues/17)
kubens vault
kubectl exec -ti vault-0 -- vault operator init

# unseal vault (repeat this 3x times)
 kubectl exec -ti vault-0 -- vault operator unseal
##Unseal Key (will be hidden):
##Key                Value
##---                -----
##Seal Type          shamir
##Initialized        true
##Sealed             true
##Total Shares       5
##Threshold          3
##Unseal Progress    1/3
##Unseal Nonce       ce19b942-d591-1fab-1f15-70442c53052b
##Version            1.12.1
##Build Date         2022-10-27T12:32:05Z
##Storage Type       file
##HA Enabled         false
```

After the helm install, vault's liveness probe will fail. It requires initialisation - this will provide the initial token for login - and unsealing, with the keys generated during initialisation.

Once it is unsealed (3x), you should be able to port-forward vault, and access it from the browser, or - if you followed the previous steps and enabled `metallb` - expose it using the load balancer range

> Important: you may want to store the token and the keys that is outputed at `vault operator init` command, as you will need these to use the command line tool.

### Working with the Vault

You can use `brew install vault` to install the command line interface and interact with the Hashicorp vault.

In order to do this, you will need to store two env variables:

```bash
# replace this with the token from the previous step
export VAULT_TOKEN=<your vault token>
# replace the IP with what Metallb allocated (in a production environment this should be fixed with the "loadBalancerIP:" value in the helm values)
export VAULT_ADDR='http://10.0.2.200:8200'
```

At this point `vault status` should display the status of the vault

```bash
fabrice in 🌐 central-1 in mikrok8s on  main [!?] took 2s
❯ export VAULT_TOKEN=hvs.7Ol8RLgjnblm4pwPIo45oSUs

fabrice in 🌐 central-1 in mikrok8s on  main [!?]
❯ export VAULT_ADDR='http://10.0.2.200:8200'

fabrice in 🌐 central-1 in mikrok8s on  main [!?]
❯ vault status
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    5
Threshold       3
Version         1.12.1
Build Date      2022-10-27T12:32:05Z
Storage Type    file
Cluster Name    vault-cluster-23585a63
Cluster ID      e9880361-edd5-77cc-0fa0-a938ed946e48
HA Enabled      false
```