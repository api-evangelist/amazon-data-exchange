---
title: "Managing COVID-19 exposure with crowd tracing"
url: "https://aws.amazon.com/blogs/big-data/managing-covid-19-exposure-with-crowd-tracing/"
date: "Thu, 19 Nov 2020 14:00:33 +0000"
author: "Aspire Ventures"
feed_url: "https://aws.amazon.com/blogs/big-data/category/analytics/aws-data-exchange/feed/"
---
<p><em>This is a guest blog post by AWS partner Aspire Ventures</em></p> 
<p>As we enter winter, with fewer options to be outdoors, our personal choices can impact our risk of contracting the COVID-19 virus even more. <a href="https://www.nejm.org/doi/full/10.1056/NEJMp2026913" rel="noopener noreferrer" target="_blank">The New England Journal of Medicine publication</a> showed real-world examples of the effectiveness of masks and social distancing in mitigating severity of COVID-19 infection and keeping people asymptomatic. <a href="https://www.cnn.com/2020/09/10/health/restaurant-dining-covid-19-cdc-study-wellness/index.html" rel="noopener noreferrer" target="_blank">CNN reported</a> on a study that showed people who contracted COVID-19 were twice as likely to have visited a restaurant in the prior two weeks. What if we had actionable crowding, mask usage, and social distancing data that we could analyze to inform our daily decisions to keep us safe?</p> 
<p>Aspire Ventures, an AWS partner, has developed the <a href="https://events.clio.health/" rel="noopener noreferrer" target="_blank">Clio GO</a> pass system — a new venue-entry system that helps track COVID-19 exposure through kiosks and mobile phones in a completely privacy-preserving way. It uses a new technology called <em>crowd tracing</em>, which allows users to assess whether certain locations and venues meet their risk profile. Crowd tracing data is COVID-19 location-scouting data, which helps answer the question of how much risk may be associated with entering a particular crowd.</p> 
<p>Today, Aspire Ventures is collaborating with AWS to open source anonymized crowd tracing data from the Clio GO pass system and make it available in the public AWS COVID-19 data lake. Aspire Ventures is a venture fund dedicated to fast-tracking precision medicine technologies and practices that leverage AI and IoT to deliver affordable, individualized solutions at a massive scale. The AWS COVID-19 data lake is a public repository of up-to-date and curated datasets on or related to COVID-19 to help experts track, contain, and neutralize the virus causing the illness. With the Clio GO pass system and the open-source crowd tracing dataset in the AWS COVID-19 data lake, we believe the global community can come together and develop techniques to better fight the COVID-19 pandemic.</p> 
<p>The Clio GO app functions like an airline mobile boarding pass system. Prior to arrival, you check in via the app by answering a few questions and receive a mobile entry GO Pass in either QR-code or NFC ticket format. When you arrive at a venue, you validate your GO Pass by scanning it at a kiosk or smartphone. GO Pass is being used by thousands of venues who have tens of millions of annual visitors. These venues run the spectrum from schools and medical practices to office buildings and food manufacturing facilities. Further adoption of Aspire’s Clio GO app will generate more anonymized data that AWS will make available for advancing COVID-19 solutions.</p> 
<h2>Crowd tracing vs. contact tracing</h2> 
<p>As Dr. Fauci said, <a href="https://twitter.com/CBSNews/status/1276576403929157632?s=20" rel="noopener noreferrer" target="_blank">contact tracing is “not working.”</a> As the primary technology used by public health authorities, it’s fraught with <a href="https://www.axios.com/coronavirus-contact-tracing-isnt-working-0d8ec92c-ec1c-4b46-a736-844649b760dd.html" rel="noopener noreferrer" target="_blank">poor adoption, poor accuracy, high cost, and serious privacy concerns</a>. To understand the issues with contact tracing, we analogize to a first-person video game to introduce the <a href="https://www.thelancet.com/journals/laninf/article/PIIS1473-3099(20)30232-2/fulltext" rel="noopener noreferrer" target="_blank">immunological concepts</a> of <a href="https://www.axios.com/viral-load-dose-coronavirus-246b334d-5420-488d-a1b1-ec9a39c55f58.html?utm_campaign=organic&amp;utm_medium=socialshare&amp;utm_source=email" rel="noopener noreferrer" target="_blank">viral dose</a>, <a href="https://www.bmj.com/content/369/bmj.m1443.long" rel="noopener noreferrer" target="_blank">viral load</a><u>,</u> and <a href="https://www.nejm.org/doi/full/10.1056/NEJMp2015897" rel="noopener noreferrer" target="_blank">undetectable</a> <a href="https://www.advisory.com/daily-briefing/2020/08/10/asymptomatic" rel="noopener noreferrer" target="_blank">asymptomatic carriers</a>.</p> 
<p>Imagine a video game scenario with attackers and shields to protect from attacks. <em>Viral dose</em> is analogous to incremental hits that weaken a player’s shield. Avoiding those hits prevents your shield from collapsing.</p> 
<p><em>Viral load</em> is analogous to how strongly any one attacker can hit a player’s shield. Certain infected individuals who are more progressed in their infection may hit you harder. Just as in the game, your shields may be destroyed by many weak hits from multiple attackers or one very strong hit from a single attacker.</p> 
<p><em>Asymptomatic carriers</em> are like players who, from a distance, look like they have no weapons. <em>Undetectable asymptomatic carriers</em> are like players whose weapons can’t even be detected when you search their belongings—the science indicates that infectious asymptomatic carriers <a href="https://www.nejm.org/doi/full/10.1056/NEJMp2015897" rel="noopener noreferrer" target="_blank">may not be detectable with COVID-19 PCR swab tests</a>. <a href="https://www.cdc.gov/coronavirus/2019-ncov/cases-updates/commercial-lab-surveys.html" rel="noopener noreferrer" target="_blank">CDC blood test surveys</a> show <a href="https://jamanetwork.com/journals/jamainternalmedicine/fullarticle/2768834?guestAccessKey=7a5c32e6-3c27-41b3-b46c-43c4a38bbe00&amp;utm_source=For_The_Media&amp;utm_medium=referral&amp;utm_campaign=ftm_links&amp;utm_content=tfl&amp;utm_term=072120" rel="noopener noreferrer" target="_blank">between 6 times to 25 times</a> as many asymptomatic carriers are lurking out there for any single known case.</p> 
<p>Clio GO uses crowd tracing and <a href="https://youtu.be/EVnZpdZzlK4" rel="noopener noreferrer" target="_blank">adaptive artificial intelligence (A<sup>2</sup>I)</a> to progressively improve the estimates of each player’s shield strength, the intensity of hits you might encounter, and the likely hits from attackers whose weapons are completely undetectable.</p> 
<p>In contrast, contact tracing requires that an attacker have a visible weapon, and if so, it assumes shields are obliterated immediately. However, if there is no visible weapon, the shields remain at 100%. In either case, contact tracing doesn’t decrease your shield level based on cumulative small hits (viral dose) or how intense the hits (viral load of others) are by taking into account the conditions at the time, such as mask usage, social distancing, and duration of contact.</p> 
<p><img alt="" class="alignnone size-full wp-image-12896" height="826" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/10/28/aspire-crowd-tracing-1.jpg" style="margin: 10px 0px 10px 0px; border: 1px solid #cccccc;" width="448" /></p> 
<p>The more people who have checked in to the same place, the lower each person’s shield is computed to be. Shield levels are further refined by reported mask usage and social distancing within the crowd. The venue never sees the person’s shield level. It sees a green or red check mark indicating if you’re entering with a valid pass and meets their entry requirements, but no symptom data is shared with the venue<em>.</em></p> 
<p>After your visit, you can rate the venue’s use of masks and social distancing, and this report helps compute your own shield level and that of others. When using the Clio GO app to scout venues prior to visiting, the venue’s listing shows the aggregate mask and social distancing ratings by visitors.</p> 
<p>As part of our collaboration with AWS, and to broaden the adoption of crowd tracing, Clio GO app users can now pre-screen their visitors at no cost for personal, non-profit, faith- based, educational, and amateur athletic event use.</p> 
<h2>How you can contribute</h2> 
<p>We welcome everyone to participate in this collaborative effort. Using the app improves your and your visitor’s safety while contributing anonymized crowd tracing data to the open-source public AWS COVID-19 data lake.</p> 
<p>In just a few minutes, you can <a href="https://events.clio.health" rel="noopener noreferrer" target="_blank">get a free Clio GO account</a> and use the Clio GO app to <a href="https://events.clio.health" rel="noopener noreferrer" target="_blank">pre-screen people attending personal events and private clubs</a>—whether small dinner parties, soccer matches, or religious gatherings. You can <a href="https://gopass.clio.health" rel="noopener noreferrer" target="_blank">purchase additional hardware</a> for unmanned door screening or mobile kiosk functionality, as well as solutions for commercial enterprises.</p> 
<h2>AWS COVID-19 data lake and crowd tracing data</h2> 
<p>To make the data from the AWS COVID-19 data lake available in the Data Catalog in your AWS account, create a CloudFormation stack using the following&nbsp;<a href="https://us-east-2.console.aws.amazon.com/cloudformation/home?region=us-east-2#/stacks/create/review?templateURL=https://covid19-lake.s3.us-east-2.amazonaws.com/cfn/CovidLakeStack.template.json&amp;stackName=CovidLakeStack">template</a>. This template creates a covid-19 database in your Glue Data Catalog and tables that point to the public AWS COVID-19 data lake. This includes a&nbsp;<code>aspirevc_crowd_tracing&nbsp;</code>table which points to up-to-date crowd tracing data, and also a&nbsp;<code>aspirevc_crowd_tracing_zipcode_3digits&nbsp;</code>table which points to a lookup which translates 3 digits zip codes used in the&nbsp;<code>aspirevc_crowd_tracing&nbsp;</code>table to the respective states.</p> 
<p>You can query these tables using Amazon Athena. Athena is a serverless interactive query service that makes it easy to analyze the data in the AWS COVID-19 data lake. Athena supports SQL, a common language that data analysts use for analyzing structured data. To query the data, complete the following steps:</p> 
<ol> 
 <li>Sign in to the <a href="https://us-east-2.console.aws.amazon.com/athena/home" rel="noopener noreferrer" target="_blank">Athena console</a>. 
  <ol type="a"> 
   <li>If this is the first time you are using Athena, you must <a href="https://docs.aws.amazon.com/athena/latest/ug/querying.html#query-results-specify-location" rel="noopener noreferrer" target="_blank">specify a query result location</a> on <a href="http://aws.amazon.com/s3" rel="noopener noreferrer" target="_blank">Amazon S3</a>.</li> 
  </ol> </li> 
 <li>From the drop-down menu, choose the <code>covid-19</code> database.</li> 
 <li>Enter your query.</li> 
