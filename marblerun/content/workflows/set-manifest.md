# Setting a manifest

The manifest is a JSON document that defines which services span the mesh and how they should be configured.
It further defines what Infrastructure providers are allowed.
You can set a manifest through MarbleRun's Client HTTP REST API.
The endpoint for all manifest operations is `/manifest`.

See the following manifest for example (`manifest.json`).

```json
{
    "Packages": {
        "backend": {
            "UniqueID": "6b2822ac2585040d4b9397675d54977a71ef292ab5b3c0a6acceca26074ae585",
            "ProductID": 43,
            "SecurityVersion": 1,
            "Debug": false
        },
        "frontend": {
            "SignerID": "43361affedeb75affee9baec7e054a5e14883213e5a121b67d74a0e12e9d2b7a",
            "ProductID": 42,
            "SecurityVersion": 3,
            "Debug": true
        }
    },
    "Marbles": {
        "backend_first": {
            "Package": "backend",
            "MaxActivations": 1,
            "Parameters": {
                "Files": {
                    "/tmp/defg.txt": "foo",
                    "/tmp/jkl.mno": "bar"
                },
                "Env": {
                    "IS_FIRST": "true",
                    "ROOT_CA": "{{ pem .Marblerun.RootCA.Cert }}",
                    "SEAL_KEY": "{{ hex .Marblerun.SealKey }}",
                    "MARBLE_CERT": "{{ pem .Marblerun.MarbleCert.Cert }}",
                    "MARBLE_KEY": "{{ pem .Marblerun.MarbleCert.Private }}"
                },
                "Argv": [
                    "--first",
                    "serve"
                ]
            }
        },
        "backend_other": {
            "Package": "backend",
            "Parameters": {
                "Env": {
                    "ROOT_CA": "{{ pem .Marblerun.RootCA.Cert }}",
                    "SEAL_KEY": "{{ hex .Marblerun.SealKey }}",
                    "MARBLE_CERT": "{{ pem .Marblerun.MarbleCert.Cert }}",
                    "MARBLE_KEY": "{{ pem .Marblerun.MarbleCert.Private }}"
                },
                "Argv": [
                    "serve"
                ]
            }
        },
        "frontend": {
            "Package": "frontend",
            "Parameters": {
                "Env": {
                    "ROOT_CA": "{{ pem .Marblerun.RootCA.Cert }}",
                    "SEAL_KEY": "{{ hex .Marblerun.SealKey }}",
                    "MARBLE_CERT": "{{ pem .Marblerun.MarbleCert.Cert }}",
                    "MARBLE_KEY": "{{ pem .Marblerun.MarbleCert.Private }}"
                }
            }
        }
    },
    "RecoveryKeys":
    {
        "recoveryKey1:" "-----BEGIN PUBLIC KEY-----\nMIIBpTANBgkqhkiG9w0BAQEFAAOCAZIAMIIBjQKCAYQAyokHE545y3lU4xsxrqXJ\n58jiaXN8yEdjjuKk0903zMT+FV62UeX17BQhrtdOIf4l4/V/xipqI+osAHBQpRY1\nwM1NCIFFlXUQGgXdtoWiAS7zfFKC+mNlB63Z0Z/50Iw9pl6AFWBQ+16lfmsPMnIu\nLHf4AL3KXVlpgPn6cmRfUoDBx6ITm2QrCDFlVu4j4isgnaZrw6VD0V+G9Mcpgs/0\n0XNmz72eMULfuW+4ULJI9Fx88wiNWWHeSI4vz83ylM5+1QntFROSYWBjgmCnm25j\nKbzV765CVTIU3qq3qkYmclpHfKKt7/TOgVOauvkMCYXyLJkSd1LGLIctWK8tCs1K\nnB237nNg+dZ67Zz9lBYKfNnFoudoc85+vXBRKIfV56FXiXrB32hF1DEj11viMPUr\nroMokLFtDCoAk0Xok4AFQDOgxTw7F8cHskjIYWVCmCqmDUI+FGttyVrc5YLSHAuR\nxQ2oxD0F44JXwxDc/C+OYzOApYl25rmR2nuqioDGpL6/ELRRAgMBAAE=\n-----END PUBLIC KEY-----\n"
    }
}
```

To set the manifest we can use the command line interface, which automatically performs remote attestation before uploading the manifest.

```bash
marblerun manifest set manifest.json $MARBLERUN
```

If the manifest contains a `RecoveryKeys` entry, you will receive a JSON reply including a recovery secret, encrypted with the public key supplied in `RecoveryKeys`. The reply will look like this, with `[base64]` as your encrypted recovery secret.

`{"EncryptionKey":"[base64]"}`

!> It is important that you keep this value stored somewhere safe. Without it, you will not be able to perform a recovery step in case the SGX seal key changed.
