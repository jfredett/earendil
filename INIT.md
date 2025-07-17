# Preliminaries

`earendil` is the primary collection of helm charts and instructions for setting up my k3s environment. This document
is the 'how to go from nothing to a functional kubernetes environment with all the services I expect'.

The minimal tools you need should be provided by the `minas-tarwon` flake shell, but explicitly they are:

1. kubectl and friends
2. the correct config for the single node

You can do much of this with `k9s` or other all-in-one tools, and generally I do, but the examples here are generally
assuming you are doing it via `kubectl`.

# Stage 1: Initial Bootstrapping

We'll assume this starts with all the fixings available from the `laurelin` module. While the 'default' services:

- vault + vault operator
- cert-manager
- nfs-subdir-provisioner
- longhorn (NOTE: Not working at TOW)

are technically optional, it's hard to get much done without them. The intent is generally that they will be present.
We'll also be setting up OLM as part of this stage, and additional services that come in the form of operators.

The main goal is to have a functional vault, set up for a multinode setup (but not yet scaling to multiple nodes). Stage
2 will cover scaling to a full control plane and additional workers.

## 1.1 Vault

### Initial unseal

We need to unseal the vault and record the unseal keys somewhere safe. What that means may vary by target, so I won't
specify where 'safe' is.

`kubectl exec -n kube-infra exec -it vault-0 -- vault operator init` 

By default, `laurelin.services.k3s` installs the auto-charts to the `kube-infra`, if your vault lives in another
namsepace by configuration, switch as appropriate.

This will generate output containing all the relevant 'unseal' keys and root token, we'll now unseal the vault:

`kubectl exec -n kube-infra exec -it vault-0 -- vault operator unseal`

This will prompt you to provide unseal keys, do this until the vault is unsealed. Now run:

```bash
➜ kubectl -n kube-infra exec -it vault-0 -- vault operator unseal
Unseal Key (will be hidden): 
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
# ... snip ###
Version         1.19.0
HA Enabled      false

➜ kubectl -n kube-infra get pods vault-0
NAME      READY   STATUS    RESTARTS   AGE
vault-0   1/1     Running   0          22h
```

### Ingress and Endpoints

Ingress is handled by the module via an IngressRoute.

### Additional Setup

This repo contains a terraform to enable various policies, definitions, and other items, see the [relevant
section](#Terraform) for more details.

## 1.2 Cert-Manager

TODO: any post work for cert-man

## 1.3 Initial Terraform

TODO: Apply terraform to vault, etc.
