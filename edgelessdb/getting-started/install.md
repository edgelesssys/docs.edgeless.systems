# Install EDB

EDB is provided as a Docker image. There are two flavors:
* `ghcr.io/edgelesssys/edb-sgx-1gb` with 1 GB of enclave heap memory
* `ghcr.io/edgelesssys/edb-sgx-4gb` with 4 GB of enclave heap memory

Use `edb-sgx-1gb` primarily to test EDB on machines with limited resources. Prefer `edb-sgx-4gb` for production deployments.

?> A future version of EDB will have a dynamic heap size.

## Prepare the SGX system
Skip this section if you want to run EDB in simulation mode.

### Hardware
The hardware must support SGX-FLC and it must be enabled in the BIOS:
```sh
$ sudo apt install cpuid
$ cpuid | grep SGX
      SGX: Software Guard Extensions supported = true
      SGX_LC: SGX launch config supported      = true
   SGX capability (0x12/0):
      SGX1 supported                         = true
```
* `SGX: Software Guard Extensions supported` is true if the hardware supports it.
* `SGX_LC: SGX launch config supported` is true if the hardware also supports FLC.
* `SGX1 supported` is true if it is enabled in the BIOS.

### Driver
The SGX driver exposes a device:
```sh
ls /dev/*sgx*
```

If the output is empty, install the driver:
```sh
wget https://download.01.org/intel-sgx/sgx-linux/2.12/distro/ubuntu`lsb_release -rs`-server/sgx_linux_x64_driver_1.36.2.bin
chmod +x sgx_linux_x64_driver_1.36.2.bin
sudo ./sgx_linux_x64_driver_1.36.2.bin
```

## Run the Docker image
Run EDB on an SGX-capable system:
```sh
docker run -t \
  --name my-edb \
  -p3306:3306 \
  -p8080:8080 \
  --privileged -v /dev/sgx:/dev/sgx \
  ghcr.io/edgelesssys/edb-sgx-1gb
```

Or try it in simulation mode on any system:
```sh
docker run -t \
  --name my-edb \
  -p3306:3306 \
  -p8080:8080 \
  -e OE_SIMULATION=1 \
  ghcr.io/edgelesssys/edb-sgx-1gb
```

This exposes two services:
* The MySQL interface served on port 3306
* The HTTP REST API on port 8080

## Storage
If EDB is run with one of the commands above, all data is stored inside the docker container. For a production deployment, consider using one of the [data management approaches of Docker](https://docs.docker.com/storage). E.g., to mount a directory on the host system, add `-v /my/own/datadir:/data` to the command line.