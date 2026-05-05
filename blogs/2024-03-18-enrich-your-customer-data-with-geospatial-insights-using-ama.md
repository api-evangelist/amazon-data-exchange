---
title: "Enrich your customer data with geospatial insights using Amazon Redshift, AWS Data Exchange, and Amazon QuickSight"
url: "https://aws.amazon.com/blogs/big-data/enrich-your-customer-data-with-geospatial-insights-using-amazon-redshift-aws-data-exchange-and-amazon-quicksight/"
date: "Mon, 18 Mar 2024 16:07:23 +0000"
author: "Tony Stricker"
feed_url: "https://aws.amazon.com/blogs/big-data/category/analytics/aws-data-exchange/feed/"
---
<p>It always pays to know more about your customers, and <a href="https://aws.amazon.com/data-exchange/" rel="noopener" target="_blank">AWS Data Exchange</a> makes it straightforward to use publicly available census data to enrich your customer dataset.</p> 
<p>The United States Census Bureau conducts the US census every 10 years and gathers household survey data. This data is anonymized, aggregated, and made available for public use. The smallest geographic area for which the Census Bureau collects and aggregates data are census blocks, which are formed by streets, roads, railroads, streams and other bodies of water, other visible physical and cultural features, and the legal boundaries shown on Census Bureau maps.</p> 
<p>If you know the census block in which a customer lives, you are able to make general inferences about their demographic characteristics. With these new attributes, you are able to build a segmentation model to identify distinct groups of customers that you can target with personalized messaging. This data is available to subscribe to on AWS Data Exchange—and with data sharing, you don’t need to pay to store a copy of it in your account in order to query it.</p> 
<p>In this post, we show how to use customer addresses to enrich a dataset with additional demographic details from the US Census Bureau dataset.</p> 
<h2>Solution overview</h2> 
<p>The solution includes the following high-level steps:</p> 
<ol> 
 <li>Set up an <a href="https://aws.amazon.com/redshift/redshift-serverless/" rel="noopener" target="_blank">Amazon Redshift Serverless</a> endpoint and load customer data.</li> 
 <li>Set up a place index in <a href="https://aws.amazon.com/location/" rel="noopener" target="_blank">Amazon Location Service</a>.</li> 
 <li>Write an <a href="http://aws.amazon.com/lambda" rel="noopener" target="_blank">AWS Lambda</a> user-defined function (UDF) to call Location Service from <a href="http://aws.amazon.com/redshift" rel="noopener" target="_blank">Amazon Redshift</a>.</li> 
 <li>Subscribe to census data on AWS Data Exchange.</li> 
 <li>Use geospatial queries to tag addresses to census blocks.</li> 
 <li>Create a new customer dataset in Amazon Redshift.</li> 
 <li>Evaluate new customer data in <a href="https://aws.amazon.com/quicksight" rel="noopener" target="_blank">Amazon QuickSight</a>.</li> 
</ol> 
<p>The following diagram illustrates the solution architecture.</p> 
<p><img alt="architecture diagram" class="aligncenter wp-image-59737 size-full" height="471" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/08/BDB-3404-ADX_Blog_Arch.jpg" style="margin: 10px 0px 10px 0px;" width="630" /></p> 
<h2>Prerequisites</h2> 
<p>You can use the following <a href="http://aws.amazon.com/cloudformation" rel="noopener" target="_blank">AWS CloudFormation</a> <a href="https://d2e-demos-published.s3.amazonaws.com/adx-demo/adx-blog-stack.yaml" rel="noopener" target="_blank">template</a> to deploy the required infrastructure. Before deployment, you need to sign up for QuickSight access through the <a href="http://aws.amazon.com/console" rel="noopener" target="_blank">AWS Management Console</a>.</p> 
<h2>Load generic address data to Amazon Redshift</h2> 
<p>Amazon Redshift is a fully managed, petabyte-scale data warehouse service in the cloud. Redshift Serverless makes it straightforward to run analytics workloads of any size without having to manage data warehouse infrastructure.</p> 
<p>To load our address data, we first create a Redshift Serverless workgroup. Then we use Amazon Redshift Query Editor v2 to load customer data from <a href="http://aws.amazon.com/s3" rel="noopener" target="_blank">Amazon Simple Storage Service</a> (Amazon S3).</p> 
<h3>Create a Redshift Serverless workgroup</h3> 
<p>There are two primary components of the Redshift Serverless architecture:</p> 
<ul> 
 <li><strong>Namespace</strong> – A collection of database objects and users. Namespaces group together all of the resources you use in Redshift Serverless, such as schemas, tables, users, datashares, and snapshots.</li> 
 <li><strong>Workgroup</strong> – A collection of compute resources. Workgroups have network and security settings that you can configure using the Redshift Serverless console, the <a href="http://aws.amazon.com/cli" rel="noopener" target="_blank">AWS Command Line Interface</a> (AWS CLI), or the Redshift Serverless APIs.</li> 
