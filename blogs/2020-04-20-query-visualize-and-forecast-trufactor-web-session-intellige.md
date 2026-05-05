---
title: "Query, visualize, and forecast TruFactor web session intelligence with AWS Data Exchange"
url: "https://aws.amazon.com/blogs/big-data/query-visualize-and-forecast-trufactor-web-session-intelligence-with-aws-data-exchange/"
date: "Mon, 20 Apr 2020 16:16:30 +0000"
author: "Jay Park"
feed_url: "https://aws.amazon.com/blogs/big-data/category/analytics/aws-data-exchange/feed/"
---
<p>Given the infinite nature of data, finding the right data set to gain business insights can be a challenge. You can improve your business by having access to a central repository of various data sets to query, visualize, and forecast. With <a href="https://aws.amazon.com/data-exchange/" rel="noopener noreferrer" target="_blank">AWS Data Exchange</a>, finding the right data set has become much simpler. As an example, you can use data sets on web session visitation and demographics to help you understand which demographic groups visit your website most frequently. You can then improve your business through machine learning (ML) models and visitation forecasts.</p> 
<p>AWS Data Exchange makes it easy to find, subscribe to, and use third-party data in the cloud. After you subscribe to a data product within AWS Data Exchange, you can use the AWS Data Exchange API, <a href="http://aws.amazon.com/cli" rel="noopener noreferrer" target="_blank">AWS CLI</a>, or the <a href="http://aws.amazon.com/console" rel="noopener noreferrer" target="_blank">AWS Management Console</a> to load data into <a href="http://aws.amazon.com/s3" rel="noopener noreferrer" target="_blank">Amazon S3</a> directly. You can then analyze the imported data with a wide variety of AWS services, ranging from analytics to machine learning.</p> 
<p>This post showcases <a href="https://aws.amazon.com/marketplace/search/results?page=1&amp;filters=FulfillmentOptionType%2Cvendor_id&amp;FulfillmentOptionType=AWSDataExchange&amp;vendor_id=948e985d-d81e-4649-a1ec-79bdc2ba5bda" rel="noopener noreferrer" target="_blank">TruFactor Intelligence-as-a-Service data</a> on AWS Data Exchange. TruFactor’s anonymization platform and proprietary AI ingests, filters, and transforms more than 85 billion high-quality raw signals daily from wireless carriers, OEMs, and mobile apps into a unified <em>phygital</em> consumer graph across physical and digital dimensions. TruFactor intelligence is application-ready for use within any AWS analytics or ML service to power your models and applications running on AWS, with no additional processing required. Common use cases include the following:</p> 
<ul> 
 <li> <strong>Consumer segmentation</strong> – Web intelligence on internet browsing behavior in the US provides a complete view of the consumer, including interests, opinions, values, digital behavior, and sentiment, to inform segmentation of your customers and those of your competitors.</li> 
 <li> <strong>Customer acquisition or churn campaigns</strong> – Internet browsing behavior can identify affinity properties for new prospects as well as switching to competitors’ websites.</li> 
</ul> 
<p>This walkthrough uses TruFactor’s <a href="https://aws.amazon.com/marketplace/pp/prodview-bbsc65s6e4vnm?qid=1586212005676&amp;sr=0-2&amp;ref_=srh_res_product_title#overview" rel="noopener noreferrer" target="_blank">Daily Mobile Web Session Index</a> and <a href="https://aws.amazon.com/marketplace/pp/prodview-lsbj3dfcgkutq?qid=1586212161560&amp;sr=0-2&amp;ref_=srh_res_product_title" rel="noopener noreferrer" target="_blank">Daily Demographics by Mobile Web Sessions</a> data sets, which are both available for free subscription through the AWS Data Exchange console. While there are commercial data sets available for purchase in AWS Data Exchange, this post uses trial data sets to showcase the breadth and depth of analytics possible with TruFactor’s intelligence.</p> 
<p>This TruFactor intelligence is aggregated on over 3 billion records from telco carrier networks and mobile apps per day, originating from approximately 30 million consistent users, distilled into session-level information that provides a complete view of user digital interests. The accuracy, breadth of data provided, and the persistency of the panel deliver a unified view of consumers that can inform insights or power analytic models or applications on AWS.</p> 
<p>These two data sets have applications across verticals such as retail, financial services, and advertising. Common use cases include creating detailed customer segmentation (for example, full DNA maps of consumers based on visits to specific web HTTP hosts), identifying affinity properties, and estimating demand for apps or services. This intelligence is also ideal for identifying trends and changes over time.</p> 
<h2>Solution overview</h2> 
<p>The following diagram illustrates the architecture of the solution.</p> 
<p><img alt="" class="alignnone size-full wp-image-9586" height="448" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange1.png" style="margin: 20px 0px 20px 0px;" width="800" /></p> 
<p>The workflow is comprised of the following steps:</p> 
<ol> 
 <li>Subscribe to a data set from AWS Data Exchange and export to Amazon S3</li> 
 <li>Run an <a href="https://aws.amazon.com/glue" rel="noopener noreferrer" target="_blank">AWS Glue</a> crawler to load product data</li> 
 <li>Perform queries with <a href="http://aws.amazon.com/athena" rel="noopener noreferrer" target="_blank">Amazon Athena</a> </li> 
 <li>Visualize the queries and tables with <a href="https://aws.amazon.com/quicksight" rel="noopener noreferrer" target="_blank">Amazon QuickSight</a> </li> 
 <li>Run an ETL job with AWS Glue</li> 
 <li>Create a time series forecast with <a href="https://aws.amazon.com/forecast/" rel="noopener noreferrer" target="_blank">Amazon Forecast</a> </li> 
 <li>Visualize the forecasted data with Amazon QuickSight</li> 
