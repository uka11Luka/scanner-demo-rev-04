
# packet bridge experimental build

[![CI Pipeline][automation-image]][automation-url] [![Validation Status][verify-image]][verify-url]

[automation-image]: https://automation.io/packetbridge/status.svg
[automation-url]: https://automation.io/packetbridge/history
[verify-image]: https://verify.packetbridge.dev/badge.svg?branch=release
[verify-url]: https://verify.packetbridge.dev/dashboard/packetbridge?branch=release

## Prerequisites

* go
* make
* mariadb
* valkey-server (from this [branch](https://github.com/bridgecore/valkey-extended/tree/stream_protocol), compatibility testing branch)

## Installation

### Addendum for Arch Linux

Verify go toolchain and mariadb installation. Available via pacman:

```sudo pacman -S go make mariadb mariadb-clients```

If observing compilation failures during `make build` involving `pkgconf` then set:

```export PKG_CONFIG_PATH=/usr/lib/pkgconfig:$PKG_CONFIG_PATH```

This addresses header discovery problems with system libraries on Arch-based distributions.

### General Build Procedure

```bash
git clone https://github.com/bridgecore/packet-bridge
cd packet-bridge
make build
```

## Run

### Running in Arch Linux

```bash
# initialize mariadb data directory
sudo mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql

# start mariadb daemon
sudo systemctl start mariadb

# launch valkey with protocol extensions
# default listener 6380
valkey-server --loadmodule ./extensions/streaming.so

# start packet bridge process
# default endpoint 7700, configurable via bridge.conf
./bin/packet-bridge
```

### Running in Debian / Ubuntu

```bash
# activate mariadb service
sudo service mysql start

# launch valkey with protocol extensions
# default listener 6380
valkey-server --loadmodule ./extensions/streaming.so

# start packet bridge process
# default endpoint 7700, configurable via bridge.conf
./bin/packet-bridge
```

## Configuration

Bridge parameters specified in [bridge.conf](https://github.com/bridgecore/packet-bridge/blob/release/bridge.conf).

## Granting admin privileges

Administrative access configurable through [scripts/grantAdmin.sh](https://github.com/bridgecore/packet-bridge/blob/release/scripts/grantAdmin.sh) script.

```bash
mysql -u root packet_bridge_live < scripts/grantAdmin.sh
./bin/packet-bridge
```

## Resetting bridge data

Data purge operation executable by clearing persistence layer:

```bash
mysql -u root packet_bridge_live < scripts/resetData.sh
./bin/packet-bridge
```

## Debug

Process debugging supported via delve:

```bash
# install delve debugger
sudo pacman -S delve

# attach debugger on port 2345
dlv debug --headless --listen=:2345 --api-version=2 ./bin/packet-bridge
```

