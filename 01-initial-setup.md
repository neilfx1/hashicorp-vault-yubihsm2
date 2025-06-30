# YubiHSM2 Setup for HashiCorp Vault with PKCS#11

This guide walks through setting up a YubiHSM2 to work with HashiCorp Vault using PKCS#11 as a seal and KMS plugin.

---

## 🔧 Prerequisites

- Vault Enterprise 1.17.2+ (`vault +ent.hsm` build)
- Valid Vault license
- YubiHSM2 hardware device with firmware 2.4 or higher  
  _(Note: FIPS edition YubiHSM2s at the time of publishing do not support AES)_
- YubiHSM2 SDK 2025.x installed
- `yubihsm-shell`
- `yubihsm-connector`  

---

1. Create PKCS#11 Configuration File

Download the yubihsm_pkcs11.conf from this repository and store it in a system accessible location. If you already have this file, ensure the connector= line matches the location of your yubihsm-connector instance.  If your YubiHSM2 is in a different server to your Vault instance, change the connector URL to the IP address of that server.  

```ini
connector=http://127.0.0.1:12345
```

If the connector is running on a remote system, replace 127.0.0.1 with a valid IP address.

---

2. Start yubihsm-connector either as a service or in interactive mode.  If you are running this across a network (not on the Vault server), start the yubihsm-connector with either the actual IP of the server or the following command to listen on all interfaces:  

```bash
yubihsm-connector -C 0.0.0.0:12345
```

If you are running on the yubihsm-connector on the same server as vault, 127.0.0.1 is sufficient.

```bash
yubihsm-connector -C 127.0.0.1:12345
```

In either case, this should match your `yubihsm_pkcs11.conf`

⚠️ For production, replace 0.0.0.0 with the actual IP and use firewall rules to protect TCP port 12345.  You should also consider using TLS.  Please see the my guide at https://github.com/neilfx1/yubihsm2-adcs-tls-connector for instructions on how to achieve this.

---

3. Run yubihsm-shell with the following command:

```bash
yubihsm-shell -C <ipaddress>:12345
```
At the prompt:

```bash
yubihsm> connect
yubihsm> session open 1 password
```

_Note: This assumes a brand new YubiHSM2 where the default authentication-key is still in place.  Alter the command to use your valid authentication-key with enough rights to create aes and hmac keys in the appropriate domain._

**Create a symmetric AES key**
```bash
yubihsm> generate symmetric 0 0 vaultkey 1 encrypt-cbc:decrypt-cbc aes128
```

**Create a HMAC key**
```bash
yubihsm> generate hmackey 0 0 hmac_key 1 all hmac-sha256
```

_In the above command, hmac_key is the label.  “all” gives this key access to all capabilities (including export) within domain 1. Consider restricting this as appropriate for your environment._

---

4. Create the following environment variables, ideally system wide

```bash
export VAULT_ADDR='http://127.0.0.1:8200'
export YUBIHSM_PKCS11_CONF=/your/valid/path/yubihsm_pkcs11.conf
```

---

5. Prepare a Vault directory structure if you do not already have one

```bash
mkdir -p /tmp/vault/raft_vault
mkdir -p /tmp/vault/config
mv vault /tmp/vault
mv vault.hclic /tmp/vault
```

---

6. Create a Vault configuration file for this scenario.

Save this as `/tmp/vault/config/config-hsm.hcl`:

```bash
storage "raft" {
  path    = "/tmp/vault/raft_vault/"
  node_id = "vault"
}

listener "tcp" {
  address         = "127.0.0.1:8200"
  cluster_address = "127.0.0.1:8201"
  tls_disable     = true
}

api_addr       = "http://127.0.0.1:8200"
ui             = true
disable_mlock = true
cluster_addr   = "http://127.0.0.1:8201"
license_path   = "/tmp/vault/vault.hclic"

seal "pkcs11" {
  lib            = "/usr/lib/x86_64-linux-gnu/pkcs11/yubihsm_pkcs11.so"
  slot           = "0"
  pin            = "0001password"
  key_label      = "vaultkey"
  hmac_key_label = "hmac_key"
  generate_key   = "false"
}

kms_library "pkcs11" {
  name    = "yubihsm"
  library = "/usr/lib/x86_64-linux-gnu/pkcs11/yubihsm_pkcs11.so"
}
```

_Ensure the `yubihsm_pkcs11.so` location matches the location on your server.  Replace 0001password with your YubiHSM2 authentication key and password.  In this example that is object ID 1 and the password is password._

---
7. Start Vault interactively


```bash
cd /tmp/vault
./vault server -config=/tmp/vault/config/config-hsm.hcl
```

The vault will start and repeat an error stating it is unable to unseal the core - this can be ignored as it will be created in the next step.

---

8. Initialize Vault

Open another terminal session run the following:

```bash
cd /tmp/vault
export VAULT_ADDR='http://127.0.0.1:8200'
./vault operator init
```

Make a note of the recovery keys and root token.

```bash
export VAULT_NAMESPACE="root"
export VAULT_TOKEN="<root token from previous output>"
```

Vault is now ready for testing with the YubiHSM2 PKCS#11 seal and KMS plugin.
