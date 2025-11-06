# Webeet-Refactoring-Initative
In this project Python scripts are used to populate a database with multiple layers of Points of Interest (POIs). Each layer represents a different category or type of POI, allowing for structured, scalable data population.  During the 8 weeks of internship at Webeet.io I worked on Webeet's refactoring initiative.

---
 
## 1. Tasks Step 1 – Layer-by-Layer Review of Existing Data Sources
The goal is to review each existing layer in the repository and evaluate whether its data can be migrated to the OpenStreetMap (OSM) API.

### 🎯 1.1 Objectives
- I went through each layer directory in the repository and reviewed the existing code to determine:
 - Which data source is currently used.
 - Which columns/fields are being extracted and stored in the database.
 - Check if the same data can be retrieved from the OpenStreetMap API.
 - Look up the equivalent data in the OpenStreetMap API and determined if OSM can provide the same data
- I developed a strategy for OSM use, especially defined the columns that should be used in each layer if OSM is used
 - I provided a list of standard columns with their details like type, label and the mapping instruction. Additional information may be added according to the layer's needs.
    
There are layers where OSM can be used:
- For these layers I identified if OSM can fully replace the old data source.
There are layers where only partial replacement is possible:
- For these layers I determined which columns still need to come from the original source.
- I proposed a matching strategy (mapping between OSM fields and old source columns).

For each layer, it is clearly documented:
- Whether it can be migrated to OSM.
- If yes, whether OSM fully or partially replaces the source.
- What additional columns are needed beyond OSM.
- A proposed field mapping exists for each OSM-enabled layer.

### 1.2 Results for step 1

#### 1.2.1 General OSM information
🌍 OpenStreetMap (OSM) is a free, collaborative map of the world that anyone can edit.
- It contains geospatial data: roads, railways, buildings, parks, rivers, amenities (cafés, hospitals, schools, etc.).
- Data is stored as:
  - Nodes (points, e.g. bus stops, cafés).
  - Ways (lines/polygons, e.g. roads, parks, building outlines).
  - Relations (groupings, e.g. bus routes, administrative boundaries).
- Everything is tagged with simple key=value pairs (e.g. amenity=cafe, highway=residential).

Most OSM objects will at least have:
- Identity (name, operator, brand)
- Location (addr:*)
- Availability (opening_hours, access)
- Contact (phone, website)
- Accessibility (wheelchair)

Data can be retrieved by using the following Python code:
```
import osmnx as ox
import geopandas as gpd
import pandas as pd

# populating a dataframe
tags = {"amenity": "university"}
university_gdf = ox.features_from_place("Berlin, Germany", tags)
university_gdf.info()
```
Example output:
```
<class 'geopandas.geodataframe.GeoDataFrame'>
MultiIndex: 117 entries, ('node', np.int64(278540049)) to ('way', np.int64(1211946568))
Data columns (total 87 columns):
 #   Column                     Non-Null Count  Dtype   
---  ------                     --------------  -----   
 0   geometry                   117 non-null    geometry
 1   addr:city                  62 non-null     object  
 2   addr:country               42 non-null     object  
 3   addr:housenumber           61 non-null     object  
 4   addr:postcode              61 non-null     object  
 5   addr:street                62 non-null     object  
 6   addr:suburb                44 non-null     object  
 7   amenity                    117 non-null    object  
 8   contact:phone              8 non-null      object  
 9   name                       109 non-null    object  
 10  operator                   59 non-null     object  
 ...
```
geometry contains information about longitude and latitude of an object and is provided in the format: POINT (13.35074 52.42464)

#### 1.2.2 Columns of OSM to be used in all layers

| Columns name | Description | OSM object name |  mapping instruction | example |
| ----------- | ----------- | ----------- | ----------- | ----------- |
| name | stores the primary, commonly used name of an object | name | 1:1 | Volkspark Friedrichshain, Café Einstein | 
| operator | organization that runs it | operator | 1:1 | S-Bahn Berlin GmbH | 
| brand | chain brand | brand | 1:1 | Starbucks | 
| longitude | east–west position on Earth | geometry | retrieve from first number in brackets | 13.34455 | 
| latitude | specifies how far north or south a point is from the Equator (0°) | geometry | retrieve from second number in brackets | 52.4988 | 
| country | country where object is located | addr:country | rename | DE | 
| city | city where object is located | addr:city | rename | Berlin | 
| street | street where object is located | addr:street | rename | Friedrichstraße | 
| housenumber | housenumber of street where object is located | addr:housenumber | rename | 180 | 
| postcode | postcode of city and street where object is located | addr:postcode | rename | 10117 | 
| neighborhood | neighborhood of object | addr:suburb | rename | Mitte | 
| phone | contact: phone number | contact:phone or phone | rename or 1:1 | +493045040 | 
| email | contact: email | contact:email or email | rename or 1:1 | |
| website | contact: website | contact:website or website| rename or 1:1 | | 
| wheelchair |accessibility for wheelchair | wheelchair | 1:1 | yes/no/limited |
| wheelchair_toilets |toilets accessible for wheelchairs | toilets:wheelchair | 1:1 | yes/no |
| opening_hours |opening hours of object in a standard machine-readable syntax| opening_hours | 1:1 | Mo-Su 08:00-20:00 |

 **in addition 🟢 district and district_id 🟢 shouild be added to all layers**

