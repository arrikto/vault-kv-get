# Introduction

`vault-kv-get` is a small Go CLI that:
1. Receives input from stdin
2. Performs regex matching on the input to find references of the form `<vault-kv-get>.*</vault-kv-get>`
3. Uses the `vault` CLI to make a call `vault kv get your-arguments-go-here` if it finds `<vault-kv-get>your-arguments-go-here</vault-kv-get>`.
4. Replace `<vault-kv-get>.*</vault-kv-get>` with the retrieved value
5. Prints the result to stdout


# Usage

`vault-kv-get` was built in order to facilitate a workflow where you have a HashiCorp Vault server that contains secrets you would like to render into a YAML template. For example,
the following template can be use to fetch value in the field "foo" from version 1 of the secret "secret/my-secret"

`input.yaml`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
stringData:
  someSecret: <vault-kv-get> -field=foo -version=1 secret/my-secret </vault-kv-get>
```

`cat input.yaml | vault-kv-get` would yield:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
stringData:
  someSecret: someValue
```

In order for this to work, `vault-kv-get` (and by extension the `vault` CLI) needs to have access to a Hashicorp Vault server. This is handled in the environment; `vault-kv-get` does not offer any way to manage the connection. If you have a local vault server (or have port-forwarded a server deployed on kubernetes with `kubectl port-forward services/vault 8200:8200 -n vault`) the following will allow you to connect and then use `vault-kv-get`.

```yaml
export VAULT_ADDR=http://127.0.0.1:8200
export VAULT_TOKEN=root # set a token here. "root" is the default Vault token if deployed in "dev" mode
cat input.yaml | vault-kv-get
```



# Installation

## Prerequisites:

As a minimum, you will require the `vault` binary.

```bash
VAULT_VER=1.9.3
wget https://releases.hashicorp.com/vault/${VAULT_VER}/vault_${VAULT_VER}_linux_amd64.zip
unzip vault_${VAULT_VER}_linux_amd64.zip
sudo mv vault /usr/local/bin/vault
chmod +x /usr/local/bin/vault
```

## Download the binary:

```bash
VAULT_KV_GET_VER=1.0.0
wget https://github.com/karlschriek/vault-kv-get/releases/download/${VAULT_KV_GET_VER}/vault-kv-get
sudo mv vault-kv-get /usr/local/bin/vault-kv-get
chmod +x /usr/local/bin/vault-kv-get
```

## Build from source

Clone this repo and from the root run:
```bash
go build
sudo mv vault-kv-get /usr/local/bin/vault-kv-get
chmod +x /usr/local/bin/vault-kv-get
```

# Contributing

To contribute, please branch or fork from master, make your changes and submit a PR onto master. To test locally you will need access to a HashiCorp Vault Server. To test run `cat test_input.yaml | go run main.go`.

# Releasing

A GitHub Action will automatically release a new version on merge to master. Versioning follows the convention `MAJOR.MINOR.PATCH`. By default, `PATCH` will be incremented on every
merge to master, e.g. `1.0.1` -> `1.0.2`. To trigger a `MINOR` change (typically when adding a non-breaking feature) including the text "MINOR" somehwere in your commit message. 
To trigger a `MAJOR` change (i.e. something that breaks existing functionality) include the text "MAJOR" somewhere in your commit message.