</ol> 
<p>The following query returns statistics including the number of people marked as <code>symptoms</code>, <code>diagnosed</code>, <code>contact</code>, and <code>near</code> for the given scan date:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">SELECT
  cast(from_iso8601_timestamp(scandate) as date) as date,
  COUNT_IF(symptoms) as symptoms_count,
  COUNT_IF(diagnosed) as diagnosed_count,
  COUNT_IF(contact) as contact_count,
  COUNT_IF(near) as near_count, COUNT(*) as total_count
FROM "covid-19"."aspirevc_crowd_tracing"
WHERE from_iso8601_timestamp(scandate)
BETWEEN parse_datetime('2020-10-01:00:00:00','yyyy-MM-dd:HH:mm:ss')
AND parse_datetime('2020-10-16:00:00:00','yyyy-MM-dd:HH:mm:ss')
GROUP BY 1
ORDER BY 1
</code></pre> 
</div> 
<p><code>symptoms</code>: Past 2 weeks, have you had any of the following symptoms: shortness of breath, fever, loss of taste or smell, new cough?</p> 
<p><code>diagnosed</code>: Past 2 weeks, have you been diagnosed with COVID or are waiting for COVID test results?</p> 
<p><code>contact</code>: Past 2 weeks, have you been in contact with anyone who has been diagnosed with COVID or is waiting for COVID test results?</p> 
<p><code>near</code>: Past 2 weeks, have you been near anyone with the following symptoms: shortness of breath, fever, loss of taste or smell, new cough?</p> 
<p>The following screenshot shows the results of this query:</p> 
<p><img alt="" class="alignnone size-full wp-image-12897" height="679" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/10/28/aspire-crowd-tracing-2.jpg" style="margin: 10px 0px 10px 0px; border: 1px solid #cccccc;" width="1000" /></p> 
<p>To see more details, you can run the following query to retrieve the same statistics per <code>state</code> per <code>risklevel</code> for the given scan date:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">SELECT
  cast(from_iso8601_timestamp(scandate) as date) as date,
  SUBSTR(scannerdevice_zipcode, 1, 3) as zip,
  state,
  risklevel,
  result,
  COUNT_IF(symptoms) as symptoms_count,
  COUNT_IF(diagnosed) as diagnosed_count,
  COUNT_IF(contact) as contact_count,
  COUNT_IF(near) as near_count, 
  COUNT(*) as total_count
