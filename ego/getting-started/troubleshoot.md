# Troubleshooting

## SGX
If EGo works in simulation mode (with `OE_SIMULATION=1`), but not with SGX, check the following.

### Operating system
EGo currently supports Ubuntu 18.04 and 20.04.

### Hardware
The hardware must support SGX and it must be enabled in the BIOS:
```shell-session
$ sudo apt install cpuid
$ cpuid | grep SGX
      SGX: Software Guard Extensions supported = true
      SGX_LC: SGX launch config supported      = true
   SGX capability (0x12/0):
      SGX1 supported                         = true
```
* `SGX: Software Guard Extensions supported` is true if the hardware supports it.
* `SGX_LC: SGX launch config supported` is true if the hardware also supports FLC. This is required for attestation.
* `SGX1 supported` is true if it is enabled in the BIOS.

### Driver
The SGX driver exposes a device:
```bash
ls /dev/*sgx*
```
If the output is empty, install the driver.

The easiest way to install the driver is using ego install:
```bash
ego install sgx-driver
```

### Required packages
If your system doesn't support FLC, install the `libsgx-launch` package:
```bash
ego install libsgx-launch
```

## Attestation
If EGo works in SGX mode (i.e., without `OE_SIMULATION`), but attestation fails, check the following.

### FLC
Attestation only works on [SGX-FLC](#hardware) systems.

### Quote provider
You must install a [quote provider](../reference/quoteprov.md).
