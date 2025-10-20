# **Developer's Guide: Maryland iMap Geospatial Feature Services**

This document provides technical details for an AI assistant on how to integrate Maryland's official geospatial boundary data (GeoJSON) into interactive data visualizations.

---

## **0. Guiding Principles for AI Implementation**

### **Primary Directive: Live API Integration**

The primary goal is to generate code that performs a **live, real-time HTTP GET request** to the Maryland iMap ArcGIS FeatureServer endpoint. The final generated code **must not** contain simulated, mocked, or locally stored GeoJSON files. The data must be fetched dynamically from the live service at runtime.

* **DO:** Write code that dynamically calls the `fetch` API (or a similar HTTP client) to retrieve GeoJSON data.
* **DO NOT:** Create a static variable or constant (e.g., `const countyBoundaries = {...}`) to represent the API's response.

### **Rationale**

Using the live API is essential to ensure that the geospatial data is the most current and authoritative version available from the state. This guide's purpose is to facilitate the creation of functional, data-driven applications that reflect real-world geographic boundaries.

### **Implementation Checklist**

When generating code, ensure the following criteria are met:

* [ ] The code makes a `fetch` (or equivalent) `GET` request to a specified FeatureServer URL.
* [ ] The request URL includes the `f=geojson` query parameter.
* [ ] The JSON response from the live API is parsed and processed at runtime.
* [ ] The code is prepared to join the fetched GeoJSON with a separate, non-geospatial dataset using a common key.
* [ ] There are no static variables or objects containing pre-downloaded GeoJSON data.

---

## **1. Core Concepts**

### **Base URL and Query Structure**

The endpoints in this guide are part of Maryland's ArcGIS REST Services. They follow a standard structure for querying vector data layers.

* **Structure**: `https://mdgeodata.md.gov/imap/rest/services/{Service}/{LayerName}/FeatureServer/{LayerID}/query`
* **Key Parameters**: To get usable map data, you must append two query parameters:
    * `where=1=1`: An SQL-like clause that selects all features in the layer.
    * `f=geojson`: Specifies the output format. This is the most important parameter.

### **Coordinate Reference System (CRS)**

This is a critical concept for using mapping data correctly.

* **API's Native CRS**: The iMap server stores data in **EPSG:3857** (Pseudo-Mercator).
* **GeoJSON Standard CRS**: The GeoJSON specification mandates the use of **EPSG:4326** (WGS 84), which represents standard latitude and longitude.
* **ACTION REQUIRED**: **None.** By using the `f=geojson` query parameter, you are instructing the API to perform the coordinate transformation for you. The resulting data will be in the correct EPSG:4326 format, ready to be used directly by mapping libraries like Leaflet, Mapbox, or Google Maps. **Do not attempt to re-project the coordinates.**

### **Authentication**

No API key or authentication is required to access these public data layers.

---

## **2. Geospatial Boundary Layers**

These endpoints provide the geometric shapes (polygons) needed to create choropleth maps (maps where areas are colored based on a data variable).

### **2.1. Maryland County Boundaries**

Provides the 24 major political subdivisions of Maryland (23 counties and Baltimore City).

* **URL**: `https://mdgeodata.md.gov/imap/rest/services/Boundaries/MD_PhysicalBoundaries/FeatureServer/0/query?outFields=*&where=1%3D1&f=geojson`
* **Method**: `GET`
* **Key Property for Joining**: The `properties` object of each GeoJSON feature contains a `COUNTY` field. This is the field to use when joining this layer with other datasets.

#### **Available Properties**

Each feature's `properties` object will contain the following fields:

* `OBJECTID`: Unique integer identifier for the feature.
* `COUNTY`: The full name of the county or Baltimore City.
* `DISTRICT`: The district number.
* `COUNTY_FIP`: The 3-digit FIPS code for the county.
* `COUNTYNUM`: An internal numeric identifier for the county.
* `CREATION_D`: The feature's creation date as a Unix timestamp.
* `LAST_UPDAT`: The feature's last update date as a Unix timestamp.
* `Shape__Area`: The area of the polygon in square meters (based on the native CRS).
* `Shape__Length`: The length of the polygon's perimeter in meters (based on the native CRS).

