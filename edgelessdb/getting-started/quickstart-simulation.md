# Quickstart: simulation mode
In this guide, you will set up EdgelessDB with a minimal manifest and connect to it with the `mysql` client.

!> Simulation mode is only suitable for evaluating EdgelessDB. The setup is not secure in terms of confidential computing. For a setup in production, continue with [SGX mode](quickstart-sgx.md).

## Start EdgelessDB
Run the EdgelessDB Docker image:
```bash
docker run --name my-edb -p3306:3306 -p8080:8080 -e OE_SIMULATION=1 -t ghcr.io/edgelesssys/edgelessdb-sgx-1gb
```
This should give the following output:
```shell-session
[erthost] running in simulation mode
[erthost] loading enclave ...
[erthost] entering enclave ...
[EDB] 2021/07/27 15:50:12 DB has not been initialized, waiting for manifest.
...
ERROR: SGX Plugin _get_report(): failed to get ecdsa report. OE_PLATFORM_ERROR (oe_result_t=OE_PLATFORM_ERROR) [openenclave-src/enclave/sgx/attester.c:_get_report:320]
Failed to get quote: OE_PLATFORM_ERROR
```

The error is expected, because EdgelessDB can't get an SGX attestation quote in simulation mode. EdgelessDB is now waiting for the [manifest](concepts.md#manifest).

## Generate certificates and create a manifest
You will now create a manifest that defines a root user. This user is authenticated by an X.509 certificate.

Generate a certificate authority (CA) and a corresponding user certificate:
```bash
openssl req -x509 -newkey rsa -nodes -subj '/CN=My CA' -keyout ca-key.pem -out ca-cert.pem
openssl req -newkey rsa -nodes -subj '/CN=rootuser' -keyout key.pem -out csr.pem
openssl x509 -req -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial -in csr.pem -out cert.pem
```

Escape the line breaks of the CA certificate:
```bash
awk 1 ORS='\\n' ca-cert.pem
```

Create a file `manifest.json` with the following contents:
```json
{
    "sql": [
        "CREATE USER root REQUIRE ISSUER '/CN=My CA' SUBJECT '/CN=rootuser'",
        "GRANT ALL ON *.* TO root WITH GRANT OPTION"
    ],
    "ca": "-----BEGIN CERTIFICATE-----\n...\n-----END CERTIFICATE-----\n"
}
```

`sql` is a list of SQL statements that define the initial state of the database. The two statements above create a root user that is authenticated by the user certificate you just generated.

Replace the value of `ca` with the escaped content of `ca-cert.pem`.

## Verify your EdgelessDB instance
Before you can trust your EdgelessDB instance, you first need to verify that it is in a good shape. You can use the [Edgeless Remote Attestation (era)](https://github.com/edgelesssys/era) tool for this. If you're just getting started, you may also skip this part.

Once you've installed `era`, you can get the attested root certificate of your EdgelessDB instance as follows:
```bash
wget https://github.com/edgelesssys/edgelessdb/releases/latest/download/edgelessdb-sgx.json
era -c edgelessdb-sgx.json -h localhost:8080 -output-root edb.pem -skip-quote
```

Here, `edgelessdb-sgx.json` contains the expected properties of your EdgelessDB instance. However, in simulation mode, you need to skip the actual verification of the properties via the `-skip-quote` option.

```shell-session
WARNING: Skipping quote verification
Root certificate written to edb.pem
```

## Set the manifest
You're now ready to send the manifest over a secure TLS connection based on the attested root certificate of your EdgelessDB instance:
```bash
curl --cacert edb.pem --data-binary @manifest.json https://localhost:8080/manifest
```

In case you skipped the verification step above, just replace `--cacert edb.pem` with `-k` in the above command.

## Use EdgelessDB
Now you can use EdgelessDB like any other SQL database:
```bash
mysql -h127.0.0.1 -uroot --ssl-ca edb.pem --ssl-cert cert.pem --ssl-key key.pem
```

For an example of EdgelessDB's confidential-computing features, see the [demo of a secure multi-party data processing app](https://github.com/edgelesssys/edgelessdb/tree/main/demo).
