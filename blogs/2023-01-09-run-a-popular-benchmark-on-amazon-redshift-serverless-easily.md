---
title: "Run a popular benchmark on Amazon Redshift Serverless easily with AWS Data Exchange"
url: "https://aws.amazon.com/blogs/big-data/run-a-popular-benchmark-on-amazon-redshift-serverless-easily-with-aws-data-exchange/"
date: "Mon, 09 Jan 2023 19:07:49 +0000"
author: "Jon Roberts"
feed_url: "https://aws.amazon.com/blogs/big-data/category/analytics/aws-data-exchange/feed/"
---
<p>Amazon Redshift is a fast, easy, secure, and economical cloud data warehousing service designed for analytics. AWS announced Amazon Redshift Serverless general availability in July 2022, providing an easier experience to operate Amazon Redshift. Amazon Redshift Serverless makes it simple to run and scale analytics without having to manage your data warehouse infrastructure. Amazon Redshift Serverless automatically provisions and intelligently scales data warehouse capacity to deliver fast performance for even the most demanding and unpredictable workloads, and you pay only for what you use.</p> 
<p>Amazon Redshift Serverless measures data warehouse capacity in Redshift Processing Units (RPUs). You pay for the workloads you run in RPU-hours on a per-second basis (with a 60-second minimum charge), including queries that access external data in open file formats like CSV and Parquet stored in <a href="http://aws.amazon.com/s3" rel="noopener" target="_blank">Amazon S3</a>. For more information on RPU pricing, refer to <a href="https://aws.amazon.com/redshift/pricing/" rel="noopener" target="_blank">Amazon Redshift pricing</a>.</p> 
<p>AWS Data Exchange makes it easy to find, subscribe to, and use third-party data in the cloud. With AWS Data Exchange for Amazon Redshift, customers can start querying, evaluating, analyzing, and joining third-party data with their own first-party data without requiring any extracting, transforming, and loading (ETL). Data providers can list and offer products containing Amazon Redshift datasets in the AWS Data Exchange catalog, granting subscribers direct, read-only access to the data stored in Amazon Redshift. This feature empowers customers to quickly query, analyze, and build applications with these third-party datasets.</p> 
<p><a href="https://www.tpc.org/tpcds" rel="noopener" target="_blank">TPC-DS</a> is a commonly used benchmark for measuring the query performance of data warehouse solutions such as Amazon Redshift. The benchmark is useful in proving the query capabilities of executing simple to complex queries in a timely manner. It is also used to measure the performance of different database configurations, different concurrent workloads, and also against other database products.</p> 
<p>This blog post walks you through the steps you’ll need to set up Amazon Redshift Serverless and run the SQL queries derived from the TPC-DS benchmark against data from the AWS Data Exchange.</p> 
<h2>Solution overview</h2> 
<p>We will get started by creating an Amazon Redshift Serverless workgroup and namespace. A namespace is a collection of database objects and users while a workgroup is a collection of compute resources. To simplify executing the benchmark queries, a Linux EC2 instance will also be deployed.</p> 
<p>Next, a <a href="https://github.com/aws-samples/redshift-benchmarks/tree/main/adx-tpc-ds" rel="noopener" target="_blank">GitHub repo</a> containing the TPC-DS derived queries will be used. The TPC-DS benchmark is frequently used for evaluating the performance and functionality of cloud data warehouses. The TPC-DS benchmark includes additional steps and requirements to be considered official, but for this blog post, we are focused on only executing the SQL SELECT queries from this benchmark.</p> 
<p>The last component of the solution is data. The TPC-DS benchmark includes binaries for generating data, but this is time-consuming to run. We have avoided this problem by generating the data, and we have made this available freely in the <a href="https://aws.amazon.com/data-exchange/" rel="noopener" target="_blank">AWS Data Exchange</a>.</p> 
<h2>Automated setup: CloudFormation</h2> 
<h2><a href="https://console.aws.amazon.com/cloudformation/home?#/stacks/new?stackName=rs-serverless-adx&amp;templateURL=https://redshift-blogs.s3.amazonaws.com/redshift-serverless-adx/rs_serverless.json" rel="noopener" target="_blank"><img alt="BDB-2063-launch-cloudformation-stack" class="alignnone wp-image-31116 size-full" height="20" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/06/22/image003-5.png" style="font-size: 16px;" width="107" /></a></h2> 
<p>Click the <strong>Launch Stack</strong> link above to launch the CloudFormation stack, which will automate the deployment of resources needed for the demo. The template deploys the following resources in your default VPC:</p> 
<ul> 
 <li>Amazon Compute Cloud (Amazon EC2) instance with the latest version of Amazon Linux</li> 
 <li>Amazon Redshift Serverless workgroup and namespace</li> 
 <li>IAM role with <code>redshift-serverless:GetWorkgroup</code> action granted; this is attached to the EC2 instance so that a command line interface (CLI) command can run to complete the instance configuration</li> 
 <li>Security group with inbound port 22 (ssh) and connectivity between the EC2 instance and the Amazon Redshift Serverless workgroup</li> 
 <li>The GitHub repo is downloaded in the EC2 instance</li> 
