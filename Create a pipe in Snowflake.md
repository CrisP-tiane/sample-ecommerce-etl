# Create a pipe in Snowflake

Now that your AWS and Snowflake accounts have the right security conditions, complete Snowpipe 
setup with S3 event notifications.

### Create a Database, Table, Stage, and Pipe
On a fresh Snowflake web console worksheet, use the commands below to create the objects needed for Snowpipe ingestion.

#### Create Database

```snowflake
create or replace database S3_db;
```
Execute the above command to create a database called **S3_db**.

#### Create Table

This command will make a table by the name of ‘S3_table' on the S3_db database.

```snowflake
create or replace table S3_table(files string);
```

#### Create Stage

```snowflake
use schema S3_db.public;
create or replace stage S3_stage
  url = ('s3://<bucket>/<path>/')
  storage_integration = S3_role_integration;
```
To make the external stage needed for our S3 bucket, use this command. 
Be mindful to replace the `<bucket>` and `<path>` with your S3 bucket name and file path.

#### Create Pipe

The magic of automation is in the create pipe parameter `auto_ingest=true`. With auto_ingest set to true, data stagged 
will automatically integrate into your database.

```snowflake
create or replace pipe S3_db.public.S3_pipe auto_ingest=true as
  copy into S3_db.public.S3_table
  from @S3_db.public.S3_stage;
```
### Configure Snowpipe User Permissions

To ensure the Snowflake user associated with executing the Snowpipe actions had sufficient permissions, create a unique 
role to manage Snowpipe security privileges. Do not employ the user account you're currently utilizing, instead create a 
new user to assign to Snowpipe within the web console.

```snowflake
-- Create Role
use role securityadmin;
create or replace role S3_role;

-- Grant Object Access and Insert Permission
grant usage on database S3_db to role S3_role;
grant usage on schema S3_db.public to role S3_role;
grant insert, select on S3_db.public.S3_table to role S3_role;
grant usage on stage S3_db.public.S3_stage to role S3_role;

-- Bestow S3_pipe Ownership
grant ownership on pipe S3_db.public.S3_pipe to role S3_role;

-- Grant S3_role and Set as Default
grant role S3_role to user <username>;
alter user <username> set default_role = S3_role;
```
