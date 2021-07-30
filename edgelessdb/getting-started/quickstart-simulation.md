# Quickstart: simulation mode
In this guide, you will set up EDB with a minimal manifest and connect to it with the `mysql` client.

!> Simulation mode is only suitable for evaluating EDB. The setup is not secure in terms of confidential computing. For a setup in production, continue with [SGX mode](quickstart-sgx.md).

## Start EDB
Run the EDB Docker image:
```console
$ docker run --name my-edb -p3306:3306 -p8080:8080 -e OE_SIMULATION=1 -t ghcr.io/edgelesssys/edgelessdb-sgx-1gb

[erthost] running in simulation mode
[erthost] loading enclave ...
[erthost] entering enclave ...
[EDB] 2021/07/27 15:50:12 DB has not been initialized, waiting for manifest.
...
ERROR: SGX Plugin _get_report(): failed to get ecdsa report. OE_PLATFORM_ERROR (oe_result_t=OE_PLATFORM_ERROR) [openenclave-src/enclave/sgx/attester.c:_get_report:320]
Failed to get quote: OE_PLATFORM_ERROR
```

Note that EDB is now waiting for the [manifest](concepts.md#manifest). Also note that EDB can't get an SGX attestation quote in simulation mode.

## Generate certificates and create a manifest
You will now create a manifest that defines a root user. This user is authenticated by an X.509 certificate.

Generate a CA to issue user certificates. Generate a user certificate and sign it:
```sh
openssl req -x509 -newkey rsa -nodes -subj '/CN=My CA' -keyout ca-key.pem -out ca-cert.pem
openssl req -newkey rsa -nodes -subj '/CN=rootuser' -keyout key.pem -out csr.pem
openssl x509 -req -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial -in csr.pem -out cert.pem
```

Escape the line breaks of the CA certificate:
```sh
awk 1 ORS='\\n' ca-cert.pem
```

Create a file `manifest.json`:
```json
{
    "sql": [
        "CREATE USER root REQUIRE ISSUER '/CN=My CA' SUBJECT '/CN=rootuser'",
        "GRANT ALL ON *.* TO root WITH GRANT OPTION"
    ],
    "ca": "-----BEGIN CERTIFICATE-----\n...\n-----END CERTIFICATE-----\n"
}
```

`sql` is a list of SQL statements that define the initial state of the database. The two statements above create a root user that is authenticated by the user certificate you generated.

Replace the value of `ca` with the escaped content of `ca-cert.pem`.

## Initialize EDB with the manifest
Obtain the attested EDB root certificate so that you can send the manifest securely to EDB.

Install the [Edgeless remote attestation (era)](https://github.com/edgelesssys/era) tool.

Then get the EDB attestation configuration and use `era` to get the root certificate of your EDB instance:
```console
$ wget https://github.com/edgelesssys/edgelessdb/releases/latest/download/edgelessdb-sgx.json
$ era -c edgelessdb-sgx.json -h localhost:8080 -output-root edb.pem -skip-quote

WARNING: Skipping quote verification
Root certificate writen to edb.pem
```
In simulation mode you must skip quote verification with `-skip-quote`.

Initialize EDB with the manifest:
```sh
curl --cacert edb.pem --data-binary @manifest.json https://localhost:8080/manifest
```

## Use EDB
Now you can use EDB like any other SQL database:
```sh
mysql -h127.0.0.1 -uroot --ssl-ca edb.pem --ssl-cert cert.pem --ssl-key key.pem
```

For a more advanced example of EDB's CC features, see the [demo that shows a secure multi-party data processing](https://github.com/edgelesssys/edgelessdb/tree/main/demo) scenario.