#### 1.2.3 Layer-by-Layer Review of Existing Data Sources
All columns shown in the table above should be added to all layers, independently if they are available in the legacy database or not.
In addition to these OSM or non-OSM columns have to be added according to legacy database.



---

##### ✅ Main Table (Compact Overview)

| Layer | Source | OSM Replace? | Tag in OSM | Scope |
|-------|--------|--------------|------------|--------|
| banks | OSM | yes | amenity=bank | full |
| boundary / mileuschutz | gdi.berlin.de | no | – | – |
| colleges | edurank.org | yes | amenity=university | partial |
| crime_statistics | Berlin Police Dept. | no | – | – |
| dental_offices | OSM / Berlin Open Data | partial | healthcare=dentist | – |
| gym | OSM | yes | leisure=fitness_centre | full |
| hospitals | deutsches-krankenhaus-verzeichnis.de | yes | amenity=hospital | partial |
| pools | baederleben.de | yes | leisure=swimming_pool | full |
| population_statistics | statistik-berlin-brandenburg.de | no | – | – |
| post_offices | Deutsche Post | yes | amenity=post_office | partial |
| public_transport | vbb.de, gtfs.de, bvg.de, OSM | yes | public_transport=stop | full |
| real_estate_statistics | IBB Wohnungsmarktbericht | no | – | – |
| recreational_zone | Berlin Open Data | yes | leisure=park / landuse=recreation_ground | full |
| s-bahn | Berlin Open Data | partial | railway=station | partial |
| schools | (unknown) | yes | amenity=school | partial |
| tram_bus | BVG Public API | yes | railway=tram_stop / highway=bus_stop | full |
| venues | Overpass API | yes | amenity=cafe/restaurant/bar/etc. | full |
| vet_clinics | OSM | yes | amenity=veterinary | full |
| short_term_listing | Airbnb | yes | tourism=apartment/hotel/etc. | partial |
| ubahn | Wikidata via SPARQL | yes | railway=station | full |

---

##### 📂 Details by Layer

<details>
<summary><strong>🏥 Hospitals – Details</strong></summary>

**✔ OSM Tags Used:**  
- `amenity=hospital`, optional: `healthcare=hospital`  
- Scope: **partial replacement**

**✅ Extra OSM Columns (already exist in many cases):**
- `beds=*` (incomplete data)
- `emergency=yes/no/designated`

**➕ Proposed New Columns to Add in OSM:**
- `healthcare:speciality=*` (cardiology, oncology, psychiatry, geriatrics, etc.)  
- `parking=*` or linking `amenity=parking`  
- `ambulance=*` or nearby `emergency=ambulance_station`

**❌ Missing from OSM (only in legacy source):**
- `cases` (annual patient cases)

**💡 Notes / Matching Strategy:**
- Combine OSM hospital nodes with legacy list using name + address + coordinates  
- Missing specialties can be crowd-sourced or added manually

</details>

---

<details>
<summary><strong>🎓 Colleges / Universities</strong></summary>

**✅ General Information**  
| Field | Value |
|-------|-------|
| Data Source | edurank.org |
| OSM Replaceable | Yes |
| Relevant OSM Tags | amenity=university |
| Scope | Partial |

**🧩 Additional OSM Attributes Used**
| Attribute | Description |
|-----------|-------------|
| university_id | Legacy ID |
| rank_in_berlin_brandenburg | Ranking |
| rank_in_germany | National ranking |
| enrollment | Number of students |
| founded | Year founded |

**➕ Suggested New Columns**
| Column | Description |
|--------|-------------|
| – | – |

**⚠️ Missing or Incomplete Data**
| Issue | Impact |
|-------|--------|
| Some ranks or enrollment info | Not in OSM |

**🛠 Matching / Transformation Notes**
- Use name + address + coordinates for mapping legacy data to OSM nodes
</details>

---

<details>
<summary><strong>🦷 Dental Offices</strong></summary>

**✅ General Information**  
| Field | Value |
|-------|-------|
| Data Source | OSM / Berlin Open Data Portal |
| OSM Replaceable | Partial |
| Relevant OSM Tags | healthcare=dentist |
| Scope | – |

**🧩 Additional OSM Attributes Used**
| Attribute | Description |
|-----------|-------------|
| name | Clinic name |
| address | Street and number |
| opening_hours | Working hours |

**➕ Suggested New Columns**
| Column | Description |
|--------|-------------|
| dentist_id | Legacy database ID |
| specialty | e.g., orthodontics, pediatric |

**⚠️ Missing or Incomplete Data**
| Issue | Impact |
|-------|--------|
| Some specialties may be missing | Needs enrichment from legacy DB |

**🛠 Matching / Transformation Notes**
- Match by coordinates + name; enrich missing data from Berlin Open Data
</details>

---

<details>
<summary><strong>🏊 Swimming Pools / Pools</strong></summary>

**✅ General Information**  
| Field | Value |
|-------|-------|
| Data Source | baederleben.de |
| OSM Replaceable | Yes |
| Relevant OSM Tags | leisure=swimming_pool |
| Scope | Full |

**🧩 Additional OSM Attributes Used**
| Attribute | Description |
|-----------|-------------|
| pool_type | indoor / outdoor |
| access | public / private / members |
| depth | Pool depth |
| length | Pool length |
| heated | Yes / No |
| fee | Entrance fee |

**➕ Suggested New Columns**
| Column | Description |
|--------|-------------|
| pool_id | Legacy DB ID |
| open_all_year | Boolean for seasonality |

