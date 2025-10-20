# LLM-Context

This repository provides context files designed for use with large language models (such as Gemini 2.5 Pro) and interactive tools like Canvas. The included context files help LLMs use geocoding, mapping, and open data tools more effectively.


## Contents

- `imap_geocoder.md`: Context for enabling LLMs to use geocoding and mapping tools.
- `socrata_api.md`: Context for enabling LLMs to access and work with Socrata API data (e.g., Maryland open data).
- `imap_boundaries.md`: Context for enabling LLMs to use Maryland's official geospatial boundary data for live, interactive mapping.
## imap_boundaries.md

**Purpose:**  
Provides context for integrating Maryland's official geospatial boundary data (GeoJSON) into interactive data visualizations. This context enables LLMs to generate code that fetches live boundary data (counties, municipalities, legislative districts, ZIP codes) from Maryland iMap's ArcGIS FeatureServer endpoints, ensuring the most current and authoritative geographic information is used.

**Example Use-Case:**  
With Gemini 2.5 Pro and the Canvas tool active, you might use the following prompt:

```
Use the attached markdown to create a map of Maryland legislative districts. When I click on a district I want to learn the legislator's name.
```

---

## imap_geocoder.md

**Purpose:**  
Provides context that enables LLMs to use geocoding and mapping tools. This context helps the LLM interpret and process address data, allowing it to interact with tools that can geocode addresses and display them on interactive maps.

**Example Use-Case:**  
With Gemini 2.5 Pro and the Canvas tool active, you might use the following prompt:

```
List 15 real addresses around the state of Maryland, and then use the attached context to build an interactive map of their locations.
```

---

## socrata_api.md

**Purpose:**  
Provides context that enables LLMs to access and work with data from Socrata APIs. This context helps the LLM interpret API data structures and interact with tools for data analysis and visualization. While it is general-purpose for Socrata APIs, a common use-case is working with Maryland open data.

**Example Use-Case:**  
With Canvas, you might use the following prompt and data:

```
Build me a dashboard using this context and this data URL: https://opendata.maryland.gov/resource/9qyj-bhez.json
```

**Title:**

Annual practices (Cover Crops, Conservation Tillage Management, Soil Conservation and Water Quality Plans, Nutrient Management) for 2010 through 2024, by FIPs. WIP goals for progress year 2025.

**First row:**
```json
{
  "agriculture_practices": "Commodity Cover Crop",
  "unit": "Acres",
  "fips": "24001",
  "_2010_progress": "10",
  "_2011_progress": "128",
  "_2012_progress": "56",
  "_2013_progress": "42",
  "_2014_progress": "297",
  "_2015_progress": "69",
  "_2016_progress": "222",
  "_2017_progress": "139",
  "_2018_progress": "0",
  "_2019_progress": "0",
  "_2020_progress": "0",
  "_2021_progress": "0",
  "_2022_progress": "143",
  "_2023_progress": "92",
  "_2024_progress": "134",
  "_2025_goal": "206.00"
}
```

---

## Getting Started

1. Review the context files.
2. Use them as context in your preferred LLM interface (e.g., Gemini 2.5 Pro with Canvas).
3. For best results, follow the example use-cases above.

---

## License

See [LICENSE](LICENSE) for details.
