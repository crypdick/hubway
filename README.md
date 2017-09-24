# Hubway Data

Create two appropriately named tables and load the Hubway csv files into the tables. Include any reasonable integrity constraints in your “create table” commands. Please at least include a foreign key constraint. Read the README file!

This file contains metadata for both the Trips and Stations table.

Trips Table Variables: (current as of end of November 2013)
- seq_id: unique record ID
- hubway_id: trip id
- status: trip status; "closed" indicates a trip has terminated
- duration: time of trip in seconds
- start_date: start date of trip with date and time, in EST
- strt_statn: id of start station
- end_date: end date of trip with date and time, in EST
- end_statn: station id of end station
- bike_nr: id of bicycle used
- subsc_type: subscription type - "Registered" is user with membership; "Casual" is user without membership
- zip_code: zipcode of user (only available for registered users) **data includes an apostrophe(') prefix**
- birth_date: birth year of user
- gender: gender of user

Database format:
  seq_id 	SEQUENCE primary key,
  hubway_id 	bigint,
  status 	character varying(10),
  duration 	integer,
  start_date 	timestamp without time zone,
  strt_statn 	integer,
  end_date 	timestamp without time zone,
  end_statn 	integer,
  bike_nr 	character varying(20),
  subsc_type 	character varying(20),
  zip_code 	character varying(6),
  birth_date 	integer,
  gender 	character varying(10)



Stations Table Variables: (current as of end of November 2013)

Note on variables id versus terminal: Hubway-assigned terminal names (see variable terminalName below) correspond to physical stations. If stations move from one year to the next, the terminal name does not change. This may cause confusion if analysts try to compare station data from different years. Therefore, MAPC added a station id (see variable id below) that corresponds to stations' latitude and longitude. If stations move, they receive a new station id, even if their terminal name does not change. In most cases, station movements are small (across a street, or a block down the same street). 

- id: station id assigned by MAPC; corresponds to start_station and end_station_id in trips table
- terminal: Hubway-assigned station identifier
- station: station name
- municipal: Municipality
- lat: station latitude
- lng: station longitude
- status: Existing station locations and ones that have been removed or relocated

# 1
Create a database called hubway\_<lastname> from the data in compsci01:/usr/share/databases/Hubway. Create two appropriately named tables and load the Hubway csv files into the tables. Include any reasonable integrity constraints in your “create table” commands. Please at least include a foreign key constraint. Read the README file!

[brew services start postgresql]
psql -p5432 -d "username"
CREATE DATABASE hubway_duffrindecal;
\connect hubway_duffrindecal
CREATE TABLE trips (
  seq_id  serial primary key,
  hubway_id   bigint,
  status  character varying(10),
  duration  integer,
  start_date  timestamp without time zone,
  strt_statn  integer,
  end_date  timestamp without time zone,
  end_statn   integer,
  bike_nr   character varying(20),
  subsc_type  character varying(20),
  zip_code  character varying(6),
  birth_date  integer,
  gender  character varying(10));

CREATE TABLE stations (
  id  serial primary key,
  terminal   char(6),
  station  varchar(80),
  municipal  varchar(20),
  lat  double precision,
  lng  double precision,
  status  varchar(8));

\copy stations FROM data/hubway_stations.csv  CSV HEADER;
\copy trips FROM data/hubway_trips.csv  CSV HEADER;

# 2
Write these queries.

## a
Find the first 10 station names whose status correspond to 'Removed', sorted by station, ascending.

SELECT station FROM stations WHERE status = 'Removed' ORDER BY station ASC LIMIT 10;
                              station                              
-------------------------------------------------------------------
 Andrew Station - Dorchester Ave at Humboldt Pl
 Boston Medical Center - 721 Mass. Ave.
 Boylston at Fairfield
 Brookline Town Hall / Library Washington St
 Charles Circle - Charles St. at Cambridge St.
 Dudley Square
 Harvard  University River Houses at DeWolfe St / Cowperthwaite St
 Mayor Thomas M. Menino - Government Center
 New Balance - 38 Guest St.
 Overland St at Brookline Ave
(10 rows)

## b
Find the first 10 station names that are located inside the bounding box formed by two given (latitude, longitude) points, sorted by station, ascending.

SELECT min(lat) AS min_lat, max(lat) AS max_lat, min(lng) AS min_lng, max(lng) AS max_lng FROM stations;
  min_lat  | max_lat  |  min_lng   |  max_lng   
-----------+----------+------------+------------
 42.309467 | 42.40449 | -71.146452 | -71.035705
(1 row)

lat = 42.35 42.4
lng = -71.1 -71.05
       low    hi

SELECT station FROM stations WHERE lat > 42.35 AND lat < 42.4 AND lng > -71.1 AND lng < -71.05 ORDER BY station ASC LIMIT 10;
                        station                         
--------------------------------------------------------
 Aquarium Station - 200 Atlantic Ave.
 Beacon St / Mass Ave
 Biogen Idec - Binney St / Sixth St
 Boylston St / Berkeley St
 Boylston St / Washington St
 Boylston St. at Arlington St.
 Cambridge St - at Columbia St / Webster Ave
 Cambridge St. at Joy St.
 CambridgeSide Galleria - CambridgeSide PL at Land Blvd
 Charles Circle - Charles St. at Cambridge St.
(10 rows)



## c
Find the first 10 trips' ids (hubway\_id) that started or ended at stations within a bounding boxed formed by two given (latitude, longitude) points, sorted by id, ascending.


CREATE TEMP TABLE stations_in_box AS (SELECT id FROM stations WHERE lat > 42.35 AND lat < 42.4 AND lng > -71.1 AND lng < -71.05);

SELECT hubway_id FROM trips, stations_in_box WHERE id = strt_statn OR id = end_statn LIMIT 10;
 hubway_id 
-----------
         8
         9
        10
        11
        12
        13
        14
        15
        16
        17
(10 rows)

# 3
Write another query to find records in the hubway tables with suspect values of zip code or duration. Start by snooping around the database manually to see how data can be bad. Find any other bad data you can.
Your query will create a table whose first column is called problem, a short description of the problem, and whose other columns are enough to identify the bad record and illustrate the problem with the data. Show five examples of each problem.

Problem records can have invalid zip codes, a duration that doesn't match the time between start and end (while still accounting for daylight savings), and trips that are less than 5 minutes, with an emphasis on 0 minute trips.

All zip codes are suspect, but assumed to be purely uncleaned data from the leading apostraphe.

(zip, length, discrepancy, duration)
first column - problem (zip not 5 digits, length is negative, length is 0, discrepancy between computed duration and given, duration less than 5 minutes, duration 0, duration negative)
other columns - zip_code, start_date, end_date, duration, seq_id

CREATE TEMP TABLE problem_trips AS (SELECT 'zip not 5 digits' AS problem, zip_code, start_date, end_date, duration, seq_id FROM trips WHERE length(replace(zip_code, '''', '')) != 5);

     problem      | zip_code |     start_date      |      end_date       | duration | seq_id 
------------------+----------+---------------------+---------------------+----------+--------
 zip not 5 digits | 2114     | 2012-03-21 19:19:00 | 2012-03-21 19:31:00 |      723 | 145217
 zip not 5 digits | 2114     | 2012-03-22 08:31:00 | 2012-03-22 08:43:00 |      738 | 145556
 zip not 5 digits | 2114     | 2012-03-27 14:22:00 | 2012-03-27 14:37:00 |      888 | 150973
 zip not 5 digits | 2114     | 2012-03-28 07:54:00 | 2012-03-28 08:05:00 |      622 | 151525
 zip not 5 digits | 2134     | 2012-03-29 19:27:00 | 2012-03-29 19:39:00 |      749 | 153077
(5 rows)


INSERT INTO problem_trips (SELECT 'recorded duration is negative' AS problem, zip_code, start_date, end_date, duration, seq_id FROM trips WHERE duration < 0);

       problem        | zip_code |     start_date      |      end_date       | duration | seq_id 
----------------------+----------+---------------------+---------------------+----------+--------
 duration is negative |          | 2012-11-04 01:49:00 | 2012-11-04 01:01:00 |    -6480 | 634341
 duration is negative |          | 2012-11-04 01:50:00 | 2012-11-04 01:02:00 |    -6480 | 634342
 duration is negative |          | 2012-11-04 01:53:00 | 2012-11-04 01:21:00 |    -5520 | 634343
 duration is negative |          | 2012-11-04 01:53:00 | 2012-11-04 01:21:00 |    -5520 | 634344
 duration is negative |          | 2012-11-04 01:53:00 | 2012-11-04 01:34:00 |    -4740 | 634345
(5 rows)

INSERT INTO problem_trips (SELECT 'recorded duration is zero' AS problem, zip_code, start_date, end_date, duration, seq_id FROM trips WHERE duration = 0);

     problem      | zip_code |     start_date      |      end_date       | duration | seq_id 
------------------+----------+---------------------+---------------------+----------+--------
 duration is zero | '02139   | 2012-11-03 21:01:00 | 2012-11-03 21:01:00 |        0 | 634131
 duration is zero | '53207   | 2012-11-03 22:46:00 | 2012-11-03 22:46:00 |        0 | 634215
 duration is zero | '02143   | 2012-11-04 12:21:00 | 2012-11-04 12:21:00 |        0 | 634957
 duration is zero | '02446   | 2012-11-04 12:41:00 | 2012-11-04 12:41:00 |        0 | 635018
 duration is zero | '02446   | 2012-11-04 12:48:00 | 2012-11-04 12:48:00 |        0 | 635038
(5 rows)

INSERT INTO problem_trips (SELECT 'discrepancy between computed and recorded duration' AS problem, zip_code, start_date, end_date, duration, seq_id FROM trips WHERE abs((extract(epoch FROM end_date - start_date)/60) - (duration / 60)) > 1 AND abs((extract(epoch FROM end_date - start_date)/60) - (duration / 60)) < 59);

                      problem                       | zip_code |     start_date      |      end_date       | duration | seq_id 
----------------------------------------------------+----------+---------------------+---------------------+----------+--------
 discrepancy between computed and recorded duration |          | 2011-07-28 16:19:00 | 2011-07-28 17:10:00 |     2951 |    160
 discrepancy between computed and recorded duration |          | 2011-07-29 02:11:00 | 2011-07-29 02:38:00 |     1511 |    440
 discrepancy between computed and recorded duration |          | 2011-07-29 13:07:00 | 2011-07-29 13:18:00 |      444 |    617
 discrepancy between computed and recorded duration |          | 2011-07-30 22:02:00 | 2011-07-30 22:10:00 |      233 |   1716
 discrepancy between computed and recorded duration |          | 2011-07-31 17:21:00 | 2011-07-31 17:49:00 |     1575 |   2439
(5 rows)

# 4
Create a database called fanfiction\_<lastname>. Create a stories\_orig table with url as the primary key. 

## a
Use \copy to load Fanfiction/stories\_orig.csv into the table. Despite what Postgres may say, the problem isn’t that url is the primary key; the problem is in the data. Explain. 

## b
Work around the data problem temporarily by adding a sequence column to the table, removing url as a primary key, and loading the table. (Consult the Postgres docs and/or StackOverflow.)

## c
Fix the data problem using a select statement that joins stories\_orig with itself (!), checking for records with the same url. Try your sql with explain with and without an index, but run it with an index.

# 5
Write two python programs to fix the data problem. These are two strategies 

## a
One program sorts the rows of stories\_orig.csv and traverses rows in sorted order.

## b
The other program uses a hash:

### i
Find a way to map each row r to a number h(r) between 0 and N = 220-1. 
One suggestion (among many): represent published as mmddyy and words as nnn...n, and set h(r) to mmddyynnn...n mod N.

### ii
Create a python list of N empty lists.

### iii
Add each row r to the h(r)th list. 

### iv
Traverse the list of lists. If any list contains more than one row, compare them. 

## c
Which program runs faster? Explain.
