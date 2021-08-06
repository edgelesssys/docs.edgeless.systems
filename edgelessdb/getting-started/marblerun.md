# Running EdgelessDB under MarbleRun

If you want to run EdgelessDB as a distributed application in a confidential cluster, you can combine it with [MarbleRun](https://marblerun.sh).

## Changes
When you are running EdgelessDB as a Marble, keys and credentials will be managed by MarbleRun. EdgelessDB will no longer generate its own root certificate nor its sealing key. Instead, they will be provided by MarbleRun's [Secrets management](https://docs.edgeless.systems/marblerun/#/content/features/secrets-management). The root certificate for EdgelessDB needs to be explicitly defined in MarbleRun's manifest. We explain this in more detail in the next section below. Given now MarbleRun will handle EdgelessDB's certificate and key management, EdgelessDB's own recovery method will be unavailable. Instead, MarbleRun will handle recovery for your entire cluster.

## Setup

### Extend the MarbleRun manifest
To add EdgelessDB to your MarbleRun cluster, add to the [MarbleRun manifest](https://docs.edgeless.systems/marblerun/#/workflows/define-manifest)
* the `edgelessdb` package
* an encryption key `edb_masterkey`
* a root certificate `edb_rootcert`
* and a Marble `edb_marble` that applies the secrets.

Here's a template:
```json
{
    "Packages": {
        "edgelessdb": {
            "SecurityVersion": 1,
            "ProductID": 16,
            "SignerID": "67d7b00741440d29922a15a9ead427b6faf1d610238ae9826da345cea4fee0fe"
        }
    },
    "Marbles": {
        "edb_marble": {
            "Package": "edgelessdb",
            "Parameters": {
                "Env": {
                    "EROCKSDB_MASTERKEY": "{{ hex .Secrets.edb_masterkey.Private }}",
                    "EDB_ROOT_CERT": "{{ pem .Secrets.edb_rootcert.Cert }}",
                    "EDB_ROOT_KEY": "{{ pem .Secrets.edb_rootcert.Private }}"
                }
            }
        }
    },
    "Secrets": {
        "edb_masterkey": {
            "Type": "symmetric-key",
            "Size": 128
        },
        "edb_rootcert": {
            "Type": "cert-ecdsa",
            "Size": 256,
            "Cert": {
                "IsCA": true,
                "Subject": {
                    "Organization": [
                        "My EdgelessDB root"
                    ]
                }
            }
        }
    }
}
```

### Launch the MarbleRun Coordinator
[Set up the MarbleRun Coordinator](https://docs.edgeless.systems/marblerun/#/deployment/cloud?id=deploy-marblerun) and [set the MarbleRun manifest](https://docs.edgeless.systems/marblerun/#/workflows/set-manifest).

## Launch as a Marble
To run EdgelessDB as a Marble, you have to add `-marble` as a parameter and define the required Marble definitions as environment variables. Assuming you are using the default settings for the MarbleRun coordinator deployed with the example manifest shown above, you can do this in the following way:

```sh
docker run \
--name my-edb \
--privileged \
--network host \
-p3306:3306 \
-p8080:8080 \
-e EDG_MARBLE_TYPE=edb_marble \
-e EDG_MARBLE_COORDINATOR_ADDR=localhost:2001 \
-e EDG_MARBLE_UUID_FILE=uuid \
-e EDG_MARBLE_DNS_NAMES=localhost \
-v /dev/sgx:/dev/sgx \
-t ghcr.io/edgelesssys/edgelessdb-sgx-1gb \
-marble
```

!> The example command uses `--network host` to simplify the required network configuration to discover the coordinator. **This is not necessarily safe for production**, as this disables Docker's network isolation. For production use, please configure adequately such that the EdgelessDB container is able to access the MarbleRun coordinator.


For 4 GB of enclave heap memory, replace `edgelessdb-sgx-1gb` with `edgelessdb-sgx-4gb`.

## Remote attestation
When running as a Marble, you can either attest an EdgelessDB instance by itself or by attesting the whole cluster once through the MarbleRun Coordinator. Given that EdgelessDB's certificates are issued and provided by MarbleRun, you can establish trust via MarbleRun's public key infrastructure (PKI) to your EdgelessDB instances.