**⚠️ Missing or Incomplete Data**
| Issue | Impact |
|-------|--------|
| Some depth or heating info | May require manual addition |

**🛠 Matching / Transformation Notes**
- Merge OSM data with legacy ID where coordinates match
</details>

---

<details>
<summary><strong>📮 Post Offices</strong></summary>

**✅ General Information**  
| Field | Value |
|-------|-------|
| Data Source | Deutsche Post |
| OSM Replaceable | Yes |
| Relevant OSM Tags | amenity=post_office |
| Scope | Partial |

**🧩 Additional OSM Attributes Used**
| Attribute | Description |
|-----------|-------------|
| operating_hours | Opening times |
| locationName | Name of branch |
| primaryKeyZipRegion | ZIP code info |

**➕ Suggested New Columns**
| Column | Description |
|--------|-------------|
| pfServiceType | Service type: copy, print, parcel_pickup, etc. |
| additionalInfo | Extra notes |

**⚠️ Missing or Incomplete Data**
| Issue | Impact |
|-------|--------|
| Some service type info | Not in OSM, legacy DB needed |

**🛠 Matching / Transformation Notes**
- Use address + coordinates to link legacy post office IDs with OSM nodes
</details>

---

<details>
<summary><strong>🏫 Schools</strong></summary>

**✅ General Information**  
| Field | Value |
|-------|-------|
| Data Source | Unknown |
| OSM Replaceable | Yes |
| Relevant OSM Tags | amenity=school, amenity=college |
| Scope | Partial |

**🧩 Additional OSM Attributes Used**
| Attribute | Description |
|-----------|-------------|
| isced:level | Education level (1=primary, 2=lower secondary, etc.) |
| school_type_de | Type in German |
| school_category_de / en | School category |
| students_total / students_f / students_m | Student counts |
| teachers_total / teachers_f / teachers_m | Teacher counts |

**➕ Suggested New Columns**
| Column | Description |
|--------|-------------|
| startchancen_flag | Indicator from legacy DB |
| ownership_en | School ownership info |

**⚠️ Missing or Incomplete Data**
| Issue | Impact |
|-------|--------|
| Some ISCED levels may be missing | Needs enrichment from legacy DB |

**🛠 Matching / Transformation Notes**
- Use coordinates + name to link with legacy DB
- Gender and ownership mapping may require manual checking
</details>

---

<details>
<summary><strong>🚆 S-Bahn</strong></summary>

**✅ General Information**  
| Field | Value |
|-------|-------|
| Data Source | Berlin Open Data Portal |
| OSM Replaceable | Partial |
| Relevant OSM Tags | railway=station, stop_position, public_transport=stop_position |
| Scope | Partial |

**🧩 Additional OSM Attributes Used**
| Attribute | Description |
|-----------|-------------|
| name | Station name |
| lines | Associated S-Bahn lines |
| platform | Platform info |

**➕ Suggested New Columns**
| Column | Description |
|--------|-------------|
| station_id | Legacy DB ID |
| neighborhood | Locality mapping |

**⚠️ Missing or Incomplete Data**
| Issue | Impact |
|-------|--------|
| Some platforms or line references may be incomplete | May require manual enrichment |

**🛠 Matching / Transformation Notes**
- Use coordinates + station name to match legacy data
- Some line information may need combination of `route=*` + `ref=*` tags
</details>

---

<details>
<summary><strong>🚇 U-Bahn</strong></summary>

**✅ General Information**  
| Field | Value |
|-------|-------|
| Data Source | Wikidata via SPARQL |
| OSM Replaceable | Yes |
| Relevant OSM Tags | railway=station, station=subway, platform, public_transport=platform |
| Scope | Full |

**🧩 Additional OSM Attributes Used**
| Attribute | Description |
|-----------|-------------|
| line | Line reference (e.g., U1, U2) |
| platform | Platform nodes |
| level | Floor / underground level info |

**➕ Suggested New Columns**
| Column | Description |
|--------|-------------|
| station_id | Legacy DB ID |
| neighborhood | Area or quarter |

**⚠️ Missing or Incomplete Data**
| Issue | Impact |
|-------|--------|
| Complex mapping of multiple tags required | Need careful QA to combine lines/platforms |

**🛠 Matching / Transformation Notes**
- Map subway stations using `station=subway_entrance` + `platform` nodes  
- Combine with legacy DB for full attributes
</details>

---

<details>
<summary><strong>🚋 Tram / Bus</strong></summary>

**✅ General Information**  
| Field | Value |
|-------|-------|
| Data Source | BVG Public API |
| OSM Replaceable | Yes |
| Relevant OSM Tags | railway=tram_stop, highway=bus_stop |
| Scope | Full |

**🧩 Additional OSM Attributes Used**
| Attribute | Description |
|-----------|-------------|
| stop_id | Legacy stop ID |
| name | Stop name |
| lines | Tram / bus line numbers |

**➕ Suggested New Columns**
| Column | Description |
|--------|-------------|
| accessibility | Barrier-free access info |
| shelter | Shelter availability |
| coordinates | Lat/Lon from legacy DB |

**⚠️ Missing or Incomplete Data**
| Issue | Impact |
|-------|--------|
| Some stops missing line assignments | May need cross-check with GTFS |

**🛠 Matching / Transformation Notes**
- Match stops by coordinates and name  
- GTFS integration may improve line assignment completeness
</details>

