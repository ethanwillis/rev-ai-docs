# Getting Started with Rev.AI

## Pre-requisites 

1. Signup for an account [here](https://www.rev.ai/account/auth/signup)
2. Generate API keys on your [account settings](https://www.rev.ai/settings) page.
3. Install curl, postman, python, or node.js on your machine.



## Introduction

The RevSpeech API is an advanced speech recognition platform developed by the makers of Temi and Rev.com. This Quickstart guide will quickly get you up to speed so that you can begin submitting orders and receiving transcriptions! 

Once you've completed this guide you should know how to do the following:

1. Submit a new job to Rev.AI
2. Get a list of your pending and completed jobs for your account
3. Fetch a transcription for a completed job.

Currently this guide is broken up into 4 different versions so that you can follow along in whichever tooling you're most comfortable with:

1. [cURL](#getting-started-with-curl)
2. [Postman](#getting-started-with-postman)
3. [Our Python SDK](#getting-started-with-python-sdk)
4. [Node.js ](#getting-started-with-node.js)

### Brief Overview of Rev.AI API usage Lifecycle

The typical flow of API usage for Rev.AI consists of the following steps:

1. Place a new order by passing us a Job object.
2. Poll the Job status until it's complete or until you receive a callback notification from us that it is complete.
3. Once the Job is complete fetch your completed transcript. 



## Getting Started with cURL

### Placing a new Job order

#### Placing a new job order - Media from a URL

Placing a new order using a URL to an existing piece of media just requires specification of an options object 

```shell
$ curl -X POST "https://api.rev.ai/revspeech/v1beta/jobs" \
	-H "Authorization: Bearer <api_key>" \
	-H "Content-Type: application/json" \
	-d "{\"media_url\":\"https://support.rev.com/hc/en-us/article_attachments/200043975/FTC_Sample_1_-_Single.mp3\",\"metadata\":\"This is a sample submit jobs option\"}"

```

After sending this request to the API you will get a response body with the following format:

```json
{
    "id": "111111",
    "created_on": "2019-01-25T20:40:03.49Z",
    "metadata": "This is a sample submit jobs option",
    "media_url": "https://support.rev.com/hc/en-us/article_attachments/200043975/FTC_Sample_1_-_Single.mp3",
    "status": "in_progress"
}
```



#### Placing a new job order - Media from a local file

If instead you have a file on your local machine that you would like to be processed you can use cURL with the -f flag. Replace /path/to/media_file.mp3 with the location of the file on your local computer that you would like to send.

```shell
$ curl -X POST "https://api.rev.ai/revspeech/v1beta/jobs" \
	-H "Authorization: Bearer <api_key>" \
	-H "Content-Type: multipart/form-data" \
    -F "media=@/path/to/media_file.mp3;type=audio/mp3" \
    -F "options={\"metadata\":\"This is a sample submit jobs option for multipart\"}"
```

After sending this request to the API you will get a response body with the following format:

```json
{
  "id": "111111",
  "status": "in_progress",
  "created_on": "2018-05-05T23:23:22.29Z",
  "metadata": "This is a sample submit jobs option"
}
```

#### Checking your job status

Once you have placed an order and have your Job ID you will need to wait until the order is completed before taking any further action. There are two ways to check the Job status: Automatic notifications via webhooks and through long-polling of the Job you submitted. 

For this guide we'll be focusing on the latter. *For more information on how to receive notifications you can read more about utilizing webhooks [here](https://docs.rev.ai/en/latest/models/webhooks.html)*

Taking your "id" from the response after placing your order you will fire off a GET request to the /jobs/{id} endpoint:

```shell
$ curl -X GET "https://api.rev.ai/revspeech/v1beta/jobs/{id}" \
	-H "Authorization: Bearer <api_key>"
```

After sending this request you will get the following response body:

```json
{
    "id": "111111",
    "created_on": "2019-01-25T20:40:03.49Z",
    "completed_on": "2019-01-25T20:41:55.412Z",
    "name": "FTC Sample 1 - Single.mp3",
    "metadata": "This is a sample submit jobs option",
    "media_url": "https://support.rev.com/hc/en-us/article_attachments/200043975/FTC_Sample_1_-_Single.mp3",
    "status": "transcribed",
    "duration_seconds": 107
}
```

#### Getting your transcripts

When a job object has a returned status of "transcribed" it is now ready for you to fetch the completed transcript.

You will need your order ID and the transcript encoding you would like to receive. The order ID can be found in the response bodies from the previous steps. 

**Transcript Encodings:**

*We currently support two transcript encodings: "text/plain" or "application/vnd.rev.transcript.v1.0+json" for a Rev.ai Transcript.*

*You can read more about the [Rev.ai Transcript here](https://docs.rev.ai/en/latest/models/transcript.html#transcript-model)* *and you can see an example of the text/plain version of transcripts [here](https://docs.rev.ai/en/latest/endpoints/transcript.html) under Responses*

For our example we will use the Rev.ai Transcript. Your chosen transcript type will be passed in as an Accept header on your request as shown below.

```sh
curl -X GET \
  https://api.rev.ai/revspeech/v1beta/jobs/111111/transcript \
  -H 'accept: application/vnd.rev.transcript.v1.0+json' \
  -H 'authorization: Bearer <api_key>'
```

Once you send the above request you will receive the following response. This response has been truncated to only show the general form of the response you will receive. s

```json
{
    "monologues": [
        {
            "speaker": 0,
            "elements": [
                {
                    "type": "text",
                    "value": "Hi",
                    "ts": 0.26999999999999996,
                    "end_ts": 0.48,
                    "confidence": 1
                },
                {
                    "type": "punct",
                    "value": ", "
                },
                {
                    "type": "text",
                    "value": "my",
                    "ts": 0.51,
                    "end_ts": 0.65999999999999992,
                    "confidence": 1
                },
                {
                    "type": "punct",
                    "value": " "
                },
...
```



