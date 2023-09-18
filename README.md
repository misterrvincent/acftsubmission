# acftsubmission
This is technical write-up of how Zack and I created a web app to push fitness score submissions using a complete cloud environment on AWS. While the application itself is a very rudimentary representation of python's capabilities, our focus was in demonstrating deep knowledge on deploying an application on the Amazon Web Services (AWS) platform. It follows the Six Pillars of the Well-Architected Framework provided by AWS. We heavily focused on the most secure protocols considering the Army's ACFT scores generally contain a small amount of PII, but for all intents and purposes we did not include PII submission with the application as it is just a "simulated" HIPAA envrionment. In conjunction with secure protocols, we implemented the lowest-cost environment considering the application is showcasing our deep knowledge of AWS, we found it unnecessary to make it highly available and only moderately resilient. 

![image](https://github.com/misterrvincent/acftsubmission/assets/145295366/1fb7097f-3bd5-44ec-ab49-fbafcd27c3f3)
**Image 1.0**

Image 1.0 is our workflow in Amazon Web Services. Most of the architecture is built to be extremely secure, moderately resilient but not at all available. Reason being is highly available architectures aren't much more expensive, but for a display application with not an extreme amount of functionality would be an unnecessary expense on our part. We wanted to keep the project to under $10 a month to host on AWS. Our most expensive cost for the project so far is the wAF. It will cost us about $8 for the WAF and a few cents for the rest of the architecture. 

**1.** Steps to gain access to the static website hosted on the S3 Bucket which hosts the submission form.
   
1a. All internet traffic will flow from the Cloudfront Edge Location to the WAF, where it will have to pass checks through the WebACL and geolocation restrictions. The WAF will also protect the CF distribution from common web attacks such as SQL injections, XSS. While AWS Shield is enabled by default to protect against DDoS attacks, Advanced which will offer more protection against DDos attacks is not enabled by dafault and costs quite a bit of money so it was not enabled for the application. 

1b. The WAF allows access to an end-user to the distribution. The distribution then proceeds to initiate the next steps:
  1c. Check for the .html file in the cache, if there is a cache hit, the website will return the file.
  1d. If cache miss, the distribution routes to the source .html file in the S3 bucket using it's Origin Access Control to access the bucket. 
  
**2.**  The S3 bucket checks it's bucket policy for the Cloudformation Distribution "Allow" rule using the distribution's ARN as it's source ARN. If true, it provides the file and also decrypts the file using the SSE-KMS customer managed key. 

**3.** The distribution then sends it to the CF Edge Location, stores it in the cache and then displays it to the end-user. 

**4.** The user enters their ACFT data into the static page via a javascript form. The "Submit" button initiates javascript to convert form entries into python which is sent to an API gateway.

**5.** The API gateway triggers an SQS message which will include the python code with the ACFT entries.

**6.** Lambda polls the message, ingests the data and then converts the code to SQL to send to an RDS database.

**7.** The lambda function assumes the IAM role that will allow "UPDATE" SQL commands to the MySQL RDS Instance and uses the decryption/encryption IAM policy to modify and then reencrypt the TABLE "acft".

**8. thru 9.** The MySQL Instance invokes an event to Eventbridge which will trigger another SQS queue.

**10.** Another lambda function polls the SQS queue. 

**11. thru 12.** The function will execute an "UPDATE" command to the MySQL instance to update a "VIEW" in the RDS, create a .html file and replace the .html file (separate from the index.html file) that has the initial view configuration in it. It will assume the role which will give access to the S3 bucket and decryption method for KMS.

There will be additional tasks we want to add to the workflow such as AWS Quicksights to create graphs and visual statistics for the data but that will be implemented post-initial deployment of the application. 
