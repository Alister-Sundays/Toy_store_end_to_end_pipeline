```sql
CREATE OR REPLACE STORAGE INTEGRATION my_gcs_integration
TYPE = EXTERNAL_STAGE
STORAGE_PROVIDER = 'GCS'
ENABLED = TRUE
STORAGE_ALLOWED_LOCATIONS = ('gcs://toy_sales/');

DROP TABLE  TOYSTORE.STAGING.STG_CALENDAR;


-- Set up the defaults
USE WAREHOUSE COMPUTE_WH;
USE DATABASE TOYSTORE;
USE SCHEMA STAGING;

-- Set up an External Stage 
CREATE OR REPLACE STAGE my_gcs_stage
URL = 'gcs://toy_sales/'
STORAGE_INTEGRATION = my_gcs_integration; -- Pre-configured in Snowflake


-- Create our three tables and import the data from S3
CREATE OR REPLACE TABLE stg_calendar
                    (
                     date date);
                    
COPY INTO stg_calendar (date
)
                   from @my_gcs_stage/calendar.csv
                    FILE_FORMAT = (type = 'CSV' skip_header = 1 FIELD_OPTIONALLY_ENCLOSED_BY = '"');
                    

CREATE OR REPLACE TABLE stg_dictionary
                    (table_name string,
                    field string,
                    description string);
                    
COPY INTO stg_dictionary (table_name,field, description)
                   from @my_gcs_stage/data_dictionary.csv
                    FILE_FORMAT = (type = 'CSV' skip_header = 1
                    FIELD_OPTIONALLY_ENCLOSED_BY = '"');
                    

CREATE OR REPLACE TABLE stg_inventory
                    (store_id integer,
                     product_id integer,
                     stock_on_hand integer
                     );
                    
COPY INTO stg_inventory (store_id,product_id,stock_on_hand)
                   from @my_gcs_stage/inventory.csv
                    FILE_FORMAT = (type = 'CSV' skip_header = 1
                    FIELD_OPTIONALLY_ENCLOSED_BY = '"');

CREATE OR REPLACE TABLE stg_products
                    (product_id integer,
                     product_name string,
                     product_category string,
                     product_cost string,
                     product_price string
                     );
                    
COPY INTO stg_products (product_id, product_name, product_category,product_cost,product_price)
                   from @my_gcs_stage/products.csv
                    FILE_FORMAT = (type = 'CSV' skip_header = 1
                    FIELD_OPTIONALLY_ENCLOSED_BY = '"');

CREATE OR REPLACE TABLE stg_sales
                    (sale_id integer,
                     date date,
                     store_id integer,
                     product_id integer,
                     unit integer
                     );
                    
COPY INTO stg_sales (sale_id,date,store_id,product_id,unit)
                   from @my_gcs_stage/sales.csv
                    FILE_FORMAT = (type = 'CSV' skip_header = 1
                    FIELD_OPTIONALLY_ENCLOSED_BY = '"');

CREATE OR REPLACE TABLE stg_stores
                    (store_id integer,
                     store_name string,
                     store_city string,
                     store_location string,
                     store_open_date date
                     );
                    
COPY INTO stg_stores (store_id,store_name, store_city,store_location,store_open_date)
                   from @my_gcs_stage/stores.csv
                    FILE_FORMAT = (type = 'CSV' skip_header = 1
                    FIELD_OPTIONALLY_ENCLOSED_BY = '"');