</ol> 
<p>This post looks at the demographic distributions across various websites and how to use ML to forecast website visitation.</p> 
<h2>Walkthrough overview</h2> 
<p>The walkthrough includes the following steps:</p> 
<ol> 
 <li>Subscribe to a TruFactor data set from the AWS Data Exchange console and export the data set to Amazon S3</li> 
 <li>Use an AWS Glue crawler to load the product data into an AWS Glue Data Catalog</li> 
 <li>Use Amazon Athena for SQL querying</li> 
 <li>Visualize the query views and tables with Amazon QuickSight</li> 
 <li>Use AWS Glue jobs to extract, transform, and load your data for forecasting with Amazon Forecast</li> 
 <li>Use Amazon Forecast to create a time series forecast of the transformed data</li> 
 <li>Visualize the forecasted web visitation data with Amazon QuickSight</li> 
</ol> 
<p>You do not have to perform additional processing or manipulation of the TruFactor intelligence for this walkthrough.</p> 
<h2>The data sets</h2> 
<p>The TruFactor data sets this post uses are in Parquet format and snappy compression. The following section provides additional details and schema for each data set.</p> 
<h3>TruFactor Daily Mobile Web Session Index (US – Nationwide) — Trial</h3> 
<p>The <a href="https://aws.amazon.com/marketplace/pp/prodview-bbsc65s6e4vnm?qid=1586212005676&amp;sr=0-2&amp;ref_=srh_res_product_title#overview" rel="noopener noreferrer" target="_blank">TruFactor Daily Mobile Web Session Index (US – Nationwide) — Trial</a> data set provides aggregate information per HTTP host as a view of the internet browsing behavior in the US. TruFactor generates the data from high-quality packet layer data sourced from mobile carriers that includes the mobile internet traffic originating from a user’s device. TruFactor derives the projected counts from observed counts that are filtered for exclusion and anonymized to make sure users cannot be re-identified. It extrapolates values from US Census data using a proprietary algorithm. For the avoidance of doubt, this data set does not include user-level data.</p> 
<p>The following screenshot shows the schema for the mobile web session data set by HTTP host, session time, MB transferred, number of events, sessions, users, and dates.</p> 
<p><img alt="" class="alignnone size-full wp-image-9587" height="181" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange2.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" /></p> 
<h3>TruFactor Daily Demographics by Mobile Web Session (US) — Trial</h3> 
<p>The <a href="https://aws.amazon.com/marketplace/pp/prodview-lsbj3dfcgkutq?qid=1586212161560&amp;sr=0-2&amp;ref_=srh_res_product_title" rel="noopener noreferrer" target="_blank">TruFactor Daily Demographics by Mobile Web Session (US) — Trial</a> data set includes aggregate demographics: a projected distribution of users per HTTP host as a view of the internet browsing behavior in the US. TruFactor generates the data from high-quality packet layer data sourced from mobile carriers that includes the mobile internet traffic originating from a user’s device. TruFactor derives the distribution from observed counts that are filtered for exclusion and anonymized to make sure users cannot be re-identified. It extrapolates values from US Census data using a proprietary algorithm. Demographics include gender, age range, ethnicity, and income range.</p> 
<p>The following screenshot shows the partial schema for the demographics by web session data set. The full schema includes the following attributes: HTTP host, age ranges, genders, ethnicity, income ranges, and date.</p> 
<p><img alt="" class="alignnone size-full wp-image-9588" height="202" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange3.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" /></p> 
<h2>Prerequisites</h2> 
<p>To complete this walkthrough successfully, you must have the following resources:</p> 
<ul> 
 <li>An AWS account.</li> 
 <li>Familiarity with AWS core services and concepts.</li> 
 <li>The ability to launch new resources in your account. Some resources may not be eligible for Free Tier usage and might incur costs.</li> 
 <li>Subscription to TruFactor’s Daily Mobile Web Session Index (US – Nationwide) – Trial and Daily Demographics by Mobile Web Session (US) – Trial data sets. For instructions on subscribing to a data set on AWS Data Exchange, see <a href="https://aws.amazon.com/blogs/aws/aws-data-exchange-find-subscribe-to-and-use-data-products/" rel="noopener noreferrer" target="_blank">AWS Data Exchange – Find, Subscribe To, and Use Data Products</a>.</li> 
