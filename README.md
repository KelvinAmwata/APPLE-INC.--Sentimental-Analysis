## Introduction 

Apple Inc. is one of the leading tech companies in the world. Headquartered in Cupertino, the company specializes in personal digital devices such as the iPhone, MacBook, iPad, and others. Apple products have become an integral part of our daily lives. If it's not an iPhone, you may be using a MacBook or an iPad. Even if you don't use these products yourself, chances are you have a friend who does. It's no surprise, then, the level of attention the company garners during product releases and conferences. At their WWDC in 2024, Apple received significant attention from various social media sources. It is against this backdrop that I decided to perform sentiment analysis based on the Twitter audience. 

# Data Sources 
- The source of data for this project is X tweets(formerly known as Twitter)
- X link: [https://x.com/home]
## Tools, Services, and Software
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
## Create Kinesis Firehose Data Stream 
- Kinesis firehose data stream is a near real-time data ingestion service that ingests data from a source and loads it to a storage service. In this project, kinesis firehose will load data from lambda and load it into an S3 bucket.
-  Steps to creating firehose data stream:[https://docs.aws.amazon.com/firehose/latest/dev/basic-create.html]
## Create Athena: 
- After Kinesis firehose pushes the data into an S3 bucket, the next step is to query the data using Athena so that the data can be ready for visualizations and other business needs
- Search for Athena on the search bar.
- Click on settings, and then click on manage settings. Here we want to set the location where we will store the results of our queries. So open a duplicate go back to S3 and create another bucket. Remember to adhere to s3 naming conventions
- Now we create the database. On the query editor, enter DDL: CREATE DATABASE database name. Replace the database name with the name you want for your database
- We now have Athena set up and we are ready to crawl the data

# Data Ingestion 

## Setting up environment variables

- Since we are ingesting data from Twitter, we need to set up certain variables that will enable us to extract the data
- First, we will need an API key and the associated secret key
- 
