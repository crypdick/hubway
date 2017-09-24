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

# 2
Write these queries.

## a
Find the first 10 the station names whose status correspond to 'Removed', sorted by station, ascending.

## b
Find the first 10 the station names that are located inside the bounding box formed by two given (latitude, longitude) points, sorted by station, ascending.

## c
Find the first 10 trips' ids (hubway\_id) that started or ended at stations within a bounding boxed formed by two given (latitude, longitude) points, sorted by id, ascending.

# 3
Write another query to find records in the hubway tables with suspect values of zip code or duration. Start by snooping around the database manually to see how data can be bad. Find any other bad data you can.
Your query will create a table whose first column is called problem, a short description of the problem, and whose other columns are enough to identify the bad record and illustrate the problem with the data. Show five examples of each problem.

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
