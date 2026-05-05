---
title: "Share and publish your Snowflake data to AWS Data Exchange using Amazon Redshift data sharing"
url: "https://aws.amazon.com/blogs/big-data/share-and-publish-your-snowflake-data-to-aws-data-exchange-using-amazon-redshift-data-sharing/"
date: "Thu, 17 Nov 2022 16:43:32 +0000"
author: "Raks Khare"
feed_url: "https://aws.amazon.com/blogs/big-data/category/analytics/aws-data-exchange/feed/"
---
<p><a href="http://aws.amazon.com/redshift" rel="noopener" target="_blank">Amazon Redshift</a> is a fully managed, petabyte-scale data warehouse service in the cloud. You can start with just a few hundred gigabytes of data and scale to a petabyte or more. Today, tens of thousands of AWS customers—from Fortune 500 companies, startups, and everything in between—use Amazon Redshift to run mission-critical business intelligence (BI) dashboards, analyze real-time streaming data, and run predictive analytics. With the constant increase in generated data, Amazon Redshift customers continue to achieve <a href="https://aws.amazon.com/redshift/customer-success/" rel="noopener" target="_blank">successes</a> in delivering better service to their end-users, improving their products, and running an efficient and effective business.</p> 
<p>In this post, we discuss a customer who is currently using Snowflake to store analytics data. The customer needs to offer this data to clients who are using Amazon Redshift via <a href="https://aws.amazon.com/data-exchange/" rel="noopener" target="_blank">AWS Data Exchange</a>, the world’s most comprehensive service for third-party datasets. We explain in detail how to implement a fully integrated process that will automatically ingest data from Snowflake into Amazon Redshift and offer it to clients via AWS Data Exchange.</p> 
<h2>Overview of the solution</h2> 
<p>The solution consists of four high-level steps:</p> 
<ol> 
 <li>Configure Snowflake to push the changed data for identified tables into an <a href="http://aws.amazon.com/s3" rel="noopener" target="_blank">Amazon Simple Storage Service</a> (Amazon S3) bucket.</li> 
 <li>Use a custom-built <a href="https://github.com/aws-samples/amazon-redshift-infrastructure-automation/tree/main/Redshift-Loader" rel="noopener" target="_blank">Redshift Auto Loader</a> to load this Amazon S3 landed data to Amazon Redshift.</li> 
 <li>Merge the data from the change data capture (CDC) S3 staging tables to Amazon Redshift tables.</li> 
 <li>Use Amazon Redshift data sharing to license the data to customers via AWS Data Exchange as a public or private offering.</li> 
</ol> 
<p>The following diagram illustrates this workflow.</p> 
<h2><img alt="Solution Architecture Diagram" class="alignnone size-full wp-image-36588" height="766" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/10/27/image001-9.jpg" width="1379" /></h2> 
<h2>Prerequisites</h2> 
<p>To get started, you need the following prerequisites:</p> 
<ul> 
 <li>A Snowflake account in the same Region as your Amazon Redshift cluster.</li> 
 <li>An S3 bucket. Refer to <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/creating-bucket.html" rel="noopener" target="_blank">Create your first S3 bucket</a> for more details.</li> 
 <li>An Amazon Redshift cluster with encryption enabled and an <a href="http://aws.amazon.com/iam" rel="noopener" target="_blank">AWS Identity and Access Management</a> (IAM) role with permission to the S3 bucket. See <a href="https://docs.aws.amazon.com/redshift/latest/gsg/rs-gsg-launch-sample-cluster.html" rel="noopener" target="_blank">Create a sample Amazon Redshift cluster</a> and <a href="https://docs.aws.amazon.com/redshift/latest/dg/c-getting-started-using-spectrum-create-role.html" rel="noopener" target="_blank">Create an IAM role for Amazon Redshift</a> for more details.</li> 
 <li>A database schema from Snowflake to Amazon Redshift that is migrated using the <a href="https://aws.amazon.com/dms/schema-conversion-tool/" rel="noopener" target="_blank">AWS Schema Conversion Tool</a> (AWS SCT). For more information, refer to <a href="https://aws.amazon.com/blogs/big-data/accelerate-snowflake-to-amazon-redshift-migration-using-aws-schema-conversion-tool/" rel="noopener" target="_blank">Accelerate Snowflake to Amazon Redshift migration using AWS Schema Conversion Tool</a>.</li> 
 <li>An IAM role and external Amazon S3 stage for Snowflake access to the S3 bucket you created earlier. For instructions, refer to <a href="https://docs.snowflake.com/en/user-guide/data-load-s3-config.html" rel="noopener" target="_blank">Configuring Secure Access to Amazon S3</a>. Name this external stage unload_to_s3, pointing to the s3-redshift-loader-source folder of the target S3 bucket. It will be referenced in COPY commands later in this post for offloading the data to Amazon S3. Once created, you should see an external stage created as shown in the following screenshot.<br /> <img alt="pre-req-1" class="alignnone size-full wp-image-36836" height="524" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/01/Screen-Shot-2022-11-01-at-6.07.59-PM.png" style="margin: 10px 0px 10px 0px;" width="2948" /></li> 
 <li>You must be a registered provider on AWS Data Exchange. For more information, see <a href="https://docs.aws.amazon.com/data-exchange/latest/userguide/providing-data-sets.html" rel="noopener" target="_blank">Providing data products on AWS Data Exchange</a>.</li> 
