# Coordinator client API

## JSend response style

The Client API is designed as an HTTP REST interface. Responses follow the [JSend](https://github.com/omniti-labs/jsend) style, though only the response types `success` and `error` are returned so far.

In general, a successful API call (HTTP Code 200) will return a response in the following style:

```json
{
    "status": "success",
    "data": {
        "ManifestSignature": "3fff78e99dd9bd801e0a3a22b7f7a24a492302c4d00546d18c7f7ed6e26e95c3"
    }
}
```

Depending on the API endpoint and the data submitted, `data` might contain a specific answer from the Coordinator, or may just be `null` to acknowledge that the requested operation was performed successfully.

Whereas an error (HTTP Code 4xx or 5xx) might look like this:

```json
{
    "status": "error",
    "data": null,
    "message": "server is not in expected state"
}
```
For errors, `data` will always be `null`, and `message` contains the specific error the Coordinator ran into when processing the request.

## Endpoints

The API currently contains the following endpoints. If an endpoint specifies *Returns* for either HTTP GET or HTTP POST, it means that the specified data can be found encoded inside the `data` block if the response was successful. If no returns are specified for a given endpoint, or in case all possible return values for an endpoint are declared as optional, `data` can just be `null`.

## /manifest

For deploying and verifying the manifest.

* Before deploying the application to the cluster the manifest needs to be set once by the provider
* Users can retrieve and inspect the manifest through this endpoint before interacting with the application

### GET
**Returns**:

| Field value       | Type   | Description                                                                                     |
| ----------------- | ------ | ----------------------------------------------------------------------------------------------- |
| ManifestSignature | string | A SHA-256 of the currently set manifest. Does not change when an update has been applied.       |
| Manifest          | bytes  | The currently set manifest in base64 encoding. Does not change when an update has been applied. |


### POST
**Returns**:

| Field value     | Type             | Description                                                                                                                                                                                                |
| --------------- | ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| RecoverySecrets | array (optional) | An array containing key-value mapping for encrypted secrets to be used for recovering the Coordinator in case of disaster recovery. The key matches each supplied key from `RecoveryKeys` in the manifest. |


Example for setting the manifest (HTTP POST):

```bash
curl --cacert marblerun.crt --data-binary @manifest.json "https://$MARBLERUN/manifest"
```

Example for verifying the deployed manifest (HTTP GET):

```bash
curl --cacert marblerun.crt "https://$MARBLERUN/manifest" | jq '.data.ManifestSignature' --raw-output
```

## /quote

For retrieving a remote attestation quote over the whole cluster and the root certificate.
The quote is an SGX-DCAP quote, you can learn more about DCAP in the [official Intel DCAP orientation](https://download.01.org/intel-sgx/sgx-dcap/1.9/linux/docs/Intel_SGX_DCAP_ECDSA_Orientation.pdf).
Both the provider and the users of the confidential application can use this endpoint to verify the integrity of the Coordinator and the cluster at any time.

### GET
**Returns**:

| Field value | Type   | Description                                                                                                                                                               |
| ----------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Cert        | string | A PEM-encoded certificate chain containing the Coordinator's Root CA and Intermediate CA, which can be used for trust establishment between a client and the Coordinator. |
| Quote       | string | Base64-encoded quote which can be used for Remote Attestation, as described in [Verifying a deployment](workflows/verification.md)                     |


Example for retrieving a quote

```bash
curl -k "https://$MARBLERUN/quote"
```

We provide a tool to automatically verify the quote and output the trusted certificate:

```bash
# Either install era for the current user
wget -P ~/.local/bin https://github.com/edgelesssys/era/releases/latest/download/era
chmod +x ~/.local/bin/era

# Or install it globally on your machine (requires root permissions)
sudo wget -O /usr/local/bin/era https://github.com/edgelesssys/era/releases/latest/download/era
sudo chmod +x /usr/local/bin/era

era -c coordinator-era.json -h $MARBLERUN -o marblerun.crt
```

?> On Ubuntu, `~/.local/bin` is only added to PATH when the directory exists when initializing your bash environment during login. You might need to re-login after creating the directory. Also, non-default shells such as `zsh` do not add this path by default. Therefore, if you receive `command not found: era` as an error message for a local user installation, either make sure `~/.local/bin` was added to your PATH successfully or simply use the machine-wide installation method.

The file `coordinator-era.json` contains the *Packages* information for the Coordinator. For our testing image this can be pulled from our GitHub releases:

```bash
wget https://github.com/edgelesssys/marblerun/releases/latest/download/coordinator-era.json
```

## /recover

For recovering the Coordinator in case unsealing the existing state failed.

### POST

This API endpoint is only available when the Coordinator is in recovery mode. Before you can use the endpoint, you need to decrypt the recovery secret which you may have received when setting the manifest initially. See [Recovering the Coordinator](workflows/recover-coordinator.md) to retrieve the recovery key needed to use this API endpoint correctly.

Example for recovering the Coordinator:

```bash
curl -k -X POST --data-binary @recovery_key_decrypted "https://$MARBLERUN/recover"
```

## /secrets

For setting and retrieving secrets.

This API endpoint only works when `Users` were defined in the manifest. For more information, look up [Managing secrets](workflows/managing-secrets.md).

### GET
**Returns**:

| Field value                  | Type | Description                                                |
| ---------------------------- | ---- | ---------------------------------------------------------- |
| \<SecretName\> (one or more) | map  | A map containing key-value pairs for the requested secret. |


Each GET requests allows specifying one or more secrets in the form of a query string, where each parameter `s` specifies one secret.
A query string for the secrets `symmetric_key_shared` and `cert_shared` may look like the following:
```
s=symmetric_key_shared&s=cert_shared
```

Example for retrieving the secrets `symmetric_key_shared` and `cert_shared`:
```bash
curl --cacert marblerun.crt --cert user_certificate.crt --key user_private.key https://$MARBLERUN/secrets?s=symmetric_key_shared&s=cert_shared
```

### POST
Setting secrets requires uploading them in JSON format using a POST request. For more information refer to [Managing secrets](workflows/managing-secrets.md).

Example for setting secrets from the file `secrets.json`:
```bash
curl --cacert marblerun.crt --cert user_certificate.crt --key user_private.key --data-binary @secrets.json https://$MARBLERUN/secrets
```

## /status

For returning the current state of the Coordinator.

### GET
**Returns**:

| Field value   | Type   | Description                                                                                       |
| ------------- | ------ | ------------------------------------------------------------------------------------------------- |
| StatusCode    | int    | A status code that matches the internal code of the Coordinator's current state.                  |
| StatusMessage | string | A descriptive status message of what the Coordinator expects the user to do in its current state. |


**Possible values**:


| StatusCode | StatusMessage                                                                                                                                                             |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1          | Coordinator is in recovery mode. Either upload a key to unseal the saved state, or set a new manifest. For more information on how to proceed, consult the documentation. |
| 2          | Coordinator is ready to accept a manifest.                                                                                                                                |
| 3          | Coordinator is running correctly and ready to accept marbles.                                                                                                             |


Example for getting the status:

```bash
curl -k "https://$MARBLERUN/status"
```

It may be useful to use this API endpoint and use it for other monitoring tools. More information can be found under [Monitoring and Logging](workflows/monitoring.md)

## /update

For updating the packages specified in the currently set manifest or retrieving a log of all performed updates.

### GET
**Returns:**

A structured log of all updates performed via the `/update` or `/secrets` endpoint, including timestamp, author, and affected resources.

Example for getting the update log:
```bash
curl -k "https://$MARBLERUN/update"
```

### POST
This API endpoint only works when `Users` were defined in the manifest. For more information, look up ["updating a manifest"](workflows/update-manifest.md)

Example for updating the manifest:

```bash
curl --cacert marblerun.crt --cert user_certificate.crt --key user_private.key -w "%{http_code}" --data-binary @update_manifest.json https://$MARBLERUN/update
```
