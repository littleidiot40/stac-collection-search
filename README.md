# STAC API - Collection Search Specification

- STAC API - Collection Search Specification
  - [Link Relations](#link-relations)
  - [Endpoints](#endpoints)
  - [Query Parameters and Fields](#query-parameters-and-fields)
    - [Query Examples](#query-examples)
    - [Query Parameters Table](#query-parameters-table)
  - [Response](#response)
    - [Pagination](#pagination)
  - [HTTP Request Methods and Content Types](#http-request-methods-and-content-types)
    - [GET](#get)
    - [POST](#post)
  - [Example Landing Page for STAC API - Collection Search](#example-landing-for-stac-api---collection-search)
  - [Example catalog result for STAC API - Collection Search](#example-catalog-result-for-stac-api---collection-search)
  - [Extensions](#extensions)

- **OpenAPI specification:** [openapi.yaml](openapi.yaml)
- **Conformance URIs:**
  - <https://api.stacspec.org/v1.0.0-rc.2/collection-search>
- **Maturity Classification:** Proposal
- **Dependencies**:
  - [Core](https://github.com/radiantearth/stac-api-spec/tree/main/core)

A collection search endpoint provides the ability to filter STAC collections. 
It retrieves a group of collections that match the provided parameters, wrapped in a [STAC Catalog](https://github.com/radiantearth/stac-api-spec/tree/main/stac-spec/catalog-spec).

Implementing GET /search is required, POST /search is optional, but recommended.

## Link Relations
The following Link relation must exist in the Landing Page (root).

| **rel**        | **href**       | **From**  | **Description**                                     |
| -------------- | -------------  | --------- | --------------------------------------------------- |
| `search`       | `/collections` | Extension | **REQUIRED** URL for the Collection Search endpoint |

This search link relation must have a type of `application/json`. If no method attribute is specified, 
it is assumed to represent a GET request. If the server supports both GET and POST requests, two links should be included, 
one with a method of GET one with a method of POST.

## Endpoints

This conformance class also requires for the endpoints in the [STAC API - Core](https://api.stacspec.org/v1.0.0-rc.2/core) 
conformance class to be implemented.

These endpoints are required, with details provided in this [OpenAPI specification document](openapi.yaml).

| Endpoint       | Returns                                                                                      | Description                          |
| -------------- | -------------------------------------------------------------------------------------------- | ------------------------------------ |
| `/collections` | [Catalog](https://github.com/radiantearth/stac-api-spec/tree/main/stac-spec/catalog-spec) | Returns a filterable set of [collections](https://docs.opengeospatial.org/is/17-069r3/17-069r3.html#_collections_) |

## Query Parameters and Fields

The following list of parameters is used to narrow search results. They can all be represented as query string parameters in a GET request, 
or as JSON entity fields in a POST request. For filters that represent a set of values, 
query parameters must use comma-separated string values with no enclosing brackets (\[ or \]),
no whitespace between values, and JSON entity attributes must use JSON Arrays.

### Query Examples

`GET /collections?bbox=-10.415,36.066,3.779,44.213&limit=200&datetime=2017-05-05T00:00:00Z`

```json
{
    "bbox": [10.415,36.066,3.779,44.213],
    "limit": 200,
    "datetime": "2017-05-05T00:00:00Z"
}
```
### Query Parameters Table
The core parameters for STAC collection search are analogous to [STAC API - Item Search](https://github.com/radiantearth/stac-api-spec/tree/master/item-search) 
with the addition of several query parameters from [OGC API-Records](https://docs.ogc.org/DRAFTS/20-004.html#_query_parameters) 
to facilitate efficient collection discovery.

| Parameter   | Type             | Source API | Description                                                                                                                                                                     |
| ----------- | ---------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| limit       | integer          | [OAFeat](https://docs.ogc.org/DRAFTS/20-004.html#_parameter_limit)     | **REQUIRED** The maximum number of results to return (page size).                                                                                       |
| bbox        | \[number]        | [OAFeat](https://docs.ogc.org/DRAFTS/20-004.html#_parameter_bbox)     | **REQUIRED** Requested bounding box.                                                   |
| datetime    | string           | [OAFeat](https://docs.ogc.org/DRAFTS/20-004.html#_parameter_datetime)     | **REQUIRED** Single date+time, or a range ('/' separator), formatted to [RFC 3339, section 5.6](https://tools.ietf.org/html/rfc3339#section-5.6). Use double dots `..` for open date ranges. |
| intersects  | GeoJSON Geometry | STAC       | Searches Collections by performing intersection between their geometry and provided [GeoJSON geometry](https://geojson.org/)    .  All GeoJSON geometry types must be supported.           |
| ids         | \[string]        | STAC       | **REQUIRED** Set of Collections to return defined by their ids.                                                                 |
| q           | \[string]        | [OGC API-Records](https://docs.ogc.org/DRAFTS/20-004.html#_parameter_q) | **REQUIRED** String value for textual search.   |
| type        | \[string]        | [OGC API-Records](https://docs.ogc.org/DRAFTS/20-004.html#core-query-parameters-type) | Set values associated with an enumeration of types of collection |
| externalId  | \[string]        | [OGC API-Records](https://docs.ogc.org/DRAFTS/20-004.html#core-query-parameters-externalid) | Set of Collections to return defined by their external ids |

**limit** The `limit` parameter follows the same semantics of the OAFeat Item resource limit parameter. 
The value is a suggestion to the server as to the maximum number of collection objects the client would prefer in the response. 
The OpenAPI specification defines the default and maximum values for this parameter. 
The base specifications define these with a default of 10 and a maximum of 10000, 
but implementers may choose other values to advertise through their service-desc endpoint.
If the `limit` parameter value is greater than the advertised maximum limit, 
the server must return the maximum possible number of items (ideally, 
the number as the advertised maximum limit), rather than responding with an error.

**datetime** The `datetime` parameter uses the same allowed values as the OAF datetime parameter. 
This allows for either a single RFC 3339 datetime or an open or closed interval that also uses RFC 3339 datetimes.

**bbox** Represented using either 2D or 3D geometries. The length of the array must be 2*n where n is the number of dimensions. 
The array contains all axes of the southwesterly most extent followed by all axes of the northeasterly most extent 
specified in Longitude/Latitude or Longitude/Latitude/Elevation based on WGS 84. When using 3D geometries, 
the elevation of the southwesterly most extent is the minimum elevation in meters and the elevation of the northeasterly most extent is the maximum. 
When filtering with a 3D bbox over Items with 2D geometries, it is assumed that the 2D geometries are at elevation 0.

**intersects** The `intersects` parameter allow for filtering by more complex geometries than bounding box. Any [GeoJSON geometry](https://geojson.org/) may be supplied and the response will only contain collections whose spatial extent instersects with the geometry supplied

*Note:* Only one of either intersects or bbox may be specified. If both are specified, a 400 Bad Request response must be returned.

**ids** The `ids` parameter should contain one or more values corresponding to IDs of collections with the catalog.
If the `ids` parameter is provided by the client, only collections, 
as indicated by the value(s) provided, whose IDs are equal to one of the listed values shall be in the result set.

**q** Keyword searches using the `q` parameter SHALL be case insensitive. 
The specific set of text keys/fields/properties of a resource to which the q operator is applied 
SHALL be left to the discretion of the implementation.

**type** The `type` parameter should contain one or more values of an enumeration of known collection types. 
If the `type` parameter is provided by the client, only collections whose type, 
as indicated by the value(s) provided, are equal to one of the listed values shall be in the result set.

**externalId** The `externalId` parameter should contain one or more values corresponding to external IDs of collections with the catalog.
If the `externalId` parameter is provided by the client, only collections, 
as indicated by the value(s) provided, whose external IDs are equal to one of the listed values shall be in the result set.

## Response

The response to a request (GET or POST) to the collection search endpoint must always be a 
Catalog object that consists entirely of STAC Collection objects.

### Pagination

OGC API supports paging through hypermedia links and STAC follows the same pattern for a collection search. 
For GET requests, a link with `rel` type `next` is supplied. 
This link may contain any URL parameter that is necessary for the implementation to understand how to provide the next page of 
results, eg: `page`, `next`, `token`, etc. 
The parameter name is defined by the implementor and is not necessarily part of the API specification. For example:

```json
{
    "type": "Catalog",
    "collections": [],
    "links": [
        {
            "rel": "prev",
            "href": "https://stac-api.example.com/collections?page=1"
        }
        ,
        {
            "rel": "next",
            "href": "https://stac-api.example.com/collections?page=3"
        }
    ],
```
The href may contain any arbitrary URL parameter:

- `https://stac-api.example.com/collections?page=2`
- `https://stac-api.example.com/collections?next=8a35eba9c`
- `https://stac-api.example.com/collections?token=f32890a0bdb09ac3`

Implementations may also add link relations prev, first, and last, though these are not required and may be infeasible 
to implement in some data stores.

OAFeat does not support POST requests for searches, however the STAC API spec does. Hypermedia links are not designed 
for anything other than GET requests, 
so providing a next link for a POST search request becomes problematic. 
STAC has decided to extend the Link object to support additional fields that provide hints to the client as to 
how it must execute a subsequent request for the next page of results.

The following fields have been added to the Link object specification for the API spec:

| **Parameter** | **Type** | **Description** |
| ------------- | -------- | --------------- |
| method        | string   | The HTTP method of the request, usually `GET` or `POST`. Defaults to `GET` |
| headers       | object   | A dictionary of header values that must be included in the next request    |
| body          | object   | A JSON object containing fields/values that must be included in the body of the next request | 
| merge         | boolean  | If `true`, the headers/body fields in the next link must be merged into the original request and be sent combined in the next request. Defaults to `false` |

The implementor has the freedom to decide exactly how to apply these extended fields for their particular pagination mechanism. 
The same freedom that exists for GET requests, where the actual URL parameter used to defined the next page of 
results is purely up to the implementor and not defined in the API spec, if the implementor decides to use headers, 
there are no specific or required header names defined in the specification. Implementors may use any names or fields of their choosing. 
Pagination can be provided solely through header values, solely through body values, or through some combination.

To avoid returning the entire original request body in a POST response which may be arbitrarily large, the merge property can be specified. 
This indicates that the client must send the same post body that it sent in the original request, 
but with the specified headers/body values merged in. 
This allows servers to indicate what needs to change to get to the next page without mirroring the entire query structure back to the client.

## HTTP Request Methods and Content Types

STAC APIs follow the modern web API practices of using HTTP Request Methods ("verbs") and the Content-Type header '
to drive behavior on resources ("nouns"). 
This section describes how these are used with the `/collection` endpoint.

### GET

**Required:** Collection search requires GET queries for all implementations, 
following OAFeat's precedent of making GET required (it only specifies GET so far).

### POST

**Recommended** Collection search is strongly recommended to implement POST `Content-Type: application/json`, 
where the content body is a JSON object representing a query and filter, as defined in this document.

It is recommended that clients use POST for querying (if the STAC API supports it), especially when using the intersects query parameter, 
for two reasons:

In practice, the allowed size for an HTTP GET request is significantly less than that allowed for a POST request, s
o if a large geometry is used in the query it may cause a GET request to fail.
The parameters for a GET request must be escaped properly, making it more difficult to construct when using JSON
parameters (such as intersect, as well as additional filters from the query extension).

## Example Landing for STAC API - Collection Search

```json
{
    "stac_version": "1.0.0",
    "id": "example-stac",
    "title": "A simple STAC API Example",
    "description": "This Catalog aims to demonstrate the a simple landing page",
    "type": "Catalog",
    "conformsTo" : [
        "https://api.stacspec.org/v1.0.0-rc.2/core",
        "https://api.stacspec.org/v1.0.0-rc.2/collection-search"
    ],
    "links": [
        {
            "rel": "self",
            "type": "application/json",
            "href": "https://stac-api.example.com"
        },
        {
            "rel": "root",
            "type": "application/json",
            "href": "https://stac-api.example.com"
        },
        {
            "rel": "service-desc",
            "type": "application/vnd.oai.openapi+json;version=3.0",
            "href": "https://stac-api.example.com/api"
        },
        {
            "rel": "service-doc",
            "type": "text/html",
            "href": "https://stac-api.example.com/api.html"
        },
        {
            "rel": "search",
            "type": "application/json",
            "href": "https://stac-api.example.com/collections",
            "method": "GET"
        },
        {
            "rel": "search",
            "type": "application/json",
            "href": "https://stac-api.example.com/collections",
            "method": "POST"
        }
    ]
```
## Example Catalog result for STAC API - Collection Search

```json
{
    "id": "Collection-search-results",
    "stac_version": "1.0.0",
    "description": "Collections matching query 'bbox=-10.415,36.066,3.779,44.213&limit=200&datetime=2017-05-05T00:00:00Z'",
    "license": "not-provided",
    "type": "Catalog",
    "links": [
        {
            "rel": "self",
            "href": "/collections?bbox=-10.415,36.066,3.779,44.213&limit=200&datetime=2017-05-05T00:00:00Z",
            "title": "Collections matching query 'foo'",
            "type": "application/json"
        },
        {
            "rel": "root",
            "href": "/",
            "title": "CMR-STAC Root",
            "type": "application/json"
        },
        {
            "rel": "next",
            "href": "/collections?bbox=-10.415,36.066,3.779,44.213&limit=200&datetime=2017-05-05T00:00:00Z&page=2"
        }
    ],
    "collections": [
        {
            "id": "ACEPOL_AircraftRemoteSensing_AirHARP_Data.v1",
            "stac_version": "1.0.0",
            "license": "not-provided",
            "title": "ACEPOL Airborne Hyper Angular Rainbow Polarimeter (AirHARP) Remotely Sensed Data Version 1",
            "type": "Collection",
            "description": "ACEPOL Airborne Hyper Angular Rainbow Polarimeter (AirHARP) Remotely Sensed Data (ACEPOL_AircraftRemoteSensing_AirHARP_Data) are remotely sensed measurements collected by the Airborne Hyper Angular Rainbow Polarimeter (AirHARP) onboard the ER-2 during ACEPOL. In order to improve our understanding of the effect of aerosols on climate and air quality, measurements of aerosol chemical composition, size distribution, height profile, and optical properties are of crucial importance. In terms of remotely sensed instrumentation, the most extensive set of aerosol properties can be obtained by combining passive multi-angle, multi-spectral measurements of intensity and polarization with active measurements performed by a High Spectral Resolution Lidar. During Fall 2017, the Aerosol Characterization from Polarimeter and Lidar (ACEPOL) campaign, jointly sponsored by NASA and the Netherlands Institute for Space Research (SRON), performed aerosol and cloud measurements over the United States from the NASA high altitude ER-2 aircraft. Six instruments were deployed on the aircraft. Four of these instruments were multi-angle polarimeters: the Airborne Hyper Angular Rainbow Polarimeter (AirHARP), the Airborne Multiangle SpectroPolarimetric Imager (AirMSPI), the Airborne Spectrometer for Planetary Exploration (SPEX Airborne) and the Research Scanning Polarimeter (RSP). The other two instruments were lidars: the High Spectral Resolution Lidar 2 (HSRL-2) and the Cloud Physics Lidar (CPL). The ACEPOL operation was based at NASA’s Armstrong Flight Research Center in Palmdale California, which enabled observations of a wide variety of scene types, including urban, desert, forest, coastal ocean and agricultural areas, with clear, cloudy, polluted and pristine atmospheric conditions. The primary goal of ACEPOL was to assess the capabilities of the different polarimeters for retrieval of aerosol and cloud microphysical and optical parameters, as well as their capabilities to derive aerosol layer height (near-UV polarimetry, O2 A-band). ACEPOL also focused on the development and evaluation of aerosol retrieval algorithms that combine data from both active (lidar) and passive (polarimeter) instruments. ACEPOL data are appropriate for algorithm development and testing, instrument intercomparison, and investigations of active and passive instrument data fusion, which is a valuable resource for remote sensing communities as they prepare for the next generation of spaceborne MAP and lidar missions.",
            "links": [
                {
                    "rel": "self",
                    "href": "/collections/ACEPOL_AircraftRemoteSensing_AirHARP_Data.v1",
                    "title": "Info about this collection",
                    "type": "application/json"
                },
                {
                    "rel": "root",
                    "href": "/",
                    "title": "Root catalog",
                    "type": "application/json"
                },
                {
                    "rel": "parent",
                    "href": "/",
                    "title": "Parent catalog",
                    "type": "application/json"
                },
                {
                    "rel": "items",
                    "href": "/collections/ACEPOL_AircraftRemoteSensing_AirHARP_Data.v1/items",
                    "title": "Granules in this collection",
                    "type": "application/json"
                },
                {
                    "rel": "about",
                    "href": "https://cmr.earthdata.nasa.gov/search/concepts/C1758588261-LARC_ASDC.html",
                    "title": "HTML metadata for collection",
                    "type": "text/html"
                },
                {
                    "rel": "via",
                    "href": "https://cmr.earthdata.nasa.gov/search/concepts/C1758588261-LARC_ASDC.json",
                    "title": "CMR JSON metadata for collection",
                    "type": "application/json"
                }
            ],
            "extent": {
                "spatial": {
                    "bbox": [
                        [
                            -130,
                            25,
                            -100,
                            45
                        ]
                    ]
                },
                "temporal": {
                    "interval": [
                        [
                            "2017-10-18T00:00:00.000Z",
                            "2020-11-20T23:59:59.999Z"
                        ]
                    ]
                }
            }
        },
        {
            "id": "ACTIVATE_TraceGas_AircraftInSitu_Falcon_Data.v1",
            "stac_version": "1.0.0",
            "license": "not-provided",
            "title": "ACTIVATE Falcon In Situ Trace Gas Data",
            "type": "Collection",
            "description": "ACTIVATE_TraceGas_AircraftInSitu_Falcon_Data is the trace gas data collected onboard the HU-25 Falcon aircraft via in-situ instrumentation during the ACTIVATE project. ACTIVATE is a 5-year NASA Earth-Venture Sub-Orbital field campaign, with a target completion of December 2023. Data collection is still ongoing. \r\n\r\nMarine boundary layer clouds play a critical role in Earth’s energy balance and water cycle. These clouds cover more than 45% of the ocean surface and exert a net cooling effect. The Aerosol Cloud meTeorology Interactions oVer the western Atlantic Experiment (ACTIVATE) project is a five-year project (January 2019-December 2023) that will provide important globally-relevant data about changes in marine boundary layer cloud systems, atmospheric aerosols and multiple feedbacks that warm or cool the climate. ACTIVATE studies the atmosphere over the western North Atlantic and samples its broad range of aerosol, cloud and meteorological conditions using two aircraft, the UC-12 King Air and HU-25 Falcon. The UC-12 King Air will primarily be used for remote sensing measurements while the HU-25 Falcon will contain a comprehensive instrument payload for detailed in-situ measurements of aerosol, cloud properties, and atmospheric state. A few trace gas measurements will also be onboard the HU-25 Falcon for the measurements of pollution traces, which will contribute to airmass classification analysis. A total of 150 coordinated flights over the western North Atlantic are planned through 6 deployments from 2020-2022. The ACTIVATE science observing strategy intensively targets the shallow cumulus cloud regime and aims to collect sufficient statistics over a broad range of aerosol and weather conditions which enables robust characterization of aerosol-cloud-meteorology interactions. This strategy is implemented by two nominal flight patterns: Statistical Survey and Process Study. The statistical survey pattern involves close coordination between the remote sensing and in-situ aircraft to conduct near coincident sampling at and below cloud base as well as above and within cloud top. The process study pattern involves extensive vertical profiling to characterize the target cloud and surrounding aerosol and meteorological conditions.",
            "links": [
                {
                    "rel": "self",
                    "href": "/collections/ACTIVATE_TraceGas_AircraftInSitu_Falcon_Data.v1",
                    "title": "Info about this collection",
                    "type": "application/json"
                },
                {
                    "rel": "root",
                    "href": "/",
                    "title": "Root catalog",
                    "type": "application/json"
                },
                {
                    "rel": "parent",
                    "href": "/",
                    "title": "Parent catalog",
                    "type": "application/json"
                },
                {
                    "rel": "items",
                    "href": "/collections/ACTIVATE_TraceGas_AircraftInSitu_Falcon_Data.v1/items",
                    "title": "Granules in this collection",
                    "type": "application/json"
                },
                {
                    "rel": "about",
                    "href": "https://cmr.earthdata.nasa.gov/search/concepts/C1994460919-LARC_ASDC.html",
                    "title": "HTML metadata for collection",
                    "type": "text/html"
                },
                {
                    "rel": "via",
                    "href": "https://cmr.earthdata.nasa.gov/search/concepts/C1994460919-LARC_ASDC.json",
                    "title": "CMR JSON metadata for collection",
                    "type": "application/json"
                }
            ],
            "extent": {
                "spatial": {
                    "bbox": [
                        [
                            -85,
                            25,
                            -58.5,
                            50
                        ]
                    ]
                },
                "temporal": {
                    "interval": [
                        [
                            "2017-02-14T00:00:00.000Z",
                            null
                        ]
                    ]
                }
            }
        }
    ]
}    
```

## Extensions

None.