---


<details>
<summary><strong>☕ Venues (Cafes, Restaurants, Bars, Pubs)</strong></summary>

**✅ General Information**  
| Field | Value |
|-------|-------|
| Data Source | Overpass API |
| OSM Replaceable | Yes |
| Relevant OSM Tags | amenity=cafe, restaurant, bar, pub, beer_garden |
| Scope | Full |

**🧩 Additional OSM Attributes Used**
| Attribute | Description |
|-----------|-------------|
| cuisine | Type of cuisine (e.g., coffee_shop, italian) |
| outdoor_seating | yes/no |
| smoking | yes/no/separated |
| wifi | yes/no/free |
| diet:vegetarian/vegan/halal/gluten_free | Dietary options |
| delivery | yes/no |
| drink:cocktail/beer/wine | Available drinks |

**➕ Suggested New Columns**
| Column | Description |
|--------|-------------|
| venue_id | Legacy DB ID |
| capacity | Number of seats (if available) |
| opening_hours | Legacy operating hours |

**⚠️ Missing or Incomplete Data**
| Issue | Impact |
|-------|--------|
| Some dietary or drink info may be missing | Needs enrichment from legacy DB |

**🛠 Matching / Transformation Notes**
- Match by name + coordinates  
- Can enhance OSM nodes with legacy ID for reporting
</details>

---

<details>
<summary><strong>🏠 Short-term Listings</strong></summary>

**✅ General Information**  
| Field | Value |
|-------|-------|
| Data Source | Airbnb |
| OSM Replaceable | Yes |
| Relevant OSM Tags | tourism=apartment, guest_house, hotel, hostel, chalet, camp_site, motel |
| Scope | Partial |

**🧩 Additional OSM Attributes Used**
| Attribute | Description |
|-----------|-------------|
| beds | Number of beds |
| accommodates | Number of guests |
| room_type | Type of room |
| amenities | e.g., wifi, kitchen |
| delivery / service info | Optional |

**➕ Suggested New Columns**
| Column | Description |
|--------|-------------|
| host_ID | Unique host identifier |
| property_type | Apartment, hotel, hostel, etc. |
| short_term_ID | Legacy DB ID |
| minimum_nights / maximum_nights | Booking info |
| reviews / review_scores | Ratings and feedback |
| price | Price per night |

**⚠️ Missing or Incomplete Data**
| Issue | Impact |
|-------|--------|
| Some host info or ratings missing | Needs enrichment from legacy DB |

**🛠 Matching / Transformation Notes**
- Match via coordinates + name  
- OSM coverage may be partial; legacy DB required for full listings
</details>

---

<details>
<summary><strong>🌳 Recreational Zones</strong></summary>

**✅ General Information**  
| Field | Value |
|-------|-------|
| Data Source | Berlin Open Data – Parks & Playgrounds |
| OSM Replaceable | Yes |
| Relevant OSM Tags | leisure=park, garden, playground; landuse=recreation_ground; boundary=national_park |
| Scope | Full |

**🧩 Additional OSM Attributes Used**
| Attribute | Description |
|-----------|-------------|
| green_area_type | Surface type: grass, sand, rubber, etc. |
| access | Public / Private / Restricted |
| playground equipment | swing, slide, climbingframe |
| age | Recommended age for playgrounds |
| area_sqm | Area in square meters |

**➕ Suggested New Columns**
| Column | Description |
|--------|-------------|
| planning_area_name | From legacy DB |
| created_at / updated_at | Metadata |

**⚠️ Missing or Incomplete Data**
| Issue | Impact |
|-------|--------|
| Some playground or garden details missing | Needs enrichment from legacy DB |

**🛠 Matching / Transformation Notes**
- Merge OSM parks/playgrounds with legacy DB using coordinates and name  
- Smaller gardens may need special handling
</details>

---

<details>
<summary><strong>🏦 Banks</strong></summary>

**✅ General Information**  
| Field | Value |
|-------|-------|
| Data Source | OSM |
| OSM Replaceable | Yes |
| Relevant OSM Tags | amenity=bank |
| Scope | Full |

**🧩 Additional OSM Attributes Used**
| Attribute | Description |
|-----------|-------------|
| name | Bank name |
| atm | yes/no |
| operator | Bank operator |

**➕ Suggested New Columns**
| Column | Description |
|--------|-------------|
| bank_id | Legacy DB ID |
| branch_type | Branch or ATM |

**⚠️ Missing or Incomplete Data**
| Issue | Impact |
|-------|--------|
| Some branches may not be mapped in OSM | Use legacy DB for completeness |

**🛠 Matching / Transformation Notes**
- Match by coordinates + name for merging with legacy DB
</details>

---

<details>
<summary><strong>💪 Gyms / Fitness Centers</strong></summary>

**✅ General Information**  
| Field | Value |
|-------|-------|
| Data Source | OSM |
| OSM Replaceable | Yes |
| Relevant OSM Tags | leisure=fitness_centre |
| Scope | Full |

**🧩 Additional OSM Attributes Used**
| Attribute | Description |
|-----------|-------------|
| name | Gym name |
| opening_hours | Working hours |
| operator | Operator name |

**➕ Suggested New Columns**
| Column | Description |
|--------|-------------|
| gym_id | Legacy DB ID |
| membership_type | Type: public/private |

