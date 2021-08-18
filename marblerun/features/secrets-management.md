# Secrets management

The generation and the management of cryptographic keys and certificates for Marbles (i.e., containers running enclaves) are central duties of the Coordinator. Keys and certificates are passed to Marbles on startup via placeholders defined in the manifest. You can learn more about this mechanism in the `Secrets` section from our [manifest definition hands-on](workflows/define-manifest.md?id=secrets). Specifically, the Coordinator provides the following to Marbles.

* [Symmetric keys](#symmetric-keys)
* [Certificates](#certificates)
* [User-defined secrets](#user-defined-secrets)

## Symmetric keys
MarbleRun allows you to define symmetric keys that can be user per-Marble or shared between multiple ones. For example, these keys can be used to encrypt and decrypt data inside your Marble, without having to store the key in your application logic by yourself. 

To pass them to your application, you can use a template placeholder definition like `{{ raw .Secrets.MySecretKey }}` in your manifest and pass it to your application either as a file, over via an environment variable. From these two points, your application can retrieve the passed secret key and use it for cryptographic purposes.

If you use one key throughout multiple Marbles, care has to be taken to not repeat nonces between Marbles when using shared keys with AES-GCM or similar encryption algorithms.

## Certificates
Upon launch of a Marble, the Coordinator will generate a private key for each new Marble and issue a corresponding X.509 certificate. Both are automatically added to the Marble's environment variables as `MARBLE_PREDEFINED_MARBLE_CERTIFICATE_CHAIN` and `MARBLE_PREDEFINED_PRIVATE_KEY`. In addition, you can choose to define them to other environment variables or copy them to a file only available to the Marble through by defining template placeholders such as `{{ pem .MarbleRun.MarbleCert.Cert }}` or `{{ pem .MarbleRun.MarbleCert.Private }}` in the manifest. The certificate is signed with the Coordinator's intermediate CA. Marbles can use this certificate to establish secure communication channels with other Marbles and external clients (i.e., users of your app). Clients only need to verify the Coordinator's root CA once before they can securely communicate with any Marble, as is described in more detail in our [verification hands-on](workflows/verification.md).

Next to this automatically generated certificate, you can also define additional ones  as secrets in your manifest. These are also signed by the Coordinator's intermediate CA and generated upon each launch, but do not contain a specific Marble's by default. You can include them via placeholders such as `{{ pem .Secrets.MySecretCertificate }}` in your manifest.

!> Certificate secrets are always regenerated upon launch of a Marble, making them not suitable for certificate pinning.

## User-defined secrets

In addition to automatically generated secrets, MarbleRun also allows you to define secrets which can be uploaded and updated after the initial deployment of your manifest. This can be used to store non-random sensitivate data, such as passwords, tokens, certificates, or other types of credentials that should not be stored in the manifest directly. They can be injected via the placeholders in the same way as other secret types. Changes to user-defined secrets are logged and can be audited over via the `/update` or `/secrets` HTTP REST API endpoint of the Coordinator. You can learn more about this in the section about [Managing secrets](workflows/managing-secrets).

