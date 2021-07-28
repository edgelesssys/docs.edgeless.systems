# Configuration
EDB is configured via environment variables. Add them as flags `-e NAME=value` to the `docker run` command line.

* `EDG_EDB_DATABASE_ADDR`: The network address of the MySQL interface. Defaults to `0.0.0.0:3306`.
* `EDG_EDB_API_ADDR`: The network address of the HTTP REST API. Defaults to `0.0.0.0:8080`.
* `EDG_EDB_DEBUG`: set to `1` to enable debug logging to the terminal. The [manifest](manifest.md) must allow this because logs may leak data.
* `EDG_EDB_LOG_DIR`: like `EDG_EDB_DEBUG`, but log to files. Set this, e.g., to `/log` and mount a host directory by adding `-v /path/to/log:/log` to the `docker run` command line.