**⚠️ Missing or Incomplete Data**
| Issue | Impact |
|-------|--------|
| Some gyms may lack operator info | Cross-check with legacy DB |

**🛠 Matching / Transformation Notes**
- Merge OSM gyms with legacy DB using coordinates + name
</details>

---

<details>
<summary><strong>👥 Population Statistics</strong></summary>

**✅ General Information**  
| Field | Value |
|-------|-------|
| Data Source | statistik-berlin-brandenburg.de |
| OSM Replaceable | No |
| Relevant OSM Tags | – |
| Scope | – |

**🧩 Additional Attributes Used**
| Attribute | Description |
|-----------|-------------|
| population_total | Total population per area |
| demographics | Age, gender distribution |
| neighborhood | Locality info |

**➕ Suggested New Columns**
| Column | Description |
|--------|-------------|
| population_id | Legacy DB ID |
| migration_data | Optional: inflow/outflow |

**⚠️ Missing or Incomplete Data**
| Issue | Impact |
|-------|--------|
| Some neighborhoods may be missing detailed stats | Needs enrichment from official statistics |

**🛠 Matching / Transformation Notes**
- Use area codes, names, or geo-polygons to merge with other datasets
</details>

---

<details>
<summary><strong>🏘 Real Estate Statistics</strong></summary>

**✅ General Information**  
| Field | Value |
|-------|-------|
| Data Source | IBB Wohnungsmarktbericht 2024, FIS-Broker Berlin, Immobilienmarktberichte |
| OSM Replaceable | No |
| Relevant OSM Tags | – |
| Scope | – |

**🧩 Additional Attributes Used**
| Attribute | Description |
|-----------|-------------|
| ID | Legacy DB ID |
| property_type | Apartment, house, commercial |
| locality | Neighborhood or area |

**➕ Suggested New Columns**
| Column | Description |
|--------|-------------|
| price | Rent or sale price |
| size_sqm | Square meters |
| rooms | Number of rooms |

**⚠️ Missing or Incomplete Data**
| Issue | Impact |
|-------|--------|
| Some locality mappings may not match OSM neighborhoods | Needs manual verification |

**🛠 Matching / Transformation Notes**
- Check `locality` field to map to OSM neighborhood for reporting
</details>

---

<details>
<summary><strong>🚨 Crime Statistics</strong></summary>

**✅ General Information**  
| Field | Value |
|-------|-------|
| Data Source | Berlin Police Department |
| OSM Replaceable | No |
| Relevant OSM Tags | – |
| Scope | – |

**🧩 Additional Attributes Used**
| Attribute | Description |
|-----------|-------------|
| crime_type | Type of crime |
| incident_count | Number of incidents |
| area | Neighborhood / locality |

**➕ Suggested New Columns**
| Column | Description |
|--------|-------------|
| police_id | Legacy DB ID |
| reporting_period | Date or year of statistics |

**⚠️ Missing or Incomplete Data**
| Issue | Impact |
|-------|--------|
| Some areas may not have detailed data | Only legacy DB can provide full coverage |

**🛠 Matching / Transformation Notes**
- Match by neighborhood name or geographic boundaries to combine with other datasets
</details>

---


## 2. Tasks Step 2: Work on Hospital Layer to use OSM Data

### 2.1 🎯 Objectives
I refactored the existing hospital layer script so that it pulls data from the OpenStreetMap API instead of (or in addition to) the current source (deutsches-krankenhaus-verzeichnis.de).
Additionally I evaluated whether OSM can fully or partially replace the current hospital dataset.

#### 2.1.1 Source considerations
- Current sources should be replaced
#### 2.1.2 OSM data should be used
content of old data columns should be replaced by OSM data if possible, investigations whether old columns can be reused and joined
content should be easily retrieved
required updates should be easily implemented in future releases
it should be challenged whether "old" content is really required in the light of future updates and extension to other cities
### 2.2 OSM data
##### 2.2.1 vailable tags in OSM:
amenity: hospital
amenity: clinic
healthcare: hospital, clinic
#### 2.2.1 Amenity: hospital
58 entries, 89 columns
contains not only hopsitals, but also clinics, e.g. Gemeinschaftspraxis Michael Balschin, Vadim Rubinstein, Irina Rabinovich
#### 2.2.2 Amenity: clinic
180 entries, 88 columns (no bed column)
contains also hopsitals, e.g. Charité Comprehensive Cancer Center
### 2.3 Which data should go in this layer and who decided what is needed?
#### 2.3.1 Row deletion
my proposal: delete Ärtze, Ärztehaus, Versorgungszentrum, etc. --> but for future use also in other cities this might not be a clean option
after deletion 22 entries in clincis and 58 in hospitals
total 80 entries

### 2.4 Relevant columns:
"name", "geometry", "operator", "brand", "addr:city", "addr:street", "addr:housenumber", "addr:postcode", "addr:suburb", "phone","email", "website", "wheelchair", "toilets:wheelchair", "beds", "emergency", "healthcare:speciality","opening_hours","source"