</ul> 
<h2>Configure Snowflake to track the changed data and unload it to Amazon S3</h2> 
<p>In Snowflake, identify the tables that you need to replicate to Amazon Redshift. For the purpose of this demo, we use the data in the <code class="lang-sql">TPCH_SF1</code> schema’s <code class="lang-sql">Customer</code>, <code class="lang-sql">LineItem</code>, and <code class="lang-sql">Orders</code> tables of the <code class="lang-sql">SNOWFLAKE_SAMPLE_DATA</code> database, which comes out of the box with your Snowflake account.</p> 
<ol> 
 <li>Make sure that the Snowflake external stage name <code class="lang-sql">unload_to_s3</code> created in the prerequisites is pointing to the S3 prefix <code class="lang-sql">s3-redshift-loader-source</code>created in the previous step.</li> 
 <li>Create a new schema <code class="lang-sql">BLOG_DEMO</code> in the <code class="lang-sql">DEMO_DB</code> database:<code>CREATE SCHEMA demo_db.blog_demo;<br /> </code></li> 
 <li>Duplicate the <code class="lang-sql">Customer</code>, <code class="lang-sql">LineItem</code>, and <code class="lang-sql">Orders</code> tables in the <code class="lang-sql">TPCH_SF1</code> schema to the <code class="lang-sql">BLOG_DEMO</code> schema: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">CREATE TABLE CUSTOMER AS 
SELECT * FROM snowflake_sample_data.tpch_sf1.CUSTOMER;
CREATE TABLE ORDERS AS
SELECT * FROM snowflake_sample_data.tpch_sf1.ORDERS;
CREATE TABLE LINEITEM AS 
SELECT * FROM snowflake_sample_data.tpch_sf1.LINEITEM;</code></pre> 
  </div> </li> 
 <li>Verify that the tables have been duplicated successfully: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">SELECT table_catalog, table_schema, table_name, row_count, bytes
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA = 'BLOG_DEMO'
ORDER BY ROW_COUNT;</code></pre> 
  </div> <p><img alt="unload-step-4" class="alignnone wp-image-36591 size-full" height="422" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/10/27/image003.png" style="margin: 10px 0px 10px 0px;" width="1380" /></p></li> 
 <li>Create <a href="https://docs.snowflake.com/en/user-guide/streams.html" rel="noopener" target="_blank">table streams</a> to track data manipulation language (DML) changes made to the tables, including inserts, updates, and deletes: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">CREATE OR REPLACE STREAM CUSTOMER_CHECK ON TABLE CUSTOMER;
CREATE OR REPLACE STREAM ORDERS_CHECK ON TABLE ORDERS;
CREATE OR REPLACE STREAM LINEITEM_CHECK ON TABLE LINEITEM;</code></pre> 
  </div> </li> 
 <li>Perform DML changes to the tables (for this post, we run UPDATE on all tables and MERGE on the <code class="lang-sql">customer</code> table): 
  <div class="hide-language"> 
   <pre><code class="lang-sql">UPDATE customer 
SET c_comment = 'Sample comment for blog demo' 
WHERE c_custkey between 0 and 10; 
UPDATE orders 
SET o_comment = 'Sample comment for blog demo' 
WHERE o_orderkey between 1800001 and 1800010; 
UPDATE lineitem 
SET l_comment = 'Sample comment for blog demo' 
WHERE l_orderkey between 3600001 and 3600010;</code></pre> 
   <div class="hide-language"> 
    <pre><code class="lang-sql">MERGE INTO customer c 
