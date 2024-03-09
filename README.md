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

```
heritage_site=# CREATE TABLE unesco (id SERIAL, name VARCHAR(200), description VARCHAR(2000), justification VARCHAR(4000), year SMALLINT, LONGITUDE FLOAT, LATITUD
E FLOAT, area_hectores FLOAT, category_id INTEGER, state_id INTEGER, region_id INTEGER, iso_id INTEGER);
```
![image](https://github.com/SkywalkerZ/heritage_site_normalization/assets/6307592/f650d427-187d-4928-91be-e8ba6426ed6b)




