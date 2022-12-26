##       The Cloud Resume Challenge Project  

I came across **The Cloud Resume Challenge** (To learn more about the challenge, you can visit the webpage [here](https://cloudresumechallenge.dev/docs/the-challenge/aws/).) when I was looking for hands-on labs to practice my Cloud skills.  I spent a lot of my time working on this project,  and the result is that I built a serverless resume website which is completely hosted on AWS:  **[yidichen.com]()**       

This post describes the steps I took to finish this project, from building the front end of the website using AWS services such as S3, CloudFront, and Route53 to setting up a visitor counter using services including API Gateway, Lambda, and DynamoDB. 

**Table of Contents:**

[TOC]

 **High-level architecture diagram:** 

![](Q:\NetOps\Devops\Project\The Cloud resume challenge\The Cloud Resume Challenge Project\Cloud resume architecture diagram.PNG)



#### Requirement for this challenge

Need to be AWS certified.  

AWS Solutions Architect - Associate  :ballot_box_with_check:



### Configure frontend of the static website 



#### Create S3 bucket 

First, I created an S3 bucket to host my static website.

Next, I enabled **static website hosting** for the bucket and uploaded the HTML index file and other necessary files for my website to the bucket.

![](Q:\NetOps\Devops\Project\The Cloud resume challenge\The Cloud Resume Challenge Project\Screenshots\Capture2.PNG)



#### Set up CloudFront to distribute web content 

Amazon CloudFront is a CDN managed by AWS, using CloudFront helps to improve website performance in which the global users can access web content that is cached in edge locations directly.

I chose my S3 bucket as the Origin domain for CloudFront and selected **OAI** to limit S3 bucket access to CloudFront only. 

![](Q:\NetOps\Devops\Project\The Cloud resume challenge\The Cloud Resume Challenge Project\Screenshots\cloudfront1.PNG)



After the distribution is created, make sure to update the bucket policy to allow access to CloudFront IAM service principal role.

![](Q:\NetOps\Devops\Project\The Cloud resume challenge\The Cloud Resume Challenge Project\Screenshots\bucketPolicy.PNG)



Set **Viewer protocol policy** to "Redirect HTTP to HTTPS"  since the website is required to use HTTPS for security. Leave other settings of CloudFront as default. 



#### Set up Route53 

Before setting up Route53, a domain name is required for the website. I purchased the domain name "yidichen.com" from Namecheap. Then I added name servers for the domain in my Namecheap account.

I also requested a certificate for the domain using **Amazon Certificate Manager (ACM)**. once the ACM is validated, I created another CloudFront distribution and added records in Route53, the internet traffic for my domain will be routed to the CloudFront distribution.  Next, I deleted the CloudFront distribution I created previously.

![](Q:\NetOps\Devops\Project\The Cloud resume challenge\The Cloud Resume Challenge Project\Screenshots\Route53_recordsUpdate.png)



Make sure to specify the default root object in the CloudFront Distribution.

![](Q:\NetOps\Devops\Project\The Cloud resume challenge\The Cloud Resume Challenge Project\Screenshots\RootObject.PNG)                                       



### Configure serverless backend of the website 

The resume challenge requires that your resume webpage include a visitor counter which tracks and shows how many people have visited the site. To achieve this function, the JavaScript in my static webpage invokes API URL  GET request to API Gateway.  Trigger lambda function. 

#### Create DynamoDB Table

I created a DynamoDB table named "visitor_count", then I added an item named "visitor_count" to the table and set its initial value to 1. 

![](Q:\NetOps\Devops\Project\The Cloud resume challenge\The Cloud Resume Challenge Project\Screenshots\DBTable.PNG)



#### Create Lambda Function 



Attach *AmazonDynamoDBFullAccess* policy to the IAM role created by Lambda 





#### Create API Gateway to call Lambda function 

I created a new REST API in the API Gateway console, then I created a GET method API and integrated it with the Lambda function I created earlier. Once the API is deployed and tested,  I enabled Cross Origin Resource Sharing (CORS) on the API.

![](Q:\NetOps\Devops\Project\The Cloud resume challenge\The Cloud Resume Challenge Project\Screenshots\API Gateway_GetMethod.PNG)

An endpoint is created when the API is deployed,  the endpoint URL will be used on my website to invoke the API. 

I created a Javascript function in my webpage to call the API endpoint every time the webpage is loaded. 

```
function visitorCount(){
    		fetch('https://t8k12v6an6.execute-api.us-east-2.amazonaws.com/dev') //API gateway endpoint URL
      		.then(response => response.json())
      		.then((data) => {
        	document.getElementById('visitor_count').innerHTML = data.body;
			console.log(data);
           
      		})
		}
		
   visitorCount();
```

The visitor counter displayed on my website: 

![](Q:\NetOps\Devops\Project\The Cloud resume challenge\The Cloud Resume Challenge Project\Screenshots\visitor_count.PNG)



### Conclusion 