USING 
( 
SELECT n_nationkey 
FROM snowflake_sample_data.tpch_sf1.nation s 
WHERE n_name = 'UNITED STATES') n 
ON n.n_nationkey = c.c_nationkey 
WHEN MATCHED THEN UPDATE SET c.c_comment = 'This is US based customer1';</code></pre> 
   </div> 
  </div> </li> 
 <li>Validate that the stream tables have recorded all changes: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">SELECT * FROM CUSTOMER_CHECK; 
SELECT * FROM ORDERS_CHECK; 
SELECT * FROM LINEITEM_CHECK;</code></pre> 
   <p>For example, we can query the following customer key value to verify how the events were recorded for the MERGE statement on the customer table:</p> 
   <p><code>SELECT * FROM CUSTOMER_CHECK where c_custkey = 60027;</code></p> 
   <p>We can see the <code>METADATA$ISUPDATE</code> column as <code>TRUE</code>, and we see DELETE followed by INSERT in the <code>METADATA$ACTION</code> column.<br /> <img alt="unload-val-step-7" class="alignnone wp-image-36592 size-full" height="295" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/10/27/image004.png" style="margin: 10px 0px 10px 0px;" width="1379" /></p> 
  </div> </li> 
 <li>Run the COPY command to offload the CDC from the stream tables to the S3 bucket using the external stage name <code>unload_to_s3</code>.In the following code, we’re also copying the data to S3 folders ending with <code>_stg</code> to ensure that when Redshift Auto Loader automatically creates these tables in Amazon Redshift, they get created and marked as staging tables: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">COPY INTO @unload_to_s3/customer_stg/
FROM (select *, sysdate() as LAST_UPDATED_TS from demo_db.blog_demo.customer_check)
FILE_FORMAT = (TYPE = PARQUET)
OVERWRITE = TRUE HEADER = TRUE;</code></pre> 
  </div> 
  <div class="hide-language"> 
   <pre><code class="lang-sql">COPY INTO @unload_to_s3/customer_stg/
FROM (select *, sysdate() as LAST_UPDATED_TS from demo_db.blog_demo.customer_check)
FILE_FORMAT = (TYPE = PARQUET)
OVERWRITE = TRUE HEADER = TRUE;</code></pre> 
  </div> 
  <div class="hide-language"> 
   <pre><code class="lang-sql">COPY INTO @unload_to_s3/lineitem_stg/ 
FROM (select *, sysdate() as LAST_UPDATED_TS from demo_db.blog_demo.lineitem_check) 
FILE_FORMAT = (TYPE = PARQUET) 
OVERWRITE = TRUE HEADER = TRUE;</code></pre> 
  </div> </li> 
 <li>Verify the data in the S3 bucket. There will be three sub-folders created in the s3-redshift-loader-source folder of the S3 bucket, and each will have .parquet data files.<img alt="unload-step-9-val" class="alignnone size-full wp-image-36593" height="634" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/10/27/image005-7.png" style="margin: 10px 0px 10px 0px;" width="1378" /><img alt="unload-step-9-val" class="alignnone size-full wp-image-36594" height="508" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/10/27/image006-5.png" style="margin: 10px 0px 10px 0px;" width="1378" />You can also automate the preceding COPY commands using tasks, which can be scheduled to run at a set frequency for automatic copy of CDC data from Snowflake to Amazon S3.</li> 
 <li>Use the <code>ACCOUNTADMIN</code> role to assign the <code>EXECUTE TASK</code> privilege. In this scenario, we’re assigning the privileges to the <code>SYSADMIN</code> role: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">USE ROLE accountadmin;
GRANT EXECUTE TASK, EXECUTE MANAGED TASK ON ACCOUNT TO ROLE sysadmin;</code></pre> 
  </div> </li> 
 <li>Use the <code>SYSADMIN</code> role to create three separate tasks to run three COPY commands every 5 minutes: <code>USE ROLE sysadmin;</code> 
  <div class="hide-language"> 
   <pre><code class="lang-sql">/* Task to offload Customer CDC table */ 
CREATE TASK sf_rs_customer_cdc 
WAREHOUSE = SMALL 
SCHEDULE = 'USING CRON 5 * * * * UTC' 
AS 
COPY INTO @unload_to_s3/customer_stg/ 
FROM (select *, sysdate() as LAST_UPDATED_TS from demo_db.blog_demo.customer_check) 
FILE_FORMAT = (TYPE = PARQUET) 
OVERWRITE = TRUE 
HEADER = TRUE;</code></pre> 
   <div class="hide-language"> 
    <pre><code class="lang-sql">/*Task to offload Orders CDC table */ 