</ul> 
<p>To create your namespace and workgroup, refer to <a href="https://docs.aws.amazon.com/redshift/latest/gsg/new-user-serverless.html#serverless-console-resource-creation" rel="noopener" target="_blank">Creating a data warehouse with Amazon Redshift Serverless</a>. For this exercise, name your workgroup sandbox and your namespace adx-demo.</p> 
<h3>Use Query Editor v2 to load customer data from Amazon S3</h3> 
<p>You can use Query Editor v2 to submit queries and load data to your data warehouse through a web interface. To configure Query Editor v2 for your AWS account, refer to <a href="https://aws.amazon.com/blogs/big-data/data-load-made-easy-and-secure-in-amazon-redshift-using-query-editor-v2/" rel="noopener" target="_blank">Data load made easy and secure in Amazon Redshift using Query Editor V2</a>. After it’s configured, complete the following steps:</p> 
<ul> 
 <li>Use the following SQL to create the <code>customer_data</code> schema within the dev database in your data warehouse:</li> 
</ul> 
<div class="hide-language"> 
 <pre><code class="lang-sql">CREATE SCHEMA customer_data;</code></pre> 
</div> 
<ul> 
 <li>Use the following SQL DDL to create your target table into which you’ll load your customer address data:</li> 
</ul> 
<div class="hide-language"> 
 <pre><code class="lang-sql">CREATE TABLE customer_data.customer_addresses (
    address character varying(256) ENCODE lzo,
    unitnumber character varying(256) ENCODE lzo,
    municipality character varying(256) ENCODE lzo,
    region character varying(256) ENCODE lzo,
    postalcode character varying(256) ENCODE lzo,
    country character varying(256) ENCODE lzo,
    customer_id integer ENCODE az64
) DISTSTYLE AUTO;</code></pre> 
</div> 
<ul> 
 <li>Load the <a href="https://d2e-demos-published.s3.amazonaws.com/adx-demo/address_list.csv" rel="noopener" target="_blank">address_list.csv</a> file to the table you just created. For instructions, refer to <a href="https://aws.amazon.com/blogs/big-data/data-load-made-easy-and-secure-in-amazon-redshift-using-query-editor-v2/" rel="noopener" target="_blank">Data load made easy and secure in Amazon Redshift using Query Editor V2</a>.</li> 
</ul> 
<p>The file has no column headers and is pipe delimited (|). For information on how to load data from either Amazon S3 or your local desktop, refer to <a href="https://docs.aws.amazon.com/redshift/latest/mgmt/query-editor-v2-loading.html" rel="noopener" target="_blank">Loading data into a database</a>.</p> 
<h2>Use Location Service to geocode and enrich address data</h2> 
<p>Location Service lets you add location data and functionality to applications, which includes capabilities such as maps, points of interest, geocoding, routing, geofences, and tracking.</p> 
<p>Our data is in Amazon Redshift, so we need to access the Location Service APIs using SQL statements. Each row of data contains an address that we want to enrich and geotag using the Location Service APIs. Amazon Redshift allows developers to create UDFs using a SQL SELECT clause, Python, or Lambda.</p> 
<p>Lambda is a compute service that lets you run code without provisioning or managing servers. With Lambda UDFs, you can write custom functions with complex logic and integrate with third-party components. Scalar Lambda UDFs return one result per invocation of the function—in this case, the Lambda function runs one time for each row of data it receives.</p> 
<p>For this post, we write a Lambda function that uses the Location Service API to geotag and validate our customer addresses. Then we register this Lambda function as a UDF with our Redshift instance, allowing us to call the function from a SQL command.</p> 
<p>For instructions to create a Location Service place index and create your Lambda function and scalar UDF, refer to <a href="https://aws.amazon.com/blogs/big-data/access-amazon-location-service-from-amazon-redshift/" rel="noopener" target="_blank">Access Amazon Location Service from Amazon Redshift</a>. For this post, we use ESRI as a provider and name the place index <code>placeindex.redshift</code>.</p> 
<p>Test your new function with the following code, which returns the coordinates of the White House in Washington, DC:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">select public.f_geocode_address('1600 Pennsylvania Ave.','Washington','DC','20500','USA');</code></pre> 
</div> 
<h2>Subscribe to demographic data from AWS Data Exchange</h2> 
<p>AWS Data Exchange is a data marketplace with more than 3,500 products from over 300 providers delivered—through files, APIs, or Amazon Redshift queries—directly to the data lakes, applications, analytics, and machine learning models that use it.</p> 
<p>First, we need to give our Redshift namespace permission via <a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management</a> (IAM) to access subscriptions on AWS Data Exchange. Then we can subscribe to our sample demographic data. Complete the following steps:</p> 
<ol> 
 <li>On the IAM console, add the <code>AWSDataExchangeSubscriberFullAccess</code> managed policy to your Amazon Redshift commands access role you assigned when creating the namespace.</li> 
 <li>On the AWS Data Exchange console, navigate to the dataset <a href="https://us-east-2.console.aws.amazon.com/dataexchange/home?region=us-east-2#/products/prodview-ftz5ddfcci33c" rel="noopener" target="_blank">ACS – Sociodemographics (USA, Census Block Groups, 2019)</a>, provided by CARTO.</li> 
 <li>Choose <strong>Continue to subscribe</strong>, then choose <strong>Subscribe</strong>.</li> 