#### **County Names for Joining**

The `COUNTY` field will contain one of the following 24 string values:
`Allegany`, `Anne Arundel`, `Baltimore`, `Baltimore City`, `Calvert`, `Caroline`, `Carroll`, `Cecil`, `Charles`, `Dorchester`, `Frederick`, `Garrett`, `Harford`, `Howard`, `Kent`, `Montgomery`, `Prince George's`, `Queen Anne's`, `Somerset`, `St. Mary's`, `Talbot`, `Washington`, `Wicomico`, `Worcester`.

### **2.2. Maryland Municipal Boundaries**

Provides the boundaries for incorporated cities and towns within Maryland.

* **URL**: `https://mdgeodata.md.gov/imap/rest/services/Boundaries/MD_PoliticalBoundaries/FeatureServer/2/query?outFields=*&where=1%3D1&f=geojson`
* **Method**: `GET`
* **Key Properties for Joining**:
    * `MUN_NAME`: The name of the municipality (e.g., "ANNAPOLIS", "OCEAN CITY").
    * `JURSCODE`: A four-letter code often used in other government datasets.

#### **Available Properties**

Each feature's `properties` object will contain the following fields:

* `OBJECTID`: Unique integer identifier for the feature.
* `JURSCODE`: A four-letter jurisdictional code.
* `Shape__Area`: The area of the polygon in square meters.
* `Shape__Length`: The length of the polygon's perimeter in meters.
* `MUN_NAME`: The full name of the municipality in uppercase.

#### **Municipality Names for Joining**

The `MUN_NAME` field will contain one of the following string values:
`ABERDEEN`, `ACCIDENT`, `ANNAPOLIS`, `BALTIMORE CITY`, `BARCLAY`, `BARNESVILLE`, `BARTON`, `BEL AIR`, `BERLIN`, `BERWYN HEIGHTS`, `BETTERTON`, `BLADENSBURG`, `BOONSBORO`, `BOWIE`, `BRENTWOOD`, `BROOKEVILLE`, `BROOKVIEW`, `BRUNSWICK`, `BURKITTSVILLE`, `CAMBRIDGE`, `CAPITOL HEIGHTS`, `CECILTON`, `CENTREVILLE`, `CHARLESTOWN`, `CHESAPEAKE BEACH`, `CHESAPEAKE CITY`, `CHESTERTOWN`, `CHEVERLY`, `CHEVY CHASE SECTION 3`, `CHEVY CHASE SECTION 5`, `CHEVY CHASE TOWN`, `CHEVY CHASE VIEW`, `CHEVY CHASE VILLAGE`, `CHURCH CREEK`, `CHURCH HILL`, `CLEAR SPRING`, `COLLEGE PARK`, `COLMAR MANOR`, `COTTAGE CITY`, `CRISFIELD`, `CUMBERLAND`, `DEER PARK`, `DELMAR`, `DENTON`, `DISTRICT HEIGHTS`, `EAGLE HARBOR`, `EAST NEW MARKET`, `EASTON`, `EDMONSTON`, `ELDORADO`, `ELKTON`, `EMMITSBURG`, `FAIRMOUNT HEIGHTS`, `FEDERALSBURG`, `FOREST HEIGHTS`, `FREDERICK`, `FRIENDSVILLE`, `FROSTBURG`, `FRUITLAND`, `FUNKSTOWN`, `GAITHERSBURG`, `GALENA`, `GALESTOWN`, `GARRETT PARK`, `GLEN ECHO`, `GLENARDEN`, `GOLDSBORO`, `GRANTSVILLE`, `GREENBELT`, `GREENSBORO`, `HAGERSTOWN`, `HAMPSTEAD`, `HANCOCK`, `HAVRE DE GRACE`, `HEBRON`, `HENDERSON`, `HIGHLAND BEACH`, `HILLSBORO`, `HURLOCK`, `HYATTSVILLE`, `INDIAN HEAD`, `KEEDYSVILLE`, `KENSINGTON`, `KITZMILLER`, `LA PLATA`, `LANDOVER HILLS`, `LAUREL`, `LAYTONSVILLE`, `LEONARDTOWN`, `LOCH LYNN HEIGHTS`, `LONACONING`, `LUKE`, `MANCHESTER`, `MARDELA SPRINGS`, `MARTINS ADDITIONS`, `MARYDEL`, `MIDDletown`, `MIDLAND`, `MILLINGTON`, `MORNINGSIDE`, `MOUNT AIRY`, `MOUNT RAINIER`, `MOUNTAIN LAKE PARK`, `MYERSVILLE`, `NEW CARROLLTON`, `NEW MARKET`, `NEW WINDSOR`, `NORTH BEACH`, `NORTH BRENTWOOD`, `NORTH CHEVY CHASE`, `NORTH EAST`, `OAKLAND`, `OCEAN CITY`, `OXFORD`, `PERRYVILLE`, `PITTSVILLE`, `POCOMOKE CITY`, `POOLESVILLE`, `PORT DEPOSIT`, `PORT TOBACCO`, `PRESTON`, `PRINCESS ANNE`, `QUEEN ANNE`, `QUEENSTOWN`, `RIDGELY`, `RISING SUN`, `RIVERDALE PARK`, `ROCK HALL`, `ROCKVILLE`, `ROSEMONT`, `SALISBURY`, `SEAT PLEASANT`, `SECRETARY`, `SHARPSBURG`, `SHARPTOWN`, `SMITHSBURG`, `SNOW HILL`, `SOMERSET`, `ST. MICHAELS`, `SUDLERSVILLE`, `SYKESVILLE`, `TAKOMA PARK`, `TANEYTOWN`, `TEMPLEVILLE`, `THURMONT`, `TRAPPE`, `UNION BRIDGE`, `UNIVERSITY PARK`, `UPPER MARLBORO`, `VIENNA`, `WALKERSVILLE`, `WASHINGTON GROVE`, `WESTERNPORT`, `WESTMINSTER`, `WILLARDS`, `WILLIAMSPORT`, `WOODSBORO`.

