<div class="shields">

[![License](https://img.shields.io/github/license/edgelesssys/ego)](https://github.com/edgelesssys/ego/blob/master/LICENSE)
[![GoReportCard](https://goreportcard.com/badge/github.com/edgelesssys/ego)](https://goreportcard.com/report/github.com/edgelesssys/ego)
[![Go Reference](https://pkg.go.dev/badge/github.com/edgelesssys/ego.svg)](https://pkg.go.dev/github.com/edgelesssys/ego)
[![Discord](https://img.shields.io/discord/823900998606651454?label=chat%20on%20discord)](https://discord.gg/rH8QTH56JN)
[![Twitter](https://img.shields.io/twitter/follow/EdgelessSystems)](https://twitter.com/EdgelessSystems)

</div>

# Welcome to EGo! 🎉

EGo is a framework for building *confidential apps* in Go. Confidential apps run in secure execution environments called *enclaves*. Enclaves are strongly isolated, runtime encrypted, and attestable. Enclaves can be created on Intel processors that have the SGX (Software Guard Extensions) feature.

In essence, EGo lets you run any Go program inside an enclave - without requiring modifications. Apps that benefit from enclaves are typically server applications that deal with sensitive data like cryptographic keys or payment data. HashiCorp Vault is a great example of such an app.

## Philosophy 🎓

EGo's goal is to bridge the gap between cloud-native and confidential computing. EGo's philosophy is to get as many enclave specifics out of your way as possible. Fundamentally, building and running an enclave with EGo is as simple as building and running an app with normal Go:

```bash
ego-go build myapp.go
ego sign myapp
ego run myapp
```

## Architecture 🏗

In a nutshell, EGo comprises a modified Go compiler, additional tooling, and a Go library.

The compiler compiles your code in such a way that it runs inside an enclave. With the help of some magic ✨, EGo-compiled apps can also still be run like normal apps outside enclaves. This makes the dev process and debugging easy.

EGo's tools take care of things like signing enclaves. EGo also comes with a GDB debugger that lets you debug Go code inside enclaves.

The library makes two key features of enclaves easily accessible from Go:

**Remote attestation** 🔍: prove to a client that you're indeed running inside an enclave and that you have a certain hash. This is typically used to bootstrap attested TLS connections to an enclave.

**Sealing** 📧: securely store data to the untrusted disk.
