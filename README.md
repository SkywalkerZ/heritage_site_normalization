# heritage_site_normalization

## Introduction
In this project, I create a raw table in postgreSQL, injest data into the table, analyse the data lengths and create new tables to aid in normalization. 
All activities done using PSQL, which is a Postgre CLI, more commonly used in Linux systems.

## Raw Table creation

```
heritage_site=# CREATE TABLE unesco_raw
heritage_site-#  (name TEXT, description TEXT, justification TEXT, year INTEGER,
heritage_site(#     longitude FLOAT, latitude FLOAT, area_hectares FLOAT,
heritage_site(#     category TEXT, category_id INTEGER, state TEXT, state_id INTEGER,
heritage_site(#     region TEXT, region_id INTEGER, iso TEXT, iso_id INTEGER);
```
![image](https://github.com/SkywalkerZ/heritage_site_normalization/assets/6307592/395f7265-583e-42a2-8d14-edc94c716959)

## Raw Data injestion

```
heritage_site=# \copy unesco_raw(name,description,justification,year,longitude,latitude,area_hectares,category,state,region,iso) FROM 'C:\Users\abraa\Documents\Po
stgres csv files\whc-sites-2018-small.csv' WITH DELIMITER ',' CSV HEADER;
```
![image](https://github.com/SkywalkerZ/heritage_site_normalization/assets/6307592/ee2cca92-157d-4d45-8432-c80f35757b1f)

As you may have noticed, many of the columns in the create statement, such as category_id, state_id etc, are not being populated. We will do this manually with UPDATE statements later.

## Data lengths analysis

```
heritage_site=# SELECT max(length(name)) AS max_len_name, max(length(description)) AS max_len_description, max(length(justification)) AS max_len_justification, ma
x(length(category)) AS max_len_category, max(length(state)) AS max_len_state, max(length(region)) AS max_len_region, max(length(iso)) AS max_len_iso FROM unesco_r
aw;
```
![image](https://github.com/SkywalkerZ/heritage_site_normalization/assets/6307592/c8920ab2-aec7-453e-b0eb-c1642589f999)

## Normalized Table Design

Lets use the lengths discovered in previous query to design a more structured table.

### UNESCO Table
```
heritage_site=# CREATE TABLE unesco (id SERIAL, name VARCHAR(200), description VARCHAR(2000), justification VARCHAR(4000), year SMALLINT, LONGITUDE FLOAT, LATITUD
E FLOAT, area_hectores FLOAT, category_id INTEGER, state_id INTEGER, region_id INTEGER, iso_id INTEGER, PRIMARY KEY(id));
```
![image](https://github.com/SkywalkerZ/heritage_site_normalization/assets/6307592/f650d427-187d-4928-91be-e8ba6426ed6b)

```
heritage_site=# ALTER TABLE unesco ADD CONSTRAINT fk_category FOREIGN KEY(category_id) REFERENCES category(id),ADD CONSTRAINT fk_state FOREIGN KEY(state_id) REFER
ENCES state(id),ADD CONSTRAINT fk_region FOREIGN KEY(region_id) REFERENCES region(id),ADD CONSTRAINT fk_iso FOREIGN KEY(iso_id) REFERENCES iso(id);
```

### Category Table
```
CREATE TABLE category (id SERIAL, name VARCHAR(20), PRIMARY KEY(id));
```
![image](https://github.com/SkywalkerZ/heritage_site_normalization/assets/6307592/47ed0b98-2582-43e2-8eca-09f9c951d57c)

### State Table
```
heritage_site=# CREATE TABLE state (id SERIAL, name VARCHAR(100), PRIMARY KEY(id));
```
![image](https://github.com/SkywalkerZ/heritage_site_normalization/assets/6307592/91e3a8fc-baea-48f9-ae65-737c9aa29ba2)

### Region Table
```
heritage_site=# CREATE TABLE region (id SERIAL, name VARCHAR(100), PRIMARY KEY(id));
```
![image](https://github.com/SkywalkerZ/heritage_site_normalization/assets/6307592/31be55a7-2918-44db-a990-838da2408560)

### ISO Table
```
heritage_site=# CREATE TABLE iso (id SERIAL, name VARCHAR(5), PRIMARY KEY(id));
```
![image](https://github.com/SkywalkerZ/heritage_site_normalization/assets/6307592/15802090-71f6-4d02-bda3-35eb4b47f02d)

## Normalized Data Ingestion

### Category Table
```
heritage_site=# INSERT INTO category(name) SELECT DISTINCT(category) FROM unesco_raw;
```
![image](https://github.com/SkywalkerZ/heritage_site_normalization/assets/6307592/77d58afd-5d7a-415d-91e6-9333625e0150)

### State Table
```
heritage_site=# INSERT INTO state(name) SELECT DISTINCT(state) FROM unesco_raw;
```
![image](https://github.com/SkywalkerZ/heritage_site_normalization/assets/6307592/05221769-b7a2-49e5-bc4d-c0e455aee630)

### Region Table
```
heritage_site=# INSERT INTO region(name) SELECT DISTINCT(region) FROM unesco_raw;
```
![image](https://github.com/SkywalkerZ/heritage_site_normalization/assets/6307592/73123f13-637d-434c-abd8-7049aa0814c4)

### ISO Table
```
heritage_site=# INSERT INTO iso(name) SELECT DISTINCT(iso) FROM unesco_raw;
```
![image](https://github.com/SkywalkerZ/heritage_site_normalization/assets/6307592/8db75e11-9321-41e8-bb03-5b0a57e10831)

### UNESCO_RAW Table
```
heritage_site=# UPDATE unesco_raw SET category_id = (SELECT category.id FROM category WHERE category.name = unesco_raw.category);

heritage_site=# UPDATE unesco_raw SET state_id = (SELECT state.id FROM state WHERE state.name = unesco_raw.state);

heritage_site=# UPDATE unesco_raw SET region_id = (SELECT region.id FROM region WHERE region.name = unesco_raw.region);

heritage_site=# UPDATE unesco_raw SET iso_id = (SELECT iso.id FROM iso WHERE iso.name = unesco_raw.iso);
```
![image](https://github.com/SkywalkerZ/heritage_site_normalization/assets/6307592/baf3ff0a-07e4-4787-baca-d28feea5fd11)

### UNESCO Table
```
heritage_site=# INSERT INTO unesco(name,description,justification,year,longitude,latitude, area_hectores, category_id, state_id, region_id, iso_id) SELECT name,de
scription,justification,year,longitude,latitude,area_hectares,category_id,state_id,region_id,iso_id FROM unesco_raw;
```
![image](https://github.com/SkywalkerZ/heritage_site_normalization/assets/6307592/d3b9aaff-6f63-4cc7-b24b-7926eac72ea4)

## Analysis

Let's join the tables to view the data in the right manner, rather than the foreign keys which could mean anything

```
heritage_site=# SELECT unesco.name AS heritage_site, unesco.year AS year, category.name AS category, state.name AS state, region.name AS region,iso.name AS iso FR
OM unesco JOIN category ON unesco.category_id = category.id JOIN iso ON unesco.iso_id = iso.id JOIN state ON unesco.state_id = state.id JOIN region ON unesco.regi
on_id = region.id ORDER BY iso.name, unesco.name LIMIT 50;
```
![image](https://github.com/SkywalkerZ/heritage_site_normalization/assets/6307592/665fe6ae-5218-4676-a435-a4aa6a888285)





