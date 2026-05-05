---
title: "Accelerate machine learning with AWS Data Exchange and Amazon Redshift ML"
url: "https://aws.amazon.com/blogs/big-data/accelerate-machine-learning-with-aws-data-exchange-and-amazon-redshift-ml/"
date: "Fri, 08 Jul 2022 15:46:46 +0000"
author: "Yadgiri Pottabhathini"
feed_url: "https://aws.amazon.com/blogs/big-data/category/analytics/aws-data-exchange/feed/"
---
<p><em><strong>July 2023: This post was reviewed for accuracy and updated.</strong></em></p> 
<p><a href="https://aws.amazon.com/redshift/features/redshift-ml/" rel="noopener noreferrer" target="_blank">Amazon Redshift ML</a> makes it easy for SQL users to create, train, and deploy ML models using familiar SQL commands. Redshift ML allows you to use your data in Amazon Redshift with <a href="https://aws.amazon.com/sagemaker/" rel="noopener noreferrer" target="_blank">Amazon SageMaker</a>, a fully managed ML service, without requiring you to become an expert in ML.</p> 
<p><a href="https://aws.amazon.com/data-exchange/" rel="noopener noreferrer" target="_blank">AWS Data Exchange</a> makes it easy to find, subscribe to, and use third-party data in the cloud. <a href="https://aws.amazon.com/redshift/features/aws-data-exchange-for-amazon-redshift/" rel="noopener noreferrer" target="_blank">AWS Data Exchange for Amazon Redshift</a> enables you to access and query tables in Amazon Redshift without extracting, transforming, and loading files (ETL).</p> 
<p>As a subscriber, you can browse through the AWS Data Exchange catalog and find data products that are relevant to your business with data stored in Amazon Redshift, and <a href="https://aws.amazon.com/blogs/aws/new-aws-data-exchange-for-amazon-redshift/" rel="noopener noreferrer" target="_blank">subscribe</a> to the data from the providers without any further processing, and no need for an ETL process.</p> 
<p>If the provider data is not already available in Amazon Redshift, many providers will add the data to Amazon Redshift upon request.</p> 
<p>In this post, we show you the process of subscribing to datasets through AWS Data Exchange without ETL, running ML algorithms on an Amazon Redshift cluster, and performing local inference and production.</p> 
<h2>Solution overview</h2> 
<p>The use case for the solution in this post is to predict ticket sales for worldwide events based on historical ticket sales data using a regression model. The data or ETL engineer can build the data pipeline by subscribing to the <a href="https://aws.amazon.com/marketplace/pp/prodview-4ozlpl4r3k7cg?sr=0-3&amp;ref_=beagle&amp;applicationId=AWSMPContessa" rel="noopener noreferrer" target="_blank">Worldwide Event Attendance</a> product on AWS Data Exchange without ETL. You can then create the ML model in Redshift ML using the time series ticket sales data and predict future ticket sales.</p> 
<p>To implement this solution, you complete the following high-level steps:</p> 
<ol> 
 <li>Subscribe to datasets using AWS Data Exchange for Amazon Redshift.</li> 
 <li>Connect to the datashare in Amazon Redshift.</li> 
 <li>Create the ML model using the SQL notebook feature of the Amazon Redshift query editor V2.</li> 
</ol> 
<p>The following diagram illustrates the solution architecture.</p> 
<p><img alt="" class="size-full wp-image-31256 aligncenter" height="442" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/06/27/image001.jpg" style="margin: 10px 0px 10px 0px;" width="1074" /></p> 
<h2>Prerequisites</h2> 
<p>Before starting this walkthrough, you must complete the following prerequisites:</p> 
<ol> 
 <li>Make sure you have an existing Amazon Redshift cluster with RA3 node type. If not, you can <a href="https://docs.aws.amazon.com/redshift/latest/gsg/rs-gsg-launch-sample-cluster.html" rel="noopener noreferrer" target="_blank">create a provisioned Amazon Redshift cluster</a>.</li> 
 <li>Make sure the Amazon Redshift cluster is encrypted, because the data provider is encrypted and Amazon Redshift data sharing requires homogeneous encryption configurations. For more details on homogeneous encryption, refer to <a href="https://docs.aws.amazon.com/redshift/latest/dg/considerations.html" rel="noopener noreferrer" target="_blank">Data sharing considerations in Amazon Redshift</a>.</li> 
 <li>Create an<a href="https://aws.amazon.com/iam/" rel="noopener noreferrer" target="_blank"> AWS Identity Access and Management</a> (IAM) role with access to SageMaker and <a href="http://aws.amazon.com/s3" rel="noopener noreferrer" target="_blank">Amazon Simple Storage Service</a> (Amazon S3) and attach it to the Amazon Redshift cluster. Refer to <a href="https://docs.aws.amazon.com/redshift/latest/dg/admin-setup.html#cluster-setup" rel="noopener noreferrer" target="_blank">Cluster setup for using Amazon Redshift ML</a> for more details.</li> 
 <li>Create an S3 bucket for storing the training data and model output.</li> 
