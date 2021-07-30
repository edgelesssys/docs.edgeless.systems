# Building a service: Graphene
Running a Graphene app with MarbleRun requires some changes to its manifest. These are explained in the following. See also the [helloworld example](https://github.com/edgelesssys/marblerun/tree/master/samples/graphene-hello).

## Requirements
First, get Graphene up and running. You can use either the [Building](https://graphene.readthedocs.io/en/latest/building.html) or [Cloud Deployment](https://graphene.readthedocs.io/en/latest/cloud-deployment.html) guide to build and initially set up Graphene.

Before running your application, make sure you got the prerequisites for ECDSA remote attestation installed on your system. You can collectively install them with the following command:
```sh
sudo apt install libsgx-quote-ex-dev
```
## Configuration
### Entrypoint and argv
We provide the `premain-libos` executable with the [MarbleRun Releases](https://github.com/edgelesssys/marblerun/releases). It will contact the Coordinator, set up the environment, and run the actual application.

Set the premain executable as the entry point of the Graphene project and place the actual entry point in argv0:
```toml
libos.entrypoint = "file:premain-libos"
sgx.trusted_files.premain = "file:premain-libos"

# argv0 needs to contain the name of your executable
loader.argv0_override = "hello"
```
After the premain is done running, it will automatically spawn your application.

For a better illustration of the differences between the two premain variants, check out our ["Hello World" sample on GitHub.](https://github.com/edgelesssys/marblerun/tree/master/samples/graphene-hello)

### Host environment variables
The premain needs access to some host [environment variables for configuration](content/workflows/add-service.md#step-3-start-your-service):
```toml
loader.insecure__use_host_env = 1
```
The premain will remove all other variables before the actual application is launched, but there may still be risks. Don't use this on production until [secure forwarding of host environment variables](https://github.com/oscarlab/graphene/issues/2356) will be available.

### uuid file
The Marble must be able to store its uuid:
```toml
sgx.allowed_files.uuid = "file:uuid"
```

### Remote attestation
The Marble will send an SGX quote to the Coordinator for remote attestation:
```toml
sgx.remote_attestation = 1
```

### Enclave size and threads
The premain process is written in Go. The enclave needs to have enough resources for the Go runtime:
```toml
sgx.enclave_size = "1024M"
sgx.thread_num = 16
```

If your application has high memory demands, you may need to increase the size even further.
### Secret files
A Marble's secrets, e.g. a certificate and private key, can be provisioned as files. Ideally, these would be placed in the Marble's in-memory filesystem. Graphene does not support this yet, but you can fall back on *Graphene Protected Files* instead:
```toml
sgx.protected_files.cert = "file:server.crt"
sgx.protected_files.privkey = "file:server.key"
```
You can specify the files' content in the MarbleRun manifest:
```javascript
...
    "Parameters": {
        "Files": {
            "/dev/attestation/protected_files_key": "{{ hex .Marblerun.SealKey }}",
            "server.crt": "{{ pem .Secrets.server_cert.Cert }}",
            "server.key": "{{ pem .Secrets.server_cert.Private }}"
        }
    }
...
```
Note that Graphene requires to initialize the protected files key by writing it hex-encoded to the virtual `protected_files_key` device. This can be easily done through the above manifest configuration.

You can see this in action in the [nginx example](https://github.com/edgelesssys/marblerun/tree/master/samples/graphene-nginx).

## Troubleshooting
### aesm_service returned error: 30
If you receive the following error message on launch:

```
aesm_service returned error: 30
load_enclave() failed with error -1
```

Make sure you installed the Intel AESM ECDSA plugins on your machine. You can do this by installing the `libsgx-quote-dev` package mentioned in the requirements above.
