1. Create the Backup Table
First, ensure you have a backup table with the same structure as the original table. This backup table will store the data that is older than 6 months.

Example Table Structure:

Assuming your original table is employees and you have a column hire_date:

sql
Copy code
CREATE TABLE employees_backup AS
SELECT *
FROM employees
WHERE 1=0;  -- Creates an empty table with the same structure as 'employees'
2. Create a PL/SQL Procedure to Move Data
Write a PL/SQL procedure that transfers data older than 6 months from the original table to the backup table and then deletes the old data from the original table.

Procedure:

sql
Copy code
CREATE OR REPLACE PROCEDURE move_old_employees IS
BEGIN
    -- Insert old data into the backup table
    INSERT INTO employees_backup
    SELECT *
    FROM employees
    WHERE hire_date < ADD_MONTHS(SYSDATE, -6);

    -- Delete old data from the original table
    DELETE FROM employees
    WHERE hire_date < ADD_MONTHS(SYSDATE, -6);

    -- Commit the transaction
    COMMIT;
END;
/
3. Schedule the Job Using DBMS_SCHEDULER
Create a scheduled job to execute this procedure periodically (e.g., monthly).

Scheduled Job:

sql
Copy code
BEGIN
    DBMS_SCHEDULER.create_job (
        job_name        => 'MOVE_OLD_EMPLOYEES_JOB',
        job_type        => 'PLSQL_BLOCK',
        job_action      => 'BEGIN move_old_employees; END;',
        start_date      => SYSTIMESTAMP,
        repeat_interval => 'FREQ=MONTHLY; BYMONTHDAY=1; BYHOUR=0; BYMINUTE=0; BYSECOND=0',
        enabled         => TRUE
    );
END;
/
Explanation:

job_name: The name of the job.
job_type: Specifies that this job runs a PL/SQL block.
job_action: The PL/SQL block to execute, which calls the move_old_employees procedure.
start_date: When to start the job. SYSTIMESTAMP starts the job immediately.
repeat_interval: Defines the frequency of the job. In this case, it runs monthly on the first day of the month at midnight.
4. Verify the Scheduled Job
After creating the job, you can verify its status and details using the following queries:

Check Job Status:

sql
Copy code
SELECT job_name, enabled, last_start_date, next_run_date
FROM DBA_SCHEDULER_JOBS
WHERE job_name = 'MOVE_OLD_EMPLOYEES_JOB';
Check Job Runs:

sql
Copy code
SELECT job_name, log_date, status
FROM DBA_SCHEDULER_JOB_RUN_DETAILS
WHERE job_name = 'MOVE_OLD_EMPLOYEES_JOB';
Summary
By following these steps, you set up a scheduled job that automatically moves data older than 6 months from an original table to a backup table. This approach ensures that your data is managed efficiently and archived as needed.
