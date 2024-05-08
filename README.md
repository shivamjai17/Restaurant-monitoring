# Restaurant Tracking & Monitoring System

<i>NOTE: To run this backend solution on your system, follow the steps:
1. Open your terminal and change the directory to the desired folder.
2. Create a Python virtual environment using your terminal.
3. Clone the repository into the desired directory.
4. Download the required modules  (pip install -r requirements.txt)
5. Execute the run.py file.
6. Use Postman/Thunderclient, etc. for executing API requests and responses. </i>


<b><u>Problem Statement</u>: </b><i> several restaurants in the United States need to monitor if the store is online or not. All restaurants are supposed to be online during their business hours. Due to some unknown reasons, a store might go inactive for a few hours. Restaurant owners want to get a report of how often this happened in the past.</i>   

<b><u>Aim</u>: </b>To develop optimized and efficient solutions by building backend APIs that will help restaurant owners achieve this goal. 

<b><u>About the Data</u>: </b> We have 3 sources of data --
1. We poll every store roughly every hour and have data about whether the store was active or not in a CSV.  The CSV has 3 columns (`store_id, timestamp_utc, status`) where status is active or inactive.  All timestamps are in **UTC**<br>
2. We have the business hours of all the stores - the schema of this data is `store_id, dayOfWeek(0=Monday, 6=Sunday), start_time_local, end_time_local`<br>
    1. These times are in the **local time zone**<br>
    2. If data is missing for a store, assume it is open 24*7<br>
3. Timezone for the stores - schema is `store_id, timezone_str`<br>
    1. If data is missing for a store, assume it is America/Chicago<br>
    2. This is used so that data sources 1 and 2 can be compared against each other. <br>


<b><u>System Requirements</u>: </b>
- Do not assume that this data is static and precompute the answers as this data will keep getting updated every hour.
- You need to store these CSVs in a relevant database and make API calls to get the data.
  (Currently, SQLite has been used as a lightweight database, however, extended implementation would involve PostgreSQL to leverage the power of multi-threading and caching)


<b><u>Data Output Requirements</u>: </b>
Generate a report to the user with the schema:
`store_id, uptime_last_hour(in minutes), uptime_last_day(in hours), update_last_week(in hours), downtime_last_hour(in minutes), downtime_last_day(in hours), downtime_last_week(in hours)` 

1. Uptime and downtime should only include observations within business hours. 
2. You need to extrapolate uptime and downtime based on the periodic polls we have ingested, to the entire time interval.
    i. eg, business hours for a store are 9 AM to 12 PM on Monday
        a. we only have 2 observations for this store on a particular date (Monday) in our data at 10:14 AM and 11:15 AM.
        b. we need to fill the entire business hours interval with uptime and downtime from these 2 observations based on some sane interpolation logic.


<b><u>API requirement</u>: </b> We build two APIs for triggering & generating reports and one API for data import and caching. 
 
1. <i><b>/trigger_report</b></i> endpoint that will trigger report generation from the data provided (stored in DB)<br>
        1. No input <br>
        2. Output - report_id (random string) <br>
        3. report_id will be used for polling the status of report completion<br>
2. <i><b>/get_report</b></i> endpoint that will return the status of the report or the csv<br>
        1. Input - report_id<br>
        2. Output<br>
            - if report generation is not complete, return “Running” as the output<br>
            - if report generation is complete, return “Complete” along with the CSV file with the schema described above.<br>
3. <i><b>/import_data</b></i> endpoint that is responsible for importing data from CSV to the database and caching the following data for faster in-memory access