</ol> 
<p>The subscription may take a few minutes to configure.</p> 
<ol> 
 <li>When your subscription is in place, navigate back to the Redshift Serverless console.</li> 
 <li>In the navigation pane, choose <strong>Datashares</strong>.</li> 
 <li>On the <strong>Subscriptions</strong> tab, choose the datashare that you just subscribed to.</li> 
 <li>On the datashare details page, choose <strong>Create database from datashare</strong>.</li> 
 <li>Choose the namespace you created earlier and provide a name for the new database that will hold the shared objects from the dataset you subscribed to.</li> 
</ol> 
<p>In Query Editor v2, you should see the new database you just created and two new tables: one that holds the block group polygons and another that holds the demographic information for each block group.</p> 
<p><img alt="Query Editor v2 data source explorer" class="aligncenter wp-image-59738" height="363" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/08/BDB-3404-query_editor_v2_explorer.jpg" style="margin: 10px 0px 10px 0px;" width="650" /></p> 
<h2>Join geocoded customer data to census data with geospatial queries</h2> 
<p>There are two primary types of spatial data: raster and vector data. Raster data is represented as a grid of pixels and is beyond the scope of this post. Vector data is comprised of vertices, edges, and polygons. With geospatial data, <em>vertices</em> are represented as latitude and longitude points and <em>edges</em> are the connections between pairs of vertices. Think of the road connecting two intersections on a map. A <em>polygon</em> is a set of vertices with a series of connecting edges that form a continuous shape. A simple rectangle is a polygon, just as the state border of Ohio can be represented as a polygon. The geography_usa_blockgroup_2019 dataset that you subscribed to has 220,134 rows, each representing a single census block group and its geographic shape.</p> 
<p>Amazon Redshift supports the storage and querying of vector-based spatial data with the <a href="https://docs.aws.amazon.com/redshift/latest/dg/geospatial-overview.html" rel="noopener" target="_blank">GEOMETRY and GEOGRAPHY data types</a>. You can use Redshift SQL functions to perform queries such as a point in polygon operation to determine if a given latitude/longitude point falls within the boundaries of a given polygon (such as state or county boundary). In this dataset, you can observe that the <code>geom</code> column in <code>geography_usa_blockgroup_2019</code> is of type GEOMETRY.</p> 
<p>Our goal is to determine which census block (polygon) each of our geotagged addresses falls within so we can enrich our customer records with details that we know about the census block. Complete the following steps:</p> 
<ul> 
 <li>Build a new table with the geocoding results from our UDF:</li> 
</ul> 
<div class="hide-language"> 
 <pre><code class="lang-sql">CREATE TABLE customer_data.customer_addresses_geocoded AS 
select address
    ,unitnumber
    ,municipality
    ,region
    ,postalcode
    ,country
    ,customer_id
    ,public.f_geocode_address(address||' '||unitnumber,municipality,region,postalcode,country) as geocode_result
FROM customer_data.customer_addresses;</code></pre> 
</div> 
<ul> 
 <li>Use the following code to extract the different address fields and latitude/longitude coordinates from the JSON column and create a new table with the results:</li> 
</ul> 
<div class="hide-language"> 
 <pre><code class="lang-sql">CREATE TABLE customer_data.customer_addresses_points AS
SELECT customer_id
    ,geo_address
    address
    ,unitnumber
    ,municipality
    ,region
    ,postalcode
    ,country
    ,longitude
    ,latitude
    ,ST_SetSRID(ST_MakePoint(Longitude, Latitude),4326) as address_point
            --create new geom column of type POINT, set new point SRID = 4326
