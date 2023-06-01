---
layout: "post"
title:  "Nearby places search with data from OSM using PostgreSQL & PostGIS extension"
date:   2023-6-1 09:28:37 +0700
categories: sql stuff 
permalink: "/postgis1/"
author: "heabeoun himself"
---
# Nearby places search with data from OSM using PostgreSQL & PostGIS extension

![image](\assets\img\portrait1.jpeg){: width="600"}
For this demo we will learn how to find the distance from a given longitude/latitude coordinates
to another longitude/latitude tagged place of our interest. 
## Installing required thingies:
###  PostgreSQL 
*there are many tutorials out there :>*
### PostGIS
you can follow the tutorial on how to install for your system [here](https://postgis.net/documentation/getting_started/)
after you're done installing PostGIS , Create a database and then activate PostGIS for your database:
*connect to your database using psql or pgAdmin* then run:
```sql
CREATE EXTENSION postgis; 
-- the postgis extension
CREATE EXTENSION hstore; 
-- hstore extension for storing unstructured data types
```
***optional***
because we are going to import osm data for this tutorial , we will be using a tool called `osm2pgsql`
to install it simply run:
```shell
apt install osm2pgsql
```
### Getting data
we will be importing data from [OpenStreetMap](https://www.openstreetmap.org), then you can get export data from you location of choice by simply clicking "export" on the top left and then you will see a bar then press the blue "export" button again it will download a file called "map.osm" , then to import osm data into your database
```shell
osm2pgsql -U admin -W -d osm -H yourhostname -P yourportnumber --hstore --hstore-add-index map.osm  
```

Start out with the `osm2pgsql` command and add `-U` with your database username (usually `admin`) and `-W` indicating that you need a password prompt. Next, add `-d` and your database name (here _osm_). The `-H` option is the deployment's hostname and `-P` is the port number. We've added the `--hstore` to create _tags_ columns for each table that contains the supplemental non-standardized data and the `--hstore-add-index` option sets up indexes on those columns. Finally, we add the `map.osm` file that we downloaded. Now your database should have four tables (excluding : *spatial_ref_sys* as PostGIS requires this table to work)

## The fun part
### 1.  *Getting standard longitude/latitude coordinates from our imported osm data:*
In our imported osm data there are 4 tables in total, Since we want to get the distance from our location to a place of our interest in standard longitude/latitude format, We will be using the `planet_osm_points` since that table stores landmarks. Now to get the coordinates of locations labeled as `amenity` we can run :
```sql
SELECT ST_X(ST_AsText(ST_Transform(way, 4326))) as longitude
,ST_Y(ST_AsText(ST_Transform(way, 4326))) as latitude, 
amenity,name 
FROM planet_osm_point WHERE amenity IS NOT NULL;
```
it will output:

| longitude | latitude | amenity | name|
| -- | -- | --| --|
| 104.92080459999998 | 11.556113299541718 | atm | ANZ Royal ATM |
| 104.92778869999998 | 11.553392399541488 | restaurant | Il Forno|
now,  what makes PostGIS special is it's built in functions, We will go through the functions together. First , in our select command is `ST_Transform`, Since the column `way` is stored in PostGIS's  `geometry` datatype, which is just computer gibberish. In order for it to make sense to us we need to convert it to a human readable format , or at least a human readable spatial reference system, which in our case we want it to be in *WGS84*  which has a number of 4326 in the [ESPG](https://en.wikipedia.org/wiki/EPSG_Geodetic_Parameter_Dataset) identifier *( WGS84 is a standard coordinate frame for the Earth, a datum/reference ellipsoid for raw altitude data. WGS84 and ESPG is out of the scope of discussion for this article )* , then we want to convert the geometry format into plain text therefore we wrap it in `ST_AsText`, so currently if we run 
```sql
SELECT ST_AsText(ST_Transform(way, 4326)) 
FROM planet_osm_point WHERE amenity IS NOT NULL;
```
we get
| st_astext | 
|--|
| "POINT(104.92080459999998 11.556113299541718)" |
| "POINT(104.92778869999998 11.553392399541488)" |
....
it is in PostGIS's point format so, this is where `ST_X` and `ST_Y` comes in, as it is a handy function to map the Points to longitude and latitude according to the X and Y respectively, so the final result is like at the top.

### 2. Getting the distance from a given lat/long to another lat/long
it is very easy to get the distance from one point to another by using `ST_Distance`
*for instance:*
```sql
SELECT ST_Distance(
    'SRID=4326;POINT(104.92080459999998 11.556113299541718)'::geography,
    'SRID=4326;POINT(104.9259047 11.556225899541724)'::geography
);
```	
we will get:
|st_distance|
|--|
|556.44590038|
now, you can read about what is the `geography` thingy [*here*](http://postgis.net/workshops/postgis-intro/geography.html), in the mean time our distance between the points are `556.44590038` , this number is in meters. *very easy !*
### 3. Putting it all together
If we put the two things we learned at the top together we can get query for the nearest location within a , let's say 2km radius
```sql
SELECT 
ST_Distance(
	'SRID=4326;POINT(104.92379749999999 11.535633599540022)'::geography,
	ST_AsEWKT(ST_Transform(way, 4326))::geography
) AS distance,
 name, amenity 
 FROM planet_osm_point 
 WHERE amenity = 'fast_food' 
 AND ST_Distance(
	'SRID=4326;POINT(104.92379749999999 11.535633599540022)'::geography,
	ST_AsEWKT(ST_Transform(way, 4326))::geography
) <= 2000; --this is the radius 
```
*note: `ST_AsEWKT` is just `ST_AsText` with the SRID included in the text so it is easier for us to cast it as a geography type when we run 	`ST_Distance`*
it will output:

| distance | name | amenity |
| -- | -- | --|
| 1958.01669185 | Krispy Kreme | fast_food| 
| 1850.08767635 | null | fast_food |
note : the distance is in *meters*.

but wait, there *is* also another built in PostGIS just to find points in a radius called `ST_DWithin` so , we can revise our sql command to:
```sql
SELECT 
ST_Distance(
	'SRID=4326;POINT(104.92379749999999 11.535633599540022)'::geography,
	ST_AsEWKT(ST_Transform(way, 4326))::geography
) AS distance,
 name, amenity 
 FROM planet_osm_point 
 WHERE ST_DWithin('SRID=4326;POINT(104.92379749999999 11.535633599540022)'::geography,
				 ST_AsEWKT(ST_Transform(way, 4326))::geography, 2000 --this is the radius
				 ) AND amenity IS NOT NULL;
```

## Building our own partner locations finder
With what we learned above we can build our very own table in PostgreSQL to find our bussiness partners within a given radius, we only need to fit it to our own use case therefore making our own table, which would look something like:
| id [PK] | site (varchar) | longlat(geography) | type(varchar) |
|--|--|--|--|
| 3124 | beoun.net |SRID=4326;POINT(104.92379749999999 11.535633599540022)|shop|
| 4232 | goofyahh.bar |SRID=4326;POINT(104.92379749999999 11.535633599540022)|bar|

## More reading, if you are interested in diving into this 
[Postgis tutorial/demo](https://postgis.net/workshops/postgis-intro/)
[World Geodetic System, aka what that WGS84 thingy means](https://en.wikipedia.org/wiki/World_Geodetic_System)


