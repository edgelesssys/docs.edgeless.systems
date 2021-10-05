# Quote provider
For remote attestation, both the attester and the verifier expect a *quote provider* to be installed on the system. If no quote provider can be found, you will see the `OE_QUOTE_PROVIDER_LOAD_ERROR` error. The required quote provider depends on your environment.

## Azure
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

## On-premises or other cloud
If the enclave app runs on-premises or on another cloud that doesn't provide its own attestation infrastructure, follow the [guide from Open Enclave](https://github.com/openenclave/openenclave/blob/master/docs/GettingStartedDocs/Contributors/NonAccMachineSGXLinuxGettingStarted.md#3-set-up-intel-dcap-quote-provider-library-qpl) to set up the required services.
