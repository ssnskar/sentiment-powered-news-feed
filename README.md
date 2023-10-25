# Sentiment Powered News Feed

## Overview

In the digital age, staying informed about current events is essential, but the sheer volume of news available can be overwhelming. To address this issue, we present the "Sentiment-Powered News Feed" project, a novel solution that leverages the power of the AWS services to deliver a more personalized and emotionally resonant news experience. 

[AWS Lambda](https://aws.amazon.com/lambda/) is a serverless, event-driven compute service that lets you run code for virtually any type of application or backend service without provisioning or managing servers. 

[Amazon Comprehend](https://aws.amazon.com/comprehend/) is a natural-language processing (NLP) service that uses machine learning to uncover valuable insights and connections in text. 

[Amazon DynamoDB](https://aws.amazon.com/dynamodb/) is a fully managed, serverless, key-value NoSQL database designed to run high-performance applications at any scale. 

[Amazon API Gateway](https://aws.amazon.com/api-gateway/) is a fully managed service that makes it easy for developers to create, publish, maintain, monitor, and secure APIs at any scale.

## Objective

The primary objective of the "Sentiment-Powered News Feed" project is to revolutionize the way people consume news by tailoring news articles to their emotional preferences.

## Architecture

![NewDiagran](https://github.com/ssnskar/sentiment-powered-news-feed/assets/89893005/291b4a66-af66-4472-82ec-e802ac1079c6)

[News API](https://newsapi.org/) is a simple, easy-to-use REST API that returns JSON search results for current and historic news articles published by over 80,000 worldwide sources. 

## Prerequisites

- AWS account
- Python 3.11
- Requests Package as Layer

## Instructions

### Get your API KEY from NEWS API

You will need to log in [News API](https://newsapi.org/) to get your personalised API Key. This key will be used in the Lambda function 1 to extract the news headlines.

### Creation of DynamoDB Table

We would need to create a DynamoDB table to store the news based on sentiment and timestamp.
Here,
PARTITION KEY : sentiment (String)
SORT KEY : timestamp (String)

You can refer to the DynamoDB.yml for the Clouformation script of the same.

AWS CLI [command](https://docs.aws.amazon.com/cli/latest/reference/dynamodb/create-table.html) for the table creation :

```
aws dynamodb create-table \
    --table-name news \
    --attribute-definitions AttributeName=sentiment,AttributeType=S AttributeName=timestamp,AttributeType=S \
    --key-schema AttributeName=sentiment,KeyType=HASH AttributeName=timestamp,KeyType=RANGE \
    --billing-mode PAY_PER_REQUEST
```

### Creation of the Lambda function 1

This function is responsible for extracting the current news using the **NEWS API**, calls the **Amazon Comprehend** to determine the sentiments of the news and categorize them in three categories : Positive, Negative and Neutral. Finally this Lambda function stores this data in a **DynamoDB** table.

The function uses Python 3.11 runtime and calls the various AWS services using [**Boto3**](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html).

The function code is available at [**Derive.py**](Functions/DeriveNews.py)

**NOTE** : This function requires **REQUESTS** module to work. Therefore, make sure to add the Request module as a lambda Layer.

In order to save the Request Module, find refer [here](https://www.keyq.cloud/en/blog/creating-an-aws-lambda-layer-for-python-requests-module). To create a Lambda layer using the zip file, refer [here](https://docs.aws.amazon.com/lambda/latest/dg/creating-deleting-layers.html#layers-create).

### Execution of the Lambda function 1

**NOTE** : The Lambda function execution role requires permission for Amazon Comprehend (**comprehend:DetectSentiment**) and Amazon DynamoDB (**dynamodb:PutItem**) to perform the execution. Refer to policy.json for the inline policy required to perform these actions.

This Lambda function needs to be executed on a regular basis in order to store the news headline. Hence, you can make use of the **EventBridge** to create a [rule](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-events-rule.html#cfn-events-rule-scheduleexpression) to execute the function on a scheduled time interval.

### Creation of API Gateway
### Creation of the Lambda function 2