CREATE TASK sf_rs_orders_cdc 
WAREHOUSE = SMALL 
SCHEDULE = 'USING CRON 5 * * * * UTC' 
AS 
COPY INTO @unload_to_s3/orders_stg/ 
FROM (select *, sysdate() as LAST_UPDATED_TS from demo_db.blog_demo.orders_check)
FILE_FORMAT = (TYPE = PARQUET)
OVERWRITE = TRUE HEADER = TRUE;</code></pre> 
   </div> 
  </div> 
  <div class="hide-language"> 
   <pre><code class="lang-sql">/* Task to offload Lineitem CDC table */ 
CREATE TASK sf_rs_lineitem_cdc 
WAREHOUSE = SMALL 
SCHEDULE = 'USING CRON 5 * * * * UTC' 
AS 
COPY INTO @unload_to_s3/lineitem_stg/ 
FROM (select *, sysdate() as LAST_UPDATED_TS from demo_db.blog_demo.lineitem_check)
FILE_FORMAT = (TYPE = PARQUET)
OVERWRITE = TRUE HEADER = TRUE;</code></pre> 
  </div> <p>When the tasks are first created, they’re in a <code>SUSPENDED</code> state.</p></li> 
 <li>Alter the three tasks and set them to RESUME state: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">ALTER TASK sf_rs_customer_cdc RESUME;
ALTER TASK sf_rs_orders_cdc RESUME;
ALTER TASK sf_rs_lineitem_cdc RESUME;</code></pre> 
  </div> </li> 
 <li>Validate that all three tasks have been resumed successfully: <code>SHOW TASKS;</code><img alt="unload-setp-13-val" class="alignnone size-full wp-image-36595" height="273" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/10/27/image007-1.png" style="margin: 10px 0px 10px 0px;" width="1264" />Now the tasks will run every 5 minutes and look for new data in the stream tables to offload to Amazon S3.As soon as data is migrated from Snowflake to Amazon S3, Redshift Auto Loader automatically infers the schema and instantly creates corresponding tables in Amazon Redshift. Then, by default, it starts loading data from Amazon S3 to Amazon Redshift every 5 minutes. You can also <a href="https://github.com/aws-samples/amazon-redshift-infrastructure-automation/tree/main/Redshift-Loader#viewing-schedules-in-event-bridge" rel="noopener" target="_blank">change the default setting</a> of 5 minutes.</li> 
 <li>On the Amazon Redshift console, launch the <a href="https://docs.aws.amazon.com/redshift/latest/mgmt/query-editor-v2-using.html" rel="noopener" target="_blank">query editor v2</a> and connect to your Amazon Redshift cluster.</li> 
 <li>Browse to the <code>dev</code> database, <code>public</code> schema, and expand <strong>Tables</strong>.<br /> You can see three staging tables created with the same name as the corresponding folders in Amazon S3.</li> 
 <li>Validate the data in one of the tables by running the following query:<code>SELECT * FROM "dev"."public"."customer_stg";</code><img alt="unload-step-16-val" class="alignnone size-full wp-image-36596" height="432" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/01/image008-3.png" style="margin: 10px 0px 10px 0px;" width="1379" /></li> 
