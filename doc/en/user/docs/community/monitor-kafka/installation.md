# Installing the Kafka Monitor Extension {: #monitor_kafka_installation }

As a community module, the package needs to be downloaded from the [nightly builds](https://build.geoserver.org/geoserver/), picking the community folder of the corresponding GeoServer series (e.g. if working on GeoServer main development branch nightly builds, pick the zip file form `main/community-latest`).

To install the module, unpack the zip file contents into GeoServer own `WEB-INF/lib` directory and restart GeoServer.

For the module to work, the [Monitoring](../../extensions/monitoring/index.md) extensions must also be installed.