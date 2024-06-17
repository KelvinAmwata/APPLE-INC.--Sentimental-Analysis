## Introduction 

Apple Inc. is one of the leading tech companies in the world. Headquartered in Cupertino, the company specializes in personal digital devices such as the iPhone, MacBook, iPad, and others. Apple products have become an integral part of our daily lives. If it's not an iPhone, perhaps you're using a MacBook or an iPad. Even if you don't use these products yourself, chances are you have a friend who does. It's no surprise, then, the level of attention the company garners during product releases and conferences. At their WWDC in 2024, Apple received significant attention from various social media sources. It is against this backdrop that I decided to perform sentiment analysis based on the Twitter audience. 

# Data Sources 
- The source of data for this project is X tweets(formerly known as twitter)
- X link: [https://x.com/home]
## Tools,Services, and Software
- AWS Account
- AWS Lambda
- AWS Firehose
- AWS Athena
- AWS Glue
- AWS S3
- AWS Quicksight
- AWS 

# Project Steps 

## Creating an AWS Account: 
Note: This project assumes that you already have an AWS Account. If you do not have one, please follow this link: [https://docs.aws.amazon.com/accounts/latest/reference/manage-acct-creating.html]
## Create an S3 Bucket 
- We need an S3 bucket to store our data. Amazon Simple Storage Service (S3) is an object storage service that is highly effective for data storage.
- It is from the s3 bucket that glue will crawl the data and create a schema
- Ensure the name that you give to your bucket is unique. Bucket names are global by nature. It's advisable to use descriptive names for reference purposes
-  Steps to creating a bucket: [https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html]
-  