</ul> 
<h2>Using AWS Data Exchange, Amazon S3, AWS Glue, Amazon Athena, and Amazon QuickSight</h2> 
<p>This section examines the key demographics of visitors to the top seven e-commerce websites. This information can help you understand which demographic groups are visiting your website most frequently and also help you target ads and cater to certain demographics groups. You use AWS Glue crawlers to crawl your data sets in Amazon S3, populate your AWS Glue Data Catalog, query the AWS Glue Data Catalog using Amazon Athena, and use Amazon QuickSight to visualize the queries.</p> 
<h3>Step 1: Exporting the data from AWS Data Exchange to Amazon S3</h3> 
<p>To export your TruFactor data set subscriptions into an Amazon S3 bucket, complete the following steps:</p> 
<ol> 
 <li>Create an Amazon S3 bucket in your working account. For the purposes of our demo, we have named our S3 bucket <code>trufactor-data-exchange-bucket</code>.</li> 
 <li>Create two folders within the S3 bucket: <code>web_sess</code> and <code>demo_by_web_sess</code>.</li> 
</ol> 
<p style="padding-left: 30px;">This post uses a trial data set with a sample of 14 days. A paid subscription to TruFactor’s Web Sessions data on AWS Data Exchange includes 6 months of historical data, which refreshes daily.</p> 
<p style="padding-left: 30px;">The following screenshot shows the two folders within the S3 bucket.<img alt="" class="alignnone size-full wp-image-9589" height="287" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange4.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" />You are now ready to export the data sets.</p> 
<ol start="3"> 
 <li>On the AWS Data Exchange console, under <strong>Subscriptions</strong>, locate <strong>TruFactor Daily Mobile Web Sessions Index (US – Nationwide) – Trial</strong>.</li> 
 <li>Under <strong>Revisions</strong>, choose the most recent <strong>Revision ID</strong>.</li> 
 <li>Choose all assets except the <code>manifest.json</code> files.</li> 
 <li>Choose <strong>Export to Amazon S3</strong>.</li> 
 <li>In the window that opens, choose the S3 bucket and folder to export the product data into. 
  <ul> 
   <li>Export all the assets into the S3 bucket’s <code>web_sess</code> folder.</li> 
  </ul> </li> 
 <li>Repeat the previous steps for the <strong>TruFactor Daily Demographics by Mobile Web Sessions (US) – Trial</strong> data set, with the following change: 
  <ul> 
   <li>Export the assets into the <code>demo_by_web_sess</code> folder in your S3 bucket.</li> 
  </ul> </li> 
 <li>Check to make sure you successfully imported the TruFactor data sets in the <strong>Overview</strong>. The following screenshot shows that the data sets are partitioned into folders by date. Each folder contains Parquet files of web session data for each day.<img alt="" class="alignnone size-full wp-image-9590" height="405" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange5.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" /> </li> 
</ol> 
<h3>Step 2: Populating your AWS Glue Data Catalog with the TruFactor data sets</h3> 
<p>Now that you have successfully exported the TruFactor data sets into an Amazon S3 bucket, you create and run an AWS Glue crawler to crawl your Amazon S3 bucket and populate the AWS Glue Data Catalog. Complete the following steps:</p> 
<ol> 
 <li>On the AWS Glue console, under <strong>Data Catalog</strong>, choose <strong>Crawlers</strong>.</li> 
 <li>Choose <strong>Add crawler</strong>.</li> 
 <li>For <strong>Crawler name</strong>, enter a name; for example, <code>trufactor-data-exchange-crawler</code>.<img alt="" class="alignnone size-full wp-image-9591" height="267" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange6.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" /> </li> 
 <li>For <strong>Crawler source type</strong>, choose <strong>Data stores</strong>.</li> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>For <strong>Choose a data store</strong>, choose <strong>S3</strong>.</li> 
 <li>For <strong>Crawl data in</strong>, select <strong>Specified path in my account</strong>.</li> 
 <li>For <strong>Include path</strong>, enter the path for the <code>web_sess</code> data set folder. The crawler points to the following path: <code>s3://&lt;trufactor-data-exchange-bucket&gt;/web_sess</code>.</li> 
 <li>Choose <strong>Next</strong>.<img alt="" class="alignnone size-full wp-image-9592" height="340" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange7.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" /> </li> 
 <li>Select <strong>Yes</strong> to <strong>Add another data store</strong>.</li> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>For <strong>Include path</strong>, enter the path for the <code>demo_by_web_sess</code> data set folder. The crawler points to the following path: <code>s3://&lt;trufactor-data-exchange-bucket&gt;/demo_by_web_sess</code>.</li> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>In the <strong>Choose an IAM role</strong> section, select <strong>Create an IAM role</strong>. This is the role that the AWS Glue crawler and AWS Glue jobs use to access the Amazon S3 bucket and its content.</li> 
 <li>For IAM role, enter the suffix <code>demo-data-exchange</code>.</li> 
 <li>Choose <strong>Next</strong>.<img alt="" class="alignnone size-full wp-image-9593" height="389" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange8.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" /> </li> 
 <li>In the <strong>schedule </strong>section, leave the <strong>Frequency </strong>with the default <strong>Run on Demand</strong>.</li> 
 <li>Choose<strong> Next</strong>.</li> 
 <li>In the <strong>Output </strong>section, choose <strong>Add database</strong>.</li> 
 <li>Enter a name for the database; for example, <code>trufactor-db</code>.</li> 
 <li>Choose <strong>Next, </strong>then choose <strong>Finish</strong>.<img alt="" class="alignnone size-full wp-image-9594" height="326" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange9.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" />This database contains the tables that the crawler discovers and populates. With these data sets separated into different tables, you join and relationalize the data.</li> 
