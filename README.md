# Keys

This repo provides keys, certificates and various other artifacts required to support
secure, unattended boot of systems.  These are distributed for demonstration
purposes only.

The keypairs are RSA with 2048 bit private keys.

    - manifest-ca: contains the private key and self-signed certificate for the rootCA
      that is used to sign product manifest certificates. A product manifest keypair and
      certificate are used to sign and verify the manifest yaml file for a product.

    - manifest: contains a sample private key and signed certificate for a product's
      manifest. The certificate is signed by the manifestCA, establishing a chain of
      trust. The product's uuid is included into the CN of the certificate. The
      private key is used to sign the product's manifest and the certificate is
      used to verify it.

    - tpmpol-admin: contains keypair and certificate used to sign and verify
      TPM EA Policy. This particular policy is used for access to the TPM Password
      stored in a TPM nvindex.

    - tpmpol-luks: contains the keypair and certificate used to sign and verify
      TPM EA Policy. This particular policy is used for access to the LUKS
      secret stored in a TPM nvindex.

    - uefi-db: contains the keypair, guid, and certificate to sign and verify
      UEFI applications. The certificate is stored in the UEFI DB.

    - uefi-pk: contains keypair, guid and cert for UEFI platform key.

    - uefi-kek: contains keypair, guid, and cert for UEFI Key Exchange Key.

    - uki-limited: contains the keypair, guid, and certificate to sign and verify
      UKIs signed with the "limited" key. This key signs and verifies special
      purpose UKIs and UEFI binaries. It does not grant privileged access to the
      TPm secret nor the LUKS secret.

    - uki-production: contains the keypair, guid, and certificate to sign and
      verify UKIs signed with the "production" key. This is the standard kernel signing
      key which protects the LUKS secret.

    - uki-tpm: contains the keypair, guid, and certificate to sign and verify UKIs
      signed with the "tpm" key. A UKI signed with the "tpm" key permit access to
      the TPM password for administration.

# Roles / Ownership
**Michael** (think microsoft) owns the firmware.
  * owns keys: uefi-pk, uefi-kek
  * public keys for uefi-pk and uefi-kek are the UEFI Platform key and UEFI Key Exchange Key respectively.
  * In secure boot mode, the firmware measures into PCR-7 the signer of an EFI application.

**Mikey** owns a key that is in the system's UEFI DB.
  * owns keys: uefi-db
  * uefi-db here is a single key that is present in the uefi-db.
  * EFI applications signed by uefi-db will be executed by the UEFI in SecureBoot mode.
  * Mikey signs Steven's shim with Mikey's uefi-db key.

**Steven** owns the shim.efi application:
  * The shim's primary purpose is to validate the kernel that it will execute, ensuring that it is signed by one of the keys in its DB.
  * Steven's builds shim.efi with these keys in its DB: uki-limited, uki-production, uki-tpm.
  * The shim measures the signer of the efi application (kernel) into PCR-7

**Arthur** is the Manifest Certificate Authority.
 * owns keys: manifest-ca
 * As the Manifest Certificate Authority (manifest-ca), Arthur signs Oliver's Certificate Signing Request (manifest).

**Katherine** owns the kernel.
  * owns keys: uki-limited, uki-production, uki-tpm.
  * it is assumed that Katherine has sole access to those 3 keys.
  * Katherine builds and signs "Universal Kernel Initrd" (UKI) EFI applications.
  * Katherine signs a UKI with one of uki-limited, uki-production or uki-tpm key.
  * Katherine includes inside her signed UKI:
    * a copy of Arthur's public manifest-ca into /manifest-ca.pem.
    * A 'mosctl' binary

Oliver owns the operating system
  * owns keys: manifest, sudi-ca
  * Oliver's CSR with a specific UUID was signed by Arthur.  He owns that UUID prefix.
  * Oliver generates and signs SUDI certificates for systems.  Those certificates are provisioned to a single machine during provisioning.
  * Oliver uses his manifest key to sign Manifests (a root filesystem and list of containers).

**Thomas** owns the tpm.
  * owns keys: tpmpol-luks tpmpol-admin
  * During provisioning (factory install):
    * the TPM admin (master password) is set and configured to allow access to via policies that are signed with tpm-admin.
    * a luks key (randomly generated long key) is generated and stored in the tpm.  Access is granted to policies signed by the tpmpol-luks key.
    * a SUDI key is registered with the TPM.  Access is granted by policies signed by the tpmpol-luks key.
  * Thomas signs policies giving access to
    * TPM admin password if the booted kernel is signed by Katherine's uki-tpm key.
    * LUKs key if the booted kernel is signed uki-production.

## Programs
**Mosctl binary**.
  * runs in the initramfs (and after).
  * during initramfs can access SUDI cert and luks passphrase.
  * before leaving the initramfs, measures a known constant value into PCR7 so that policies can match against that, and so that policies intended to give access only during initramfs will no longer have access.
  * Will only pivot into a RFS if:
    * The manifest is signed by a certificate that has been signed /manifest-ca.pem (from Arthur)
    * The UUID prefix found sudi certificate is owned by the manifest signer.
    * The serial number in DMI information matches the one found in the SUDI certificate.

**SUDI key**
 * The Secure Unique Device Identifier
 * A certificate that is programmed into the TPM at provisioning time.
 * The certificate contains in it:
    * The systems' serial number
    * The UUID
 * It is unique to the device and secure.  Operations using this signing key are guaranteed to have originated from the device.


# PRs

Any changes to the layout here must be accompanied by a corresponding change
to (trust)[https://github.com/project-machine/trust] to effect the same change
in keysets it generates.

