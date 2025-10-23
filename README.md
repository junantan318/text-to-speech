# Serverless Text-to-Speech App

**Live Demo:** [View the app here]([https://your-s3-bucket-name.s3.amazonaws.com/index.html](http://www-audioposts-154.s3-website.eu-north-1.amazonaws.com))

This project demonstrates how to use Amazon Web Services (AWS) to convert text into natural-sounding speech.
It follows the official AWS tutorial for using Amazon Polly and can synthesize speech from plain text input.

## features
- Converts text to lifelike speech using AWS TTS (e.g., Amazon Polly)
- Supports multiple voices and languages
- Outputs audio in MP3 or WAV format
- Simple command-line or API-based usage

## tech stack
AWS: Amazon S3 , API Gateway , AWS Lambda , Dynamo DB , Amazon Polly , Amazon SNS
Frontend: HTML , Javascript

<img width="1336" height="706" alt="text to speech drawio" src="https://github.com/user-attachments/assets/0578d62e-2ab6-441d-8b27-61ce8a78326c" />

The application is a serverless text-to-speech web app built using AWS services.

- The frontend is a static website hosted on Amazon S3, which lets users enter text to convert into speech.

- When the user submits text, it’s sent through an API Gateway, which triggers an AWS Lambda function.

- This Lambda function stores the request details in Amazon DynamoDB, keeping track of all text-to-speech requests.

- Another Lambda function (triggered asynchronously via Amazon SNS) takes the text, uses Amazon Polly to generate the MP3 audio, and saves the result back into Amazon S3.

- Finally, the DynamoDB record is updated with the link to the generated MP3 file, which the frontend can display or play back.

<img width="1857" height="471" alt="image" src="https://github.com/user-attachments/assets/c2d3bd29-e4c7-4a85-9d81-84663639c1aa" />
The user can input any text and select a preferred voice. The application then converts the text into speech using AWS services and generates a unique Post ID.
This Post ID can be used in the search field to retrieve the generated result.
When retrieved, the application provides both an MP3 audio file and a preview player so the user can listen to the generated voice.


## Quick Start
1) **Prerequisites**

- AWS account + credentials configured locally (aws configure)

- An S3 bucket for the static website: YOUR_WEB_BUCKET

- An S3 bucket for audio files (mp3): YOUR_AUDIO_BUCKET

- Two Lambda functions:

  - NewPost (creates a post ID and writes to DynamoDB, publishes to SNS)

  - ConvertToAudio (reads post, calls Polly, writes MP3 to S3, updates DynamoDB)

- A DynamoDB table: Posts (PK: postId as STRING)

- An SNS topic: tts-new-posts

- API Gateway REST API with endpoints that invoke NewPost and a “Get Post” Lambda

Tip: In your frontend app.js (or similar), set API_BASE_URL to your API Gateway invoke URL.

2) **Deploy the frontend (static site)**
- build or upload your own html/js/css file

3) **Environment variables (Lambda)**

Set these on both Lambdas (or where appropriate):

- POSTS_TABLE=Posts

- AUDIO_BUCKET=YOUR_AUDIO_BUCKET

4) **Test it**

1. Open your S3 website URL (or CloudFront URL).

2. Enter text, choose a voice, click Say it!

3. Copy the Post ID shown.

4. Use the search box to retrieve it; the row should update to UPDATED with a playable MP3.

**IAM: Minimal Policies (copy & adapt)**
Execution role for NewPost (Lambda)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    { "Effect": "Allow", "Action": ["logs:CreateLogGroup","logs:CreateLogStream","logs:PutLogEvents"], "Resource": "arn:aws:logs:*:*:*" },
    { "Effect": "Allow", "Action": ["dynamodb:PutItem","dynamodb:UpdateItem","dynamodb:GetItem"], "Resource": "arn:aws:dynamodb:REGION:ACCOUNT_ID:table/Posts" },
    { "Effect": "Allow", "Action": ["sns:Publish"], "Resource": "arn:aws:sns:REGION:ACCOUNT_ID:tts-new-posts" }
  ]
}
```
Execution role for ConvertToAudio (Lambda)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    { "Effect": "Allow", "Action": ["logs:CreateLogGroup","logs:CreateLogStream","logs:PutLogEvents"], "Resource": "arn:aws:logs:*:*:*" },
    { "Effect": "Allow", "Action": ["polly:SynthesizeSpeech"], "Resource": "*" },
    { "Effect": "Allow", "Action": ["s3:PutObject","s3:GetObject"], "Resource": "arn:aws:s3:::YOUR_AUDIO_BUCKET/*" },
    { "Effect": "Allow", "Action": ["dynamodb:UpdateItem","dynamodb:GetItem"], "Resource": "arn:aws:dynamodb:REGION:ACCOUNT_ID:table/Posts" }
  ]
}
```
## Troubleshooting Log

1) KeyError: 'Records' in Lambda
Fix
- Use an SNS-shaped test event:
```
{
  "Records": [
    { "Sns": { "Message": "12345" } }
  ]
}
```

2) DynamoDB permissions (IAM)

Symptom
Access errors or inability to read items.

Fix
- Attached an inline policy to the Lambda execution role with the table ARN:
```
{
  "Version":"2012-10-17",
  "Statement":[
    {
      "Effect":"Allow",
      "Action":[ "dynamodb:GetItem","dynamodb:Query","dynamodb:Scan" ],
      "Resource":"arn:aws:dynamodb:eu-north-1:854681582170:table/posts"
    }
  ]
}
```
3) 405 from S3 Website — wrong endpoint in scripts.js

Error (browser console)

~~~
POST http://www-audioposts-154.s3-website.eu-north-1.amazonaws.com/...
405 (Method Not Allowed)
~~~

Cause
API_ENDPOINT pointed to the S3 website with the API URL mashed into the path.

Fix
- Pointed JavaScript directly at the API Gateway invoke URL:

```
// BEFORE (wrong)
var API_ENDPOINT = "api gateway - https://ojn7442whi.execute-api.eu-north-1.amazonaws.com/Dev";

// AFTER (correct)
var API_ENDPOINT = "https://ojn7442whi.execute-api.eu-north-1.amazonaws.com/Dev";
```