### 2.5 Data derivation
longitude, latitude using geometry
district, district_id, neighborhood using reverse geolocation
### 2.6 Proposed dataset: 80 columns, 24 rows
### 2.7 Outstanding discussion about columns with many missings, e.g. opening_hours, source, brand
source could be hard-coded with OSM
### 2.8 Old data
75 rows, 12 columns
contains also information grouped as clinic data, e.g. Tagesklinik
3 additional columns: bed, distance (to a reference point), cases
is there any option to get updated data if required?
what does distance mean? to Berlin Mitte?
### 2.9 Join old and new data
#### 2.9.1 Joined by name: cumbersome a typos and different spelling of same hospital name
rename of old hospital name to match OSM data
105 rows:
both 50
new 30
old 25
Do we really want to jump through hoops to get the data, just for bed, distance and cases?
proposal: use OSM data and don't try to add and clean data to get similar result as in old dataset
data not static
easy update
extension to other cities others than Berlin
## 3. Decision
OSM data is used without any modification/deletion of rows
OSM data is used with tag information (amenity, healthcare, hospital or clinic)
old data, especially beds, distance and cases will not be added to OSM dataset
OSM dataset will be name hospital_osm

## 4. Python Script
<details>
<summary><strong>Install required libraries and packages, istall packages</strong></summary>

```
# 1. Install required libraries and packages
pip install osmnx geopandas pandas

## 1.1 install packages
import osmnx as ox
import geopandas as gpd
import pandas as pd
import requests
import time
import logging
from sqlalchemy import create_engine, text
import psycopg2
import warnings

warnings.filterwarnings("ignore")

```
</details>

---

<details>
<summary><strong>Display all rows and columns in pandas DataFrames, disable disk caching</strong></summary>

```
# Display all rows and columns in pandas DataFrames
pd.set_option("display.max_rows", None)
pd.set_option("display.max_columns", None)
pd.set_option("display.max_colwidth", None)

# disable disk caching
ox.settings.use_cache = False

```
</details>

---

<details>
<summary><strong>get hospital data from OSM, ensure geometry type is Point for lat/lon extraction, set Country to Germany </strong></summary>

```

# 2. get hospital data from OSM
tags = {
    "healthcare": ["hospital","clinic"],  # amenity tagging
    "amenity": ["hospital","clinic"]  # healthcare tagging
 }
hospital_gdf = ox.features_from_place("Berlin, Germany", tags)

# Ensure geometry type is Point for lat/lon extraction

hospital_gdf = hospital_gdf.to_crs(epsg=4326)

hospital_gdf['geometry'] = hospital_gdf['geometry'].apply(lambda geom: geom if geom.geom_type == 'Point' else geom.representative_point())

## 2.1 extract longitude and latitude
hospital_gdf["latitude"] = hospital_gdf.geometry.y
hospital_gdf["longitude"] = hospital_gdf.geometry.x
hospital_gdf.head(3)

## 2.2 set Country to Germany
# Update only where city is Berlin AND country is null/NaN/empty
hospital_gdf.loc[(hospital_gdf["addr:city"] == "Berlin") & (hospital_gdf["addr:country"].isna()), "addr:country"] = "DE"
```
</details>

---

<details>
<summary><strong>select required columns</strong></summary>

```

# 3. select required columns

# Select the columns  and rename accordingly

selected_columns = [
         "name", "operator", "brand", "country", "addr:city", "addr:street", "addr:housenumber", "addr:postcode", "addr:suburb", 
         "phone","email", "website", "wheelchair", "toilets:wheelchair", "beds", "emergency", "healthcare:speciality",
         "opening_hours","latitude", "longitude","geometry", "source"]

hospital_df = hospital_gdf[selected_columns]
hospital_df.head(3)

```
</details>

---
<details>
<summary><strong>Sanity checks </strong></summary>

```

# 4. sanitary checks
## 4.1 name missing
nan_names = hospital_df[hospital_df["name"].isna()]

print(nan_names)
print(len(nan_names), "rows with NaN name")

## 4.2 top 50 hospital names
print("\nTop 10 hospitals:")
print(hospital_df["name"].value_counts().head(10))

## 4.3 difference of healtcare and amenity
# Find rows where the two columns are different
 
diff = hospital_df[hospital_df["amenity"] != hospital_df["healthcare"]][["name", "amenity", "healthcare"]]
print(diff)

# Print total count
print("\nTotal mismatches:", len(diff))

```
</details>

---
<details>
<summary><strong>Rename Columns </strong></summary>

```

## 5. rename columns
rename_map = {
    "addr:street": "street",
    "addr:housenumber": "housenumber",
    "addr:postcode": "postcode",
    "addr:city": "city",
    "addr:country": "country",
    "addr:suburb": "neighborhood",
    "healthcare:speciality": "speciality",
    "toilets:wheelchair": "toilets_wheelchair",
    "amenity": "amenity_tag",
    "healthcare": "healthcare_tag"}

# Rename the columns
hospital_df = hospital_df.rename(columns=rename_map)
hospital_df.head(3)

```
</details>

---
<details>
<summary><strong>retrieve district and district_id and neighborhoods </strong></summary>