</ul> 
<h3>Template parameters</h3> 
<ul> 
 <li><code>Stack</code>: CloudFormation term used to define all of the resources created by the template.</li> 
 <li><code>KeyName</code>: This is the name of an existing key pair. If you don’t have one already, create a key pair that is used to connect by SSH to the EC2 instance. More information on <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html" rel="noopener" target="_blank">key pairs</a>.</li> 
 <li><code>SSHLocation</code>: The CIDR mask for incoming connections to the EC2 instance. The default is 0.0.0.0/0, which means any IP address is allowed to connect by SSH to the EC2 instance, provided the client has the key pair private key. The best practice for security is to limit this to a smaller range of IP addresses. For example, you can use sites like <a href="http://www.whatismyip.com/" rel="noopener" target="_blank">www.whatismyip.com</a> to get your IP address.</li> 
 <li><code>RedshiftCapacity</code>: This is the number of RPUs for the Amazon Redshift Serverless workgroup. The default is 128 and is recommended for analyzing the larger TPC-DS datasets. You can update the RPU capacity after deployment if you like or redeploy it with a different capacity value.</li> 
</ul> 
<h2>Manual setup</h2> 
<p>If you choose not to use the CloudFormation template, deploy the EC2 instance and Amazon Redshift Serverless with the following instructions. The following steps are only needed if you are manually provisioning the resources rather than using the provided CloudFormation template.</p> 
<h3>Amazon Redshift setup</h3> 
<p>Here are the high-level steps to create an Amazon Redshift Serverless workgroup. You can get more detailed information from this <a href="https://aws.amazon.com/blogs/aws/amazon-redshift-serverless-now-generally-available-with-new-capabilities/" rel="noopener" target="_blank">News Blog post</a>.</p> 
<p>To create your workgroup, complete the following steps:</p> 
<ol> 
 <li>On the Amazon Redshift console, navigate to the <a href="https://console.aws.amazon.com/redshiftv2/home?r#serverless-dashboard" rel="noopener" target="_blank">Amazon Redshift Serverless dashboard</a>.</li> 
 <li>Choose <strong>Create workgroup</strong>.</li> 
 <li>For <strong>Workgroup name</strong>, enter a name.</li> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>For <strong>Namespace</strong>, enter a unique name.</li> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>Choose <strong>Create</strong>.</li> 
</ol> 
<p>These steps create an Amazon Redshift Serverless workgroup with 128 RPUs. This is the default; you can easily adjust this up or down based on your workload and budget constraints.</p> 
<h3>Linux EC2 instance setup</h3> 
<ul> 
 <li>Deploy a virtual machine in the same AWS Region as your Amazon Redshift database using the Amazon Linux 2 AMI.</li> 
 <li>The Amazon Linux 2 AMI (64-bit x86) with the t2.micro instance type is an inexpensive and tested configuration.</li> 
 <li>Add the security group configured for your Amazon Redshift database to your EC2 instance.</li> 
 <li>Install psql with&nbsp;<code>sudo yum install postgresql.x86_64 -y</code></li> 
 <li>Download this GitHub repo. 
  <div class="hide-language"> 
   <pre><code class="lang-bash">git clone --depth 1 https://github.com/aws-samples/redshift-benchmarks /home/ec2-user/redshift-benchmarks</code></pre> 
  </div> </li> 
 <li>Set the following environment variables for Amazon Redshift: 
  <ul> 
   <li><code>PGHOST</code>: This is the endpoint for the Amazon Redshift database.</li> 
   <li><code>PGPORT</code>: This is the port the database listens on. The Amazon Redshift default is 5439.</li> 
   <li><code>PGUSER</code>: This is your Amazon Redshift database user.</li> 
   <li><code>PGDATABASE</code>: This is the database name where your external schemas are created. This is NOT the database created for the data share. We suggest using the default “dev” database.</li> 
  </ul> </li> 