FROM
(
select customer_id
    ,address
    ,unitnumber
    ,municipality
    ,region
    ,postalcode
    ,country
    ,cast(json_extract_path_text(geocode_result, 'Label', true) as VARCHAR) as geo_address
    ,cast(json_extract_path_text(geocode_result, 'Longitude', true) as float) as longitude
    ,cast(json_extract_path_text(geocode_result, 'Latitude', true) as float) as latitude
        --use json function to extract fields from geocode_result
from customer_data.customer_addresses_geocoded) a;</code></pre> 
</div> 
<p>This code uses the <code>ST_POINT</code> function to create a new column from the latitude/longitude coordinates called <code>address_point</code> of type GEOMETRY and subtype POINT. &nbsp;&nbsp;It uses the <code>ST_SetSRID</code> geospatial function to set the spatial reference identifier (SRID) of the new column to 4326.</p> 
<p>The SRID defines the spatial reference system to be used when evaluating the geometry data. It’s important when joining or comparing geospatial data that they have matching SRIDs. You can check the SRID of an existing geometry column by using the <code>ST_SRID</code> function. For more information on SRIDs and GEOMETRY data types, refer to <a href="https://docs.aws.amazon.com/redshift/latest/dg/geospatial-overview.html" rel="noopener" target="_blank">Querying spatial data in Amazon Redshift</a>.</p> 
<ul> 
 <li>Now that your customer addresses are geocoded as latitude/longitude points in a geometry column, you can use a join to identify which census block shape your new point falls within:</li> 
</ul> 
<div class="hide-language"> 
 <pre><code class="lang-sql">CREATE TABLE customer_data.customer_addresses_with_census AS
select c.*
    ,shapes.geoid as census_group_shape
    ,demo.*
from customer_data.customer_addresses_points c
inner join "carto_census_data"."carto".geography_usa_blockgroup_2019 shapes
on ST_Contains(shapes.geom, c.address_point)
    --join tables where the address point falls within the census block geometry
inner join carto_census_data.usa_acs.demographics_sociodemographics_usa_blockgroup_2019_yearly_2019 demo
on demo.geoid = shapes.geoid;</code></pre> 
</div> 
<p>The preceding code creates a new table called <code>customer_addresses_with_census</code>, which joins the customer addresses to the census block in which they belong as well as the demographic data associated with that census block.</p> 
<p>To do this, you used the <code>ST_CONTAINS</code> function, which accepts two geometry data types as an input and returns TRUE if the 2D projection of the first input geometry contains the second input geometry. In our case, we have census blocks represented as polygons and addresses represented as points. The join in the SQL statement succeeds when the point falls within the boundaries of the polygon.</p> 
<h2>Visualize the new demographic data with QuickSight</h2> 
<p>QuickSight is a cloud-scale business intelligence (BI) service that you can use to deliver easy-to-understand insights to the people who you work with, wherever they are. QuickSight connects to your data in the cloud and combines data from many different sources.</p> 
<p>First, let’s build some new calculated fields that will help us better understand the demographics of our customer base. We can do this in QuickSight, or we can use SQL to build the columns in a Redshift view. The following is the code for a Redshift view:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">CREATE VIEW customer_data.customer_features AS (
SELECT customer_id 
    ,postalcode
    ,region
    ,municipality
    ,geoid as census_geoid
    ,longitude
    ,latitude
    ,total_pop
    ,median_age
    ,white_pop/total_pop as perc_white
    ,black_pop/total_pop as perc_black
    ,asian_pop/total_pop as perc_asian
    ,hispanic_pop/total_pop as perc_hispanic
    ,amerindian_pop/total_pop as perc_amerindian
    ,median_income
    ,income_per_capita
    ,median_rent
    ,percent_income_spent_on_rent
    ,unemployed_pop/coalesce(pop_in_labor_force) as perc_unemployment
    ,(associates_degree + bachelors_degree + masters_degree + doctorate_degree)/total_pop as perc_college_ed
    ,(household_language_total - household_language_english)/coalesce(household_language_total) as perc_other_than_english
FROM "dev"."customer_data"."customer_addresses_with_census" t );</code></pre> 
</div> 
<p>To get QuickSight to talk to our Redshift Serverless endpoint, complete the following steps:</p> 
<ul> 
 <li>Manually authorize connections from QuickSight to Redshift clusters. For instructions, refer to <a href="https://docs.aws.amazon.com/quicksight/latest/user/enabling-access-redshift.html" rel="noopener" target="_blank">Authorizing connections from Amazon QuickSight to Amazon Redshift clusters</a> (stop after Step 19).</li> 
 <li><a href="https://docs.aws.amazon.com/quicksight/latest/user/vpc-creating-a-connection-in-quicksight-console.html" rel="noopener" target="_blank">Configure the VPC connection</a> between QuickSight and the Redshift Serverless endpoint.</li> 