</ol> 
<h2>Configure the Redshift Auto Loader utility</h2> 
<p>The Redshift Auto Loader makes data ingestion to Amazon Redshift significantly easier because it automatically loads data files from Amazon S3 to Amazon Redshift. The files are mapped to the respective tables by simply dropping files into preconfigured locations on Amazon S3. For more details about the architecture and internal workflow, refer to the <a href="https://github.com/aws-samples/amazon-redshift-infrastructure-automation/tree/main/Redshift-Loader" rel="noopener" target="_blank">GitHub repo</a>.</p> 
<p>We use an <a href="http://aws.amazon.com/cloudformation" rel="noopener" target="_blank">AWS CloudFormation</a> template to set up Redshift Auto Loader. Complete the following steps:</p> 
<ol> 
 <li>Launch the CloudFormation <a href="https://console.aws.amazon.com/cloudformation/home?#/stacks/new?stackName=RedshiftLoader&amp;templateURL=https://redshift-demos.s3.amazonaws.com/redshift-loader/redshift-s3-data-autoloader.yaml" rel="noopener" target="_blank">template</a>.</li> 
 <li>Choose <strong>Next</strong>.<br /> <img alt="autoloader-step-2" class="alignnone wp-image-36597 size-full" height="589" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/10/27/image009-4.png" style="margin: 10px 0px 10px 0px;" width="1377" /></li> 
 <li>For <strong>Stack name</strong>, enter a name.</li> 
 <li>Provide the parameters listed in the following table.<br /> 
  <table border="1px" style="border-color: #000000;"> 
   <tbody> 
    <tr style="background-color: #000000;"> 
     <td><span style="color: #ffffff;"><strong>CloudFormation Template Parameter</strong></span></td> 
     <td><span style="color: #ffffff;"><strong>Allowed Values</strong></span></td> 
     <td><span style="color: #ffffff;"><strong>Description</strong></span></td> 
    </tr> 
    <tr> 
     <td><code>RedshiftClusterIdentifier</code></td> 
     <td>Amazon Redshift cluster identifier</td> 
     <td>Enter the Amazon Redshift cluster identifier.</td> 
    </tr> 
    <tr> 
     <td><code>DatabaseUserName</code></td> 
     <td>Database user name in the Amazon Redshift cluster</td> 
     <td>The Amazon Redshift database user name that has access to run the SQL script.</td> 
    </tr> 
    <tr> 
     <td><code>DatabaseName</code></td> 
     <td>S3 bucket name</td> 
     <td>The name of the Amazon Redshift primary database where the SQL script is run.</td> 
    </tr> 
    <tr> 
     <td><code>DatabaseSchemaName</code></td> 
     <td>Database name in Amazon Redshift</td> 
     <td>The Amazon Redshift schema name where the tables are created.</td> 
    </tr> 
    <tr> 
     <td><code>RedshiftIAMRoleARN</code></td> 
     <td>Default or the valid IAM role ARN attached to the Amazon Redshift cluster</td> 
     <td>The IAM role ARN associated with the Amazon Redshift cluster. Your default IAM role is set for the cluster and has access to your S3 bucket, leave it at the default.</td> 
    </tr> 
    <tr> 
     <td><code>CopyCommandOptions</code></td> 
     <td>Copy option; default is delimiter ‘|’ gzip</td> 
     <td> <p>Provide the additional COPY command data format parameters.</p> <p>If InitiateSchemaDetection = Yes, then the process attempts to detect the schema and automatically set the suitable copy command options.</p> <p>In the event of failure on schema detection or when InitiateSchemaDetection = No, then this value is used as the default COPY command options to load data.</p></td> 
    </tr> 
    <tr> 
     <td><code>SourceS3Bucket</code></td> 
     <td>S3 bucket name</td> 
     <td>The S3 bucket where the data is stored. Make sure the IAM role that is associated to the Amazon Redshift cluster has access to this bucket.</td> 
    </tr> 
    <tr> 
     <td><code>InitiateSchemaDetection</code></td> 
     <td>Yes/No</td> 
     <td> <p>Set to <strong>Yes </strong>to dynamically detect the schema prior to file load and create a table in Amazon Redshift if it doesn’t exist already. If a table already exists, then it won’t drop or recreate the table in Amazon Redshift.</p> <p>If schema detection fails, the process uses the default COPY options as specified in <code>CopyCommandOptions</code>.</p></td> 
    </tr> 
   </tbody> 
  </table> <p>The Redshift Auto Loader uses the COPY command to load data into Amazon Redshift. For this post, set <code>CopyCommandOptions</code> as follows, and configure any supported COPY command options:</p> 
  <div class="hide-language"> 
   <pre><code class="lang-sql">delimiter '|' dateformat 'auto' TIMEFORMAT 'auto'</code></pre> 
  </div> <p><img alt="autoloader-input-parameters" class="alignnone size-full wp-image-36598" height="1258" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/10/27/image010-4.png" style="margin: 10px 0px 10px 0px;" width="1320" /></p></li> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>Accept the default values on the next page and choose <strong>Next</strong>.</li> 
 <li>Select the acknowledgement check box and choose <strong>Create stack</strong>.<br /> <img alt="autoloader-step-7" class="alignnone size-full wp-image-36599" height="637" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/10/27/image011-4.png" style="margin: 10px 0px 10px 0px;" width="1379" /></li> 
 <li>Monitor the progress of the Stack creation and wait until it is complete.</li> 
 <li>To verify the Redshift Auto Loader configuration, sign in to the Amazon S3 console and navigate to the S3 bucket you provided.<br /> You should see a new directory <code>s3-redshift-loader-source</code> is created.<br /> <img alt="autoloader-step-9" class="alignnone size-full wp-image-36600" height="656" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/01/image012-4.png" style="margin: 10px 0px 10px 0px;" width="1380" /></li> 