</ol> 
<p>Please note that some of the above AWS resources in this walkthrough will incur charges. Please remember to delete the resources when you’re finished.</p> 
<h2>Subscribe to an AWS Data Exchange product with Amazon Redshift data</h2> 
<p>To subscribe to an AWS Data Exchange public dataset, complete the following steps:</p> 
<ol> 
 <li>On the AWS Data Exchange console, choose <strong>Explore available data products</strong>.</li> 
 <li>In the navigation pane, under <strong>Data available through</strong>, select <strong>Amazon Redshift</strong> to filter products with Amazon Redshift data.</li> 
 <li>Choose <strong>Worldwide Event Attendance (Test Product)</strong>.<img alt="" class="alignnone size-full wp-image-31257" height="1076" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/06/27/image002-3.jpg" style="margin: 10px 0px 10px 0px;" width="1451" /></li> 
 <li>Choose <strong>Continue to subscribe</strong>.</li> 
 <li>Confirm the catalog is subscribed by checking that it’s listed on the <strong>Subscriptions </strong>page.<img alt="" class="alignnone size-full wp-image-31259" height="818" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/06/27/image004-1.jpg" style="margin: 10px 0px 10px 0px;" width="2676" /></li> 
</ol> 
<h2>Predict tickets sold using Redshift ML</h2> 
<p>To set up prediction using Redshift ML, complete the following steps:</p> 
<ol> 
 <li>On the Amazon Redshift console, choose <strong>Datashares</strong> in the navigation pane.</li> 
 <li>On the <strong>Subscriptions</strong> tab, confirm that the AWS Data Exchange datashare is available.<img alt="" class="alignnone size-full wp-image-31260" height="726" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/06/27/image005.jpg" style="margin: 10px 0px 10px 0px;" width="2406" /></li> 
 <li>In the navigation pane, choose <strong>Query editor v2</strong>.</li> 
 <li>Connect to your Amazon Redshift cluster in the navigation pane.</li> 
</ol> 
<p>Amazon Redshift provides a feature to create notebooks and run your queries. For the remaining part of this tutorial, we run queries in the notebook and create comments in the markdown cell for each step.</p> 
<ol start="5"> 
 <li>Choose the plus sign and choose Notebook.<br /> <img alt="" class="alignnone wp-image-31262" height="111" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/06/27/image006-1.jpg" style="margin: 10px 0px 10px 0px;" width="328" /></li> 
 <li>Choose Add markdown.<br /> <img alt="" class="alignnone wp-image-31263" height="59" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/06/27/image007-1.jpg" style="margin: 10px 0px 10px 0px;" width="202" /></li> 
 <li>Enter <code>Show available data share</code> in the cell.</li> 
 <li>Choose Add SQL.<br /> <img alt="" class="alignnone wp-image-31264" height="56" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/06/27/image008-2.jpg" style="margin: 10px 0px 10px 0px;" width="162" /></li> 
 <li>Enter and run the following command to see the available datashares for the cluster: 
  <div> 
   <pre><code class="lang-sql">SHOW datashares;</code></pre> 
  </div> </li> 
</ol> 
<p>You should be able to see <code>worldwide_event_test_data</code>, as shown in the following screenshot.</p> 
<ol start="10"> 
 <li>Note the <code>producer_namespace</code> and <code>producer_account</code> values from the output, which we use in the next step.<img alt="" class="alignnone size-full wp-image-31265" height="453" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/06/27/image009-2.jpg" style="margin: 10px 0px 10px 0px;" width="2006" /></li> 
 <li>Choose <strong>Add markdown</strong> and enter <code>Create database from datashare with producer_namespace and procedure_account.</code></li> 
 <li>Choose <strong>Add SQL</strong> and enter the following code to <a href="https://docs.aws.amazon.com/redshift/latest/dg/r_CREATE_DATABASE.html" rel="noopener noreferrer" target="_blank">create a database to access the datashare</a>. Use the <code>producer_namespace</code> and <code>producer_account</code> values you copied earlier. 
  <div> 
   <pre><code class="lang-sql">CREATE DATABASE ml_blog_db FROM DATASHARE worldwide_event_test_data OF ACCOUNT 'producer_account' NAMESPACE 'producer_namespace';</code></pre> 
  </div> </li> 
 <li>Choose <strong>Add markdown</strong> and enter <code>Create new table to consolidate features</code>.</li> 
 <li>Choose <strong>Add SQL</strong> and enter the following code to create a new table called event consisting of the event sales by date and assign a running serial number to split into training and validation datasets: 
  <div> 
   <pre><code class="lang-sql">CREATE TABLE event AS
	SELECT eventname, qtysold, saletime, day, week, month, qtr, year, holiday, ROW_NUMBER() OVER (ORDER BY RANDOM()) r
	FROM "ml_blog_db"."public"."sales" s
	INNER JOIN "ml_blog_db"."public"."event" e
	ON s.eventid = e.eventid
	INNER JOIN "ml_blog_db"."public"."date" d
	ON s.dateid = d.dateid;</code></pre> 
  </div> </li> 
