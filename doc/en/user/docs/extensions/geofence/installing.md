---
render_macros: true
---

# Installing the GeoServer GeoFence extension {: #geofence_install }

For version 2.15 and later, use the standard procedure to install an extension.

1.  Visit the [website download](https://geoserver.org/download) page, locate your release, and download: `geofence`{.interpreted-text role="download_extension"}

    The download link will be in the **Extensions** section under **Other**.

    !!! warning

        Ensure to match plugin (example {{ release }} above) version to the version of the GeoServer instance.

2.  Extract the files in this archive to the **`WEB-INF/lib`** directory of your GeoServer installation.

3.  Restart GeoServer