</ul> 
<p>Now you can create a new dataset in QuickSight.</p> 
<ul> 
 <li>On the QuickSight console, choose <strong>Datasets</strong> in the navigation pane.</li> 
 <li>Choose <strong>New dataset</strong>.</li> 
</ul> 
<p><img alt="create a new dataset in quicksight" class="aligncenter wp-image-59740 size-full" height="604" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/08/BDB-3404-quicksight1.jpg" style="margin: 10px 0px 10px 0px;" width="848" /></p> 
<ul> 
 <li>We want to create a dataset from a new data source and use the <strong>Redshift: Manual connect</strong> option.</li> 
</ul> 
<p><img alt="Redshift manual connection" class="aligncenter wp-image-59739 size-medium" height="103" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/08/BDB-3404-quicksigh2-300x103.jpg" style="margin: 10px 0px 10px 0px;" width="300" /></p> 
<ul> 
 <li>Provide the connection information for your Redshift Serverless workgroup.</li> 
</ul> 
<p>You will need the endpoint for our workgroup and the user name and password that you created when you set up your workgroup. You can find your workgroup’s endpoint on the Redshift Serverless console by navigating to your workgroup configuration. The following screenshot is an example of the connection settings needed. Notice the connection type is the name of the VPC connection that you previously configured in QuickSight. When you copy the endpoint from the Redshift console, be sure to remove the database and port number from the end of the URL before entering it in the field.</p> 
<p><img alt="Redshift edit data source" class="aligncenter wp-image-59741 size-full" height="652" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/08/BDB-3404-quicksight3.jpg" style="margin: 10px 0px 10px 0px;" width="602" /></p> 
<ul> 
 <li>Save the new data source configuration.</li> 
</ul> 
<p>You’ll be prompted to choose the table you want to use for your dataset.</p> 
<ul> 
 <li>Choose the new view that you created that has your new derived fields.</li> 
</ul> 
<p><img alt="Quicksight choose your table" class="aligncenter wp-image-59742 size-full" height="496" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/08/BDB-3404-quicksight4.jpg" style="margin: 10px 0px 10px 0px;" width="602" /></p> 
<ul> 
 <li>Select <strong>Directly query your data</strong>.</li> 
</ul> 
<p>This will connect your visualizations directly to the data in the database rather than ingesting data into the QuickSight in-memory data store.</p> 
<p><img alt="Directly query your data" class="aligncenter wp-image-59743 size-full" height="310" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/08/BDB-3404-quicksight5.jpg" style="margin: 10px 0px 10px 0px;" width="600" /></p> 
<ul> 
 <li>To create a histogram of median income level, choose the blank visual on Sheet1 and then choose the histogram visual icon under <strong>Visual types</strong>.</li> 
 <li>Choose <code>median_income</code> under <strong>Fields list</strong> and drag it to the <strong>Value</strong> field well.</li> 
</ul> 
<p>This builds a histogram showing the distribution of <code>median_income</code> for our customers based on the census block group in which they live.</p> 
<p><img alt="QuickSight histogram" class="aligncenter wp-image-59744 size-full" height="680" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/08/BDB-3404-quicksight6.jpg" style="margin: 10px 0px 10px 0px;" width="674" /></p> 
<h2>Conclusion</h2> 
<p>In this post, we demonstrated how companies can use open census data available on AWS Data Exchange to effortlessly gain a high-level understanding of their customer base from a demographic standpoint. This basic understanding of customers based on where they live can serve as the foundation for more targeted marketing campaigns and even influence product development and service offerings.</p> 
<p>As always, AWS welcomes your feedback. Please leave your thoughts and questions in the comments section.</p> 
<hr /> 
<h3>About the Author</h3> 
<p><img alt="" class="size-full wp-image-59749 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/02/08/TonyStricker_Headshot_100px.jpg" width="100" /><strong>Tony Stricker</strong> is a Principal Technologist on the Data Strategy team at AWS, where he helps senior executives adopt a data-driven mindset and align their people/process/technology in ways that foster innovation and drive towards specific, tangible business outcomes. He has a background as a data warehouse architect and data scientist and has delivered solutions in to production across multiple industries including oil and gas, financial services, public sector, and manufacturing. In his spare time, Tony likes to hang out with his dog and cat, work on home improvement projects, and restore vintage Airstream campers.</p>