</ul> 
<p>Example:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">export PGUSER="awsuser" 
export PGHOST="default.01.us-east-1.redshift-serverless.amazonaws.com" 
export PGPORT="5439" 
export PGDATABASE="dev"</code></pre> 
</div> 
<p>Configure the <code>.pgpass</code> file to store your database credentials. The format for the <code>.pgpass</code> file is: <code>hostname:port:database:user:password</code></p> 
<p>Example:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">default.01.us-east-1.redshift-serverless.amazonaws.com:5439:*:user1:user1P@ss</code></pre> 
</div> 
<h2>AWS Data Exchange setup</h2> 
<p>AWS Data Exchange provides third-party data in multiple data formats, including Amazon Redshift. You can subscribe to catalog listings in multiple storage locations like Amazon S3 and Amazon Redshift data shares. We encourage you to explore the AWS Data Exchange catalog on your own because there are many datasets available that can be used to enhance your data in Amazon Redshift.</p> 
<p>First, subscribe to the AWS Marketplace listing for <a href="https://aws.amazon.com/marketplace/pp/prodview-iopazp7irqk6s" rel="noopener" target="_blank">TPC-DS Benchmark Data</a>. Select the <strong>Continue to subscribe</strong> button from the AWS Data Exchange catalog listing. After you review the offer and Terms and Conditions of the data product, choose <strong>Subscribe</strong>. Note that you will need the appropriate IAM permissions to subscribe to AWS Data Exchange on Marketplace. More information can be found at <a href="https://docs.aws.amazon.com/data-exchange/latest/userguide/security-iam-awsmanpol.html" rel="noopener" target="_blank">AWS managed policies for AWS Data Exchange</a>.</p> 
<p>TPC-DS uses 24 tables in a dimensional model that simulates a decision support system. It has store, catalog, and web sales as well as store, catalog, and web returns fact tables. It also has the dimension tables to support these fact tables.</p> 
<p>TPC-DS includes a utility to generate data for the benchmark at a given scale factor. The smallest scale factor is 1 GB (uncompressed). Most benchmark tests for cloud warehouses are run with 3–10 TB of data because the dataset is large enough to stress the system but also small enough to complete the entire test in a reasonable amount of time.</p> 
<p>There are six database schemas provided in the TPC-DS Benchmark Data subscription with 1; 10; 100; 1,000; 3,000; and 10,000 scale factors. The scale factor refers to the uncompressed data size measured in GB. Each schema refers to a dataset with the corresponding scale factor.</p> 
<table border="1" style="height: 213px;" width="733"> 
 <tbody> 
  <tr style="background-color: #000000;"> 
   <td><span style="color: #ffffff;"><strong>Scale factor (GB)</strong></span></td> 
   <td><span style="color: #ffffff;"><strong>ADX schema</strong></span></td> 
   <td><span style="color: #ffffff;"><strong>Amazon Redshift Serverless external schema</strong></span></td> 
  </tr> 
  <tr> 
   <td>1</td> 
   <td>tpcds1</td> 
   <td>ext_tpcds1</td> 
  </tr> 
  <tr> 
   <td>10</td> 
   <td>tpcds10</td> 
   <td>ext_tpcds10</td> 
  </tr> 
  <tr> 
   <td>100</td> 
   <td>tpcds100</td> 
   <td>ext_tpcds100</td> 
  </tr> 
  <tr> 
   <td>1,000</td> 
   <td>tpcds1000</td> 
   <td>ext_tpcds1000</td> 
  </tr> 
  <tr> 
   <td>3,000</td> 
   <td>tpcds3000</td> 
   <td>ext_tpcds3000</td> 
  </tr> 
  <tr> 
   <td>10,000</td> 
   <td>tpcds10000</td> 
   <td>ext_tpcds10000</td> 
  </tr> 
 </tbody> 
