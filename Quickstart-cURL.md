# Getting Started with Rev.AI

## Pre-requisites 

1. Sign up for an account [here](https://www.rev.ai/account/auth/signup)
2. Generate your API keya on your [account settings](https://www.rev.ai/settings) page.
3. [Install cURL (if you use windows)](https://stackoverflow.com/questions/9507353/how-do-i-install-and-use-curl-on-windows) on your machine. cURL is not necessary to use the API just for this quickstart.

## Introduction

Once you've completed this guide you should know how to do the following:

1. Fetch your account details from the API
2. Submit a new job to Rev.AI
3. Fetch a transcription for a completed job.

### Submitting a job

There are two ways of placing a new Job. The first is via uploading a file from the local filesystem. The second is via fetching a media file from a publicly accessible URL. In both cases we are using an MP3 file for the example, however we support many media filetypes. For an exhaustive list of supported file types & codecs please see this [reference](supported_codecs.md). In this quickstart, we'll just cover uploading a local file.

#### Submitting a local file for transcription

If you have a file on your local machine that you would like to be processed you can use cURL with the -f flag. If you don't have one, you can download [this file](https://support.rev.com/hc/en-us/article_attachments/200043975/FTC_Sample_1_-_Single.mp3\) Replace /path/to/media_file.mp3 with the location of the file on your local computer that you would like to send.

1. Open your terminal window and set your variables in the terminal window

   ```sh
    $ API_KEY="your api key goes here" 
    $ MEDIA_FILE_PATH="/path/to/media_file.mp3"
    $ MEDIA_FILE_TYPE="audio/mp3"
    $ METADATA="This is a sample submit jobs option"
   ```

2. Make your request

   ```sh
   $ curl -X POST "https://api.rev.ai/speechtotext/v1/jobs" \
   	-H "Authorization: Bearer $API_KEY" \
   	-H "Content-Type: multipart/form-data" \
       -F "media=@$MEDIA_FILE_PATH;type=$MEDIA_FILE_TYPE" \
       -F "options={\"metadata\":\"$METADATA\"}"
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

It should take no more than a few minutes for the file to finish transcribing. You can check the status of your order on the [recent jobs dashboard](https://rev.ai/jobs) or programatically via [request](https://www.rev.ai/docs#operation/GetJobById) or [webhook](https://www.rev.ai/docs#section/Webhooks).

You will need your job ID and the transcript encoding you would like to receive. The job ID can be found in the response bodies from the previous steps. 

**Transcript Encodings:**

*We support returning the transcript as plain text or as a JSON document. The desired format is specified using the Accept header (text/plain for plaintext and application/vnd.rev.transcript.v1.0+json for JSON). You can read more about the [Rev.ai JSON transcript format here](https://www.rev.ai/docs#operation/GetTranscriptById)*

For our example we will use the json transcript.

```sh
$ REV_JOB_ID = "The id you got in the response from step 3 of submitting the file"
$ curl -X GET \
  "https://api.rev.ai/speechtotext/v1/jobs/$REV_JOB_ID/transcript" \
  -H "accept: application/vnd.rev.transcript.v1.0+json" \
  -H "authorization: Bearer $API_KEY"
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
                }]
        }]
}
```