</ol> 
<ol start="22"> 
 <li>In the <strong>Review all steps</strong> section, review the crawler settings and choose <strong>Finish</strong>.</li> 
 <li>Under <strong>Data Catalog</strong>, choose <strong>Crawlers</strong>.</li> 
 <li>Select the crawler you just created.</li> 
 <li>Choose <strong>Run crawler</strong>.<img alt="" class="alignnone size-full wp-image-9596" height="211" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange10.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" />The AWS Glue crawler crawls the data sources and populates your AWS Glue Data Catalog. This process can take up to a few minutes. When the crawler is finished, you can see two tables added to your crawler details. See the following screenshot.<img alt="" class="alignnone size-full wp-image-9597" height="209" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange11.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" />You can now view your new tables.</li> 
</ol> 
<ol start="26"> 
 <li>Under <strong>Databases</strong>, choose <strong>Tables</strong>.</li> 
 <li>Choose your database.</li> 
 <li>Choose <strong>View the tables</strong>. The table names correspond to the Amazon S3 folder directory you used to point your AWS Glue crawler. See the following screenshot.<img alt="" class="alignnone size-full wp-image-9598" height="202" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange12.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" /> </li> 
</ol> 
<h3>Step 3: Querying the data using Amazon Athena</h3> 
<p>After you populate the AWS Glue Data Catalog with TruFactor’s Mobile Web Session and Demographics data, you can use Amazon Athena to run SQL queries and create views for visualization. Complete the following steps:</p> 
<ol> 
 <li>On the Amazon Athena console, choose <strong>Query Editor</strong>.</li> 
 <li>On the <strong>Database </strong>drop-down menu, choose the database you created.</li> 
 <li>To preview one of the tables in Amazon Athena, choose <strong>Preview table</strong>.<br /> On the <strong>Results</strong> section, you should see 10 records from the web_sess table. See the following screenshot.<img alt="" class="alignnone size-full wp-image-9599" height="368" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange13.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" />In this next step, you run a query that creates a view of the Web Session Index and Demographics data across a group of e-commerce HTTP hosts. This is broken down by the percentage of users categorized by age and gender, number of users, MB transferred, and number of sessions ordered by date.</li> 
 <li>Run the following SQL query in Amazon Athena: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">CREATE OR REPLACE VIEW e_commerce_web_sess_data AS 
SELECT
  "date_parse"("a"."partition_0", '%Y%m%d') "date",
  "a"."http_host",
  "a"."users",
  "a"."mb_transferred",
  "a"."number_of_sessions",
  "b"."18_to_25",
  "b"."26_to_35",
  "b"."36_to_45",
  "b"."46_to_55",
  "b"."56_to_65",
  "b"."66_to_75",
  "b"."76_plus",
  "b"."male",
  "b"."female"
FROM  
  ((
   SELECT
     "partition_0",
     "http_host",
     "users",
     "mb_transferred",
     "number_of_sessions"
   FROM
     "trufactor-db"."web_sess"
   WHERE ("http_host" IN ('www.amazon.com', 'www.walmart.com', 'www.ebay.com', 'www.aliexpress.com', 'www.etsy.com', 'www.rakuten.com', 'www.craigslist.com'))
)  a
LEFT JOIN (
   SELECT
     "http_host" "http_host_2",
     "partition_0" "partition_2",
     "age_ranges"."18_to_25",
     "age_ranges"."26_to_35",
     "age_ranges"."36_to_45",
     "age_ranges"."46_to_55",
     "age_ranges"."56_to_65",
     "age_ranges"."66_to_75",
     "age_ranges"."76_plus",
     "genders"."male",
     "genders"."female"
   FROM
     "trufactor-db"."demo_by_web_sess"
   WHERE ("http_host" IN ('www.amazon.com', 'www.walmart.com', 'www.ebay.com', 'www.aliexpress.com', 'www.etsy.com', 'www.rakuten.com', 'www.craigslist.com'))
)  b ON (("a"."http_host" = "b"."http_host_2") AND ("a"."partition_0" = "b"."partition_2")))
ORDER BY "date" ASC</code></pre> 
  </div> </li> 
 <li>After you create the view, you can preview it by repeating the above steps for previewing a table. The following screenshot shows the results, which include the number of users, user percentages by age group and gender, and a list of e-commerce hosts listed by date.<img alt="" class="alignnone size-full wp-image-9600" height="257" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange14.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" /> </li> 
