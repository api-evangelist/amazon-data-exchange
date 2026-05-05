---
title: "Use AWS Data Exchange to seamlessly share Apache Hudi datasets"
url: "https://aws.amazon.com/blogs/big-data/use-aws-data-exchange-to-seamlessly-share-apache-hudi-datasets/"
date: "Wed, 22 May 2024 19:49:49 +0000"
author: "Saurabh Bhutyani"
feed_url: "https://aws.amazon.com/blogs/big-data/category/analytics/aws-data-exchange/feed/"
---
<p><a href="https://hudi.apache.org/" rel="noopener" target="_blank">Apache Hudi</a> was originally developed by <a href="https://www.uber.com/blog/hoodie/" rel="noopener" target="_blank">Uber</a> in 2016 to bring to life a <a href="https://www.uber.com/blog/apache-hudi-graduation/" rel="noopener" target="_blank">transactional data lake</a> that could quickly and reliably absorb updates to support the massive growth of the company’s ride-sharing platform. Apache Hudi is now widely used to build very large-scale data lakes by many across the industry. Today, Hudi is the most active and high-performing open source data lakehouse project, known for fast incremental updates and a robust services layer.</p> 
<p>Apache Hudi serves as an important data management tool because it allows you to bring full online transaction processing (OLTP) database functionality to data stored in your data lake. As a result, Hudi users can store massive amounts of data with the data scaling costs of a cloud object store, rather than the more expensive scaling costs of a data warehouse or database. It also provides data lineage, integration with leading access control and governance mechanisms, and incremental ingestion of data for near real-time performance. AWS, along with its partners in the open source community, has embraced Apache Hudi in several services, offering Hudi compatibility in <a href="https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-hudi.html" rel="noopener" target="_blank">Amazon EMR</a>, <a href="https://docs.aws.amazon.com/athena/latest/ug/querying-hudi.html" rel="noopener" target="_blank">Amazon Athena</a>, <a href="https://aws.amazon.com/about-aws/whats-new/2020/09/amazon-redshift-spectrum-adds-support-for-querying-open-source-apache-hudi-and-delta-lake/" rel="noopener" target="_blank">Amazon Redshift</a>, and more.</p> 
<p><a href="https://aws.amazon.com/data-exchange/?adx-cards2.sort-by=item.additionalFields.eventDate&amp;adx-cards2.sort-order=desc" rel="noopener" target="_blank">AWS Data Exchange</a> is a service provided by AWS that enables you to find, subscribe to, and use third-party datasets in the AWS Cloud. A dataset in AWS Data Exchange is a collection of data that can be changed or updated over time. It also provides a platform through which a data producer can make their data available for consumption for subscribers.</p> 
<p>In this post, we show how you can take advantage of the data sharing capabilities in AWS Data Exchange on top of Apache Hudi.</p> 
<h2>Benefits of AWS Data Exchange</h2> 
<p>AWS Data Exchange offers a series of benefits to both parties. For subscribers, it provides a convenient way to access and use third-party data without the need to build and maintain data delivery, entitlement, or billing technology. Subscribers can find and subscribe to thousands of products from qualified AWS Data Exchange providers and use them with AWS services. For providers, AWS Data Exchange offers a secure, transparent, and reliable channel to reach AWS customers. It eliminates the need to build and maintain data delivery, entitlement, and billing technology, allowing providers to focus on creating and managing their datasets.</p> 
<p>To become a provider on AWS Data Exchange, there are a few steps to determine eligibility. Providers need to register to be a <a href="https://docs.aws.amazon.com/data-exchange/latest/userguide/provider-getting-started.html" rel="noopener" target="_blank">provider</a>, make sure their data meets the legal eligibility requirements, and create datasets, revisions, and import assets. Providers can define public offers for their data products, including prices, durations, data subscription agreements, refund policies, and custom offers. The AWS Data Exchange API and AWS Data Exchange console can be used for managing datasets and assets.</p> 
<p>Overall, AWS Data Exchange simplifies the process of data sharing in the AWS Cloud by providing a platform for customers to find and subscribe to third-party data, and for providers to publish and manage their data products. It offers benefits for both subscribers and providers by eliminating the need for complex data delivery and entitlement technology and providing a secure and reliable channel for data exchange.</p> 
<h2>Solution overview</h2> 
<p>Combining the scale and operational capabilities of Apache Hudi with the secure data sharing features of AWS Data Exchange enables you to maintain a single source of truth for your transactional data. Simultaneously, it enables automatic business value generation by allowing other stakeholders to use the insights that the data can provide. This post shows how to set up such a system in your AWS environment using <a href="http://aws.amazon.com/s3" rel="noopener" target="_blank">Amazon Simple Storage Service</a> (Amazon S3), <a href="https://aws.amazon.com/emr/" rel="noopener" target="_blank">Amazon EMR</a>, <a href="http://aws.amazon.com/athena" rel="noopener" target="_blank">Amazon Athena</a>, and AWS Data Exchange. The following diagram illustrates the solution architecture.</p> 
<p><img alt="" class="aligncenter wp-image-63191 size-full" height="380" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/05/10/BDB-3843_001.png" style="margin: 10px 0px 10px 0px;" width="936" /></p> 
<h2>Set up your environment for data sharing</h2> 
<p>You need to register as a data producer before you create datasets and list them in AWS Data Exchange as data products. Complete the following steps to register as a data provider:</p> 
<ol> 
 <li>Sign in to the <a href="https://console.aws.amazon.com/console/home?nc2=h_ct&amp;src=header-signin&amp;ref=adx_sa" rel="noopener" target="_blank">AWS account</a> that you want to use to list and manage products on AWS Data Exchange.<br /> As a provider, you are responsible for complying with these guidelines and the Terms and Conditions for AWS Marketplace Sellers and the AWS Customer Agreement. AWS may update these guidelines. AWS removes any product that breaches these guidelines and may suspend the provider from future use of the service. AWS Data Exchange may have some AWS Regional requirements; refer to Service endpoints for more information.</li> 
 <li>&nbsp;Open the <a href="https://aws.amazon.com/marketplace/management/seller-settings/register/?ref=adx_sa&amp;source=DATA_EXCHANGE" rel="noopener" target="_blank">AWS Marketplace Management Portal registration page</a> and enter the relevant information about how you will use AWS Data Exchange.</li> 
 <li>For <strong>Legal business name</strong>, enter the name that your customers see when subscribing to your data.</li> 
 <li>Review the terms and conditions and select <strong>I have read and agree to the AWS Marketplace Seller Terms and Conditions</strong>.<br /> <img alt="" class="aligncenter wp-image-63189 size-full" height="508" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/05/10/BDB-3843_002.png" style="margin: 10px 0px 10px 0px;" width="532" /></li> 
 <li>Select the information related to the types of products you will be creating as a data provider.</li> 
 <li>Choose <strong>Register &amp; Sign into Management Portal</strong>.<br /> <img alt="" class="aligncenter wp-image-63190 size-full" height="520" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/05/10/BDB-3843_003.png" style="margin: 10px 0px 10px 0px;" width="454" /></li> 
