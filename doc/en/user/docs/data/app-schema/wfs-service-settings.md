# WFS Service Settings {: #app-schema.wfs-service-settings }

There are two GeoServer WFS service settings that are strongly recommended for interoperable complex feature services. These can be enabled through the **Services --> WFS** page on the GeoServer web interface or by manually editing the **`wfs.xml`** file in the data directory,

## Canonical schema location

The default GeoServer behaviour is to encode WFS responses that include a `schemaLocation` for the WFS schema that is located on the GeoServer instance. A client will not know without retrieving the schema whether it is identical to the official schema hosted at `schemas.opengis.net`. The solution is to encode the `schemaLocation` for the WFS schema as the canonical location at `schemas.opengis.net`.

To enable this option, choose *one* of these:

1.  Either: On the **Service --> WFS** page under *Conformance* check *Encode canonical WFS schema location*.

2.  Or: Insert the following line before the closing tag in **`wfs.xml`**:

        <canonicalSchemaLocation>true</canonicalSchemaLocation>

## Encode using featureMember

By default GeoServer will encode WFS 1.1 responses with multiple features in a single `gml:featureMembers` element. This will cause invalid output if a response includes a feature at the top level that has already been encoded as a nested property of an earlier feature, because there is no single element that can be used to encode this feature by reference. The solution is to encode responses using `gml:featureMember`.

To enable this option, choose *one* of these:

1.  Either: On the **Service --> WFS** page under *Encode response with* select *Multiple "featureMember" elements*.

2.  Or: Insert the following line before the closing tag in **`wfs.xml`**:

        <encodeFeatureMember>true</encodeFeatureMember>