</ol> 
<h3>Step 4: Visualizing with Amazon QuickSight</h3> 
<p>After you query your data sets in Amazon Athena, you can use Amazon QuickSight to visualize your results. You must first grant Amazon QuickSight access to the Amazon S3 bucket that holds the TruFactor data sets, which you can do through the <strong>Manage QuickSight </strong>setting on the Amazon QuickSight console. After you grant access to the Amazon S3 bucket, you visualize the tables and queries with Amazon QuickSight. Complete the following steps:</p> 
<ol> 
 <li>In the Amazon QuickSight console, choose <strong>New Analysis</strong>.</li> 
 <li>Choose <strong>New data set</strong>.</li> 
 <li>Choose <strong>Athena </strong>as the data source.</li> 
 <li>For <strong>Data source name</strong>, enter <code>trufactor-data-exchange-source</code>.</li> 
 <li>From the drop-down menu, choose the database and view you created.</li> 
 <li>Choose <strong>Directly query your data</strong>.</li> 
 <li>Choose <strong>Visualize</strong>. Because TruFactor intelligence is application-ready, you can gain immediate insights by using Amazon Athena to query and Amazon QuickSight to visualize. This post includes visualizations of the data set for the first two weeks of October 2019. The following graph visualizes the number of users on different HTTP hosts.<img alt="" class="alignnone size-full wp-image-9601" height="417" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange15.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" />The following pie charts further filter the HTTP hosts by age range.<img alt="" class="alignnone size-full wp-image-9602" height="313" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange16.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" /><img alt="" class="alignnone size-full wp-image-9603" height="313" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange17.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" />The following bar chart offers another visualization of users by age range.<img alt="" class="alignnone size-full wp-image-9604" height="346" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange18.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" />You could add other fields such as income range, ethnicity, and gender.</li> 
</ol> 
<h2>Running AWS Glue Jobs and Amazon Forecast</h2> 
<p>This section discusses how to use AWS Glue jobs to query and export your data set for forecasting with Amazon Forecast. This walkthrough examines the amount of users’ visitation over 14 days across the top 50 HTTP hosts ranked by users’ visitation. From there, you forecast the users’ visitation for these HTTP hosts for the next three days.</p> 
<h3>Step 1: Creating and running an AWS Glue job</h3> 
<p>To create and run your AWS Glue job, complete the following steps:</p> 
<ol> 
 <li>On the AWS Glue console, under <strong>ETL</strong>, choose <strong>Jobs</strong>.</li> 
 <li>Choose <strong>Add job</strong>.</li> 
 <li>For <strong>Name</strong>, enter a name for the AWS Glue job; for example, <code>demo-glue-job</code>.</li> 
 <li>For <strong>Type </strong>and <strong>Glue version</strong>, keep the default values.</li> 
 <li>For <strong>This job runs</strong>, select <strong>A new script to be authored by you</strong>.<img alt="" class="alignnone size-full wp-image-9606" height="416" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange19.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" /> </li> 
 <li>In the <strong>Security configuration, script libraries, and job parameters (optional) </strong>section, set the <strong>Maximum capacity </strong>cluster size to 2. This reduces the cost of running the AWS Glue job. By default, the cluster size is set to 10 Data Processing Units (DPU).<img alt="" class="alignnone size-full wp-image-9607" height="313" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange20.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" /> </li> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>In the <strong>Connections</strong> section, keep the default values.</li> 
 <li>Choose <strong>Save job and edit script</strong>.</li> 
 <li>Enter the following code in the script section, and replace <code>YOUR_BUCKET_NAME</code> on line 42 with the name of your bucket. 
  <div class="hide-language"> 
   <pre><code class="lang-python">import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.dynamicframe import DynamicFrame
from awsglue.job import Job
from pyspark.sql import SparkSession
from pyspark.sql.functions import udf
from pyspark.sql.types import StringType

## @params: [JOB_NAME]
args = getResolvedOptions(sys.argv, ['JOB_NAME'])

sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

db_name = "trufactor-db"
tbl_name = "web_sess"

web_sess_dyf = glueContext.create_dynamic_frame.from_catalog(database = db_name, table_name = tbl_name, transformation_ctx = "web_sess_dyf")
web_sess_df = web_sess_dyf.toDF()
web_sess_df.createOrReplaceTempView("webSessionTable")
web_sess_sql_df = spark.sql("""
SELECT to_date(partition_0, 'yyyyMMdd') AS date,
         http_host,
         users
FROM 
    (SELECT partition_0,
         http_host,
         users,
         row_number()
        OVER ( PARTITION By partition_0
    ORDER BY users DESC ) AS rn
    FROM webSessionTable )
WHERE rn&lt;=50
ORDER BY date""")

