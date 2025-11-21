
# Tempo 30 Zones Analysis

A comprehensive project for the automated identification and visualization of Tempo 30 (20 mph / 30 km/h) zones based on OpenStreetMap (OSM) data and PostGIS analysis. A Node.js proxy is included to bypass CORS restrictions between the browser and GeoServer.

## Core Components

This project combines several tools for importing, analyzing, and providing geospatial data:

| Component | Purpose | Technology |
| :--- | :--- | :--- |
| **Import** | Loads OSM data into the database. | `osm2pgsql` |
| **Storage & Analysis** | Stores road data and performs spatial analysis. | **PostGIS** |
| **Provision** | Provides analysis results as a WFS service. | **GeoServer** |
| **Proxy** | Bypasses CORS restrictions for WFS queries. | **Express.js** |
| **Visualization** | Frontend for displaying the identified zones. | **MapLibre** |

---

## Installation and Setup

### 1. Prerequisites

Install **Node.js**, **PostgreSQL/PostGIS**, and **`osm2pgsql`** on your system.

### 2. Execute Setup

Run the setup script to install Node.js dependencies and populate the PostGIS database with OSM data.

```bash
chmod +x setup.sh
./setup.sh
````

### 3. Start Proxy

Start the Express proxy server, which acts as an intermediary between your browser and GeoServer.

```bash
node proxy.js
```

### 4. Open Application

Open the map in your browser to view the analyzed Tempo 30 zones.

```bash
open map.html
```

> **Note:** Ensure that your GeoServer instance is running before opening the application.

---

## Tempo 30 Zones Logic

The identification of road segments eligible for Tempo 30 follows a two-stage logic: **Zones** and **Connecting Segments**.

### 1. Primary Zone Identification

Road segments are assigned Tempo 30 zones if one of the following conditions is met.

#### A. Automatic Assignment

* **Residential Roads:** All roads tagged with `highway=residential` are automatically classified as Tempo 30.

#### B. Conditional Assignment (at least one condition must be met)

Applies to the following major road classes:

* `highway=primary`
* `highway=secondary`
* `highway=tertiary`

| Condition             | Description                                                      | OSM Tags (Examples)                                                                                    |
| :-------------------- | :--------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------- |
| **Noise Protection**  | Residential buildings located less than 15 meters from the road. | `building=residential`, `building=apartments`, `building=house`, `building=terrace`                    |
| **Social Facilities** | Sensitive facilities located less than 50 meters from the road.  | `amenity=school`, `amenity=kindergarten`, `amenity=hospital`, `leisure=playground`, `highway=crossing` |

### 2. Zone Extension (Connecting Segments)

After primary zones are identified, the zones are extended based on the following rules:

* **Affected Roads:** Identified zones cover a road segment of 300 meters.
* **Gap Filling:** If the distance between two identified Tempo 30 zones is less than 500 meters, the intermediate segment is also classified as Tempo 30.

```
```