```
# 6. retrieve district and district_id and neighborhood for missings
from geopy.geocoders import Nominatim
from time import sleep

def fetch_location_info(df, lat_col="latitude", lon_col="longitude", level="district", user_agent="berlin-venues-scraper/1.0", delay=1):
    """
    Fetch district or neighborhood from Nominatim for a DataFrame with lat/lon columns.
    
    Parameters:
        df: pd.DataFrame with latitude and longitude columns
        lat_col, lon_col: names of the lat/lon columns
        level: "district" or "neighborhood"
        user_agent: User-Agent string for Nominatim
        delay: delay in seconds between requests
    
    Returns:
        pd.Series with district or neighborhood names
    """
    def get_info(lat, lon):
        url = "https://nominatim.openstreetmap.org/reverse"
        params = {"lat": lat, "lon": lon, "format": "json", "addressdetails": 1}
        headers = {"User-Agent": user_agent}
        try:
            r = requests.get(url, params=params, headers=headers, timeout=10)
            r.raise_for_status()
            data = r.json()
            address = data.get("address", {})

            if level == "district":
                # Only official Bezirke
                return (
                    address.get("city_district")
                    or address.get("borough")
                    or address.get("county")
                    or address.get("state_district")
                )
            elif level == "neighborhood":
                # Include suburb / neighbourhood / city_district
                return (
                    address.get("suburb")
                    or address.get("city_district")
                    or address.get("borough")
                    or address.get("neighbourhood")
                )
            else:
                return None
        except requests.exceptions.RequestException as e:
            logging.warning(f"Error fetching {level} for ({lat}, {lon}): {e}")
            return None

    # Apply with throttling
    results = []
    for i, row in df.iterrows():
        lat, lon = row[lat_col], row[lon_col]
        if pd.notna(lat) and pd.notna(lon):
            results.append(get_info(lat, lon))
            time.sleep(delay)
        else:
            results.append(None)
    return pd.Series(results, index=df.index)

# Usage examples:
hospital_df["district"] = fetch_location_info(hospital_df, level="district")
hospital_df["neighborhood_new"] = fetch_location_info(hospital_df, level="neighborhood")

# District mapping (official codes as strings)
district_mapping = {
    'Mitte': '11001001',
    'Friedrichshain-Kreuzberg': '11002002',
    'Pankow': '11003003',
    'Charlottenburg-Wilmersdorf': '11004004',
    'Spandau': '11005005',
    'Steglitz-Zehlendorf': '11006006',
    'Tempelhof-Schöneberg': '11007007',
    'Neukölln': '11008008',
    'Treptow-Köpenick': '11009009',
    'Marzahn-Hellersdorf': '11010010',
    'Lichtenberg': '11011011',
    'Reinickendorf': '11012012'
}

# Apply mapping to create district_id column
hospital_df['district_id'] = (
    hospital_df['district']
    .map(district_mapping)
    .astype(str)
)


#for missing fill with retrieve information from neighborhood and drop the temporary column
hospital_df["neighborhood"] = hospital_df["neighborhood"].fillna(hospital_df["neighborhood_new"])
hospital_df.drop(columns=["neighborhood_new"], inplace=True)
hospital_df.head(10)

```
</details>

---
<details>
<summary><strong>Review created data frame </strong></summary>

```
# Step 7: review created data frame
## 7.1 How many rows and columns?
print("Rows, Columns:", hospital_df.shape)
## 7.2 missing values per columns
missing_count = hospital_df.isna().sum().sort_values(ascending=False)
print(missing_count)
# Number of rows (observations, hospitals)
# I need this to compute percentages of missing values below

row_count = len(hospital_df)
print(row_count)

# Build table with counts and % of missing values
# What does pd.DataFrame({...}) do? It converts that dictionary into a DataFrame (like an Excel table).
# The keys become column names.
# The values become column data.

missing = pd.DataFrame({
    "missing_count": missing_count,
    "missing_pct": (missing_count / row_count * 100).round(1)
}).sort_values(by="missing_pct", ascending=False)

print(missing)

# 7.3. Decision for keeping columns no/yes
beds                          248         98.8.    --> drop
brand                         246         98.0.    --> drop
source                        240         95.6     --> hardcode "OSM

hospital_df = hospital_df.drop(columns=["beds", "brand"])
print(hospital_df)

hospital_df.loc[hospital_df["source"].isna(), "source"] = "OSM"

```
</details>

---
<details>
<summary><strong>Handling of missing values, standardize column names </strong></summary>

```

# 8. Handling of missing value / normalization
# Replace NaN with "unknown" and standardize values

text_cols = ["name", "street", "city", "country", "website", "operator", "brand", "phone", "email", "source", "beds","housenumber", "postcode","emergency","wheelchair", "speciality" ,'opening_hours']
for col in text_cols:
    if col in hospital_df.columns:
        hospital_df[col] = hospital_df[col].astype(str).str.strip()
        hospital_df[col] = hospital_df[col].replace({"nan": "unknown", "none": "unknown", "null": "unknown"})   
hospital_df.head(10)
# Standardize column names

hospital_df.columns = hospital_df.columns.str.lower().str.strip().str.replace(" ", "_").str.replace("-", "_")

# Convert certain columns to correct type

hospital_df["housenumber"] = hospital_df["housenumber"].astype(str)   # ensure text

hospital_df["postcode"] = hospital_df["postcode"].astype(str)         # keep leading zeros

# Normalize yes/no columns into Boolean (True/False)

hospital_df["wheelchair"] = hospital_df["wheelchair"].map({"yes": True, "no": False})

# Make text values consistent (lowercase to avoid duplicates )

```
</details>

---
<details>
<summary><strong>Reset index, drop column "element" , rename "id" to "hospital_id" </strong></summary>

