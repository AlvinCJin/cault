# cault

Consul and Vault will started together in two separate, but linked, docker containers.

Vault is configured to use the `consul` [secret backend](https://www.vaultproject.io/docs/secrets/consul/).

## Start Consul and Vault

```bash
docker-compose up -d
```

## Getting Vault Ready

Login to the Vault image:

```bash
docker exec -it cault_vault_1 sh
```

Check Vault's status:

```bash
$ vault status
Error checking seal status: Error making API request.

URL: GET http://127.0.0.1:8200/v1/sys/seal-status
Code: 400. Errors:

* server is not yet initialized
```

Because Vault is not yet initialized, it is sealed, that's why Consul will show you a sealed critial status:

<p align="center"><img src="doc/img/sealed-vault.png"></p>

### Init Vault

```bash
$ vault init
Unseal Key 1: d28dc3e20848c499749450b411bdc55416cefb0ff6ddefd01ec02088aa5c90aa01
Unseal Key 2: ad2b7e9d02d0c1cb5b98fafbc2e3ea56bd4d4fa112a0c61882c1179d6c6585f302
Unseal Key 3: c393269f177ba3d07b14dbf14e25a325205dfbf5c91769b8e55bf91aff693ce603
Unseal Key 4: 87c605e5f766d2f76d39756b486cbdafbb1998e72d2f766c40911f1a288e53a404
Unseal Key 5: e97e5de7e2cdb0ec4db55461c4aaf4dc26092cb3f698d9cc270bf19dbb82eab105
Initial Root Token: 5a4a7e11-1e2f-6f76-170e-b8ec58cd2da5

Vault initialized with 5 keys and a key threshold of 3. Please
securely distribute the above keys. When the Vault is re-sealed,
restarted, or stopped, you must provide at least 3 of these keys
to unseal it again.

Vault does not store the master key. Without at least 3 keys,
your Vault will remain permanently sealed.
```

notice Vault says:

> you must provide at least 3 of these keys to unseal it again

hence it needs to be unsealed 3 times with 3 different keys (out of the 5 above)

### Unsealing Vault

```bash
$ vault unseal
Key (will be hidden):
Sealed: true
Key Shares: 5
Key Threshold: 3
Unseal Progress: 1

$ vault unseal
Key (will be hidden):
Sealed: true
Key Shares: 5
Key Threshold: 3
Unseal Progress: 2

$ vault unseal
Key (will be hidden):
Sealed: false
Key Shares: 5
Key Threshold: 3
Unseal Progress: 0
```

the Vault is now unsealed:

<p align="center"><img src="doc/img/unsealed-vault.png"></p>

### Auth with Vault

We can use the `Initial Root Token` from above to auth with the Vault:

```bash
$ vault auth
Token (will be hidden):
Successfully authenticated! You are now logged in.
token: 5a4a7e11-1e2f-6f76-170e-b8ec58cd2da5
token_duration: 0
token_policies: [root]
```

--

All done: now you have both Consul and Vault running side by side.

## Making sure it actually works

From the host environment (i.e. outside of the docker image):

```bash
alias vault='docker exec -it cault_vault_1 vault "$@"'
```

This will allow to tun `vault` commands without a need to logging in to the image.

### Watch Consul logs

In a separate terminal tail Consul logs:

```bash
$ docker logs cault_consul_1 -f
```

### Writing / Reading Secrets

```bash
$ vault write -address=http://127.0.0.1:8200 secret/billion-dollars value=behind-super-secret-password
```
```
Success! Data written to: secret/billion-dollars
```

Check the Consul log, you should see something like:

```bash
2016/12/28 06:52:09 [DEBUG] http: Request PUT /v1/kv/vault/logical/a77e1d7f-a404-3439-29dc-34a34dfbfcd2/billion-dollars (199.657µs) from=172.28.0.3:50260
```

Let's read it back:

```bash
$ vault read secret/billion-dollars
```
```
Key             	Value
---             	-----
refresh_interval	2592000
value           	behind-super-secret-password
```

And it is in fact in Consul:

<p align="center"><img src="doc/img/super-secret.png"></p>

## License

Copyright © 2016 tolitius

Distributed under the Eclipse Public License either version 1.0 or (at your option) any later version.