</ol> 
<p>If you want to submit paid products to AWS Marketplace or AWS Data Exchange, you must provide your tax and banking information. You can add this information on the <strong>Settings</strong> page:</p> 
<ol> 
 <li>Choose the <strong>Payment information</strong> tab.</li> 
 <li>Choose <strong>Complete tax information</strong> and complete the form.</li> 
 <li>Choose <strong>Complete banking information</strong> and complete the form.</li> 
 <li>Choose the <strong>Public profile</strong> tab and update your public profile.</li> 
 <li>Choose the <strong>Notifications</strong> tab and configure an additional email address to receive notifications.</li> 
</ol> 
<p>You’re now ready to configure seamless data sharing with AWS Data Exchange.</p> 
<h2>Upload Apache Hudi datasets to AWS Data Exchange</h2> 
<p>After you create your Hudi datasets and register as a data provider, complete the following steps to create the datasets in AWS Data Exchange:</p> 
<ol> 
 <li>Sign in to the <a href="https://console.aws.amazon.com/console/home?nc2=h_ct&amp;src=header-signin&amp;ref=adx_sa" rel="noopener" target="_blank">AWS account</a> that you want to use to list and manage products on AWS Data Exchange.</li> 
 <li>On the AWS Data Exchange console, choose <strong>Owned data sets</strong> in the navigation pane.</li> 
 <li>Choose <strong>Create data set</strong>.<br /> <img alt="" class="aligncenter wp-image-63188 size-full" height="216" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/05/10/BDB-3843_004.png" style="margin: 10px 0px 10px 0px;" width="936" /></li> 
 <li>Select the dataset type you want to create (for this post, we select <strong>Amazon S3 data access</strong>).<br /> <img alt="" class="aligncenter wp-image-63187 size-full" height="450" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/05/10/BDB-3843_005.png" style="margin: 10px 0px 10px 0px;" width="936" /></li> 
 <li>Choose <strong>Choose Amazon S3 locations</strong>.<br /> <img alt="" class="aligncenter wp-image-63186 size-full" height="734" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/05/10/BDB-3843_006.png" style="margin: 10px 0px 10px 0px;" width="580" /></li> 
 <li>Choose the Amazon S3 location where you have your Hudi datasets.<br /> <img alt="" class="aligncenter wp-image-63185 size-full" height="298" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/05/10/BDB-3843_007.png" style="margin: 10px 0px 10px 0px;" width="936" /></li> 
