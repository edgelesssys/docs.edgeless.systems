# Recovering the Coordinator

As described in the [recovery chapter](content/features/recovery.md), different situations can require the *recovery* of the Coordinator.
If the Coordinator finds a sealed state during its startup which it is unable to unseal using the host-specific SGX sealing key, it will wait for further instructions.
You have two options:

1. Recover the sealed state by uploading the recovery secret, which was encrypted for the `RecoveryKeys` defined in the manifest

    The recovery secret can be uploaded through the `/recover` client API endpoint. In order to do so a client needs to first extract the encrypted secret by decrypting it with the corresponding private key:

    ```bash
    base64 -d recovery_key_encrypted_base64 > recovery_key_encrypted
    openssl pkeyutl -inkey private_key.pem -in recovery_key_encrypted -pkeyopt rsa_padding_mode:oaep -pkeyopt rsa_oaep_md:sha256 -decrypt -out recovery_key_decrypted
    ```

    The extracted secret can then be uploaded using the MarbleRun CLI.

    ```bash
    marblerun recover $MARBLERUN recovery_key_decrypted
    ```

    If the recovery worked correctly, the Coordinator should apply the sealed state again without returning an error. In case the Coordinator was not able to restore the state with the uploaded key, an error will be returned in the logs and the `/recover` endpoint will stay open for further interaction.

2. Dismiss the sealed state by uploading a new manifest

    In case there is no desire to recover the old state it can simply be dismissed by [uploading a new manifest](content/workflows/set-manifest.md).

!> If a new manifest is uploaded, the old state will be overwritten on disk and the `/recover` endpoint will not be available anymore.
