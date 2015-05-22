# Sensor Data Import

Package with functions for processing environmental sensor data, create and load a PostgreSQL database and run Shiny apps specific to the project needs.


### General structure

* Functions related to creating or manipulating the PostgreSQL database are in `postgresql.R`
* The shiny apps themselves live in `inst/shiny-apps`


### In terms of the flow:

* Create a new postgreSQL database with the `create_database()` function
* Add tables with the `add_tables_db()` function (refers to an existing SQL file right now)
* runShiny("nyc") will launch the Shiny app (inst/shiny-apps/nyc)
* The server.R file sets the working directory to the shiny apps working directory
* The server.R file tries to connect to the DB using the function `get_connection()`. It reports an error if there was a problem with connection.
* The server then waits for upload.
* On upload the server tests for names that match a specific pattern. In this instance it looks for the first three letters to be ABP, GPS etc. If not it reports an error.
* The server then uses the function `process_data()` to try and process. This function is part of utils.R and it actually doesn't process the data it determines which of the processing scripts to use and then runs that script. The processing scripts are in the `sensor_processing.R` script and they each return a processed datafile.
* Note that each of the processing scripts makes use of a function called `repeatFileInfo()`. What this function does is takes the split file name ("fileinfo") as input and simply repeats this over and over. So if you have a file called GPS_123 then it creates a column where "GPS" is repeated over and over and 123 is repeated over and over. This way the table has the metadata extracted from the file name.
* The server then tries to upload the data with the function `upload_postgres()` which is a very simple script with one function for uploading. Note that this APPENDS data -- the table needs to exist.




### Including the Shiny App
http://www.r-bloggers.com/supplementing-your-r-package-with-a-shiny-app-2/

* devtools::load_all()
* runShiny("nyc")



### To Do

* PostgreSQL functions on a Mac?
* roxygen2::document() a problem with files on network drive. Issue with `file.access()` within the `digest` package.
* ~~When "objects listed as exports, but not present in namespace" delete NAMESPACE.md~~ I was getting this error I believe because I had a commented out function with roxygen comments. I deleted and the error went away
* May need to rethink the database connection and allow the user to set the ports etc. This would probably mean taking the `get_connection()` out of the Shiny app and put it either in `runShiny()` or in it's own function.
* Explicit disconnect from DB
* The `create_database` and `add_tables_db` functions are not flexible and only create the bike project DB and add tables based on the bike project SQL.
* Create a file name test function. Right now we test for whether the file name includes ABP, GPS etc right in the server and it's not flexible. Probably we want a function with test_filenames that will allow us to add rules.
* Better warning messages about DB upload problems. Use `RPostgreSQL::dbListTables(.connection$con)` to get list of tables and test.

### If you're getting errors

* Sometimes deleting NAMESPACE and re-running document(), load_all() helped
* The rows might be getting uploaded to PostgreSQL but pgAdmin may not be updating so you may need to run `count` rather than `refresh`
* How should we handle time zone -- do we need to have the user input?



# From previous notes

Short-term tasks

1. Need to review what happens with time zones and daylight savings time

2. ~~Add file name to all the tables in the database~~

3. Double check how the line in the microPEM (30-30 or a string) is being processed

4. Double check memory usage, particularly when processing many files.

5. Double check how the errors are being handled. Specifically are other files being processed after an error or does it stop there.

6. Add a check to see if the data has already been uploaded by checking file name or file name information

Longer-term tasks

1. We will need to develop and approach to assigning "session" to all the files. This could potentially be in the SQL database or maybe when the data is pulled.

2. Make the app an R package for more easy distribution

3. Need to consider how the database will get backed up

4. Rather than create new tables with session etc potentially use PostgreSQL "views"

5. Create a delete data function. We discussed adding this to Shiny but this may not be worth the effort, an R function may be enough.

6. We need some kind of "events" field or check. For example, if humidity data is bad, but other data is good for a line how do we handle.

8. We need to add data quality checks on file upload.

9. In some data files there may be missing dates or times (microPEM has "Errored lines"). This may not be an issue in the database itself, but it will be an issue in the aggregation/summarization


