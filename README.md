# Furniture Products - Amazon Vine Analysis

## Overview of Project
Since your work with Jennifer on the SellBy project was so successful, you’ve been tasked with another, larger project: analyzing Amazon reviews written by members of the paid Amazon Vine program. The Amazon Vine program is a service that allows manufacturers and publishers to receive reviews for their products. Companies like SellBy pay a small fee to Amazon and provide products to Amazon Vine members, who are then required to publish a review.  

In this project, you’ll have access to approximately 50 datasets. Each one contains reviews of a specific product, from clothing apparel to wireless products. You’ll need to pick one of these datasets and use PySpark to perform the ETL process to extract the dataset, transform the data, connect to an AWS RDS instance, and load the transformed data into pgAdmin. Next, you’ll use PySpark, Pandas, or SQL to determine if there is any bias toward favorable reviews from Vine members in your dataset. Then, you’ll write a summary of the analysis for Jennifer to submit to the SellBy stakeholders.
 
## Deliverables: 
This new assignment consists of two technical analysis deliverables and a written report.    
 
1. ***Deliverable 1:*** Perform ETL on Amazon Product Reviews
2. ***Deliverable 2:*** Determine Bias of Vine Reviews 
3. ***Deliverable 3:*** A Written Report on the Analysis [README.md](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis)


## Deliverables:
This new assignment consists of three technical analysis deliverables and a proposal for further statistical study. You’ll submit the following:

* Data Source: `SQL table schema.csv` and `Amazon ETL starter code.csv`
* Data Tools:  `Amazon_Reviews_ETL.ipynb` and `Vine_Review_Analysis.ipynb`.
* Software: `Python 3.9`, `Visual Studio Code 1.50.0`, `Anaconda 4.8.5`, `Jupyter Notebook 6.1.4` and `Pandas`


## Resources and Before Start Notes:

![logo](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis/blob/main/Resources/Images/Vine-Header.png?raw=true)


### Cloud Storage with S3 on AWS
#### Database Versus Data Storage

Data storage holds raw data such as CSVs, Excel files, and JavaScript Object Notation (JSON) files. Think of your own computer file system where you keep a ton of files as data storage. This data doesn't need to be queried and analyzed for business decisions. The files still have structure and can be reviewed, but not nearly as efficiently as a database.

A database contains cleaned, related information in tabular form. This database has been carefully planned and structured so that data can be analyzed efficiently through queries. Doing so comes at a cost of processing data to fit all the rules and structures.

Data storage is a place where large amounts of raw data can be kept without any munging or curating. Data storage allows us to keep data of different types or data we might want to parse in the future.

The benefit of having dedicated data storage is that nothing limits the intake of data. Data can flow in constantly and be saved without having to worry if it fits the criteria of the database. We have seen this with our extract, transform, and load (ETL) process—the data storage can hold raw files, such as CSV or JSON, for different needs.


#### AWS's Simple Storage Service

S3 is Amazon's cloud file storage service that uses key-value pairs. Files are stored on multiple servers and have a high rate of availability of more than 99.9%. To store files, S3 uses buckets, which are similar to folders or directories on your computer. Buckets can contain additional folders and files. Each bucket must have a unique name across all of AWS.

One of S3's perks is its fine-grained control over files. Each file or bucket can have different read and write permissions, which helps regulate what can be done with each file.

S3 is also very scalable—you are not limited to the memory of one computer. As data flows in, more and more can be stored, as opposed to a local computer that is limited by available memory. Additionally, it offers availability—several team members can access massive amounts of data from one central location.


#### PySpark and S3 Stored Data

Since PySpark is a big data tool, it has many ways of reading in files from data storage so that we can manipulate them. We have decided to use S3 as our data storage, so we'll use PySpark for all data processing.

Using PySpark is how we've been reading in our data into Google Colab so far. The format for reading in from S3 is the S3 link, followed by your bucket name, folder by each folder, and then the filename, as follows:

For US East (default region)

template_url = "https://<bucket-name>.s3.amazonaws.com/<folder-name>/<file-name>"

example_url = "https://dataviz-curriculum.s3.amazonaws.com/data-folder/data.csv"
For other regions

