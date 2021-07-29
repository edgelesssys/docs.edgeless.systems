# Welcome to EdgelessDB

In a nutshell, EdgelessDB is a SQL database architected for the SGX environment. Like a normal SQL database, EdgelessDB writes a transaction log to disk and also keeps cold data on disk. With EdgelessDB, all data on disk is strongly encrypted. Data is only ever decrypted inside SGX enclaves.

Inherent to EdgelessDB is the concept of a *manifest*. Before an instance of EdgelessDB becomes operational, it needs to be initialized with a manifest. The manifest is a simple JSON file that defines how the data stored in EdgelessDB can be accessed by different parties. Clients can (and should) verify that a given instance of EdgelessDB adheres to a certain manifest before they connect via TLS.

Clients talk to EdgelessDB via TLS. On the side of EdgelessDB, all TLS connections terminate inside secure enclaves. EdgelessDB has two interfaces: a standard SQL interface and a REST-like HTTPS interface. The SQL interface typically requires certificate-based client authentication, whereas the HTTPS interface accepts anonymous client connections.

## TODOs:

- Fill SEO fields in `index.html` and uncomment block.
- Replace logos in `/_assets` folder with EDB logos.
- Create some markdown files in the `/content` folder.
- Add new sites to the `_sidebar.md`.