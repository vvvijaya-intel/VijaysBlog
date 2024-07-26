**Using iGPT to write SQL queries for you:
**

Use igpt.intel.com for following along. 

As of 7/26/2024, iGPT is missing feature(s) to upload a data file and writing a sql query against it. Hopefully the iGPT team adds this feature quickly as it is in their roadmap. 

In the absence of upload capability, we will have to feed the schema (table columns, their datatypes, etc.) info for the agent to get contextual information from the user before it can write proper SQL queries. There are a few ways to accomplish this:

1. Provide a Text Description:
   In this method, you will provide a text description of the schema including table names, column names and data types. For example, 
   Table: employees
   Columns:
    - employee_id (INT, PRIMARY KEY)
    - first_name (VARCHAR(50))
    - last_name (VARCHAR(50))
    - email (VARCHAR(100))
    - salary (DECIMAL(10, 2))
    - department_id (INT, FOREIGN KEY references departments(department_id))
   Table: departments
   Columns:
   - department_id (INT, PRIMARY KEY)
   - department_name (VARCHAR(100))
  
2. Provide a SQL statement that references the table you will be querying against:
   - CREATE TABLE employees (
        employee_id INT PRIMARY KEY,
        first_name VARCHAR(50),
        last_name VARCHAR(50),
        email VARCHAR(100),
        salary DECIMAL(10, 2),
        department_id INT,
        FOREIGN KEY (department_id) REFERENCES departments(department_id)
    );

  - INSERT INTO departments (department_id, department_name) VALUES
    (1, 'Human Resources'),
    (2, 'Engineering');

Now that we have some context information for the agent, let's go paste the above text into the iGPT chat window input and hit Enter. You will now see that the agent sees the table info but is not sure what you are asking it to do. This is indicated by the "If you have a specific question or need assistance with a database operation related to this table, such as creating, reading, updating, or deleting records, please provide more details or a question, and I'll be glad to help you with that." response from the agent. 

Let us now provide a specific instruction to the agent. Type in "write a sql query to count number of people with salary > 60000". You should now see a SQL query for finding the count of employees with salary greater than 60000 in the response. This is just a simple 101 example. Let us move to a more advanced example.

Assume a SQL table with following schema info: 
CREATE TABLE transactions (
    date DATE,
    store_nbr INT,
    transactions INT
);

You want to create a report of the rolling 7 day average number of transactions to identify trends. So we prompt the agent with the query "write sql query to find out the rolling average of transactions for each store and date order by store, date". As expected, we get the following response: 

SELECT 
    date,
    store_nbr,
    transactions,
    AVG(transactions) OVER (
        PARTITION BY store_nbr
        ORDER BY date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS rolling_avg_transactions
FROM 
    transactions
ORDER BY 
    store_nbr, 
    date;



