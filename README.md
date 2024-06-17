# Introduction 

Apple Inc. is one of the leading tech companies in the world. Headquartered in Cupertino, the company specializes in personal digital devices such as the iPhone, MacBook, iPad, and others. Apple products have become an integral part of our daily lives. If it's not an iPhone, you may be using a MacBook or an iPad. Even if you don't use these products yourself, chances are you have a friend who does. It's no surprise, then, the level of attention the company garners during product releases and conferences. At their WWDC in 2024, Apple received significant attention from various social media sources. It is against this backdrop that I decided to perform sentiment analysis based on the Twitter audience. 

## Data Sources 
- The source of data for this project is X tweets(formerly known as Twitter)
- X link: [https://x.com/home]
### Tools, Services, and Software
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
### Getting environmental variables: 

- Since we are ingesting data from Twitter, we need to set up certain variables that will enable us to extract the data
- First, we will need consumer keys: the API key and the associated secret key. These can be found by clicking projects and apps inside the developer account:
  
<img width="670" alt="Screenshot 2024-06-17 at 12 04 44 PM" src="https://github.com/KelvinAmwata/APPLE-INC.--Sentimental-Analysis/assets/83902270/386ccb99-8daa-47de-9ee7-dfcc650978a7">
-  Below the consumer keys, we will get the bearer token and access token keys as well as their respective secrets

<img width="666" alt="Screenshot 2024-06-17 at 12 15 07 PM" src="https://github.com/KelvinAmwata/APPLE-INC.--Sentimental-Analysis/assets/83902270/448b2896-5630-4d56-b196-cce4296f7aed">

### Storing the environmental variables in the lambda environment 
- Having gotten our variables, the next step is to input them in the AWS environment. This is a good way of storing secrets. Please note that sensitive information should not be stored as plain text
-  Search for AWS Lambda on the search bar
-  Click Create a function and give your function a descriptive name
-  For the runtime, select python

<img width="854" alt="Screenshot 2024-06-17 at 12 21 49 PM" src="https://github.com/KelvinAmwata/APPLE-INC.--Sentimental-Analysis/assets/83902270/555e0f0a-fbec-489e-85f7-4cb16c931d6a">
  
-  For the permissions, ensure the role you give your function has full access to Firehose and S3
-  We are now done creating our function and we need to set the variables that we need to access the Twitter API
-  Click on configurations and then environment variables on the left side

  <img width="754" alt="Screenshot 2024-06-17 at 12 31 19 PM" src="https://github.com/KelvinAmwata/APPLE-INC.--Sentimental-Analysis/assets/83902270/14de5603-7c4b-4a39-9871-cf62d89263cb">

- Click edit and add all the variables:  Access token, API key, and bearer token. The value of the variables will be the secret keys
- Now we have all our variables set
- Let us now  write a code that will ingest data from Twitter:
  ~~
import boto3
import json
import os
import tweepy
from datetime import datetime

## Initialize the Firehose client
firehose = boto3.client('firehose')

## Initialize the Comprehend client
comprehend = boto3.client('comprehend')

## Twitter API credentials from environment variables
TWITTER_API_KEY = os.getenv('TWITTER_API_KEY')
TWITTER_API_SECRET_KEY = os.getenv('TWITTER_API_SECRET_KEY')
TWITTER_ACCESS_TOKEN = os.getenv('TWITTER_ACCESS_TOKEN')
TWITTER_ACCESS_TOKEN_SECRET = os.getenv('TWITTER_ACCESS_TOKEN_SECRET')
TWITTER_BEARER_TOKEN = os.getenv('TWITTER_BEARER_TOKEN')

## Check if all environment variables are set
if not all([TWITTER_API_KEY, TWITTER_API_SECRET_KEY, TWITTER_ACCESS_TOKEN, TWITTER_ACCESS_TOKEN_SECRET, TWITTER_BEARER_TOKEN]):
    raise EnvironmentError("One or more Twitter API credentials are not set in environment variables.")

## Tweepy client initialization
client = tweepy.Client(bearer_token=TWITTER_BEARER_TOKEN, wait_on_rate_limit=True)

# Firehose delivery stream.
DELIVERY_STREAM_NAME = 'PUT-S3-EI36D'

def lambda_handler(event, context):
    try:
        # Fetch recent tweets containing Amazon key word
        response = client.search_recent_tweets(query="amazon", max_results=100)

        if response.data:
            for tweet in response.data:
                # Structure the tweet data for Glue schema inference
                structured_tweet = {
                    'id': tweet.id,
                    'text': tweet.text,
                    'created_at': tweet.created_at.isoformat() if tweet.created_at else None,
                    'author_id': tweet.author_id,
                    'lang': tweet.lang,
                    'possibly_sensitive': tweet.possibly_sensitive if 'possibly_sensitive' in tweet.data else None,
                    'source': tweet.source
                }

                # Call Amazon Comprehend to analyze the tweet text
                comprehend_response = comprehend.detect_sentiment(Text=tweet.text, LanguageCode='en')
                structured_tweet['sentiment'] = comprehend_response['Sentiment']
                structured_tweet['sentiment_score'] = comprehend_response['SentimentScore']

                # Prepare the record data
                record_data = json.dumps(structured_tweet) + '\n'

                # Send the record to Firehose
                firehose_response = firehose.put_record(
                    DeliveryStreamName=DELIVERY_STREAM_NAME,
                    Record={'Data': record_data}
                )
                print("Firehose put_record response:", firehose_response)

        return {
            'statusCode': 200,
            'body': json.dumps('Tweets processed successfully.')
        }

    except tweepy.Unauthorized as e:
        return {
            'statusCode': 401,
            'body': json.dumps('Unauthorized access - please check your Twitter API credentials.')
        }
    except Exception as e:
        print(f'An error occurred: {str(e)}')
        return {
            'statusCode': 500,
            'body': json.dumps(f'An error occurred: {str(e)}')
        }
  ~~




