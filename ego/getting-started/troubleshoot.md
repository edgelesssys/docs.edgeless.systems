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
sudo ego install sgx-driver
```


As an  alternative way you can install it manually. 

If your system supports FLC (see above):
```bash
wget https://download.01.org/intel-sgx/sgx-linux/2.12/distro/ubuntu`lsb_release -rs`-server/sgx_linux_x64_driver_1.36.2.bin
chmod +x sgx_linux_x64_driver_1.36.2.bin
sudo ./sgx_linux_x64_driver_1.36.2.bin
```

Otherwise:
```bash
wget https://download.01.org/intel-sgx/sgx-linux/2.13.3/distro/ubuntu`lsb_release -rs`-server/sgx_linux_x64_driver_2.11.0_2d2b795.bin
chmod +x sgx_linux_x64_driver_2.11.0_2d2b795.bin
sudo ./sgx_linux_x64_driver_2.11.0_2d2b795.bin
```

### Required packages
If your system doesn't support FLC, install the `libsgx-launch` package:
```bash
sudo ego install libsgx-launch
```

Or manually:
```bash
wget -qO- https://download.01.org/intel-sgx/sgx_repo/ubuntu/intel-sgx-deb.key | apt-key add
sudo add-apt-repository "deb [arch=amd64] https://download.01.org/intel-sgx/sgx_repo/ubuntu `lsb_release -cs` main"
sudo apt install libsgx-launch
```

## Attestation
If EGo works in SGX mode (i.e., without `OE_SIMULATION`), but attestation fails, check the following.

### FLC
Attestation only works on [SGX-FLC](#hardware) systems.

### Quote provider
You must install a [quote provider](../reference/quoteprov.md).
