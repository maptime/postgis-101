PostGIS-101
===========

What is PostGIS?

---

PostGIS is a _spatial_ database.

---

Meaning,

---

it _stores_ spatial data

---

e.g.
* where are all the police stations?
* what is the temperature across the city?
* what are the boundaries of the police wards?
* where are all the ice cream parlors?

---

but also allows for the analysis of spatial data

---

e.g.
* What are the 2 closest ice cream shops to each police station?

---

Spatial & non-spatial data can be analysed at the same time

---

* What is the most cost effective ice cream shop given ice cream prices, driving distance, and current gas prices relative to each police station?

---

And the analyses are _web scale_

---

It handles vector and raster data

---

placeholder for layers

---

And it's SQL (simple query language)

---

SELECT superhero.name
FROM city, superhero
WHERE ST_Contains(city.geom, superhero.geom)
AND city.name = 'Gotham';

---

Let's do some basic work in it.  We'll make some random xy values and turn them into points:

---

-- cleanup

---

DROP TABLE IF EXISTS xyz CASCADE;

---

-- create a table

---

CREATE TABLE xyz

(

x DOUBLE PRECISION,

y DOUBLE PRECISION,

z DOUBLE PRECISION

)

WITH (OIDS=FALSE);

---

-- Add a primary key

---

ALTER TABLE xyz ADD COLUMN gid serial;

ALTER TABLE xyz ADD PRIMARY KEY (gid);

---

Insert some data

---

INSERT INTO xyz (x, y, z)
VALUES (random()*100, random()*12, random()*56);

INSERT INTO xyz (x, y, z)
VALUES (random()*12, random()*56, random()*25);

INSERT INTO xyz (x, y, z)
VALUES (random()*56, random()*25, random()*100);

INSERT INTO xyz (x, y, z)
VALUES (random()*25, random()*100, random()*12);

---

-- cleanup before creating view

---

DROP VIEW IF EXISTS xbecausez;

---

And now we'll create a spatial view that makes all our xyz values into points.

---

CREATE VIEW xbecausez AS

SELECT x, y, z, ST_SetSRID(ST_MakePoint(x,y), 4326) AS geom

FROM xyz;

---

Voila!

---

![](https://raw.githubusercontent.com/maptime/postgis-101/opengeocle/img/random_points.png)

---

Now if we get new points with just xyz values, but no spatial information, those will automatically become spatial!

---

INSERT INTO xyz (x, y, z)
VALUES (random()*100, random()*12, random()*56);

INSERT INTO xyz (x, y, z)
VALUES (random()*12, random()*56, random()*25);

INSERT INTO xyz (x, y, z)
VALUES (random()*56, random()*25, random()*100);

INSERT INTO xyz (x, y, z)
VALUES (random()*25, random()*100, random()*12);

---

![](https://raw.githubusercontent.com/maptime/postgis-101/opengeocle/img/moar_random_points.png)

---

Cool, points on a map that auto-update.  Useful, but let's go deeper.  Let's add a whole lot more points.

---


INSERT INTO xyz (x, y, z)
VALUES (random()*100, random()*12, random()*56);

INSERT INTO xyz (x, y, z)
VALUES (random()*12, random()*56, random()*25);

INSERT INTO xyz (x, y, z)
VALUES (random()*56, random()*25, random()*100);

INSERT INTO xyz (x, y, z)
VALUES (random()*25, random()*100, random()*12);

--...

---

What is the overall shape of my points? I could calculate the box they fit into.

---

We'll directly create a table this time (rather than a view).

---

DROP TABLE IF EXISTS envelope;

---

CREATE TABLE envelope AS

WITH pointz AS

	(

	SELECT x, y, z, ST_SetSRID(ST_MakePoint(x,y), 4326) AS geom FROM xyz

	)

,collectionz AS

	(

	SELECT ST_Union(geom) AS unioned FROM pointz

	)

,extentz AS

    (

    SELECT ST_Extent(unioned)::geometry AS geom FROM collectionz

	)
    
SELECT 1 AS id, geom FROM extentz;

---