web_sess_sql_df.coalesce(1).write.format("csv").option("header","false").save("s3://YOUR_BUCKET_NAME/amazon_forecast_demo/dataset/sampleset")
job.commit()</code></pre> 
  </div> <p>This code queries the top 50 HTTP hosts, ranked by users’ visitation during the first half of October and returns the users, date, and HTTP hosts columns. The query results upload to your Amazon S3 bucket in CSV format (you need the files in CSV to use Amazon Forecast).</p></li> 
 <li>Choose <strong>Save</strong> and close the AWS Glue job screen.<img alt="" class="alignnone size-full wp-image-9608" height="357" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange21.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" />Before you can run the AWS Glue job, you need to modify the IAM role associated with AWS Glue. Currently, the IAM role only has permission to get and put objects in the directories you specified earlier. You need to update the IAM policy to allow permission to get and put objects in all subdirectories of the Amazon S3 bucket.</li> 
 <li>On the IAM console, choose the role you used for this walkthrough: <code>AWSGlueServiceRole-demo-data-exchange</code>.</li> 
 <li>In the <strong>Summary </strong>section for the IAM role, on the <strong>Permissions</strong> tab, choose the IAM policy associated with the <strong>Managed policy</strong>.<img alt="" class="alignnone size-full wp-image-9609" height="387" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange22.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" /> </li> 
 <li>Choose <strong>Edit policy</strong>.</li> 
 <li>Change the view from <strong>Visual editor </strong>to <strong>JSON</strong>.</li> 
 <li>Within this JSON object, under <strong>Resource</strong>, add another resource into the list of values. The following code is the updated IAM policy: 
  <div class="hide-language"> 
   <pre><code class="lang-json">{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::trufactor-data-exchange-bucket/web_sess*",
                "arn:aws:s3:::trufactor-data-exchange-bucket/demo_by_web_sess*",
                "arn:aws:s3:::trufactor-data-exchange-bucket/*"
            ]
        }
    ]
}</code></pre> 
  </div> </li> 
 <li>Choose <strong>Review policy </strong>and <strong>Save changes</strong>.</li> 
 <li>On the AWS Glue console, under <strong>ETL</strong>, choose <strong>Jobs</strong>. Select the job you created earlier.</li> 
 <li>From the <strong>Action</strong> drop-down menu, choose <strong>Run job</strong>. On the History tab, you can see when the status changes to <code>Succeeded</code>. See the following screenshot.<img alt="" class="alignnone size-full wp-image-9610" height="351" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange23.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" />This job can take 15–20 minutes to complete.</li> 
</ol> 
<h3>Step 2: Creating a dataset group, training a predictor, and creating forecasts in Amazon Forecast</h3> 
<p>To create your dataset group, train a predictor, and create forecasts, complete the following steps:</p> 
<ol> 
 <li>On the Amazon Forecast console, choose <strong>View dataset groups</strong>.</li> 
 <li>Choose <strong>Create dataset group</strong>.</li> 
 <li>For <strong>Dataset group name</strong>, enter a name; for example, <code>users_visitation_sample_dataset_group</code>.</li> 
 <li>For <strong>Forecasting domain</strong>, choose <strong>Web traffic</strong>.</li> 
 <li>Choose <strong>Next</strong>.<img alt="" class="alignnone size-full wp-image-9611" height="405" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange24.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" /> </li> 
 <li>On the <strong>Create target time series dataset</strong> page, for <strong>Dataset name</strong>, enter the name of your dataset; for example, <code>users_visitation_sample_dataset</code>.</li> 
 <li>For <strong>Frequency of your data</strong>, choose <strong>1 day</strong>.</li> 
 <li>For <strong>Data schema</strong>, update the data schema JSON object with the following code: 
  <div class="hide-language"> 
   <pre><code class="lang-json">{
  "Attributes":[
    {
      "AttributeName": "timestamp",
      "AttributeType": "timestamp"
    },
    {
      "AttributeName": "item_id",
      "AttributeType": "string"
    },
    {
      "AttributeName": "value",
      "AttributeType": "float"
    }
  ]
}</code></pre> 
  </div> </li> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>On the <strong>Import target time series data </strong>page, for <strong>Dataset import name</strong>, enter your dataset name; for example, <code>users_visitation_sample_dataset_import</code>.</li> 
 <li>For <strong>Timestamp format</strong>, enter <code>yyyy-MM-dd</code>.</li> 
 <li>For <strong>IAM Role</strong>, create a new role and grant Amazon Forecast access to the S3 bucket that you are using for this demo.</li> 
 <li>For <strong>Data Location</strong>, use the S3 path that you exported your CSV file to after the AWS Glue job: <code>s3://&lt;trufactor-data-exchange-bucket&gt;/amazon_forecast_demo/dataset/sampleset</code>.</li> 
 <li>Review the settings for import target time series data and choose <strong>Start import</strong>.<img alt="" class="alignnone size-full wp-image-9612" height="603" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange25.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" />The process of importing the data can take approximately 10 minutes. When the status changes to <code>Active</code>, you can begin training a predictor.</li> 
