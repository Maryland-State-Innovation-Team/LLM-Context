# **Developer's Guide: Maryland iMap Geocoding Service**

This document provides the technical details required to interact with the Maryland iMap REST geocoding services. The information is derived from analyzing existing R and JavaScript implementations and is designed to be language-agnostic.

## **1\. Core Concepts**

### **Base URL**

All geocoding endpoints are part of the same ArcGIS REST Service.

* **Service URL**: https://mdgeodata.md.gov/imap/rest/services/GeocodeServices/MD\_MultiroleLocator/GeocodeServer/

### **Coordinate Reference System (CRS)**

This is the most critical concept for using the service correctly.

* **API CRS**: The service exclusively uses **EPSG:3857** (WGS 84 / Pseudo-Mercator) for all input and output coordinates.  
* **Mapping CRS**: Most standard web mapping libraries (Leaflet, Google Maps, Mapbox) use **EPSG:4326** (WGS 84), which represents standard latitude and longitude.  
* **ACTION REQUIRED**: You **must** transform coordinates between these two systems. Coordinates from the API must be converted *from* EPSG:3857 *to* EPSG:4326 before plotting them on a map. For reverse geocoding, you must transform standard latitude/longitude coordinates *from* EPSG:4326 *to* EPSG:3857 before sending them to the API.

### **Authentication**

No API key or authentication is required for this service.

## **2\. Endpoints and Methods**

### **2.1. Batch Geocoding (Multi-field)**

This endpoint is used to find coordinates for multiple addresses at once, with each address broken into its components.

* **Endpoint**: geocodeAddresses  
* **URL**: https://mdgeodata.md.gov/imap/rest/services/GeocodeServices/MD\_MultiroleLocator/GeocodeServer/geocodeAddresses  
* **Method**: POST  
* **Request Body**: The body should be application/x-www-form-urlencoded with a single key, addresses.  
* **Payload (addresses)**: The value for the addresses key is a URL-encoded JSON string. The structure of this JSON is:  
  {  
    "records": \[  
      {  
        "attributes": {  
          "OBJECTID": 1,  
          "Address": "501 E Pratt St",  
          "City": "Baltimore",  
          "Zip": "21202"  
        }  
      },  
      {  
        "attributes": {  
          "OBJECTID": 2,  
          "Address": "333 W Camden St",  
          "City": "Baltimore",  
          "Zip": "21201"  
        }  
      }  
    \]  
  }

  * OBJECTID is a unique, client-side integer used to map the results back to your original data.  
* **Response**: A JSON object containing a locations array. Each object in the array corresponds to a successfully geocoded address.  
  {  
    "spatialReference": { "wkid": 3857, "latestWkid": 3857 },  
    "locations": \[  
      {  
        "address": "501 E Pratt St, Baltimore, Maryland, 21202",  
        "location": { "x": \-8527878.29, "y": 4758235.13 },  
        "score": 100,  
        "attributes": {  
          "ResultID": 1,  
          "..."  
        }  
      }  
    \]  
  }

  * location.x and location.y are the coordinates in **EPSG:3857**.  
  * attributes.ResultID corresponds to the OBJECTID you sent in the request.

### **2.2. Batch Geocoding (Single Line)**

This is a variant of the above geocodeAddresses endpoint. The structure is identical, but the attributes object in the request takes a single field for the address.

* **Payload (addresses)**:  
  {  
    "records": \[  
      {  
        "attributes": {  
          "OBJECTID": 1,  
          "SingleLine": "501 E Pratt St, Baltimore, MD 21202"  
        }  
      }  
    \]  
  }

### **2.3. Reverse Geocoding**

Find the nearest address for a given coordinate pair.

* **Endpoint**: reverseGeocode  
* **URL**: https://mdgeodata.md.gov/imap/rest/services/GeocodeServices/MD\_MultiroleLocator/GeocodeServer/reverseGeocode  
* **Method**: GET  
* **Query Parameters**:  
  * location: A string containing the x and y coordinates in **EPSG:3857**, separated by a comma (e.g., \-8527878,4758235).  
  * distance (optional): The search radius around the location.  
  * f: json  
* **Example URL**: .../reverseGeocode?f=json\&location=-8527878,4758235  
* **Response**: A JSON object containing address and location objects.

### **2.4. Find Address Candidates**

Suggests possible address matches based on partial or complete address information.

* **Endpoint**: findAddressCandidates  
* **URL**: https://mdgeodata.md.gov/imap/rest/services/GeocodeServices/MD\_MultiroleLocator/GeocodeServer/findAddressCandidates  
* **Method**: GET  
* **Query Parameters**: Any combination of the following, plus f=json.  
  * SingleLine  
  * Address  
  * City  
  * Postal  
  * maxLocations (optional): The maximum number of candidates to return.  
* **Example URL**: .../findAddressCandidates?f=json\&SingleLine=501 E Pratt St\&maxLocations=5  
* **Response**: A JSON object containing a candidates array. Each candidate has an address, location (in **EPSG:3857**), and a score.

## **3\. Implementation Example (JavaScript)**

This example demonstrates the full flow for batch geocoding: creating the payload, making the POST request, and transforming the coordinates.

// A library like proj4.js is needed for coordinate transformation  
// import proj4 from 'proj4';

// 1\. Define the Coordinate Reference Systems  
const sourceCRS \= "EPSG:3857"; // CRS from the API  
const destCRS \= "EPSG:4326";  // Standard Lat/Lng for mapping

// 2\. Prepare the address data and format for the API  
const addresses \= \[  
    { id: 1, street: "501 E Pratt St", city: "Baltimore", zip: "21202" },  
    { id: 2, street: "333 W Camden St", city: "Baltimore", zip: "21201" }  
\];

const records \= addresses.map(addr \=\> ({  
    attributes: {  
        OBJECTID: addr.id,  
        Address: addr.street,  
        City: addr.city,  
        Zip: addr.zip  
    }  
}));

const payload \= { records: records };

// 3\. Define the API call function  
async function getCoordinates(apiPayload) {  
    const geocodeURL \= '\[https://mdgeodata.md.gov/imap/rest/services/GeocodeServices/MD\_MultiroleLocator/GeocodeServer/geocodeAddresses\](https://mdgeodata.md.gov/imap/rest/services/GeocodeServices/MD\_MultiroleLocator/GeocodeServer/geocodeAddresses)';

    try {  
        const response \= await fetch(geocodeURL, {  
            method: 'POST',  
            headers: { 'Content-Type': 'application/x-www-form-urlencoded' },  
            // Important: The JSON must be stringified AND then URL-encoded  
            body: \`f=json\&addresses=${encodeURIComponent(JSON.stringify(apiPayload))}\`  
        });

        if (\!response.ok) {  
            throw new Error(\`API responded with status: ${response.status}\`);  
        }

        const data \= await response.json();  
          
        // 4\. Process the response  
        const geocodedPoints \= data.locations.map(location \=\> {  
            const mercatorCoords \= \[location.location.x, location.location.y\];  
              
            // 5\. Transform coordinates  
            const \[lng, lat\] \= proj4(sourceCRS, destCRS, mercatorCoords);  
              
            return {  
                resultId: location.attributes.ResultID,  
                address: location.address,  
                score: location.score,  
                lat: lat,  
                lng: lng  
            };  
        });  
          
        return geocodedPoints;

    } catch (error) {  
        console.error("Geocoding failed:", error);  
        return \[\];  
    }  
}

// 6\. Execute the function  
getCoordinates(payload).then(results \=\> {  
    console.log("Geocoded and Transformed Results:", results);  
    // Now results can be plotted on a Leaflet map  
});  