```
# 9. Reset index, drop column "element" , rename "id" to "hospital_id"
# Reset the index
hospital_df= hospital_df.reset_index()

# Rename the "id" column to "hospital_id"
hospital_df = hospital_df.rename(columns={"id": "hospital_id"})  
# set the hospital_id to string
hospital_df["hospital_id"] = hospital_df["hospital_id"].astype(str)
#  
# Drop the redundant column "element", "geometry"
hospital_df= hospital_df.drop(columns=["element"],errors='ignore')
#rename final dataframe to hospital_osm
hospitals_refactored_test = hospital_df

from shapely import wkt

hospitals_refactored_test['geometry'] = hospitals_refactored_test['geometry'].apply(lambda x: x.wkt)
hospitals_refactored_test.head(10)

```
</details>

---
<details>
<summary><strong>Populate Database </strong></summary>

```

# 10. Populate Database
## 10.1 write to neondb for test purpose
# create a neon DB connection to test
#  # DB connection setup using hardcoded credentials 
conn = psycopg2.connect(
    dbname="neondb",
    user="neondb_owner",
    password="password1",
    host="ep-falling-glitter-...",
    port="1234",
    sslmode="require"
)
cur = conn.cursor()
engine = create_engine(
    "postgresql+psycopg2://neondb_owner:password1@ep-falling-glitter...:1234/neondb?sslmode=require"
)
#this is where you create table with constraints and references first
create_table_query = f"""
CREATE TABLE IF NOT EXISTS  hospitals_refactored_test (
    hospital_id VARCHAR(20) PRIMARY KEY,
    district_id VARCHAR(20),
    name VARCHAR(200),
    operator VARCHAR(200),
    country VARCHAR(100),
    city VARCHAR(100),
    street VARCHAR(200),
    housenumber VARCHAR(20),
    postcode VARCHAR(10),
    heighborhood VARCHAR(100),
    phone VARCHAR(50),
    email VARCHAR(200),
    website VARCHAR(200),
    wheelchair VARCHAR(100),
    toilets_wheelchair VARCHAR(100),
    emergency VARCHAR(200),
    speciality VARCHAR(200),
    opening_hours VARCHAR(200),
    latitude DECIMAL(9,6),
    longitude DECIMAL(9,6),
    geometry VARCHAR(200),
    source VARCHAR(100),
    amenity_tag VARCHAR(100),
    healthcare_tag VARCHAR(100),
    district VARCHAR(100),
    CONSTRAINT district_id_fk FOREIGN KEY (district_id)
        REFERENCES test_berlin_data.districts(district_id)
        ON DELETE RESTRICT
        ON UPDATE CASCADE
);
"""
# Execute the create table query
with engine.begin() as connection:
    connection.execute(text(create_table_query))
    connection.commit()
    print("Table 'hospitals_refactored_test' created or already exists.")

# to sql write into datatbase
hospitals_refactored_test.to_sql(
    'hospitals_refactored_test',      
    con=engine,   
    if_exists='replace',
    schema='test_berlin_data',
    index=False
)
# inquire the hospital_refactored_test

df= pd.read_sql("SELECT * FROM test_berlin_data.hospitals_refactored_test",engine)
print(df.head())

## 10.2 populate hospital_refactored in layereddb
hospitals_refactored = hospitals_refactored_test

# For all string/object columns, find the max length
for col in hospitals_refactored.columns:
    # Only check columns that are string dtype
    if pd.api.types.is_string_dtype(hospitals_refactored[col]):
        max_len = hospitals_refactored[col].dropna().str.len().max()
        if max_len > 200:
            print(f"Column '{col}' has max length {max_len}")


# Connect to postgres DB
user_name='user'
password='password2'

# Conection
host = 'localh'
port = '1234'
database = 'layereddb'
schema='berlin_source_data'

#connection to db after you opened tunnel
engine = create_engine(f'postgresql+psycopg2://{user_name}:{password}@{host}:{port}/{database}')

#this is where you create table with constraints and references first
create_table_query = f"""
CREATE TABLE IF NOT EXISTS {schema}.hospitals_refactored (
    hospital_id VARCHAR(20) PRIMARY KEY,
    district_id VARCHAR(20),
    name VARCHAR(200),
    operator VARCHAR(200),
    country VARCHAR(100),
    city VARCHAR(100),
    street VARCHAR(200),
    housenumber VARCHAR(20),
    postcode VARCHAR(10),
    neighborhood VARCHAR(100),
    phone VARCHAR(50),
    email VARCHAR(200),
    website VARCHAR(200),
    wheelchair VARCHAR(100),
    toilets_wheelchair VARCHAR(100),
    emergency VARCHAR(200),
    speciality VARCHAR(250),
    opening_hours VARCHAR(200),
    latitude DECIMAL(9,6),
    longitude DECIMAL(9,6),
    geometry VARCHAR(200),
    source VARCHAR(100),
    amenity_tag VARCHAR(100),
    healthcare_tag VARCHAR(100),
    district VARCHAR(100),
    CONSTRAINT district_id_fk FOREIGN KEY (district_id)
        REFERENCES berlin_source_data.districts(district_id)
        ON DELETE RESTRICT
        ON UPDATE CASCADE
);
"""
# Execute the create table query
with engine.begin() as connection:
    connection.execute(text(create_table_query))
    connection.commit()
    print("Table 'hospitals_refactored' created or already exists.")

## 10.3 query the table
##let's query test data!
query = f"""
SELECT * from berlin_source_data.hospitals_refactored
"""

# Execute the query
with engine.connect() as conn:
    df= pd.read_sql(text(query), conn)
    conn.commit()  # commit the transaction
df
```
</details>


