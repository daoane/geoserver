# Features-Templating Extension

The Features Templating plug-in works by allowing us to define a What You See Is What You Get template, that will be applied on a stream of features respecting the defined content negotiation rules. Both Simple and Complex features are supported. The following services and operations are supported:

| **Service**| **Operation**|
|-|-|
| WFS | GetFeature 
| WMS | GetFeatureInfo |
| OGCAPI Features | Collection |

The following output formats are supported:

-   `GeoJSON`
-   `GML`
-   `JSON-LD` [JSON-LD](https://json-ld.org) is a Linked Data format, based on JSON format, and revolves around the concept of "context" to provide additional mappings from JSON to an RDF model.
-   `HTML`

The first part of the plug-in documentation will go through the template syntax. The second one will show how to configure the template to apply it to a vector layer. The third part shows the backwards mapping functionality.

<div class="grid cards" markdown>

-   [Installing the GeoServer FEATURES-TEMPLATING extension](installing.md)
-   [Template Directives](directives.md)
-   [Template Configuration](configuration.md)
-   [Backward Mapping](querying.md)
-   [Features Templatring Rest API](rest.md)

</div>