</ol> 
<p>After you add the Amazon S3 location to register in AWS Data Exchange, a bucket policy is generated.</p> 
<ol start="7"> 
 <li>Copy the JSON file and update the bucket policy in Amazon S3.<br /> <img alt="" class="alignnone wp-image-63184 size-full" height="810" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/05/10/BDB-3843_008.png" style="margin: 10px 0px 10px 0px;" width="788" /></li> 
 <li>After you update the bucket policy, choose <strong>Next</strong>.</li> 
 <li>Wait for the <code>CREATE_S3_DATA_ACCESS_FROM_S3_BUCKET</code> job to show as <strong>Completed</strong>, then choose <strong>Finalize data set</strong>.<br /> <img alt="" class="aligncenter wp-image-63183 size-full" height="1110" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/05/10/BDB-3843_009.png" style="margin: 10px 0px 10px 0px;" width="866" /></li> 
</ol> 
<h2>Publish a product using the registered Hudi dataset</h2> 
<p>Complete the following steps to publish a product using the Hudi dataset:</p> 
<ol> 
 <li>On the AWS Data Exchange console, choose <strong>Products</strong> in the navigation pane.<br /> Make sure you’re in the Region where you want to create the product.</li> 
 <li>Choose <strong>Publish new product</strong> to start the workflow to create a new product.</li> 
 <li>Choose which product visibility you want to have: public (it will be publicly available in AWS Data Exchange catalog as well as the AWS Marketplace websites) or private (only the AWS accounts you share with will have access to it).</li> 
 <li>Select the sensitive information category of the data you are publishing.</li> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>Select the dataset that you want to add to the product, then choose <strong>Add selected</strong> to add the dataset to the new product.</li> 
 <li>Define access to your dataset revisions based on time. For more information, see <a href="https://docs.aws.amazon.com/data-exchange/latest/userguide/product-details.html#best-practices-revisions?ref=adx_sa" rel="noopener" target="_blank">Revision access rules</a>.</li> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>Provide the information for a new product, including a short description.<br /> One of the required fields is the product logo, which must be in a supported image format (PNG, JPG, or JPEG) and the file size must be 100 KB or less.<br /> <img alt="" class="aligncenter wp-image-63182 size-full" height="634" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/05/10/BDB-3843_010.png" style="margin: 10px 0px 10px 0px;" width="846" /></li> 
 <li>Optionally, in the <strong>Deﬁne product</strong> section, under <strong>Data dictionaries and samples</strong>, select a dataset and choose <strong>Edit</strong> to upload a data dictionary to the product.</li> 
 <li>For <strong>Long description</strong>, enter the description to display to your customers when they look at your product. Markdown formatting is supported.</li> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>Based on your choice of product visibility, configure the offer, renewal, and data subscription agreement.</li> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>Review all the products and offer information, then choose <strong>Publish</strong> to create the new private product.</li> 
</ol> 
<h2>Manage permissions and access controls for shared datasets</h2> 
<p>Datasets that are published on AWS Data Exchange can only be used when customers are subscribed to the products. Complete the following steps to subscribe to the data:</p> 
<ol> 
 <li>On the AWS Data Exchange console, choose <strong>Browse catalog</strong> in the navigation pane.</li> 
 <li>In the search bar, enter the name of the product you want to subscribe to and press <strong>Enter</strong>.</li> 
 <li>Choose the product to view its detail page.</li> 
 <li>On the product detail page, choose <strong>Continue to Subscribe</strong>.</li> 
 <li>Choose your preferred price and duration combination, choose whether to enable auto-renewal for the subscription, and review the offer details, including the data subscription agreement (DSA).<br /> The dataset is available in the US East (N. Virginia) Region.</li> 
 <li>Review the pricing information, choose the pricing offer and, if you and your organization agree to the DSA, pricing, and support information, choose <strong>Subscribe</strong>.</li> 
