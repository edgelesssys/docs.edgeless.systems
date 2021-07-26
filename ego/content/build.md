# Build your app with EGo 👷‍♀️

## `ego-go` replaces `go`

EGo provides a very simple way to build your Go project: `ego-go`, an adapted Go compiler that builds enclave-compatible executables from a given Go project—while providing the same CLI as the original Go compiler. You can build your app with

```bash
ego-go build
```

## Sign and run

After you've built your app, sign it with the [`ego sign`](content/cli.md#sign) command. Just insert the name of your binary in the following command:

```bash
ego sign <executable>
```

Run the executable with

```bash
ego run <executable>
```
You can set the `OE_SIMULATION=1` environment variable if you want to simulate the enclave, e.g. for development on hardware that does not support enclaves.

Look at our [example collection](content/examples.md) if you want to see the build process in action.
