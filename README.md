Notes on the NISO [Access and License Indicators](http://www.niso.org/workrooms/ali/) Working Group’s “[Recommended Practice](http://www.niso.org/publications/rp/rp-22-2015)” document and [schema](http://www.niso.org/schemas/ali/1.0/).

## Background

When a journal article is published, the publisher submits the article metadata as XML to CrossRef, which registers the DOI.

Many research funding organisations have mandated that journal articles resulting from research that they fund must be Open Access (or at least “free to read”) within a certain amount of time after publication. The proposal of [CHORUS](http://www.chorusaccess.org/) is that journal publishers add two new elements to the metadata submitted to CrossRef, specifying a) on which date the article will become free to read and b) the license (e.g. CC-BY) which will be applied to the article on that date.

This information would allow services to query CrossRef and retrieve only those articles which are free to read and/or have a specific license.

## XML

The proposal of the NISO Working Group for the new XML elements are these:

```xml
<license_ref xmlns=“http://www.niso.org/schemas/ali/1.0/“ start_date=“2014-01-01”>https://creativecommons.org/licenses/by/3.0/</license_ref>
```

and

```xml
<free_to_read xmlns=“http://www.niso.org/schemas/ali/1.0/“/>
```

or (with a start ± end date)

```xml
<free_to_read xmlns=“http://www.niso.org/schemas/ali/1.0/“ start_date=“2014-01-01”></free_to_read>
```

## Problems

* The [schema](http://www.niso.org/schemas/ali/1.0/ali.xsd) specifies that the `start_date` attribute (but not the `end_date` attribute) is required on the `free_to_read` element. However, this is a confusing situation: in some cases `free_to_read` can be an object with `start_date` ± `end_date` (XML), or it can be an empty `<free_to_read/>` element (XML), or it can be a boolean `”free_to_read”: true` value (JSON-LD).
* The element names are underscored, fitting CrossRef’s existing XML style. This is fine if they're only being used for submission to CrossRef, but not ideal if they're [being re-used elsewhere](http://jats.nlm.nih.gov/1.1d3/), where hyphenated XML element names are more common.
* The `start_date` and `end_date` should be full date + time with a timezone, otherwise it’s not clear when they should be applied. Are the dates inclusive, for example?
* Following common practice elsewhere, the URL would be better in an attribute rather than as the node contents.
* The “license_ref” property could have a more meaningful name.

Here’s what the XML element could look like:

```xml
<license url=“https://creativecommons.org/licenses/by/3.0/” start=“2014-01-01T00:00:00Z”/>
```

or (matching the JSON-LD example below)

```xml
<license-period license=“https://creativecommons.org/licenses/by/3.0/” start-date=“2014-01-01T00:00:00Z”/>
```

## JSON-LD

As well as the XML schema, the working group has also produced [a JSON-LD context document](http://www.niso.org/schemas/ali/1.0/), which is well-intentioned but currently quite broken.

* On the [version-less schema page](http://www.niso.org/schemas/ali/) the JSON-LD link is broken.
* On the 1.0 schema page (the current version), the file is served as “application/octet-stream” rather than “application/json”.
* The file has no “Access-Control-Allow-Origin: *” header, so it can’t be fetched automatically into [a client-side JSON-LD processor](http://json-ld.org/playground/)
* The `@vocab` URL should be “http://www.niso.org/schemas/ali/1.0/”, not “http://www.niso.org/schemas/ali/1.0/jsonld.json”.
* The `@type` attribute of several of the properties is wrong: a value of `@id` says that the value is a string value that represents an IRI, which these aren't.
* The underscored property names would be better as camelCase.
* The “uri” property should either be an “@id” property (which is a URI) or something more meaningful.
* The type of the `free_to_read` property can’t be defined, because it can be either null (in XML), “true” (in JSON-LD), or an object with “start_date” and “end_date” properties.

Here’s what the context document could look like:

```json
{
    "@context": {
        "@vocab": "http://www.niso.org/schemas/ali/1.0/",
        "startDate": {
            "@type": "http://www.w3.org/2001/XMLSchema#date"
        },
        "endDate": {
            "@type": "http://www.w3.org/2001/XMLSchema#date"
        },
        "license": {
            "@type": "@id"
        }
    }
}
```

and an example document:

```json
{
    "@context": "http://www.niso.org/schemas/ali/1.0/jsonld.json",
    "freeToRead": {
        "startDate": "2014-04-04T00:00:00Z"
    },
    "licensePeriod": [
        {
            "license": "http://creativecommons.org/licenses/by/3.0/",
            "startDate": "2014-04-04T00:00:00Z"
        },
        {
            "license": "http://creativecommons.org/licenses/by/4.0/",
            "startDate": "2015-04-04T00:00:00Z"
        }
    ]
}
```


