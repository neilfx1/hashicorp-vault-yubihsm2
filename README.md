# YubiHSM2 Vault Integration (PKCS#11)

This repository provides a complete setup guide for integrating a YubiHSM2 with [HashiCorp Vault](https://www.vaultproject.io/) using the PKCS#11 seal and KMS plugin.

It is designed for Vault Enterprise environments requiring secure key storage via hardware-backed modules, and it includes instructions for initial setup, connector configuration, environment preparation, and Vault initialization.

---

## 🚀 Purpose

To document and demonstrate how to:

- Configure YubiHSM2 for Vault PKCS#11 seal and KMS support
- Generate AES and HMAC keys within the HSM
- Use `yubihsm-shell`, `yubihsm-connector`, and `vault +ent.hsm`
- Validate correct integration of Vault with HSM-backed unsealing

---

## 📄 Contents

- `README.md`: You are here.
- `yubihsm_pkcs11.conf`: Example connector configuration file for use with PKCS#11.
- `setup.md` *(TBD)*: Full Markdown-formatted version of the step-by-step setup instructions (currently under construction or embedded in this README).

---

## 📦 Prerequisites

- Vault Enterprise 1.17.2+ (`vault +ent.hsm`)
- YubiHSM2 (firmware 2.4+ recommended)
- YubiHSM2 SDK 2025.x
- Linux environment (tested on Ubuntu 22.04)
- Working knowledge of Vault and hardware-backed key operations

---

## 🧪 Status

✅ Basic setup tested and operational  
❌ No automation or Terraform yet  
❌ No test cases for advanced Vault usage (e.g. auto-unseal with TLS-bound connector)

---

## 🔐 Related Projects

- [YubiHSM2 ADCS TLS Connector Guide](https://github.com/neilfx1/yubihsm2-adcs-tls-connector) — secure your connector with TLS
- [HashiCorp Vault](https://www.vaultproject.io/docs/secrets/pkcs11) — official documentation for PKCS#11 seal

---

## 🙋‍♂️ Notes

This guide was written for real-world lab testing and is shared as-is to help others get started. You'll likely need to adjust paths, connector IPs, and permissions to fit your environment.

> Questions or suggestions? Raise an issue or fork and adapt.
