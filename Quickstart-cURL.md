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
3. Fetch a transcription for a completed job.

### Placing a new Job order

In the following sections we describe two ways of placing a new Job order. The first is via fetching a media file from a publicly accessible URL. The second is via uploading a file from the local filesystem. In both cases we are using an MP3 file for the example, however we support many media filetypes. For an exhaustive list of supported file types & codecs please see this [reference.](supported_codecs.md). In this quickstart, we'll just cover uploading a local file.

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

### Getting your transcripts

It should take no more than a few minutes for the file to finish transcribing.

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

Once you send the above request you will receive the following response. This response has been truncated to only show the general form of the response you will receive.

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
