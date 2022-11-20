# Keys

This repo provides keys, certificates and various other artifacts required to support
secure, unattended boot of systems.  These are distributed for demonstration
purposes only.

The keypairs are RSA with 2048 bit private keys.
    - manifestCA: contains the private key and certificate for the rootCA that is
      used to sign product manifest certificates. A product manifest keypair and
      certificate are used to sign and verify the manifest yaml file for a product.

    - tpmpol-admin: contains keypair and certificate used to sign and verify
      TPM EA Policy. This particular policy is used for access to the TPM Password
      stored in a TPM nvindex.

    - tpmpol-luks: contains the keypair and certificate used to sign and verify
      TPM EA Policy. This particular policy is used for access to the LUKS
      secret stored in a TPM nvindex.

    - uefi-db: contains the keypair, guid, and certificate to sign and verify
      UEFI applications. The certificate is stored in the UEFI DB.

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