### **2.3. Maryland Legislative Districts (2022)**

Provides the boundaries for Maryland's state legislative districts.

* **URL**: `https://mdgeodata.md.gov/imap/rest/services/Boundaries/MD_ElectionBoundaries/FeatureServer/1/query?outFields=*&where=1%3D1&f=geojson`
* **Method**: `GET`
* **Key Property for Joining**: The `DISTRICT` field.

#### **Available Properties**

Each feature's `properties` object will contain the following fields:

* `OBJECTID`: Unique integer identifier.
* `DISTRICT`: The legislative district code (e.g., '10', '27A').
* `Shape__Area`: The area of the polygon in square meters.
* `Shape__Length`: The perimeter of the polygon in meters.
* `State_Senator`: Full name of the State Senator.
* `State_Representative_1`: Full name of the first State Representative/Delegate.
* `State_Representative_2`: Full name of the second State Representative/Delegate.
* `State_Representative_3`: Full name of the third State Representative/Delegate.
* `State_Senator_Party`: Political party of the State Senator.
* `State_Representative_1_Party`: Political party of the first Representative.
* `State_Representative_2_Party`: Political party of the second Representative.
* `State_Representative_3_Party`: Political party of the third Representative.
* `StateSenator_MDManualURL`: Link to the senator's official Maryland Manual biography.
* `StateRepresentative1MDManualURL`: Link to the first representative's biography.
* `StateRepresentative2MDManualURL`: Link to the second representative's biography.
* `StateRepresentative3MDManualURL`: Link to the third representative's biography.
* `Latitude`: The latitude of the district's centroid.
* `Longitude`: The longitude of the district's centroid.

#### **District Codes for Joining**

The `DISTRICT` field is a **string** and will be one of the following values:
`10`, `11A`, `11B`, `12A`, `12B`, `13`, `14`, `15`, `16`, `17`, `18`, `19`, `01A`, `01B`, `01C`, `20`, `21`, `22`, `23`, `24`, `25`, `26`, `27A`, `27B`, `27C`, `28`, `29A`, `29B`, `29C`, `02A`, `02B`, `03`, `30A`, `30B`, `31`, `32`, `33A`, `33B`, `33C`, `34A`, `34B`, `35A`, `35B`, `36`, `37A`, `37B`, `38A`, `38B`, `38C`, `39`, `04`, `40`, `41`, `42A`, `42B`, `42C`, `43A`, `43B`, `44A`, `44B`, `45`, `46`, `47A`, `47B`, `05`, `06`, `07A`, `07B`, `08`, `09A`, `09B`.

