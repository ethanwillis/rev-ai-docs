# Getting Started with Rev.AI

## Pre-requisites 

1. Signup for an account [here](https://www.rev.ai/account/auth/signup)
2. Generate API keys on your [account settings](https://www.rev.ai/settings) page.
3. Install [curl (if you have windows)](https://stackoverflow.com/questions/9507353/how-do-i-install-and-use-curl-on-windows) on your machine.



## Introduction

The RevSpeech API is an advanced speech recognition platform developed by the makers of Temi and Rev.com. This Quickstart guide will quickly get you up to speed so that you can begin submitting orders and receiving transcriptions! 

Once you've completed this guide you should know how to do the following:

1. Fetch your account details from the API
2. Submit a new job to Rev.AI
3. Check the status of a submitted job for your account
4. Fetch a transcription for a completed job.

### Brief Overview of Rev.AI API usage Lifecycle

The typical flow of API usage for Rev.AI consists of the following steps:

1. Place a new order by passing us a Job object.
2. Poll the Job status until it's complete or until you receive a callback notification from us that it is complete.
3. Once the Job is complete fetch your completed transcript. 



## Getting Started with cURL

### Testing that your API credentials work with a simple request!

Before we launch into placing orders and fetching transcripts from the API we will first try pulling back our Account information from the Rev.AI API. 

1. Set your variables on the CLI

   ```sh
   $ api_key="your api key goes here" 
   ```

2. Run the following cURL command, $api_key will be automatically replaced with the value set in step 1 via bash's variable expansion.

   ```sh
   $ curl -X GET "https://api.rev.ai/revspeech/v1beta/account" \
   -H "Authorization: Bearer $api_key"
   ```

3. Once you run this command you should get back a response similar to the following

   ```json
   {
       "email": "you@mailhost.com",
       "balance_seconds":17880
   }
   ```



### Placing a new Job order

In the following sections we describe two ways of placing a new Job order. The first is via fetching a media file from a publicly accessible URL. The second is via uploading a file from the local filesystem. In both cases we are using an MP3 file for the example, however we support many media filetypes. For an exhaustive list of supported file types & codecs please see this [reference.](supported_codecs.md) 

#### Placing a new job order - Media from a URL

Placing a new order using a URL to an existing piece of media just requires specification of an options object 

1. Set your variables on the CLI that haven't been set already.

   ```sh
   $ api_key="02H33m_P86---qBGsXE-mvt29WKWx1BUYYq17PNkpG7kYjfzs5BpGa45Kf4peNF9xio5Tb50f7HZbdQnezGBnDzsCfAZk" 
   $ media_url="https://support.rev.com/hc/en-us/article_attachments/200043975/FTC_Sample_1_-_Single.mp3"
   $ metadata="This is a sample submit jobs option"
   ```

2. Use the following cURL command that sends a POST request to the /jobs endpoint, creating a new order.

   ```sh
   $ curl -X POST "https://api.rev.ai/revspeech/v1beta/jobs" \
   	-H "Authorization: Bearer $api_key" \
   	-H "Content-Type: application/json" \
   	-d "{\"media_url\":\"$media_url\",\"metadata\":\"$metadata\"}"
   
   ```

3. After sending this request to the API you will get a response body with the following format:

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

1. Set your variables on the CLI that haven't been set already

   ```sh
   $ media_file_path="/path/to/media_file.mp3"
   $ meda_file_type="audio/mp3"
   $ metadata="This is a sample submit jobs option"
   ```

2. Make your request

   ```sh
   $ curl -X POST "https://api.rev.ai/revspeech/v1beta/jobs" \
   	-H "Authorization: Bearer $api_key" \
   	-H "Content-Type: multipart/form-data" \
       -F "media=@$media_file_path;type=$media_file_type" \
       -F "options={\"metadata\":\"$metadata\"}"
   ```

3. After sending this request to the API you will get a response body with the following format:

   ```json
   {
     "id": "111111",
     "status": "in_progress",
     "created_on": "2018-05-05T23:23:22.29Z",
     "metadata": "This is a sample submit jobs option"
   }
   ```

###  

### Checking your job status

Once you have placed an order and have your Job ID you will need to wait until the order is completed before taking any further action. There are two ways to check the Job status: Automatic notifications via webhooks and through long-polling of the Job you submitted. 

For this guide we'll be focusing on the latter. *For more information on how to receive notifications you can read more about utilizing webhooks [here](https://docs.rev.ai/en/latest/models/webhooks.html)*

Taking your "id" from the response after placing your order you will fire off a GET request to the /jobs/{id} endpoint:

1. First set a variable in your shell to correspond to the "id" you received from the response of the previous request

   ```sh
   $ rev_job_id="111111"
   ```

2.  Send your request

   ```sh
   $ curl -X GET "https://api.rev.ai/revspeech/v1beta/jobs/$rev_job_id" \
   	-H "Authorization: Bearer $api_key"
   ```

3.  After sending this request you will get the following response body:

   *note: If you submitted a local file the response body will vary in that it doesn't have a media_url field.*

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


### Getting your transcripts

When a job object has a returned status of "transcribed" it is now ready for you to fetch the completed transcript.

You will need your order ID and the transcript encoding you would like to receive. The order ID can be found in the response bodies from the previous steps. 

**Transcript Encodings:**

*We currently support two transcript encodings: "text/plain" or "application/vnd.rev.transcript.v1.0+json" for a Rev.ai Transcript.*

*You can read more about the [Rev.ai Transcript here](https://docs.rev.ai/en/latest/models/transcript.html#transcript-model)* *and you can see an example of the text/plain version of transcripts [here](https://docs.rev.ai/en/latest/endpoints/transcript.html) under Responses*

For our example we will use the Rev.ai Transcript. Your chosen transcript type will be passed in as an Accept header on your request as shown below.

```sh
$ curl -X GET \
  "https://api.rev.ai/revspeech/v1beta/jobs/$rev_job_id/transcript" \
  -H "accept: application/vnd.rev.transcript.v1.0+json" \
  -H "authorization: Bearer $api_key"
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
