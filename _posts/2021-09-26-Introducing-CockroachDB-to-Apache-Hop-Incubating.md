---
title: "Introducing CockroachDB to Apache Hop (Incubating)"
date: 2021-09-26T16:34:30-04:00
categories:
  - blog
  - Hop
tags:
  - CockroachDB
  - Hop
  - ETL
  - JDBC
  - Beginner
  - Introduction
---

In this post we'll find out how we can start development on a ETL pipeline whilst working with CockroachDB.
CockroachDB, the distributed SQL database is not a stranger to a wide variety of 3rd party tools, however there is one tool that enables developers to design, run and debug workflows that everyone can easily use that we have missed. Apache [Hop] is a visual development tool that "aims to facilitate all aspects of data and metadata orchestration".  This is the first in my series on CockroachDB paired with Apache Hop. 

[Hop]: https://hop.apache.org/

---
### Prerequisites: 
- CockroachDB (Pick one)
  - Locally [install] CockroachDB on mac/windows/linux
  - brew install cockroachdb/tap/cockroach
  - Start a free [CockroachCloud] cluster

---
### Objective: 
- Read/write to CockroachDB tables with Apache Hop

---
### TLDR for experienced Hop developers
- CockroachDB is postgres wire compliant, therefore you can setup Postgres as the connection type
- Hop tries to use multiple active portals and CRDB doesn't support this. Therefore avoid limit size in the table input step. 

---
### Tutorial:

1. Lets begin by retrieving apache-hop (incubating)
  ```bash
  # First we need to retrieve apache hop and start the gui.
  curl -O https://dlcdn.apache.org/incubator/hop/0.99/apache-hop-client-0.99-incubating.zip
  # Extract
  tar -xvzf apache-hop-client-0.99-incubating.zip
  cd hop
  # Start the hop developer user interface
  sh hop-gui.sh
  ```
2. You will be presented with a blank screen.
![image](/assets/images/blogpost1/Screenshot2021-09-28at12.27.03.png)
3. Lets setup our connection to CockroachDB. 
  First Click the metadata symbol, this will take you to the metadata pane : ![image](/assets/images/blogpost1/Screenshot2021-09-28at12.29.29.png)
4. To create a new database connection, Double click "Relational Database Connection".
  ![image](/assets/images/blogpost1/Screenshot2021-09-28at12.29.57.png)
5. Give the connection a name, add your connection details and remember to click test to verify the connection! Select PostgreSQL as the connection type. CockroachDB is postgres wire compliant, click [here] to find out more.
  ![image](/assets/images/blogpost1/Screenshot2021-09-28at12.33.53.png)
Click save, then you can exit the dialog ![image](/assets/images/blogpost1/Screenshot2021-09-28at12.36.25.png)
6.  Now lets create your first pipeline, click the + in the left hand side menu and then Pipeline.
![image](/assets/images/blogpost1/Screenshot2021-09-28at13.48.12.png)
7. You will be presented with an empty canvas, left click and a dialog box will appear to add a new step.
![image](/assets/images/blogpost1/Screenshot2021-09-28at13.49.00.png)
8. Filter for "input" and then click on "Table Input".
![image](/assets/images/blogpost1/table-input.png)
9. Click the "Table Input" step. Then click "Edit".
![image](/assets/images/blogpost1/edit.png)
10. In the dialog, input your SQL select query.
![image](/assets/images/blogpost1/Screenshot2021-09-28at13.51.22.png)
11. Don't forget to click "Preview" to check the SQL statement is correct! For this exercise, the CockroachDB example workload [movr] is used. 
![image](/assets/images/blogpost1/Screenshot2021-09-28at14.03.32.png)
12. Now we will add a text file output and write our results to CSV. You must first, add the "Text file output" step. Link the 2 steps together by holding the shift key + left clicking the "Table Input", then drag from "Table Input" to "Text file output". 
![image](/assets/images/blogpost1/Screenshot2021-09-28at14.05.18.png)
13. You should now see a basic pipeline.
![image](/assets/images/blogpost1/Screenshot2021-09-28at14.06.04window.png)
14. Double click the "Text file output" and add a filename, in this example I use a variable, you will need to provide the the variable ${output_dir} when you run the pipeline
![image](/assets/images/blogpost1/variable_text_file_output.png)
15. Click the Fields tab and add fields that you would like to output to CSV. You can use "Get fields" and "Minimal width" to autofill the fields.
![image](/assets/images/blogpost1/csvfields.png)
16. Now save your pipeline ![image](/assets/images/blogpost1/Screenshot2021-09-28at12.36.25.png)
![image](/assets/images/blogpost1/Screenshot2021-09-28at14.08.08.png)
17. Click the play button .
![image](/assets/images/blogpost1/Screenshot2021-09-28at14.10.03.png)
18. Update the output_dir variable in the variables tab and click "Launch".
![image](/assets/images/blogpost1/launch.png)
19. Success! We written some rows from the rides table to CSV.
![image](/assets/images/blogpost1/Screenshot2021-09-28at14.14.45-success.png)
20. You should be able to now see the csv.
  ```bash
  cat /var/root/sample.csv
  ```
![image](/assets/images/blogpost1/Screenshot2021-09-28at14.15.53.png)

---
### Lets read a CSV and write this back to CockroachDB
1. For this, we will use 2 steps. "CSV file input" and "Table output".
![image](/assets/images/blogpost1/overview.png)
2. CSV File output looks like so:
![image](/assets/images/blogpost1/Screenshot2021-09-28at14.20.37.png)
3. Table output, update the target schema and target table.
![image](/assets/images/blogpost1/Screenshot2021-09-28at14.24.12.png)
4. Success! We have written our CSV to a table in CockroachDB.
![image](/assets/images/blogpost1/Screenshot2021-09-28at14.24.31.png)

## Issues encountered

### Multiple active portals when using Limit size

For table inputs, avoid putting the limit size. Otherwise you will receive an error where hop tries to use multiple portals, cockroach does not support this. Simply workaround this by adding a limit clause to the query.

![image](/assets/images/blogpost1/limit.png)

### Error shown by Hop
```Java
2021/09/29 10:25:02 - Table input.0 - Caused by: org.apache.hop.core.exception.HopDatabaseException: 
2021/09/29 10:25:02 - Table input.0 - Error determining value metadata from SQL resultset metadata
2021/09/29 10:25:02 - Table input.0 - ERROR: unimplemented: multiple active portals not supported
2021/09/29 10:25:02 - Table input.0 -   Hint: You have attempted to use a feature that is not yet implemented.
2021/09/29 10:25:02 - Table input.0 - See: https://go.crdb.dev/issue-v/40195/v21.1
```

## Workaround:
```sql
SELECT * FROM movr.rides limit 10;
```
---
# Conclusion

I hope this tutorial was enough for you to get started on development with Apache Hop (Incubating) paired with CockroachDB. Its very easy to develop pipelines with Hop and I'll be developing a series on this subject, so look out for my next article. Thanks for reading!


[install]: https://www.cockroachlabs.com/docs/v21.1/install-cockroachdb-linux 
[CockroachCloud]: https://cockroachlabs.cloud/signup
[movr]: https://www.cockroachlabs.com/docs/stable/cockroach-workload.html#movr-workload
[here]: https://www.cockroachlabs.com/docs/stable/postgresql-compatibility.html
