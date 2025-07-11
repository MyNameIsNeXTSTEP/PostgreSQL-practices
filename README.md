# PostgreSQL-practices
PostgreSQL personal cheatsheets, pratical examples and solutions, application layer integration and usage

## Navigation
* [CheatSheets](#cheatsheets)
  * [Constraints](#constraints)
    * [Get constraints of a table](#get-constraints-of-a-table)
    * [Drop constraints of a table](#drop-constraints-of-a-table)
    * [List all constraints of a scheme](#list-all-constraints-of-a-scheme)
  * [Stored procedures](#stored-procedures)
    * [Create a stored procedure](#create-a-stored-procedure)
    * [Get stored procedures names](#get-stored-procedures-names)
    * [Get the inner SQL code of a procedure](#get-the-inner-sql-code-of-a-procedure)
    * [Get the full creation code of a procedure](#get-the-full-creation-code-of-a-procedure)
  * [Stored procedures](#stored-procedures)
* [Extensions](#extensions)
  * [pg_cron](#pg_cron)
    * [List all scheduled jobs](#list-all-scheduled-jobs)
    * [Schedule a cronjob call](#schedule-a-cronjob-call)
    * [Unschedule a job](#unschedule-a-job)
    * [View completed jobs](#view-completed-jobs)
    * [Installing via Docker](#installing-via-docker)


## CheatSheets

### Constraints
#### Get constraints of a table
```sql
SELECT constraint_name, table_name, column_name, ordinal_position
FROM information_schema.key_column_usage WHERE table_name = 'tableName';
```
![alt text](images/image1.png)

#### Drop constraints of a table
```sql
ALTER TABLE "table_name" DROP CONSTRAINT "constraint_name";
```

#### List all constraints of a scheme
```sql
SELECT
 tc.constraint_name,
 tc.constraint_type,
 kcu.column_name,
 ccu.table_name AS foreign_table_name,
 ccu.column_name AS foreign_column_name
FROM information_schema.table_constraints tc
JOIN information_schema.key_column_usage kcu
    ON tc.constraint_name = kcu.constraint_name
LEFT JOIN information_schema.constraint_column_usage ccu
    ON ccu.constraint_name = tc.constraint_name
```

### Stored procedures
#### Create a stored procedure
```sql
CREATE PROCEDURE clear_expired_sessions()
LANGUAGE SQL
AS $$
  DELETE FROM "Session" where expires <= Now();
$$;
```

#### Get stored procedures names
```sql
SELECT  nspname, proname 
FROM    pg_catalog.pg_namespace  
JOIN    pg_catalog.pg_proc  
ON      pronamespace = pg_namespace.oid 
WHERE   nspname = 'public' --- specify the schema anme
ORDER BY Proname;
```
![alt text](images/image2.png)

#### Get the inner SQL code of a procedure
```sql
SELECT prosrc FROM pg_proc WHERE proname = 'procedure_name';
```
![alt text](images/image3.png)

#### Get the full creation code of a procedure
```sql
SELECT pg_get_functiondef((
  SELECT oid FROM pg_proc
  WHERE proname = 'procedure_name'
));
```
![alt text](images/image4.png)

## Extensions

### pg_cron
Link to the extension - https://github.com/citusdata/pg_cron
#### List all scheduled jobs
```sql
SELECT * FROM cron.job;
```

#### Schedule a cronjob call
```sql
SELECT cron.schedule('call-procedute-clear_expired_sessions', '1 * * * *', 'CALL clear_expired_sessions()'); --- every one minute
```
> Note, that when a databse is inactive (shutdown or else) - the jobs are not invoked,
> therefore for a certain situations need to use some other mechanisms to control a job call or deal with unhandled data by it.

#### Unschedule a job
```sql
-- By using a `jobname`
SELECT cron.unschedule('sql-job-name');

-- By usin a `jobid`
SELECT cron.unschedule(1);
```

#### View completed jobs
```sql
SELECT * FROM cron.job_run_details ORDER by start_time DESC LIMIT 5;
```

#### Installing via Docker
1. Prepare the [Dockerfile](/extensions/pg_cron/Dockerfile)
2. Create extension and grant access
```sql
CREATE EXTENSION pg_cron; --- connect extension to the db as a root
GRANT USAGE ON SCHEMA cron TO postgres; -- create as a specified user
```
