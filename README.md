# Hashicorp Vault Guide

# Vault

```
Vault secures, stores, and tightly controls access to tokens, passwords, certificates, API keys, and other secrets in modern computing.
```

## TLS End to End Encryption

See steps in `hashicorp/vault/tls/ssl_generate_self_signed.txt`
You'll need to generate TLS certs (or bring your own)
Create base64 strings from the files, place it in the `server-tls-secret.yaml` and apply it.

## Deployment

```
kubectl create ns vault-example
kubectl -n vault-example apply -f server/
kubectl -n vault-example get pods
```

## Storage

```
kubectl -n vault-example get pvc
```
ensure vault-claim is bound, if not, `kubectl -n vault-example describe pvc vault-claim`
ensure correct storage class is used for your cluster.
if you need to change the storage class, deleve the pvc , edit YAML and re-apply

## Initialising Vault

```
kubectl -n vault-example exec -it adap-vault vault operator init
#unseal 3 times
kubectl -n vault-example exec -it adap-vault vault operator unseal
kubectl -n vault-example get pods
```

## Deploy the Injector

Injector allows pods to automatically get secrets from the vault.

```
kubectl -n vault-example apply -f injector/
kubectl -n vault-example get pods
```

## Injector Kubernetes Auth Policy

For the injector to be authorised to access vault, we need to enable K8s auth

```
kubectl -n vault-example exec -it adap-vault vault login
kubectl -n vault-example exec -it adap-vault vault auth enable kubernetes


kubectl -n vault-example exec -it adap-vault sh
vault write auth/kubernetes/config \
token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
kubernetes_host=https://${KUBERNETES_PORT_443_TCP_ADDR}:443 \
kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
exit

kubectl -n vault-example get pods

```

# Summary
So we have a vault, an injector, TLS end to end, stateful storage.
The injector can now inject secrets for pods from the vault.

Now we are ready to use the platform for different types of secrets:

## Secret Injection Guides

### Basic Secrets

Objective:
---------- 
* Let's create a basic secret in vault manually
* Application consumes the secret automatically

/example-apps/basic-secret/readme.md)

### Dynamic Secrets: Postgres

Objective:
---------- 
* We have a Postgres Database
* Let's delegate Vault to manage life cycles of our database credentials
* Deploy an app, that automatically gets it's credentials from vault

example-apps/basic-secret/readme.md)