### **2.4. Maryland 5-Digit ZIP Code Boundaries**

Provides generalized boundaries for USPS ZIP codes within Maryland.

* **URL**: `https://mdgeodata.md.gov/imap/rest/services/Boundaries/MD_PoliticalBoundaries/FeatureServer/4/query?outFields=*&where=1%3D1&f=geojson`
* **Method**: `GET`
* **Key Property for Joining**: The `ZIPCODE1` field.

#### **Available Properties**

Each feature's `properties` object will contain the following fields:

* `OBJECTID`: Unique integer identifier.
* `ZIPCODE1`: The 5-digit ZIP code as a string.
* `ZIPNAME`: The primary city/town name associated with the ZIP code.
* `Shape__Area`: The area of the polygon in square meters.
* `Shape__Length`: The perimeter of the polygon in meters.
* `EXISTING`: An internal data version identifier.

---

## **3. Implementation Example (JavaScript)**

This example demonstrates the full flow for building a data visualization: fetching a GeoJSON layer (counties), fetching a separate dataset, and joining them together into a single GeoJSON object ready for mapping.

```javascript
// 1. Define the endpoint for Maryland County boundaries
const countyBoundariesUrl = '[https://mdgeodata.md.gov/imap/rest/services/Boundaries/MD_PhysicalBoundaries/FeatureServer/0/query?outFields=*&where=1%3D1&f=geojson](https://mdgeodata.md.gov/imap/rest/services/Boundaries/MD_PhysicalBoundaries/FeatureServer/0/query?outFields=*&where=1%3D1&f=geojson)';

// 2. Example non-geospatial data (e.g., from a Socrata API or CSV)
// This data needs to be joined TO the GeoJSON shapes.
const countyPopulationData = [
    { countyName: 'Anne Arundel', population: 590336 },
    { countyName: 'Baltimore City', population: 576498 },
    { countyName: 'Prince George\'s', population: 955363 },
    // ... and so on for all 24 jurisdictions
];

/**
 * Fetches GeoJSON data and merges it with a separate dataset.
 * @returns {Promise<Object|null>} A GeoJSON FeatureCollection with merged data, or null on error.
 */
async function createChoroplethData() {
    try {
        // 3. Make the live API request to fetch the geographic boundaries
        const response = await fetch(countyBoundariesUrl);
        if (!response.ok) {
            throw new Error(`API responded with status: ${response.status}`);
        }
        const mdCountiesGeoJSON = await response.json();

        // 4. Join the external data with the GeoJSON
        // We iterate over each county feature from the API...
        mdCountiesGeoJSON.features.forEach(countyFeature => {
            // The key property for counties is 'COUNTY'
            const countyNameFromGeo = countyFeature.properties.COUNTY;

            // Find the matching county in our population data
            const populationRecord = countyPopulationData.find(
                data => data.countyName === countyNameFromGeo
            );

            // If a match is found, add the population to the feature's properties
            if (populationRecord) {
                countyFeature.properties.population = populationRecord.population;
            } else {
                countyFeature.properties.population = null; // No data found
            }
        });

        console.log("Successfully merged data into GeoJSON:", mdCountiesGeoJSON);
        return mdCountiesGeoJSON;

    } catch (error) {
        console.error("Failed to create geospatial data:", error);
        return null;
    }
}

// 5. Execute the function
createChoroplethData().then(mergedGeoJSON => {
    if (mergedGeoJSON) {
        // This 'mergedGeoJSON' object is now ready to be passed to a mapping
        // library like Leaflet to create a choropleth map.
        // For example: L.geoJSON(mergedGeoJSON, { style: ... }).addTo(map);
        console.log('This GeoJSON is ready for a mapping library!');
    }
});
```