</ol> 
<p>Event name, quantity sold, sale time, day, week, month, quarter, year, and holiday are columns in the dataset from AWS Data Exchange that are used as features in the ML model creation.</p> 
<ol start="15"> 
 <li>Choose <strong>Add markdown</strong> and enter <code>Split the dataset into training dataset and validation dataset</code>.</li> 
 <li>Choose <strong>Add SQL</strong> and enter the following code to split the dataset into training and validation datasets: 
  <div> 
   <pre><code class="lang-sql">CREATE TABLE training_data AS 
	SELECT eventname, qtysold, saletime, day, week, month, qtr, year, holiday
	FROM event
	WHERE r &gt;
	(SELECT COUNT(1) * 0.2 FROM event);

	CREATE TABLE validation_data AS 
	SELECT eventname, qtysold, saletime, day, week, month, qtr, year, holiday
	FROM event
	WHERE r &lt;=
	(SELECT COUNT(1) * 0.2 FROM event);</code></pre> 
  </div> </li> 
 <li>Choose <strong>Add markdown</strong> and enter <code>Create ML model</code>.</li> 
 <li>Choose <strong>Add SQL</strong> and enter the following command to create the model. Replace the <code>your_s3_bucket</code> parameter with your bucket name. 
  <div> 
   <pre><code class="lang-sql">CREATE MODEL predict_ticket_sold
	FROM training_data
	TARGET qtysold
	FUNCTION predict_ticket_sold
	IAM_ROLE 'default'
	PROBLEM_TYPE regression
	OBJECTIVE 'mse'
	SETTINGS (s3_bucket 'your_s3_bucket',
	s3_garbage_collect off,
	max_runtime 5000);</code></pre> 
  </div> </li> 
</ol> 
<p><strong>Note:</strong> It can take up to two hours to create and train the model.<br /> The following screenshot shows the example output from adding our markdown and SQL.<br /> <img alt="" class="alignnone size-full wp-image-31267" height="937" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/06/27/image010-1.jpg" style="margin: 10px 0px 10px 0px;" width="2005" /></p> 
<ol start="19"> 
 <li>Choose <strong>Add markdown</strong> and enter <code>Show model creation status</code>. Continue to next step once the Model State has changed to Ready.</li> 
 <li>Choose <strong>Add SQL</strong> and enter the following command to get the status of the model creation: 
  <div> 
   <pre><code class="lang-sql">SHOW MODEL predict_ticket_sold;</code></pre> 
  </div> <p><img alt="" class="alignnone wp-image-31268" height="465" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/06/27/image011-1.jpg" style="margin: 10px 0px 10px 0px;" width="1074" /></p></li> 
</ol> 
<p>Move to the next step after the <strong>Model State</strong> has changed to <strong>READY</strong>.</p> 
<ol start="21"> 
 <li>Choose<strong> Add markdown</strong> and enter <code>Run the inference for eventname Jason Mraz.</code></li> 
 <li>When the model is ready, you can use the SQL function to apply the ML model to your data. The following is sample SQL code to predict the tickets sold for a particular event using the <code>predict_ticket_sold</code> function created in the previous step: 
  <div> 
   <pre><code class="lang-sql">SELECT eventname,
	predict_ticket_sold(
	eventname, saletime, day, week, month, qtr, year, holiday ) AS predicted_qty_sold,
	day, week, month
	FROM event
	Where eventname = 'Jason Mraz';</code></pre> 
  </div> </li> 