</table> 
<p>The following steps will create external schemas in your Amazon Redshift Serverless database that maps to schemas found in the AWS Data Exchange.</p> 
<ul> 
 <li>Log in to the EC2 instance and create a database connection. 
  <div class="hide-language"> 
   <pre><code class="lang-bash">psql</code></pre> 
  </div> </li> 
 <li>Run the following query: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">select share_name, producer_account, producer_namespace from svv_datashares;</code></pre> 
  </div> </li> 
 <li>Use the output of this query to run the next command: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">create database tpcds_db from datashare &lt;share_name&gt; of account '<span style="color: #ff0000;">&lt;producer_account&gt;</span>' namespace '<span style="color: #ff0000;">&lt;producer_namespace&gt;</span>';</code></pre> 
  </div> </li> 
 <li>Last, you create the external schemas in Amazon Redshift: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">create external schema ext_tpcds1 from redshift database tpcds_db schema tpcds1;
create external schema ext_tpcds10 from redshift database tpcds_db schema tpcds10;
create external schema ext_tpcds100 from redshift database tpcds_db schema tpcds100;
create external schema ext_tpcds1000 from redshift database tpcds_db schema tpcds1000;
create external schema ext_tpcds3000 from redshift database tpcds_db schema tpcds3000;
create external schema ext_tpcds10000 from redshift database tpcds_db schema tpcds10000;</code></pre> 
  </div> </li> 
 <li>You can now exit psql with this command: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">\q</code></pre> 
  </div> </li> 
</ul> 
<h2>TPC-DS derived benchmark</h2> 
<p>The TPC-DS derived benchmark consists of 99 queries in four broad categories:</p> 
<ul> 
 <li>Reporting queries</li> 
 <li>Ad hoc queries</li> 
 <li>Iterative OLAP queries</li> 
 <li>Data mining queries</li> 
</ul> 
<p>In addition to running the 99 queries, the benchmark tests concurrency. During the concurrency portion of the test, there are <em>n</em> sessions (default of 5) that run the queries. Each session runs the 99 queries with different parameters and in slightly different order. This concurrency test stresses the resources of the database and generally takes longer to complete than just a single session running the 99 queries.</p> 
<p>Some data warehouse products are configured to optimize single-user performance, whereas others may not have the ability to manage the workload effectively. This is a great way to demonstrate the workload management and stability of Amazon Redshift.</p> 
<p>Since the data for each scale factor is located in a different schema, running the benchmark against each scale factor requires changing the schema you are referencing. The <code>search_path</code> defines which schemas to search for tables when a query contains objects without a schema included. For example:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">ALTER USER &lt;username&gt; SET search_path=ext_tpcds3000,public;</code></pre> 
</div> 
<p>The benchmark scripts set the <code>search_path</code> automatically.</p> 
<p>Note: The scripts create a schema called <code>tpcds_reports</code> which will store the detailed output of each step of the benchmark. Each time the scripts are run, this schema will be recreated, and the latest results will be stored. If you happen to already have a schema named <code>tpcds_reports</code>, these scripts will drop the schema.</p> 
<h3>Running the TPC-DS derived queries</h3> 
<ul> 
 <li>Connect by SSH to the EC2 instance with your key pair.</li> 
 <li>Change directory: 
  <div class="hide-language"> 
   <pre><code class="lang-bash">cd ~/redshift-benchmarks/adx-tpc-ds/</code></pre> 
  </div> </li> 
 <li>Optionally configure the variables for the scripts in <code>tpcds_variables.sh</code></li> 
</ul> 
<p>Here are the default values for the variables you can set:</p> 
<ul> 
 <li><code>EXT_SCHEMA="ext_tpcds3000"</code>: This is the name of the external schema created that has the TPC-DS dataset. The “3000” value means the scale factor is 3000 or 3 TB (uncompressed).</li> 
 <li><code>EXPLAIN="false"</code>: If set to false, queries will run. If set to true, queries will generate explain plans rather than actually running. Each query will be logged in the log directory. Default is false.</li> 
 <li><code>MULTI_USER_COUNT="5"</code>: 0 to 20 concurrent users will run the queries. The order of the queries was set with <code>dsqgen</code>. Setting to 0 will skip the multi-user test. Default is 5.</li> 
 <li>Run the benchmark: 
  <div class="hide-language"> 
   <pre><code class="lang-bash">./rollout.sh &gt; rollout.log 2&gt;&amp;1 &amp;</code></pre> 
  </div> </li> 
