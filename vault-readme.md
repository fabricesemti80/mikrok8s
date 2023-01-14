# Working with the Vault

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
❯ export VAULT_TOKEN=hvs.7Ol.... # your vault token

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

After this you could list the secret (engines) with

```bash
vault secrets list
Path          Type         Accessor              Description
----          ----         --------              -----------
cubbyhole/    cubbyhole    cubbyhole_2c621230    per-token private secret storage
identity/     identity     identity_1eab8ec6     identity store
sys/          system       system_526480d8       system endpoints used for control, policy and debugging
```