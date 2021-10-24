# Remote attestation

## Requirements

For remote attestation, make sure you have access to the following services:
- [Quote Provider](#quote-provider)
- [PCCS Server](#pccs-server)

You can test whether your setup works using our ego sample for [remote-attestation](https://github.com/edgelesssys/ego/tree/master/samples/remote_attestation).

## Quote provider
Both the attester and the verifier expect a *quote provider* to be installed on the system. If no quote provider can be found, you will see the `OE_QUOTE_PROVIDER_LOAD_ERROR` error on remote attestation. The required quote provider depends on your environment.

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
If the enclave app runs on-premises or on another cloud that doesn't provide its own attestation infrastructure, you can use `ego install` to install the required Quote Provider Library:
```bash
sudo ego install libsgx-dcap-default-qpl
```
Otherwise you can follow the [guide from Open Enclave](https://github.com/openenclave/openenclave/blob/master/docs/GettingStartedDocs/Contributors/NonAccMachineSGXLinuxGettingStarted.md#3-set-up-intel-dcap-quote-provider-library-qpl) to do the steps manually.

## PCCS Server
Some environments already provide a PCCS Server that runs by default. On Azure, e.g., the az-dcap-client automatically establishes the function of the PCCS Server.
On Alibaba Cloud you only have to configure the `/etc/sgx_default_qcnl.conf` to connect your Quote Provider to the PCCS Server. For more information please refer to the [official Alibaba Cloud instructions](https://www.alibabacloud.com/help/doc-detail/208095.htm) for setting up remote attestation in your VM.

If your cloud provider does not provide their own PCCS Server or you are using your own infrastructure, you need to setup a PCCS Server that caches the required data for remote attestation.
Follow our [instructions on GitHub](https://github.com/edgelesssys/era/blob/master/pccs/README.md) to easily run a PCCS Server using Docker.

You may also follow the [guide from Intel](https://www.intel.com/content/www/us/en/developer/articles/guide/intel-software-guard-extensions-data-center-attestation-primitives-quick-install-guide.html) to setup and configure your own PCCS Server.
