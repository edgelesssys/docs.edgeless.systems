# Configuration file 📄
Your enclave's configuration is defined in JSON and applied when signing an executable via `ego sign`.

Here's an example configuration:
```json
{
    "exe": "helloworld",
    "key": "private.pem",
    "debug": true,
    "heapSize": 512,
    "productID": 1,
    "securityVersion": 1,
    "mounts": [
        {
            "source": "/home/user",
            "target": "/data",
            "type": "hostfs",
            "readOnly": false
        },
        {
            "target": "/tmp",
            "type": "memfs"
        }
    ],
    "env": [
        {
            "name": "LANG",
            "fromHost": true
        },
        {
            "name": "PWD",
            "value": "/data"
        }
    ]
}
```

## Basic settings
`exe` is the (relative or absolute) path to the executable that should be signed.

`key` is the path to the private RSA key of the signer. When invoking `ego sign` and the key file does not exist, a key with the required parameters is automatically generated. You can also generate it yourself with:

```sh
openssl genrsa -out private.pem -3 3072
```

If `debug` is true, the enclave will be debuggable.

`heapSize` specifies the heap size available to the enclave in MB. It should be at least 512 MB.

`productID` is assigned by the developer and enables the attester to distinguish between different enclaves signed with the same key.

`securityVersion` should be incremented by the developer whenever a security fix is made to the enclave code.

## Mounts
`mounts` define custom mount points that apply to the file system presented to the enclave. This can be omitted if no mounts other than the default mounts should be performed, or you can define multiple entries with the following parameters:

  * `source` (required for `hostfs`): The directory from the host file system that should be mounted in the enclave when using `hostfs`. For `memfs`, this value will be ignored and can be omitted.
  * `target` (required): Defines the mount path in the enclave.
  * `type` (required): Either `hostfs` if you want to mount a path from the host's file system in the enclave, or `memfs` if you want to use a temporary file system similar to *tmpfs* on UNIX systems, with your data stored in the secure memory environment of the enclave.
  * `readOnly`: Can be `true` or `false` depending on if you want to mount the path as read-only or read-write. When omitted, will default to read-write.

By default, `/` is initialized as an empty `memfs` file system. To expose certain directories to the enclave, you can use the `hostfs` mounts with the options mentioned above. You can also choose to define additional `memfs` mount points, but note that there is no explicit isolation between them. They can be accessed either via the path specified in `target` or also via `/edg/mnt/<target>`, which is where the files of the additional `memfs` mount are stored internally.

!> **It is not recommended to use the mount options to remount '/' as `hostfs` other than for testing purposes**. You might involuntarily expose files to the host which should stay inside the enclave, risking the confidentiality of your data.

## Environment variables
`env` holds environment variables to set or take over from the host inside the enclave. By default, all environment variables not starting with `EDG_` are dropped when entering the enclave.

  * `name` (required): The name of the environment variable
  * `value` (required if not `fromHost`): The value of the environment variable
  * `fromHost`: When set to `true`, the current value of the requested environment variable will be copied over if it exists on the host. If the host does not hold this variable, it will either fall back to the value set in `value` (if it exists), or will not be created at all.

A special environment variable is `PWD`. Depending on the mount options you have set in your configuration, you can set the initial working directory of your enclave by specifying your desired path as a value for `PWD`. Note that this directory needs to exist in the context of the enclave, not your host file system.