</ol> 
<p>After the subscription has gone through, you will be able to see the product on the <strong>Subscriptions</strong> page.</p> 
<h2>Create a table in Athena using an Amazon S3 access point</h2> 
<p>Complete the following steps to create a table in Athena:</p> 
<ol> 
 <li>Open the Athena console.</li> 
 <li>If this is the first time using Athena, choose <strong>Explore Query Editor</strong> and set up the S3 bucket where query results will be written:<br /> Athena will display the results of your query on the Athena console, or send them through your ODBC/JDBC driver if that is what you are using. Additionally, the results are written to the result S3 bucket.<p></p> 
  <ol> 
   <li>Choose <strong>View settings.</strong></li> 
   <li>Choose <strong>Manage</strong>.</li> 
   <li>Under <strong>Query result location and encryption</strong>, choose <strong>Browse Amazon S3</strong> to choose the location where query results will be written.</li> 
   <li>Choose <strong>Save</strong>.</li> 
   <li>Choose a bucket and folder you want to automatically write the query results to.<br /> Athena will display the results of your query on the Athena console, or send them through your ODBC/JDBC driver if that is what you are using. Additionally, the results are written to the result S3 bucket.</li> 
  </ol> </li> 
 <li>Complete the following steps to create a workgroup: 
  <ol> 
   <li>In the navigation pane, choose <strong>Workgroups</strong>.</li> 
   <li>Choose <strong>Create workgroup</strong>.<br /> <img alt="" class="aligncenter wp-image-63181 size-full" height="234" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/05/10/BDB-3843_011.png" style="margin: 10px 0px 10px 0px;" width="936" /></li> 
   <li>Enter a name for your workgroup (for this post, <code>data_exchange</code>), select your analytics engine (Athena SQL), and select <strong>Turn on queries on requester pay buckets in Amazon S3</strong>.<br /> This is important to access third-party datasets.<br /> <img alt="" class="aligncenter wp-image-63192 size-full" height="928" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/05/10/BDB-3843_012.png" style="margin: 10px 0px 10px 0px;" width="936" /></li> 
   <li>In the Athena query editor, choose the workgroup you created.<br /> <img alt="" class="aligncenter wp-image-63193 size-full" height="378" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/05/10/BDB-3843_013.png" style="margin: 10px 0px 10px 0px;" width="936" /></li> 
   <li>Run the following DDL to create the table:<br /> <img alt="" class="aligncenter wp-image-63194 size-full" height="444" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/05/10/BDB-3843_014.png" style="margin: 10px 0px 10px 0px;" width="936" /></li> 
  </ol> </li> 
</ol> 
<p>Now you can run your analytical queries using Athena SQL statements. The following screenshot shows an example of the query results.</p> 
<p><img alt="" class="aligncenter wp-image-63195 size-full" height="346" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/05/10/BDB-3843_015.png" style="margin: 10px 0px 10px 0px;" width="750" /></p> 
<h2>Enhanced customer collaboration and experience with AWS Data Exchange and Apache Hudi</h2> 
<p>AWS Data Exchange provides a secure and simple interface to access high-quality data. By providing access to over 3,500 datasets, you can use leading high-quality data in your analytics and data science. Additionally, the ability to add Hudi datasets as shown in this post allows you to enable deeper integration with lakehouse use cases. There are several potential use cases where having Apache Hudi datasets integrated into AWS Data Exchange can accelerate business outcomes, such as the following:</p> 
<ul> 
 <li><strong>Near real-time updated datasets</strong> – One of Apache Hudi’s defining features is the ability to provide near real-time incremental data processing. As new data flows in, Hudi allows that data to be ingested in real time, providing a central source of up-to-date truth. AWS Data Exchange supports dynamically updated datasets, which can keep up with these incremental updates. For downstream customers that rely on the most up-to-date information for their use cases, the combination of Apache Hudi and AWS Data Exchange means that they can subscribe to a dataset in AWS Data Exchange and know that they’re getting incrementally updated data.</li> 
 <li><strong>Incremental pipelines and processing</strong> – Hudi supports incremental processing and updates to data in the data lake. This is especially valuable because it enables you to only update or process any data that has changed and materialized views that are valuable for your business use case.</li> 