</ol> 
<p>Copy all the data files exported from Snowflake under <code>s3-redshift-loader-source</code>.</p> 
<h2>Merge the data from the CDC S3 staging tables to Amazon Redshift tables</h2> 
<p>To merge your data from Amazon S3 to Amazon Redshift, complete the following steps:</p> 
<ol> 
 <li>Create a temporary staging table <code>merge_stg</code> and insert all the rows from the S3 staging table that have <code>metadata_action</code> as <code>INSERT</code>, using the following code. This includes all the new inserts as well as the update. 
  <div class="hide-language"> 
   <pre><code class="lang-sql">CREATE TEMP TABLE merge_stg 
AS
SELECT * FROM
(
SELECT *, DENSE_RANK() OVER (PARTITION BY c_custkey ORDER BY last_updated_ts DESC
) AS rnk
FROM customer_stg WHERE rnk = 1 AND metadata$action = 'INSERT'</code></pre> 
   <p>The preceding code uses a window function <code>DENSE_RANK()</code> to select the latest entries for a given <code>c_custkey</code> by assigning a rank to each row for a given <code>c_custkey</code> and arrange the data in descending order using <code>last_updated_ts</code>. We then select the rows with <code>rnk=1</code> and <code>metadata$action = ‘INSERT’</code> to capture all the inserts.</p> 
  </div> </li> 
 <li>Use the S3 staging table <code>customer_stg</code> to delete the records from the base table <code>customer</code>, which are marked as deletes or updates: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">DELETE FROM customer 
USING customer_stg 
WHERE customer.c_custkey = customer_stg.c_custkey;</code></pre> 
   <p>This deletes all the rows that are present in the CDC S3 staging table, which takes care of rows marked for deletion and updates.</p> 
  </div> </li> 
 <li>Use the temporary staging table <code>merge_stg</code> to insert the records marked for updates or inserts: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">INSERT INTO customer 
SELECT c_custkey, c_name, c_address, c_nationkey, c_phone, c_acctbal, c_mktsegment, c_comment 
FROM merge_stg;</code></pre> 
  </div> </li> 
 <li>Truncate the staging table, because we have already updated the target table:<code>truncate customer_stg;</code></li> 
 <li>You can also run the preceding steps as a stored procedure: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">CREATE OR REPLACE PROCEDURE merge_customer()
AS $$
BEGIN
/*CREATING TEMP TABLE TO GET THE MOST LATEST RECORDS FOR UPDATES/NEW INSERTS*/
CREATE TEMP TABLE merge_stg AS
SELECT * FROM
(
SELECT *, DENSE_RANK() OVER (PARTITION BY c_custkey ORDER BY last_updated_ts DESC ) AS rnk
FROM customer_stg
)
WHERE rnk = 1 AND metadata$action = 'INSERT';
/* DELETING FROM THE BASE TABLE USING THE CDC STAGING TABLE ALL THE RECORDS MARKED AS DELETES OR UPDATES*/
DELETE FROM customer
USING customer_stg
WHERE customer.c_custkey = customer_stg.c_custkey;
/*INSERTING NEW/UPDATED RECORDS IN THE BASE TABLE*/ 
INSERT INTO customer
SELECT c_custkey, c_name, c_address, c_nationkey, c_phone, c_acctbal, c_mktsegment, c_comment
FROM merge_stg;
truncate customer_stg;
END;
$$ LANGUAGE plpgsql;</code></pre> 
   <p>For example, let’s look at the before and after states of the customer table when there’s been a change in data for a particular customer.</p> 
   <p>The following screenshot shows the new changes recorded in the <code>customer_stg</code> table for <code>c_custkey = 74360</code>.<br /> <img alt="merge-process-new-changes" class="alignnone size-full wp-image-36601" height="548" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/10/27/image013-6.png" style="margin: 10px 0px 10px 0px;" width="1378" /><br /> We can see two records for a customer with <code>c_custkey=74360</code>&nbsp;one with <code>metadata$action</code> as <code>DELETE</code> and one with <code>metadata$action</code> as <code>INSERT</code>. That means the record with <code>c_custkey</code> was updated at the source and these changes need to be applied to the target <code>customer</code> table in Amazon Redshift.</p> 
   <p>The following screenshot shows the current state of the <code>customer</code> table before these changes have been merged using the preceding stored procedure:<br /> <img alt="merge-process-current-state" class="alignnone size-full wp-image-36602" height="542" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/01/image014-4.png" style="margin: 10px 0px 10px 0px;" width="1378" /></p> 
  </div> </li> 
 <li>Now, to update the target table, we can run the stored procedure as follows: <code>CALL merge_customer()</code>The following screenshot shows the final state of the target table after the stored procedure is complete.<br /> <img alt="merge-process-after-sp" class="alignnone size-full wp-image-36603" height="547" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/01/image015-4.png" style="margin: 10px 0px 10px 0px;" width="1380" /></li> 
</ol> 
<h3>Run the stored procedure on a schedule</h3> 
<p>You can also run the stored procedure on a schedule via <a href="https://aws.amazon.com/eventbridge/" rel="noopener" target="_blank">Amazon EventBridge</a>. The scheduling steps are as follows:</p> 
<ol> 
 <li>On the EventBridge console, choose <strong>Create rule</strong>.<br /> <img alt="sp-schedule-1" class="alignnone size-full wp-image-36604" height="908" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/01/image016-4.png" style="margin: 10px 0px 10px 0px;" width="2432" /></li> 
 <li>For <strong>Name</strong>, enter a meaningful name, for example, <code>Trigger-Snowflake-Redshift-CDC-Merge</code>.</li> 
 <li>For <strong>Event bus</strong>, choose <strong>default</strong>.</li> 
 <li>For <strong>Rule Type,</strong> select <strong>Schedule</strong>.</li> 
 <li>Choose <strong>Next</strong>.<br /> <img alt="sp-schedule-step-5" class="alignnone size-full wp-image-36605" height="690" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/01/image017-5.png" style="margin: 10px 0px 10px 0px;" width="1148" /></li> 
 <li>For <strong>Schedule pattern</strong>, select <strong>A schedule that runs at a regular rate, such as every 10 minutes</strong>.</li> 
 <li>For <strong>Rate expression</strong>, enter <strong>Value</strong> as 5 and choose <strong>Unit</strong> as <strong>Minutes</strong>.</li> 
 <li>Choose <strong>Next</strong>.<br /> <img alt="sp-schedule-step-8" class="alignnone size-full wp-image-36606" height="866" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/01/image018-6.png" style="margin: 10px 0px 10px 0px;" width="1812" /></li> 
 <li>For <strong>Target types</strong>, choose <strong>AWS service</strong>.</li> 
 <li>For <strong>Select a Target</strong>, choose <strong>Redshift cluster</strong>.</li> 
 <li>For <strong>Cluster</strong>, choose the Amazon Redshift cluster identifier.</li> 
 <li>For <strong>Database name</strong>, choose <strong>dev</strong>.</li> 
 <li>For <strong>Database user</strong>, enter a user name with access to run the stored procedure. It uses temporary credentials to authenticate.</li> 
 <li>Optionally, you can also use <a href="https://aws.amazon.com/secrets-manager/" rel="noopener" target="_blank">AWS Secrets Manager</a> for authentication.</li> 
 <li>For <strong>SQL statement</strong>, enter <code>CALL merge_customer()</code>.</li> 
 <li>For <strong>Execution role</strong>, select <strong>Create a new role for this specific resource</strong>.</li> 
 <li>Choose <strong>Next</strong>.<br /> <img alt="sp-schedule-step-17" class="alignnone size-full wp-image-36607" height="743" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/10/27/image019-5.png" style="margin: 10px 0px 10px 0px;" width="1175" /></li> 
 <li>Review the rule parameters and choose <strong>Create rule.</strong></li> 
</ol> 
<p>After the rule has been created, it automatically triggers the stored procedure in Amazon Redshift every 5 minutes to merge the CDC data into the target table.</p> 
<h2>Configure Amazon Redshift to share the identified data with AWS Data Exchange</h2> 
<p>Now that you have the data stored inside Amazon Redshift, you can publish it to customers using AWS Data Exchange.</p> 
<ol> 
 <li>In Amazon Redshift, using any query editor, create the data share and add the tables to be shared: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">CREATE DATASHARE salesshare MANAGEDBY ADX;
ALTER DATASHARE salesshare ADD SCHEMA tpch_sf1;
ALTER DATASHARE salesshare ADD TABLE tpch_sf1.customer;</code></pre> 
  </div> <p><img alt="ADX-step1" class="alignnone wp-image-36608 size-full" height="535" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/10/27/image020-2.jpg" style="margin: 10px 0px 10px 0px;" width="1078" /></p></li> 
 <li>On the AWS Data Exchange console, create your dataset.</li> 
 <li>Select <strong>Amazon Redshift datashare</strong>.<br /> <img alt="ADX-step3-create-datashare" class="alignnone size-full wp-image-36609" height="658" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/01/image021-1.jpg" style="margin: 10px 0px 10px 0px;" width="1029" /></li> 
 <li>Create a revision in the dataset.<br /> <img alt="ADX-step4-create-revision" class="alignnone size-full wp-image-36610" height="826" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/01/image022-2.jpg" style="margin: 10px 0px 10px 0px;" width="1027" /></li> 
 <li>Add assets to the revision (in this case, the Amazon Redshift data share).<br /> <img alt="ADX-addassets" class="alignnone size-full wp-image-36611" height="741" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/10/27/image023-1.jpg" style="margin: 10px 0px 10px 0px;" width="1026" /></li> 
 <li>Finalize the revision.<br /> <img alt="ADX-step-6-finalizerevision" class="alignnone size-full wp-image-36612" height="199" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/10/27/image024-2.jpg" style="margin: 10px 0px 10px 0px;" width="1366" /></li> 
</ol> 
<p>After you create the dataset, you can publish it to the public catalog or directly to customers as a private product. For instructions on how to create and publish products, refer to <a href="https://aws.amazon.com/blogs/aws/new-aws-data-exchange-for-amazon-redshift/" rel="noopener" target="_blank">NEW – AWS Data Exchange for Amazon Redshift</a></p> 
<h2>Clean up</h2> 
<p>To avoid incurring future charges, complete the following steps:</p> 
<ol> 
 <li>Delete the CloudFormation stack used to create the Redshift Auto Loader.</li> 
 <li>Delete the Amazon Redshift cluster created for this demonstration.</li> 
 <li>If you were using an existing cluster, drop the created external table and external schema.</li> 
 <li>Delete the S3 bucket you created.</li> 
 <li>Delete the Snowflake objects you created.</li> 
</ol> 
<h2>Conclusion</h2> 
<p>In this post, we demonstrated how you can set up a fully integrated process that continuously replicates data from Snowflake to Amazon Redshift and then uses Amazon Redshift to offer data to downstream clients over AWS Data Exchange. You can use the same architecture for other purposes, such as sharing data with other Amazon Redshift clusters within the same account, cross-accounts, or even cross-Regions if needed.</p> 
<hr /> 
<h3>About the Authors</h3> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/05/31/BDB-2083-Raks.jpg"><img alt="Raks Khare" class="wp-image-29576 size-full alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/05/31/BDB-2083-Raks.jpg" width="100" /></a>Raks Khare</strong> is an Analytics Specialist Solutions Architect at AWS based out of Pennsylvania. He helps customers architect data analytics solutions at scale on the AWS platform.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/02/ekta-ahuja-1.png"><img alt="" class="wp-image-36901 size-full alignleft" height="132" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/02/ekta-ahuja-1.png" width="100" /></a>Ekta Ahuja </strong>is a Senior Analytics Specialist Solutions Architect at AWS. She is passionate about helping customers build scalable and robust data and analytics solutions. Before AWS, she worked in several different data engineering and analytics roles. Outside of work, she enjoys baking, traveling, and board games.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/04/01/Tahir-Aziz.png"><img alt="" class="wp-image-27451 size-full alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/04/01/Tahir-Aziz.png" width="100" /></a>Tahir Aziz</strong> is an Analytics Solution Architect at AWS. He has worked with building data warehouses and big data solutions for over 13 years. He loves to help customers design end-to-end analytics solutions on AWS. Outside of work, he enjoys traveling and cooking.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2021/12/09/ahmedaws.jpg"><img alt="" class="wp-image-24619 size-full alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2021/12/09/ahmedaws.jpg" width="100" /></a>Ahmed Shehata</strong> is a Senior Analytics Specialist Solutions Architect at AWS based on Toronto. He has more than two decades of experience helping customers modernize their data platforms, Ahmed is passionate about helping customers build efficient, performant and scalable Analytic solutions.</p>