template_url = "https://<bucket-name.s3-<region>.amazonaws.com/<folder-name>/<file-name>"

example_url =" https://dataviz-curriculum.s3-us-west-1.amazonaws.com/data-folder/data.csv"


#### PySpark ETL (Extract, Transform and Load)

Let's run through a mock scenario using two different types of raw data stored in S3. Our goal is to get this raw data from S3 into an RDS database.

Assume your company already has three tables set up in the RDS database and would like to get the raw data from S3 into the database. Create a new database in pgAdmin called "my_data_class_db." We'll have it represent the company database by first running the following schema in pgAdmin for our RDS:

````sql
-- Create Active User Table
CREATE TABLE active_user (
 id INT PRIMARY KEY NOT NULL,
 first_name TEXT,
 last_name TEXT,
 username TEXT
);

CREATE TABLE billing_info (
 billing_id INT PRIMARY KEY NOT NULL,
 street_address TEXT,
 state TEXT,
 username TEXT
);

CREATE TABLE payment_info (
 billing_id INT PRIMARY KEY NOT NULL,
 cc_encrypted TEXT
);
````

**NOTE**
Table creation is not part of the ETL process. We're creating the tables to represent a pre-established database you need for the raw data. In a real-life situation, databases will already have a well-defined schema and tables for you, as the engineer, to process data into.

Start with creating a new notebook, installing Spark:

````python
import os
# Find the latest version of spark 3.0  from http://www-us.apache.org/dist/spark/ and enter as the spark version
# For example:
# spark_version = 'spark-3.0.2'
spark_version = 'spark-3.<enter version>'
os.environ['SPARK_VERSION']=spark_version

# Install Spark and Java
!apt-get update
!apt-get install openjdk-11-jdk-headless -qq > /dev/null
!wget -q http://www-us.apache.org/dist/spark/$SPARK_VERSION/$SPARK_VERSION-bin-hadoop2.7.tgz
!tar xf $SPARK_VERSION-bin-hadoop2.7.tgz
!pip install -q findspark

# Set Environment Variables
import os
os.environ["JAVA_HOME"] = "/usr/lib/jvm/java-11-openjdk-amd64"
os.environ["SPARK_HOME"] = f"/content/{spark_version}-bin-hadoop2.7"

# Start a SparkSession
import findspark
findspark.init()
````

We'll use Spark to write directly to our Postgres database. But in order to do so, there are few more lines of code we need.

First, enter the following code to download a Postgres driver that will allow Spark to interact with Postgres:

````python
!wget https://jdbc.postgresql.org/download/postgresql-42.2.16.jar
````
You should get a message containing the words "HTTP request sent, awaiting response… 200 OK," indicating that your request was processed without a problem.
Then, start a Spark session with an additional option that adds the driver to Spark:

````python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("CloudETL").config("spark.driver.extraClassPath","/content/postgresql-42.2.16.jar").getOrCreate()
````

We have performed the first two steps of the ETL process before with PySpark, so let's quickly review those.


**Extract**
We can connect to data storage, then extract that data into a DataFrame. We'll do this on two datasets, and be sure to replace the bucket name with one of your own.

We'll start by importing SparkFiles from PySpark into our notebook. This will allow Spark to add a file to our Spark project.

Next, the file is read in with the read method and combined with the csv() method, which pulls in our CSV stored in SparkFiles and infers the schema. SparkFiles.get() will have Spark retrieve the specified file, since we are dealing with a CSV.  The "," is the chosen separator, and we will have Spark determine the head for us. Enter the following code:

````python
# Read in data from S3 Buckets
from pyspark import SparkFiles
url ="https://YOUR-BUCKET-NAME.s3.amazonaws.com/user_data.csv"
spark.sparkContext.addFile(url)
user_data_df = spark.read.csv(SparkFiles.get("user_data.csv"), sep=",", header=True, inferSchema=True)
````

Finally, an action is called to show the first 10 runs and confirm our data extraction by entering the following code:

````python
# Show DataFrame
user_data_df.show()
````

Repeat a similar process to load in the other data. Enter the code:

