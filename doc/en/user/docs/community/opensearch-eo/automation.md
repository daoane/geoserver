# Automation with the administration REST API {: #opensearch_automation }

The OpenSearch module supports full automation REST API that can be used to create collections, ingest products and eventually their granules. The full API is available at this URL:

-   [/oseo](https://docs.geoserver.org/latest/en/api/#1.0.0/opensearch-eo.yaml)

In general terms, one would:

-   Create a collection, along with thumbnail, OGC links
-   Then create a product, along with thumbnail, OGC links
-   Finally, and optionally, specify the granules composing the product (actually needed only if the OpenSearch subsystem is meant to be used for publishing OGC services layers too, instead of being a simple search engine.

## Understanding the zip file uploads

The description of a collection and product is normally made of various components, in order to expedite data creation and reduce protocol chattines, it is possible to bulk-upload all files composing the description of collections and products as a single zip file.

### Collection components

A collection.zip, sent as a PUT request to `rest/collections` would contain the following files:

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Name                                             Optional   Description
  ------------------------------------------------ ---------- -----------------------------------------------------------------------------------------------------------------------
  collection.json                                  N          The collection attributes, matching the database structure (the prefixes are separated with a colon in this document)

  description.html                                 Y          The HTML description for the collection

  thumbnail.png, thumbnail.jpg or thumbnail.jpeg   Y          The collection thumbnail

  owsLinks.json                                    Y          The list of OWS links to OGC services providing access to the collection contents (typically as a time enabled layer)
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### Product components

A product.zip, sent as a POST request to `rest/collections/<theCollection>/products` would contain the following files:

||
||

It is also possible to send the zip file on the `rest/collections/<theCollection>/products/<theProduct>` resource as a PUT request, it will update an existing product by replacing the parts contained in the file. Parts missing from the file will be kept unchanged, to remove them run an explicit DELETE request on their respective REST resources.

## Usage of the API for search and integrated OGC service publishing

In this case the user intend to both use the OpenSearch module for search purposes, but also to publish actual mosaics for each collection.

In this case the following approach should is recommended:

-   Create a collection via the REST API, using the ZIP file POST upload
-   Create at least one product in the collection in the REST API, using the ZIP file POST upload and providing a full `granules.json` content with all the granules of said product
-   Post a layer publishing description file to `/oseo/collection/{COLLECTION}/layers` to have the module setup a set of mosaic configuration files, store, layer with eventual coverage view and style

A collection can have multiple layers:

-   Getting the `/oseo/collection/{COLLECTION}/layers` resource returns a list of the available ones
-   `/oseo/collection/{COLLECTION}/layers/{layer}` returns the specific configuration (PUT can be used to modify it, and DELETE to remove it).
-   Creation of a layer configuration can be done either by post-ing to `/oseo/collection/{COLLECTION}/layers` or by put-int to `/oseo/collection/{COLLECTION}/layers/{layer}`.

The layer configuration fields are:

||
||

The layer configuration specification will have different contents depending on the collection structure:

-   Single CRS, non band split, RGB or RGBA files, time configured as an "instant" (only `timeStart` used):

    ``` json
    {
        "workspace": "gs",
        "layer": "test123",
        "separateBands": false,
        "heterogeneousCRS": false,
        "timeRanges": false
    }
    ```

-   Single CRS, multiband in single file, with a gray browse style, product time configured as a range between `timeStart` and `timeEnd`:

    ``` json
    {
        "workspace": "gs",
        "layer": "test123",
        "separateBands": false,
        "browseBands": ["test123[0]"],
        "heterogeneousCRS": false,
        "timeRanges": true
    }
    ```

-   Heterogeneous CRS, multi-band split across files, with a RGB browse style ("timeRanges" not specified, implying it's handled as an instant):

    ``` json
    {
        "workspace": "gs",
        "layer": "test123",
        "separateBands": true,
        "bands": [
            "VNIR",
            "QUALITY",
            "CLOUDSHADOW",
            "HAZE",
            "SNOW"
        ],
        "browseBands": [
            "VNIR[0]", "VNIR[1]", "SNOW"
        ],
        "heterogeneousCRS": true,
        "mosaicCRS": "EPSG:4326"
    }
    ```

In terms of band naming the "bands" parameter contains coverage names as used in the "band" column of the granules table, in case a granule contains multiple bands, they can be referred by either using the full name, in which case they will be all picked, or by using zero-based indexes like `BANDNAME[INDEX]`, which allows to pick a particular band.

The same syntax is meant to be used in the `browseBands` property. In case the source is not split band, the `browseBands` can still be used to select specific bands, using the layer name as the coverage name, e.g. "test123[0]" to select the first band of the coverage.

### COG Mosaic creation

It's also possible to configure a layer on top of a COG ImageMosaic, provided that the [COG (Cloud Optimized GeoTIFF) Support](../cog/cog.md) plugin has been installed in GeoServer.

Additional fields for the layer configuration are:

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------
  Attribute             Description
  --------------------- -------------------------------------------------------------------------------------------------------------------------------------------
  cog                   Set it to true to specify the layer is made of COG datasets

  cogUser               (Optional) Credential to be set whenever basic HTTP authentication is needed to access the COG Datasets or an S3 Access KeyID is required

  cogPassword           (Optional) Password for the above user OR Secret Access Key for the above S3 KeyId.

  cogRangeReader        (Optional) Specify the desired RangeReader implementation. (default is HTTP based)
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------

See [COG RangeReader](../cog/cog.md#cog_plugin_rangereader) from the COG plugin documentation, for the list of supported RangeReader implementations.