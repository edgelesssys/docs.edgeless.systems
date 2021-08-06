
# Recovery

## Information
!> Recovery is only available when EdgelessDB is running standalone, and only when a recovery key has been defined in the manifest. When used with MarbleRun, consider using its [recovery feature](https://docs.edgeless.systems/marblerun/#/content/features/recovery).

Persistent storage for confidential applications requires a bit of attention.
By design, SGX sealing keys are unique to a single CPU, which means using the default SGX sealing methods has some caveats.
For example, sealing data while running on one host could mean the data can't be unsealed when running on another host later on.

EdgelessDB makes persistence straightforward by using a so-called *virtual sealing key*, which is a cryptographically securely generated encryption key used for sealing data to persistent storage. While being scheduled on the same CPU, EdgelessDB restarts autonomously. However, when EdgelessDB is moved to another physical host, the advantage of the *virtual sealing key* becomes evident.  Because we can securely backup the virtual sealing key to a trusted user (not possible with the real SGX sealing key), the trusted user can recover EdgelessDB later on.
To that end, [the manifest allows for specifying a designated *Recovery Key*](reference/manifest.md). The Recovery Key is a public RSA key. During the initial upload of the manifest, EdgelessDB generates its virtual sealing key and returns it RSA encrypted with the public key specified in the manifest.

!> The holder of the corresponding private key can recover the database and therefore, access the content of the database. It is important that this key is kept somewhere safe. After the initial upload of the manifest, EdgelessDB will not release the encryption key.

## Performing the recovery
If EdgelessDB is unable to decrypt the virtual sealing key upon launch, it will enter recovery mode. This will allow the user to upload the virtual sealing key via the `/recovery` endpoint of the HTTP REST API.

To do so, you need to:
* Decode the Base64 encoded output that was returned to you during the upload of the manifest
* Decrypt the decoded output with the corresponding RSA private key of the key defined in the manifest
* Upload the binary decoded and decrypted key to the `/recovery` endpoint

Assuming you saved the output from the manifest upload step in a file called `recovery_key_encrypted_base64`, you can accomplish the recovery process like this:

```bash
base64 -d recovery_key_encrypted_base64 > recovery_key_encrypted

openssl pkeyutl -inkey private_key.pem -in recovery_key_encrypted -pkeyopt rsa_padding_mode:oaep -pkeyopt rsa_oaep_md:sha256 -decrypt -out recovery_key_decrypted

curl -k -X POST --data-binary @recovery_key_decrypted "https://localhost:8080/recovery"
```

After you uploaded the key to the HTTP REST API endpoint, EdgelessDB will let you know if it was able to successfully recover your database instance.


## Resetting EdgelessDB
If you choose not to recover the current state of the database, you can reset EdgelessDB to a clean state by deleting its data directory.