````python
url ="https://YOUR-BUCKET-NAME.s3.amazonaws.com/user_payment.csv"
spark.sparkContext.addFile(url)
user_payment_df = spark.read.csv(SparkFiles.get("user_payment.csv"), sep=",", header=True, inferSchema=True)

# Show DataFrame
user_payment_df.show()
````

**Transform**
Now that the raw data stored in S3 is available in a PySpark DataFrame, we can perform our transformations.

First, join the two tables:

![d1](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis/blob/main/Resources/Images/data-16-9-1-1-Join-Two-DataFrames.png)


Next, drop any rows with null or "not a number" (NaN) values:

![d1](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis/blob/main/Resources/Images/data-16-9-1-2-Drop-Null-Values.png)

Filter for active users:

![d1](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis/blob/main/Resources/Images/data-16-9-1-3-Load-Sql-Function.png)

Next, select columns to create three different DataFrames that match what is in the AWS RDS database. Create a DataFrame to match the active_user table:

![d1](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis/blob/main/Resources/Images/data-16-9-1-4-Create-User-Dataframe-Active-User-Table.png)

Next, create a DataFrame to match the billing_info table:

![d1](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis/blob/main/Resources/Images/data-16-9-1-5-Create-User-DataFrame-Match-Billing-Info-Table.png)

Finally, create a DataFrame to match the payment_info table:

![d1](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis/blob/main/Resources/Images/data-16-9-1-6-Create-User-DataFrame-Match-Payment.png)

Once our data has been transformed to fit the tables in our database, we're ready to move on to the "Load" step.

**Load**
The final step is to get our transformed raw data into our database. PySpark can easily connect to a database to load the DataFrames into the table. First, we'll do some configuration to allow the connection with the following code:

````python
# Configure settings for RDS
mode = "append"
jdbc_url="jdbc:postgresql://<connection string>:5432/<database-name>"
config = {"user":"postgres",
          "password": "<password>",
          "driver":"org.postgresql.Driver"}
````

You'll need to provide your username and password, and also supply the AWS server name where `<connection string>` is located in the code above. To find it in PgAdmin, right-click AWS in the Server directory listing on the left side of PgAmin, and then select Properties in the drop-down menu. Select the Connection tab in the window that opens, and then select the address in the Host name/address field. Copy that address and paste it in place of `<connection string>`.


Let's further break down what's happening here:

* `mode` is what we want to do with the DataFrame to the table, such as `overwrite` or `append`. We'll append to the current table because every time we run this ETL process, we'll want more data added to our database without removing any.
* The `jdbc_url` is the connection string to our database.
- Replace `<connection string>` with the endpoint connection url found from your AWS RDS console.
- Replace `<database name>` with the name of your database you wish to connect to.
* A dictionary of configuration that includes the `user`, `password`, and `driver` to what type of database is being used.
- The `user` field is the username for your database, which should be `postgres` if you followed with the creation of the RDS instance. Otherwise, enter the one you created.
- The `password` would be the password you created when making the RDS instance.

**NOTE**
> If you forget anything like the name of the database or user name you can check on pgAdmin for these values. Be sure that you are entering the name of the database and not the name of your server in the connection string.

The cleaned DataFrames can then be written directly to our database by using the `.write.jdbc` method that takes in the parameters we set:

The connection string stored in `jdbc_url` is passed to the URL argument.
The corresponding name of the table we are writing the DataFrame to.
The mode we're using, which is "append."
The connection configuration we set up passed to the properties.
The code is as follows:

````python
# Write DataFrame to active_user table in RDS
clean_user_df.write.jdbc(url=jdbc_url, table='active_user', mode=mode, properties=config)

# Write dataframe to billing_info table in RDS
clean_billing_df.write.jdbc(url=jdbc_url, table='billing_info', mode=mode, properties=config)

# Write dataframe to payment_info table in RDS
clean_payment_df.write.jdbc(url=jdbc_url, table='payment_info', mode=mode, properties=config)

````
Let's wrap up by double-checking our work and running queries in pgAdmin on our database to confirm that the load did exactly what we wanted:

````sql
-- Query database to check successful upload
SELECT * FROM active_user;
SELECT * FROM billing_info;
SELECT * FROM payment_info;
````