FROM "covid-19"."aspirevc_crowd_tracing"
JOIN "covid-19"."aspirevc_crowd_tracing_zipcode_3digits" ON SUBSTR(scannerdevice_zipcode, 1, 3) = aspirevc_crowd_tracing_zipcode_3digits.zip
WHERE from_iso8601_timestamp(scandate)
BETWEEN parse_datetime('2020-10-15:00:00:00','yyyy-MM-dd:HH:mm:ss')
AND parse_datetime('2020-10-16:00:00:00','yyyy-MM-dd:HH:mm:ss')
AND scannerdevice_zipcode&lt;&gt;''
GROUP BY 1,2,3,4,5
ORDER BY 1,3</code></pre> 
</div> 
<p>The following screenshot shows the results of this query:</p> 
<p><img alt="" class="alignnone size-full wp-image-12898" height="700" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/10/28/aspire-crowd-tracing-3.jpg" style="margin: 10px 0px 10px 0px; border: 1px solid #cccccc;" width="1000" /></p> 
<p>You can see that there are a small number of people marked as <code>symptoms</code>, <code>diagnosed</code>, <code>contact</code>, and <code>near</code> per <code>state</code> per <code>risklevel</code>.</p> 
<p><img alt="" class="alignnone size-full wp-image-12899" height="646" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/10/28/aspire-crowd-tracing-4.jpg" style="margin: 10px 0px 10px 0px; border: 1px solid #cccccc;" width="1000" /></p> 
<p>By open-sourcing the data, we see possibilities to combine it with other AWS COVID-19 data lake datasets, such as hospitalizations or COVIDcast data. This can enable a new game feature such as a radar that predicts regional hotspot emergence.</p> 
<p>If you’re a data analyst, we encourage you to contribute to building better crowd tracing algorithms using any of the data provided in the public AWS Covid-19 data lake. Even if you’re building a different solution, you can use this dataset without license fees. The following section can help you quickly get started.</p> 
<h2>The data</h2> 
<p>Although our public AWS COVID-19 data lake has excellent data sources, such as hospitalization data down to regional levels, Clio GO provides data even at the zip-code level. Below is a description of the schema of the data made available in the data lake:</p> 
<p><img alt="" class="alignnone size-full wp-image-12905" height="646" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2020/10/29/aspire-crowd-tracing-5.jpg" style="margin: 10px 0px 10px 0px; border: 1px solid #cccccc;" width="1000" /></p> 
<h2>Strong data privacy and cryptographic pseudonymity via Powch</h2> 
<p>In contrast to the significant privacy challenges associated with contact tracing, crowd tracing and the Clio GO system don’t require disclosure of contact identities to reduce your risk for COVID-19 infection. Returning to the video game analogy, learning the identity after the fact of who hit your shields doesn’t matter. However, knowing that that you’re entering a crowded map of strongly armed assailants might cause you to choose a different crowd. Therefore, the aggregate risk of the crowd becomes the only relevant concern for your risk of contracting COVID-19.</p> 
<p>To protect your identity, the Clio GO app uses <a href="https://powch.com" rel="noopener noreferrer" target="_blank">Powch</a>, a powerful cryptographic technology that protects identity and data. Similar to Bitcoin, Powch enables pseudonymity—a way to log in without any linkage to your true identity. You don’t need to use any personally identifiable information for Clio Go registration. Instead, an unguessable and random secret ID is stored in a personal QR code, which only you have access to, and GO Pass has no knowledge about the owner of the secret ID. The secret ID is used during registration as the only form of identity in the Go Pass app.</p> 
<p>After exposure to a possibly infected individual, contact tracing requires you to exactly identify all the individuals that you interacted with during the same period of time, and compromise their privacy as well as your own.</p> 
<p>Although you may choose to be identified and share your name with the venue you visit, the Clio GO app never shows personal data, like actual temperature readings, to the venue owner. The app only tells the venue whether the GO Pass was accepted or denied based on the entry requirements of the venue.</p> 
<h2>Crowd-sourced solutions, open data, restrictions, and uses</h2> 
<p>We hope that free access to the crowd tracing data via the public AWS COVID-19 data lake encourages the development of new creative, low-cost COVID-19 mitigation solutions. You can use the data within commercial products under a creative commons license with the explicit requirement that algorithms developed from the public dataset are open and published.</p> 
<p>We encourage using the crowd tracing data in the public AWS COVID-19 data lake in conjunction with other free data sources also in the lake. A commercial data feed with fewer restrictions and data limitations is being made available via <a href="https://aws.amazon.com/data-exchange/" rel="noopener noreferrer" target="_blank">AWS Data Exchange</a> to commercial organizations who pass verification requirements.</p> 
<hr /> 
<h2>About the authors and Aspire Ventures</h2> 
<p>Aspire Ventures has developed a <a href="https://youtu.be/EVnZpdZzlK4" rel="noopener noreferrer" target="_blank">novel artificial intelligence engine called A<sup>2</sup>i</a> and joint ventures with mission-driven health systems. Aspire’s first joint venture with Penn Medical Lancaster General Health and Capital BlueCross established the <a href="http://www.ilab.health" rel="noopener noreferrer" target="_blank">Smart Health Innovation Lab</a>, an entity that accelerates healthcare technologies that impact the quadruple aim. Aspire is partnering with <a href="https://www.businesswire.com/news/home/20190514005724/en/Mor-Research-Applications-a-Division-of-Clalit-Health-Services-Partners-with-Aspire%C2%A0Ventures-and-Smart-Health-Innovation-Lab" rel="noopener noreferrer" target="_blank">Clalit, the majority health system of Israel</a>, in a similar joint venture focused on Israeli start-ups to encourage innovation in healthcare.</p> 
<p>Aspire Ventures was founded by Essam Abadir, SB MIT Mathematics, SB Sloan School of Management, and JD with Distinction from the University of Iowa School of Law. Essam founded Aspire Ventures as an impact investment AI firm in 2014 after selling an apps platform to Intel in 2013.</p> 
<p>A2i is overseen by Victor Owuor, SB &amp; MS Aeronautical and Astronautical Engineering MIT, SB &amp; MS Electrical Engineering MIT, and JD Harvard School of Law. Prior to Aspire, Victor headed a significant cloud P&amp;L at Oracle.</p> 
<p>The Aspire Ventures CIO and Smart Health Innovation Lab CEO is Kim Ireland, MSIS Penn State University. Kim formerly managed health system EHR rollouts at Cerner and was CEO of startup MedStatix.</p> 
<p>Scott Schell, PhD Immunology University of Chicago, MD University of Chicago, and MBA University of Michigan, heads Clio Health Go and is Chief Medical Officer for the Aspire portfolio. Scott was Founding Chair of Cleveland Clinic’s Department of Population Health and led development of two of healthcare’s largest platforms at Alere and UPMC.</p> 
<p>Clio Health GO’s mission is to reinvent the healthcare experience via advanced telehealth, starting with COVID-19. GO Pass and the crowd tracing data and algorithm is a product of Medstatix. Powch provides patented cryptographic privacy and security technologies. Connexion Health provides the GO Pass kiosks. Clio Health GO, Medstatix, Connexion Health, and Powch are portfolio companies of Aspire.</p>