</ul> 
<h2>Best practices and recommendations</h2> 
<p>We recommend the following best practices for security and compliance:</p> 
<ul> 
 <li>Enable <a href="https://aws.amazon.com/lake-formation/" rel="noopener" target="_blank">AWS Lake Formation</a> or other data governance systems as part of creating the source data lake</li> 
 <li>To maintain compliance, you can use the guides provided by <a href="https://aws.amazon.com/artifact/" rel="noopener" target="_blank">AWS Artifact</a></li> 
</ul> 
<p>For monitoring and management, you can enable <a href="http://aws.amazon.com/cloudwatch" rel="noopener" target="_blank">Amazon CloudWatch</a> logs on your EMR clusters along with CloudWatch alerts to maintain pipeline health.</p> 
<h2>Conclusion</h2> 
<p>Apache Hudi enables you to bring to life massive amounts of data stored in Amazon S3 for analytics. It provides full OLAP capabilities, enables incremental processing and querying, along with maintaining the ability to run deletes to remain <a href="https://aws.amazon.com/blogs/big-data/how-zoom-implemented-streaming-log-ingestion-and-efficient-gdpr-deletes-using-apache-hudi-on-amazon-emr/" rel="noopener" target="_blank">GDPR compliant</a>. Combining this with the secure, reliable, and user-friendly data sharing capabilities of AWS Data Exchange means that the business value unlocked by a Hudi lakehouse doesn’t need to remain limited to the producer that generates this data.</p> 
<p>For more use cases about using AWS Data Exchange, see <a href="https://aws.amazon.com/data-exchange/resources/?adx-cards2.sort-by=item.additionalFields.eventDate&amp;adx-cards2.sort-order=desc&amp;awsf.Audience=*all&amp;awsf.Format=*all&amp;awsf.Industry=*all" rel="noopener" target="_blank">Learning Resources for Using Third-Party Data in the Cloud</a>. To learn more about creating Apache Hudi data lakes, refer to <a href="https://aws.amazon.com/blogs/big-data/part-1-build-your-apache-hudi-data-lake-on-aws-using-amazon-emr/" rel="noopener" target="_blank">Build your Apache Hudi data lake on AWS using Amazon EMR – Part 1</a>. You can also consider using a fully managed lakehouse product such as <a href="https://www.onehouse.ai/blog/its-time-for-the-universal-data-lakehouse" rel="noopener" target="_blank">Onehouse</a>.</p> 
<hr /> 
<h3>About the Authors</h3> 
<p style="clear: both;"><strong><img alt="" class="wp-image-13110 size-full alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/11/03/Saurabh-Bhutyani.jpg" width="100" /></strong><strong>Saurabh Bhutyani</strong> is a Principal Analytics Specialist Solutions Architect at AWS. He is passionate about new technologies. He joined AWS in 2019 and works with customers to provide architectural guidance for running generative AI use cases, scalable analytics solutions and data mesh architectures using AWS services like Amazon Bedrock, Amazon SageMaker, Amazon EMR, Amazon Athena, AWS Glue, AWS Lake Formation, and Amazon DataZone.</p> 
<p style="clear: both;"><img alt="" class="wp-image-63290 size-full alignleft" height="96" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/05/15/ankith-ede.png" width="100" /><strong>Ankith Ede</strong> is a Data &amp; Machine Learning Engineer at Amazon Web Services, based in New York City. He has years of experience building Machine Learning, Artificial Intelligence, and Analytics based solutions for large enterprise clients across various industries. He is passionate about helping customers build scalable and secure cloud based solutions at the cutting edge of technology innovation.</p> 
<p style="clear: both;"><strong><img alt="" class="wp-image-63289 size-full alignleft" height="115" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/05/15/chandra-100.png" width="100" />Chandra Krishnan</strong> is a Solutions Engineer at Onehouse, based in New York City. He works on helping Onehouse customers build business value from their data lakehouse deployments and enjoys solving exciting challenges on behalf of his customers. Prior to Onehouse, Chandra worked at AWS as a Data and ML Engineer, helping large enterprise clients build cutting edge systems to drive innovation in their organizations.</p>
