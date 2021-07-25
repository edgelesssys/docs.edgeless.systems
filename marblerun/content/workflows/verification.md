# Verifying a deployment

Marblerun provides a simple HTTP REST API for clients to verify the confidentiality and integrity of the Coordinator and the deployed Marbles.

## Establishing trust in the Coordinator

Marblerun exposes the `/quote` endpoint that returns a quote and a certificate chain consisting of a root and intermediate CA. The root CA is fixed for the lifetime of your deployment, while the intermediate CA changes in case you [update](content/workflows/update-manifest.md) the packages specified in your manifest.

The simplest way to verify the quote is via the Edgeless Remote Attestation ([era](https://github.com/edgelesssys/era)) tools:

```bash
# Install era globally on your machine (requires root permissions)
sudo wget -O /usr/local/bin/era https://github.com/edgelesssys/era/releases/latest/download/era
sudo chmod +x /usr/local/bin/era

# Run era. You can remove the output parameters you do not need for your use case.
era -c coordinator-era.json -h $MARBLERUN -output-chain marblerun-chain.pem -output-root marblerun-root.pem -output-intermediate marblerun-intermedite.pem
```

Era requires the Coordinator's UniqueID (or MRENCLAVE in SGX terms) or the tuple ProductID, SecurityVersion, SignerID (MRSIGNER) to verify the quote.
In production, these would be generated when building the Coordinator and distributed to your clients.
For testing, we have published a Coordinator image at `ghcr.io/edgelesssys/coordinator`.
You can pull the corresponding `coordinator-era.json` file from our release page:

```bash
wget https://github.com/edgelesssys/marblerun/releases/latest/download/coordinator-era.json
```

After successful verification, you'll have `marblerun-chain.pem`, `marblerun-root.pem`, and `marblerun-intermediate.pem` in your directory. In case you want to pin against specific versions of your application, using the intermediate CA as a trust anchor is a good choice. Else you can pin against the root CA in which case different versions of your application can talk with each other, though you may not be able to launch them if they do not meet the minimum `SecurityVersion` specified in your original or updated manifest.

## Verifying the manifest

Establishing trust with the service mesh allows you to verify the deployed manifest in the second step.
To that end, Marblerun exposes the endpoint `/manifest`.
Using the CLI you can get the manifest's signature aka its sha256 hash and compare it against your local version:

```bash
marblerun manifest verify manifest.json $MARBLERUN
```