</ol> 
<ol start="15"> 
 <li>On the <strong>Dashboard</strong> page, choose <strong>Start</strong> next to <strong>Predictor training</strong>.</li> 
 <li>On the <strong>Train predictor</strong> page, for <strong>Predictor name</strong>, enter a name for the predictor; for example, <code>users_visitation_sample_dataset_predictor</code>.</li> 
 <li>For <strong>Forecast horizon</strong>, choose <strong>3</strong>.</li> 
 <li>For <strong>Forecast frequency</strong>, choose <strong>day</strong>.</li> 
 <li>For <strong>Algorithm selection</strong>, select <strong>Manual</strong>. If you use the other algorithm option, AutoML, you allow Amazon Forecast to choose the right algorithm based on a pre-defined objective function, which is not necessary for this walkthrough.</li> 
 <li>For <strong>Algorithm</strong>, choose <strong>Deep_AR_Plus</strong> (you use deep learning to forecast users’ visitation across 50 HTTP hosts).</li> 
 <li>Leave all other options with the default values.<img alt="" class="alignnone size-full wp-image-9613" height="673" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange26.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="650" /> </li> 
 <li>Review the settings and choose <strong>Train predictor</strong>. The predictor training process can take 20–30 minutes. When the training completes, the status changes to <code>Active</code>. To evaluate the predictor’s (ML model) accuracy, Amazon Forecast splits the input time series data into two data sets: <code>training</code> and <code>test</code>. This process tests a predictive model on historical data and is called <em>backtesting</em>. When it splits the input time series data, it maintains the data’s order, which is crucial for time series data. After training the dataset, Amazon Forecast calculates the root mean square error (RSME) and weighted quantile losses to determine how well the predictor performed. For more detailed information about backtesting and predictor metrics, see <a href="https://docs.aws.amazon.com/forecast/latest/dg/metrics.html" rel="noopener noreferrer" target="_blank">Evaluating Predictor Accuracy</a>. When the predictor is finished training, you can create a forecast.</li> 
 <li>On the <strong>Dashboard</strong> page, under <strong>Generate forecasts</strong>, choose <strong>Start</strong>.</li> 
 <li>For <strong>Forecast name</strong>, enter a forecast name; for example, <code>users_visitation_sample_forecast</code>.</li> 
 <li>For <strong>Predictor</strong>, choose your trained predictor.</li> 
 <li>For <strong>Forecast types</strong>, you can enter any values between 0.01 and 0.99 and the mean. These are percentage probabilities of satisfying the original demand. This post enters <code>.50, .90, .99, mean</code>.</li> 
 <li>Choose <strong>Create a forecast</strong>.<img alt="" class="alignnone size-full wp-image-9614" height="534" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange27.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="651" />The forecast creation process can take 15–20 minutes.</li> 
 <li>When the forecast is complete, choose <strong>Forecasts</strong>.<br /> You should see a single forecast. See the following screenshot.<br /> <img alt="" class="alignnone size-full wp-image-9616" height="220" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange28.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" />You can now export the generated forecast to a new folder within your existing Amazon S3 bucket for visualization with Amazon QuickSight.</li> 
</ol> 
<ol start="29"> 
 <li>Choose the newly generated forecast.</li> 
 <li>Under <strong>Exports</strong>, choose <strong>Create forecast export</strong>.</li> 
 <li>For <strong>Export name</strong>, enter a name for the export; for example, <code>users_visitation_sample_forecast_export</code>.</li> 
 <li>For <strong>Generated forecast</strong>, choose <strong>users_visitation_sample_forecast</strong>.</li> 
 <li>For <strong>IAM Role</strong>, choose the role you created earlier.</li> 
 <li>For <strong>S3 forecast export location</strong>, enter the S3 path to store the forecasts: <code>s3://&lt;trufactor-data-exchange-bucket&gt;/amazon_forecast_demo/forecasts/sampleset</code>.</li> 
 <li>Choose <strong>Create forecast export</strong>.<img alt="" class="alignnone size-full wp-image-9617" height="628" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange29.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="649" />The exporting process can take up to 5 minutes. Alternatively, you can visualize the user visitation forecasts for the 50 HTTP hosts directly through the Amazon Forecast console or Query API.</li> 
</ol> 
<h3>Step 3: Querying a view using Amazon Athena and downloading the forecast file</h3> 
<p>Before you visualize users’ visitation forecast data, create a view in Amazon Athena for the top 50 HTTP hosts ranked by users’ visitation over 14 days. Complete the following steps:</p> 
<ol> 
 <li>Run the following query in Amazon Athena: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">CREATE OR REPLACE VIEW "top_50_users" AS
SELECT date_format(date_parse(partition_0,
         '%Y%m%d'),'%Y-%m-%d') AS "date", http_host, users
FROM 
    (SELECT partition_0,
         http_host,
         users,
         row_number()
        OVER (PARTITION By partition_0
    ORDER BY  users DESC ) AS rn
    FROM "trufactor-db"."web_sess")
WHERE rn&lt;=50
ORDER BY date</code></pre> 
  </div> <p>The code queries the top 50 HTTP hosts ranked by users’ visitation sorted by date.</p></li> 
 <li>In the Amazon S3 console, navigate to the S3 bucket and directory holding the files: <code>s3://&lt;trufactor-data-exchange-bucket&gt;/amazon_forecast_demo/forecasts/sampleset</code>. The following screenshot shows three different files inside the folder.<img alt="" class="alignnone size-full wp-image-9618" height="372" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange30.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" /> </li> 