</ul> 
<h3>TPC-DS derived benchmark results</h3> 
<p>We performed a test with the 3 TB ADX TPC-DS dataset on an Amazon Redshift Serverless workgroup with 128 RPUs. Additionally, we disabled query caching so that query results aren’t cached. This allows us to measure the performance of the database as opposed to its ability to serve results from cache.</p> 
<p>The test comprises two sections. The first will run the 99 queries serially using one user while the second section will start multiple sessions based on the configuration file you set earlier. Each session will run concurrently, and each will run all 99 queries but in a different order.</p> 
<p>The total runtime for the single-user queries was 15 minutes and 11 seconds. As shown in the following graph, the longest-running query was query 67, with an elapsed time of only 101 seconds. The average runtime was only 9.2 seconds.</p> 
<p><img alt="1 Session TPC-DS 3TB 128 RPUs" class="aligncenter wp-image-40095 size-full" height="568" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/14/BDB-2599-1-Session.png" width="979" /></p> 
<p>With five concurrent users, the runtime was 28 minutes and 35 seconds, which demonstrates how Amazon Redshift Serverless performs well for single-user and concurrent-user workloads.</p> 
<p><img alt="5 Concurrent Sessions TPC-DS 3TB 128 RPUs" class="aligncenter wp-image-40096 size-full" height="508" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/14/BDB-2599-5-Sessions.png" width="979" /></p> 
<p>As you can see, it was pretty easy to deploy Amazon Redshift Serverless, subscribe to an AWS Data Exchange product listing, and run a fairly complex benchmark in a short amount of time.</p> 
<h2>Next steps</h2> 
<p>You can run the benchmark scripts again but with different dataset sizes or a different number of concurrent users by editing the <code>tpcds_variables.sh</code> file. You can also try resizing your Amazon Redshift Serverless workgroup to see the performance difference with more or fewer RPUs. You can also run individual queries to see the results firsthand.</p> 
<p>Another thing to try is to subscribe to other <a href="https://aws.amazon.com/marketplace/b/d5a43d97-558f-4be7-8543-cce265fe6d9d?ref_=mp_nav_category_87374d8c-7acc-49b2-bc45-1a6cb252a539&amp;category=d5a43d97-558f-4be7-8543-cce265fe6d9d&amp;FULFILLMENT_OPTION_TYPE=DATA_EXCHANGE&amp;DATA_AVAILABLE_THROUGH=REDSHIFT_DATA_SHARES&amp;filters=FULFILLMENT_OPTION_TYPE%2CDATA_AVAILABLE_THROUGH" rel="noopener" target="_blank">AWS Data Exchange products</a> and query this data from Amazon Redshift Serverless. Be curious and explore using Amazon Redshift Serverless and the AWS Data Exchange!</p> 
<h2>Clean up</h2> 
<p>If you deployed the resources with the automated solution, you just need to delete the stack created in CloudFormation. All resources created by the stack will be deleted automatically.</p> 
<p>If you deployed the resources manually, you need to delete the following:</p> 
<ul> 
 <li>The Amazon Redshift database created earlier. 
  <ul> 
   <li>If you deployed Amazon Redshift Serverless, you will need to delete both the workgroup and the namespace.</li> 
  </ul> </li> 
 <li>The Amazon EC2 instance.</li> 
</ul> 
<p>Optionally, you can unsubscribe from the TPC-DS data by going to your AWS Data Exchange <a href="https://console.aws.amazon.com/dataexchange/home?#/subscriptions" rel="noopener" target="_blank">Subscriptions</a> and then turning <strong>Renewal</strong> to <strong>Off</strong>.</p> 
<h2>Conclusion</h2> 
<p>This blog post covered deploying Amazon Redshift Serverless, subscribing to an AWS Data Exchange product, and running a complex benchmark in a short amount of time. Amazon Redshift Serverless can handle high levels of concurrency with very little effort and excels in <a href="https://aws.amazon.com/blogs/big-data/amazon-redshift-continues-its-price-performance-leadership/" rel="noopener" target="_blank">price-performance</a>.</p> 
<p>If you have any questions or feedback, please leave them in the comments section.</p> 
<hr /> 
<h2>About the author</h2> 
<p><img alt="Jon Roberts" class="wp-image-40117 alignleft" height="76" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/14/BDB-2599-JonRoberts.png" width="100" /><strong>Jon Roberts</strong> is a Sr. Analytics Specialist based out of Nashville, specializing in Amazon Redshift. He has over 27 years of experience working in relational databases. In his spare time, he runs.</p>