</ol> 
<p>The following is the output received by applying the ML function <code>predict_ticket_sold</code> on the original dataset. The output of the ML function is captured in the field <code>predicted_qty_sold</code>, which is the predicted ticket sold quantity.</p> 
<p><img alt="" class="alignnone size-full wp-image-31269" height="549" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/06/27/image012-2.jpg" style="margin: 10px 0px 10px 0px;" width="1013" /></p> 
<h2>Share notebooks</h2> 
<p>To share the notebooks, complete the following steps:</p> 
<ol> 
 <li>Create an IAM role with the managed policy <a href="https://docs.aws.amazon.com/redshift/latest/mgmt/query-editor-v2-getting-started.html#query-editor-v2-configure" rel="noopener noreferrer" target="_blank"><code>AmazonRedshiftQueryEditorV2FullAccess</code></a> attached to the role.</li> 
 <li>Add a principal tag to the role with the tag name <code>sqlworkbench-team</code>.</li> 
 <li>Set the value of this tag to the principal (user, group, or role) you’re granting access to.</li> 
 <li>After you configure these permissions, navigate to the Amazon Redshift console and choose <a href="https://docs.aws.amazon.com/redshift/latest/mgmt/query-editor-v2-using.html" rel="noopener noreferrer" target="_blank"><strong>Query editor v2 </strong></a>in the navigation pane. If you haven’t used the query editor v2 before, please&nbsp;<a class="c-link" href="https://docs.aws.amazon.com/redshift/latest/mgmt/query-editor-v2-getting-started.html" rel="noopener noreferrer" target="_blank">configure</a>&nbsp;your account to use query editor v2.</li> 
 <li>Choose <strong>Notebooks</strong> in the left pane and navigate to <strong>My notebooks</strong>.<img alt="" class="alignnone size-full wp-image-31271" height="587" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/06/27/image014-1.jpg" style="margin: 10px 0px 10px 0px;" width="1288" /></li> 
 <li>Right-click on the notebook you want to share and choose <strong>Share with my team</strong>.<img alt="" class="alignnone size-full wp-image-31272" height="579" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/06/27/image015-2.jpg" style="margin: 10px 0px 10px 0px;" width="1289" /></li> 
 <li>You can confirm that the notebook is shared by choosing <strong>Shared to my team</strong> and checking that the notebook is listed.<img alt="" class="alignnone size-full wp-image-31273" height="583" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/06/27/image016-1.jpg" style="margin: 10px 0px 10px 0px;" width="1289" /></li> 
</ol> 
<h2>Summary</h2> 
<p>In this post, we showed you how to build an end-to-end pipeline by subscribing to a public dataset through AWS Data Exchange, simplifying data integration and processing, and then running prediction using Redshift ML on the data.</p> 
<p>We look forward to hearing from you about your experience. If you have questions or suggestions, please leave a comment.</p> 
<hr /> 
<h3>About the Authors</h3> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/06/29/Yadgiri-P.png"><img alt="" class="size-full wp-image-31332 alignleft" height="112" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/06/29/Yadgiri-P.png" width="100" /></a>Yadgiri Pottabhathini</strong> is a Sr. Analytics Specialist Solutions Architect. His role is to assist customers in their cloud data warehouse journey and help them evaluate and align their data analytics business objectives with Amazon Redshift capabilities.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/06/15/image072.png"><img alt="" class="size-full wp-image-30918 alignleft" height="129" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/06/15/image072.png" width="100" /></a>Ekta Ahuja</strong> is an Analytics Specialist Solutions Architect at AWS. She is passionate about helping customers build scalable and robust data and analytics solutions. Before AWS, she worked in several different data engineering and analytics roles. Outside of work, she enjoys baking, traveling, and board games.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/11/04/BP-Yau.jpg"><img alt="" class="size-full wp-image-13156 alignleft" height="106" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/11/04/BP-Yau.jpg" width="100" /></a>BP Yau</strong> is a Sr Product Manager at AWS. He is passionate about helping customers architect big data solutions to process data at scale. Before AWS, he helped Amazon.com Supply Chain Optimization Technologies migrate its Oracle data warehouse to Amazon Redshift and build its next generation big data analytics platform using AWS technologies.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/04/01/Srikanth-sopirala.png"><img alt="" class="size-full wp-image-27450 alignleft" height="119" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/04/01/Srikanth-sopirala.png" width="100" /></a>Srikanth Sopirala</strong> is a Principal Analytics Specialist Solutions Architect at AWS. He is a seasoned leader with over 20 years of experience, who is passionate about helping customers build scalable data and analytics solutions to gain timely insights and make critical business decisions. In his spare time, he enjoys reading, spending time with his family, and road cycling.</p>
