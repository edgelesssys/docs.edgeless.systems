# CLI reference

After you've compiled your app with `ego-go`, use the `ego` tool to sign, run, and examine your enclave app.

Usage:
```
ego <command> [arguments]
```

## sign
Sign an executable built with ego-go. Executables must be signed before they can be run in an enclave.
```
ego sign [executable | config.json]
```

This command can be used in different modes:
* `ego sign <executable>`\
  Generates a new key `private.pem` and a default configuration `enclave.json` in the current directory and signs the executable.

* `ego sign`\
  Searches in the current directory for `enclave.json` and signs the therein provided executable.

* `ego sign <config.json>`\
  Signs an executable according to a given [configuration](config.md).

## run
Run a signed executable in an enclave.
```
ego run <executable> [args...]
```
You can pass arbitrary arguments to the enclave.

Environment variables are only readable from within the enclave if they start with "EDG_".

You need an SGX-enabled machine to run an enclave. For development, you can also enable simulation mode by setting `OE_SIMULATION=1`, e.g.:
```
OE_SIMULATION=1 ego run helloworld
```

## marblerun
Run a signed executable with [MarbleRun](https://marblerun.sh/). MarbleRun is an open-source and cloud-native framework for managing clusters of confidential microservices.
```
ego marblerun <executable>
```
Requires a running MarbleRun Coordinator instance.

Environment variables are only readable from within the enclave if they start with "EDG_" and will be extended/overwritten with the ones specified in the manifest.

Requires the following configuration environment variables:
* `EDG_MARBLE_COORDINATOR_ADDR`\
  The Coordinator address
* `EDG_MARBLE_TYPE`\
  The type of this Marble (as specified in the manifest)
* `EDG_MARBLE_DNS_NAMES`\
  The alternative DNS names for this Marble's TLS certificate
* `EDG_MARBLE_UUID_FILE`\
  The location where this Marble will store its UUID

Set `OE_SIMULATION=1` to run in simulation mode.

## signerid
Print the SignerID either from a signed executable or by reading a keyfile.
```
ego signerid <executable | key.pem>
```

## uniqueid
Print the UniqueID of a signed executable.
```
ego uniqueid <executable>
```

## env
Run a command within the ego build environment.
```
ego env ...
```
For example, run
```
ego env make
```
to build a Go project that uses a Makefile.

## install
Install drivers and other components.

Use `ego install` to list the available components for your system. Then install the component you want.
```
ego install
ego install <component>
```