</ol> 
<ol start="3"> 
 <li>Download the CSV file.</li> 
</ol> 
<h3>Step 4: Visualizing in Amazon QuickSight</h3> 
<p>To visualize the data in Amazon QuickSight, complete the following steps:</p> 
<ol> 
 <li>On the Amazon QuickSight console, choose <strong>Manage data</strong>.</li> 
 <li>Choose <strong>New data set</strong>.</li> 
 <li>Choose <strong>Upload a file</strong>.</li> 
 <li>Upload the CSV file that you downloaded.</li> 
 <li>On the <strong>Confirm file upload settings</strong> page, choose <strong>Next</strong>.</li> 
 <li>Choose <strong>Visualize</strong>.</li> 
 <li>Return to the Amazon QuickSight console and choose <strong>Manage data</strong>.</li> 
 <li>Choose <strong>New data set</strong> for the top 50 HTTP hosts view you queried earlier.</li> 
 <li>On the <strong>Create a Data set </strong>page, find the data source you created earlier: <code>trufactor-data-exchange-source</code>.</li> 
 <li>From the drop-down list, choose the database and view you created.</li> 
 <li>Choose <strong>Directly query your data.</strong> </li> 
 <li>Choose <strong>Visualize</strong>.</li> 
 <li>On the new Amazon QuickSight analysis page, choose the pencil icon next to <strong>Data set</strong>.</li> 
 <li>Choose <strong>Add data set</strong>.</li> 
 <li>Choose the CSV file you uploaded.</li> 
</ol> 
<p>You now have a single Amazon QuickSight analysis with multiple data sets to visualize.</p> 
<p>The following graphs highlight the historical data for the users’ visitation across 50 HTTP hosts for the first two weeks of October and the mean forecast for users’ visitation for the next three days.</p> 
<p><img alt="" class="alignnone size-full wp-image-9619" height="264" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange31.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" /></p> 
<p>The following graphs highlight the historical data and forecasted P50, P90, and P99 quantile values for www.google.com.</p> 
<p><img alt="" class="alignnone size-full wp-image-9620" height="266" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/03/31/TruFactorDataExchange32.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" /></p> 
<p>Amazon Forecast makes it easier to get started with machine learning without having to create your own ML models from scratch. You can use this information to anticipate the web traffic for the upcoming week, which can aid in scaling your resources and applications accordingly.</p> 
<h2>Cleaning up</h2> 
<p>To avoid incurring future charges, delete the following resources that you created in this walkthrough:</p> 
<ul> 
 <li>The Amazon S3 bucket <code>trufactor-data-exchange-bucket</code> </li> 
 <li>The AWS Glue crawler <code>trufactor-data-exchange-crawler</code> </li> 
 <li>The AWS Glue job <code>demo-glue-job</code> </li> 
 <li>The AWS IAM role <code>AWSGlueServiceRole-demo-data-exchange</code> </li> 
 <li>The AWS Glue database <code>trufactor-db</code> </li> 
 <li>The Amazon QuickSight demo data sets and analysis</li> 
 <li>The following Amazon Forecast resources (in this order) for <code>users_visitation_sample_dataset_group</code> via the console: 
  <ul> 
   <li>Existing forecasts under <strong>Forecasts</strong> </li> 
   <li>Existing predictors under <strong>Predictors</strong> </li> 
   <li>Existing datasets under <strong>Datasets</strong> </li> 
  </ul> </li> 
</ul> 
<h2>Conclusion</h2> 
<p>This walkthrough detailed how to import a data set to Amazon S3 from AWS Data Exchange, use AWS Glue to run crawlers and an ETL job on the data, run SQL queries with Amazon Athena, create a time series forecast of the queried data with Amazon Forecast, and visualize the queried and forecasted data with Amazon QuickSight.</p> 
<p>This post used TruFactor Intelligence-as-a-Service, one of the AWS Data Exchange launch partners, to power this walkthrough. TruFactor intelligence on AWS Data Exchange highlighted the ease of loading directly into Amazon S3 and layering advanced AWS services.</p> 
<p>For more information about TruFactor and the AWS Data Exchange, see <a href="https://trufactor.io/partners/aws-data-exchange" rel="noopener noreferrer" target="_blank">TruFactor on AWS Data Exchange</a> on the TruFactor website. You can subscribe to TruFactor Intelligence directly on AWS Data Exchange or engage with TruFactor directly to identify the right offering from the larger product portfolio of anonymized consumer intelligence.</p> 
<hr /> 
<h3>About the Authors</h3> 
<p><strong><img alt="" class="size-full wp-image-9644 alignleft" height="154" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/04/01/jeprk.png" width="113" />Jay Park is a solutions architect at AWS.</strong></p> 
<p>&nbsp;</p> 
<p>&nbsp;</p> 
<p>&nbsp;</p> 
<p>&nbsp;</p> 
<p><img alt="" class="size-full wp-image-9645 alignleft" height="143" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/04/01/ArianaR.png" width="113" /><strong>Ariana Rahgozar is a solutions architect at AWS.</strong></p>