Nice work! You now have enough knowledge and practice with PySpark and AWS to begin your client project.


> Let's move on!

# Deliverable 1:  
## Perform ETL on Amazon Product Reviews 
### Deliverable Requirements:

Using the cloud ETL process, you’ll create an AWS RDS database with tables in pgAdmin, pick a dataset from the [Amazon Review datasets](https://s3.amazonaws.com/amazon-reviews-pds/tsv/index.txt), and extract the dataset into a DataFrame. You'll transform the DataFrame into four separate DataFrames that match the table schema in pgAdmin. Then, you'll upload the transformed data into the appropriate tables and run queries in pgAdmin to confirm that the data has been uploaded.

> To Deliver. 

**Follow the instructions below:**

1. From the following [Amazon Review datasets](https://s3.amazonaws.com/amazon-reviews-pds/tsv/index.txt), pick a dataset that you would like to analyze. All the datasets have the same schemata, as shown in this image:

![d1](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis/blob/main/Resources/Images/1.png)

2. Create a new database with Amazon RDS just as you did in this module.

3. In pgAdmin, create a new database in your Amazon RDS server that you just create.

4. Download the `challenge_schema.sql` file to your computer.

5. In pgAdmin, run a new query to create the tables for your new database using the code from the `challenge_schema.sql` file.

- After you run the query, you should have the following four tables in your database: customers_table, products_table, review_id_table, and vine_table.

6. Download the `Amazon_Reviews_ETL_starter_code.ipynb` file, then upload the file as a Google Colab Notebook, and rename it `Amazon_Reviews_ETL`.

**NOTE**
> If you try to open the `Amazon_Reviews_ETL_starter_code.ipynb` with jupyter notebook it will give you an error.

7. First **extract** one of the review datasets, then create a new DataFrame.
8. Next, follow the steps below to **transform** the dataset into four DataFrames that will match the schema in the pgAdmin tables:

**NOTE**
> Some datasets have a large number of rows, which will affect the time it takes to complete the following steps.

**The customers_table DataFrame**
To create the `customers_table`, use the code in the `Amazon_Reviews_ETL_starter_code.ipynb` file and follow the steps below to aggregate the reviews by `customer_id`.

* Use the `groupby()` function on the customer_id column of the DataFrame you created in Step 6.
* Count all the customer ids using the `agg()` function by chaining it to the `groupby()` function. After you use this function, a new column will be created, `count(customer_id)`.
* Rename the `count(customer_id)` column using the `withColumnRenamed()` function so it matches the schema for the `customers_table` in pgAdmin.
* The final `customers_table` DataFrame should look like this:

![d1](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis/blob/main/Resources/Images/2.png)


**The products_table DataFrame**
To create the `products_table`, use the `select()` function to select the `product_id` and `product_title`, then drop duplicates with the `drop_duplicates()` function to retrieve only unique `product_ids`. Refer to the code snippet provided in the `Amazon_Reviews_ETL_starter_code.ipynb` file for assistance.

The final `products_table` DataFrame should look like this:

![d1](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis/blob/main/Resources/Images/3.png)

**The review_id_table DataFrame**
To create the `review_id_table`, use the `select()` function to select the columns that are in the `review_id_table` in [pgAdmin](https://docs.github.com/en/github/authenticating-to-github/removing-sensitive-data-from-a-repository), and convert the review_date column to a date using the code snippet provided in the `Amazon_Reviews_ETL_starter_code.ipynb` file.

The final `review_id_table` DataFrame should look like this:

![d1](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis/blob/main/Resources/Images/4.png)

**The vine_table DataFrame**
To create the `vine_table`, use the `select()` function to select only the columns that are in the `vine_table` in [pgAdmin](https://docs.github.com/en/github/authenticating-to-github/removing-sensitive-data-from-a-repository).

The final `vine_table` DataFrame should look like this:

![d1](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis/blob/main/Resources/Images/5.png)

**Load the DataFrames into pgAdmin**
1. Make the connection to your AWS RDS instance.
2. Load the DataFrames that correspond to tables in pgAdmin.
3. In pgAdmin, run a query to check that the tables have been populated.

**IMPORTANT**
> Before uploading anything to GitHub be sure to remove all sensitive information such as passwords and connection strings. If you have accidentally done so already see this link (Links to an external site.) for more information.

When you’re done, export your `Amazon_Reviews_ETL` Google Colab Notebook as an ipynb file, and save it to your Amazon_Vine_Analysis GitHub repository.

**NOTE**
Uploading each DataFrame can take up to 10 minutes or longer, so it’s a good idea to double-check your work before uploading. If you have problems uploading your work, you may have to shut down the pgAdmin server and restart. Alternatively, you may have to delete the tables and create them again, then re-run your `Amazon_Reviews_ETL` Google Colab Notebook.

**IMPORTANT**
Be sure that you don’t leave your RDS instance up too long. Try to get all your work for Deliverable 1 done in one sitting, then shut down your instance. Please consult the AWS clean-up videos for more information about shutting down your RDS instance. You will not be graded on anything contained strictly in your RDS, so be sure to shut it down.

#### Deliverable 1 Requirements
You will earn a perfect score for Deliverable 1 by completing all requirements below:

* The `Amazon_Reviews_ETL.ipynb` file does the following:
    * An Amazon Review dataset is extracted as a DataFrame
    * The extracted dataset is transformed into four DataFrames with the correct columns
    * All four DataFrames are loaded into their respective tables in pgAdmin



### DELIVERABLE RESULTS:

**Helpful Reviews (All) with 5 Star:**  
For all reviews and "helpful" reviews, **around half of the ratings are 5 Star**, which indicates that the Vine programs tend to give 5 Stars over any other rating.
![d1](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis/blob/main/Resources/Images/r5.png)

**Percentage of Vine Reviews are 5-star:**  
For all the Vine Reviews, we found almost the same, a little more lower ratings than 5 Star.
![d1](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis/blob/main/Resources/Images/r6.png)  


**Percentage of Non-Vine Reviews are 5-star:** 
In General, the non-Vine reviews is higher of 5 Stars on non-Vine reviews than 5 Star Vine.  
![d1](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis/blob/main/Resources/Images/r7.png)

**Vine Review vs. Non-Vine Review**:   
For the entire Furniture product review file, the majority has a small Amazon Vine review:   
![d1](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis/blob/main/Resources/Images/r1.png)

Now, applying the same analysis over smaller dataset, with "helpful" reviews, we faound an average percentage from the Vine program:  
![d1](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis/blob/main/Resources/Images/r2.png)

**5 Star Reviews Vine vs Non-Vine:** 
For the entire review dataset, we found a small 5 Star reviews from Vine reviews, **around 0.3%**  
![d1](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis/blob/main/Resources/Images/r3.png)

By Filtering the "helpful" reviews only, we saw and found a light difference; **a lower 1%** of the 5 Star review from Vine.  
![d1](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis/blob/main/Resources/Images/r4.png)


### SUMMARY


1. The majority of reviews for Furniture product are almost nothing or lower results from Vine participants: **99.6% are Non-Vine**.  
2. And overall of all 5 Star reviews are also the same as the Furniture, all are from Vine participants: **99.7% of all 5-star reviews are non-Vine**.
3. But we need to highlight that not all of the 5 Star reviews are coming from Vine participants.


### RECOMMENDATIONS:
Below some recommendations to follow:

1. The Amazon Vine Analysis provide a favorable dataset on the 5-star rating.

2. In addition, we found that much data isn't Vine reviews over specific products, that we could minimize the resluts and create a different dataset on just Vine products.

> In addition, 

The analysis gave us that **1/4 are Vine Reviews**
  
![d1](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis/blob/main/Resources/Images/r8.png)  

Specific Product provide an average of **57% 5 Star reviews**  

![d1](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis/blob/main/Resources/Images/r9.png)  

For the majority of Vine Reviews, the analysis provide a **49% of 5 Star reviews**   

![d1](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis/blob/main/Resources/Images/r10.png)   

And for the majority of the non-Vine Reviews, the analysis provide a **60% of 5 Star reviews**

![d1](https://github.com/emmanuelmartinezs/Amazon_Vine_Analysis/blob/main/Resources/Images/r11.png) 
