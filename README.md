
# call / Consume REST API and Insert its JSON Restonse into Oracle Table

This repository provides a PL/SQL solution to consume a REST API, parse the JSON response, and insert the data into an Oracle database table.

## Overview

The process involves:
1. Making a REST API request using `UTL_HTTP`.
2. Parsing the JSON response using `JSON_TABLE`.
3. Inserting the parsed data into an Oracle table.

## Table Structure

Before running the procedure, you must create a table to store the data returned by the API.

### Table Schema

```sql
CREATE TABLE api_data (
    name VARCHAR2(20),
    dt_server VARCHAR2(200),
    latitude VARCHAR2(200),
    longitude VARCHAR2(100),
    altitude_no VARCHAR2(20),
    angle_no VARCHAR2(20),
    speed_no VARCHAR2(20)
);
```
## PL/SQL Procedure to Call REST API

1. Below is the PL/SQL procedure that:

2. Sends a GET request to the REST API.

3. Parses the JSON response using JSON_TABLE.

Inserts the parsed data into the api_data table.

## PL/SQL Code

- Call REST API and JSON Response into Page Item
  
 Call rest api ```
   
   declare 
   vResult clob;

   begin
        vResult := APEX_WEB_SERVICE.make_rest_request(
                    p_url         => 'https://track.autonemogps.com/api/api.php?api=user&key=F084EFA9C2C0536E212CD63E23A4B4A5&cmd=OBJECT_GET_LOCATIONS,862292053230462;862292053225637',
                    p_http_method => 'GET'
                    ,p_wallet_path => 'file:/u02/app/oracle/wallets/ssl_wallet'
                    ,p_wallet_pwd  => ''
        );


  --dbms_output.put_line('Token:'||vResult);

    :P1_API_RESPONSE := vResult;

   end;

``

## JSON Response (:P1_API_RESPONSE) insert into Oracle Table

- insert data to Oracle Table

FOR rec IN (
        SELECT * FROM JSON_TABLE(:P1_API_RESPONSE, '$.*' 
            COLUMNS (
                name VARCHAR2(20) PATH '$.name',
                dt_server VARCHAR2(200) PATH '$.dt_server',
                latitude VARCHAR2(200) PATH '$.lat',
                longitude VARCHAR2(100) PATH '$.lng',
                altitude_no VARCHAR2(20) PATH '$.altitude',
                angle_no VARCHAR2(20) PATH '$.angle',
                speed_no VARCHAR2(20) PATH '$.speed'
            )
        )
    ) LOOP
        -- Insert the parsed data into the table
        INSERT INTO api_data (
            name,
            dt_server,
            latitude,
            longitude,
            altitude_no,
            angle_no,
            speed_no
        )
        VALUES (
            rec.name,
            rec.dt_server,
            rec.latitude,
            rec.longitude,
            rec.altitude_no,
            rec.angle_no,
            rec.speed_no
        );
    END LOOP;

    -- Commit the transaction
    COMMIT;
    DBMS_OUTPUT.put_line('Data inserted successfully.');
    
EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.put_line('Error: ' || SQLERRM);
        ROLLBACK;
END;




-- 

 # Thank you
 ## Sanjay Sikder
