# Install EdgelessDB

EdgelessDB is provided as a Docker image. There are two flavors:
* `ghcr.io/edgelesssys/edgelessdb-sgx-1gb` with 1 GB of enclave heap memory
* `ghcr.io/edgelesssys/edgelessdb-sgx-4gb` with 4 GB of enclave heap memory

Use `edgelessdb-sgx-1gb` primarily to test EdgelessDB on machines with limited resources. Prefer `edgelessdb-sgx-4gb` for production deployments.

?> A future version of EdgelessDB will have a dynamic heap size.

## Prepare the SGX system
Skip this section if you want to run EdgelessDB in simulation mode. You may also skip this section if you are running on an SGX-enabled VM in Azure (DC2 series).

### Hardware
The hardware must support SGX-FLC and it must be enabled in the BIOS. Use the following commands to check:
```bash
sudo apt install cpuid
cpuid | grep SGX
```

This should give you output like the following:
```shell-session
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
```bash
ls /dev/*sgx*
```

If the output is empty, install the driver:
```bash
wget https://download.01.org/intel-sgx/sgx-linux/2.12/distro/ubuntu`lsb_release -rs`-server/sgx_linux_x64_driver_1.36.2.bin
chmod +x sgx_linux_x64_driver_1.36.2.bin
sudo ./sgx_linux_x64_driver_1.36.2.bin
```

## Run the Docker image
Run EdgelessDB on an SGX-capable system:
```bash
docker run -t \
  --name my-edb \
  -p3306:3306 \
  -p8080:8080 \
  --privileged -v /dev/sgx:/dev/sgx \
  ghcr.io/edgelesssys/edgelessdb-sgx-1gb
```

Or try it in simulation mode on any system:
```bash
docker run -t \
  --name my-edb \
  -p3306:3306 \
  -p8080:8080 \
  -e OE_SIMULATION=1 \
  ghcr.io/edgelesssys/edgelessdb-sgx-1gb
```

This exposes two services:
* The MySQL interface served on port 3306
* The HTTP REST API on port 8080

## Storage
If EdgelessDB is run with one of the commands above, all data is stored inside the docker container in the `/data` directory. For a production deployment, consider using one of the [data management approaches of Docker](https://docs.docker.com/storage). E.g., to mount a directory on the host system, add `-v /my/own/datadir:/data` to the command line.
