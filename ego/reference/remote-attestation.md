# Remote Attestation

## Requirements

Make sure to install the following services to be able to use remote attestation:
- [Quote Provider](#quote-provider)
- [PCCS Server](#pccs-server)

After you've installed the requirements, you can test whether your setup works using our ego sample for [remote-attestation](https://github.com/edgelesssys/ego/tree/master/samples/remote_attestation).

## Quote provider
For remote attestation, both the attester and the verifier expect a *quote provider* to be installed on the system. If no quote provider can be found, you will see the `OE_QUOTE_PROVIDER_LOAD_ERROR` error. The required quote provider depends on your environment.

### Azure
Install the Azure DCAP client:
```bash
sudo ego install az-dcap-client
```
Or manually:
```bash
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add
sudo add-apt-repository "deb [arch=amd64] https://packages.microsoft.com/ubuntu/`lsb_release -rs`/prod `lsb_release -cs` main"
sudo apt install az-dcap-client
```

### On-premises or other cloud
If the enclave app runs on-premises or on another cloud that doesn't provide its own attestation infrastructure, you can use `ego install` to setup the required services:
```bash
sudo ego install libsgx-dcap-default-qpl
```

Otherwise you can follow the [guide from Open Enclave](https://github.com/openenclave/openenclave/blob/master/docs/GettingStartedDocs/Contributors/NonAccMachineSGXLinuxGettingStarted.md#3-set-up-intel-dcap-quote-provider-library-qpl) to do the steps manually.

## PCCS Server
In all cases, you need to setup a PCCS Server that caches the required data for remote attestation.
Follow our [instructions on Github](https://github.com/edgelesssys/era/blob/master/pccs/README.md) to easily run a PCCS Server using Docker.
