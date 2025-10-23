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

  -NewPost (creates a post ID and writes to DynamoDB, publishes to SNS)

  -ConvertToAudio (reads post, calls Polly, writes MP3 to S3, updates DynamoDB)

- A DynamoDB table: Posts (PK: postId as STRING)

- An SNS topic: tts-new-posts

- API Gateway REST API with endpoints that invoke NewPost and a “Get Post” Lambda

Tip: In your frontend app.js (or similar), set API_BASE_URL to your API Gateway invoke URL.
