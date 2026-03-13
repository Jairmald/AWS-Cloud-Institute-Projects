# Serverless Audio Summary Generator – Amazon Polly and AWS Lambda

![Lab Report](https://img.shields.io/badge/Status-Completed-brightgreen)
![Framework](https://img.shields.io/badge/Framework-AWS%20Best%20Practices-blue)
![Tools](https://img.shields.io/badge/Tools-Amazon%20Polly%2C%20AWS%20Lambda%2C%20Amazon%20S3%2C%20Python-orange)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)
![Duration](https://img.shields.io/badge/Duration-Hands--On%20Lab-lightblue)

---

## **1. Lab Title**

> Week 10-11 Lab: Building a Serverless Audio Summary Generator App with Amazon Polly — Event-Driven Text-to-Speech Pipeline using AWS Lambda and Amazon S3

---

## **2. Purpose and Objectives**

The objective of this lab is to build a fully serverless, **event-driven audio generation pipeline** that automatically converts text summaries uploaded to Amazon S3 into MP3 audio files using **Amazon Polly**. The pipeline is triggered automatically by an S3 event whenever a `.txt` file is uploaded, invokes an **AWS Lambda** function to process the text, and delivers the resulting audio back to S3 — requiring zero manual intervention after deployment.

**Key Learning Outcomes:**
- Create and configure an **AWS Lambda function** using the Python 3.13 runtime
- Configure an **S3 event trigger** with prefix and suffix filters for targeted invocation
- Extract source bucket and object key from a **Lambda event payload**
- Use **Amazon Polly** `synthesize_speech()` to convert text to MP3 audio
- Upload Polly's audio stream directly to an S3 destination bucket
- Implement **error handling** with `try/except` for Polly API failures
- Test error scenarios using an invalid `VoiceId` to verify exception capture

---

## **3. Frameworks, Standards, and Methodologies**

- **AWS Well-Architected Framework:** Operational Excellence and Reliability pillars — event-driven serverless architecture
- **Serverless Design Patterns:** Event-driven invocation, stateless function execution, and managed service integration
- **S3 Event Notification Architecture:** Prefix and suffix filtering for precise Lambda trigger targeting
- **IAM Least Privilege:** Lambda execution role scoped to required S3 and Polly permissions only
- **Defensive Programming:** Try/except error handling for external API failures within Lambda
- **Infrastructure-First Deployment:** Creating and verifying all AWS resources before deploying function code

---

## **4. Tools and Technologies**

- **Cloud Provider:** Amazon Web Services (AWS)
- **Compute:** AWS Lambda (Python 3.13 runtime)
- **Text-to-Speech Service:** Amazon Polly
- **Storage Service:** Amazon S3
- **IAM Role:** `LambdaRole` (with S3 read/write and Polly permissions)
- **SDK:** AWS SDK for Python — Boto3 (available natively in Lambda runtime)
- **S3 Trigger Configuration:**
  - Bucket: `appfilesbucket-` (prefix match)
  - Object Prefix: `text/`
  - Object Suffix: `.txt`
- **Polly Configuration:**
  - `OutputFormat: 'mp3'`
  - `VoiceId: 'Joanna'`
- **S3 Output:** `audio/{filename}.mp3`
- **Lambda Function Name:** `AppLambda`

---

## **5. Procedures and Implementation**

<details>
<summary>Click to expand detailed steps</summary>

1. **Task 1: Create the Lambda Function and S3 Trigger**
   - Created a new Lambda function named `AppLambda` from the AWS Console
   - Selected **Python 3.13** as the runtime
   - Assigned the pre-created `LambdaRole` as the execution role
   - Configured an **S3 trigger** on the `appfilesbucket-` bucket:
     - Event type: `PUT` (object created)
     - Prefix filter: `text/`
     - Suffix filter: `.txt`
   - Verified the trigger appeared in the Lambda Designer view
   - Confirmed the `LambdaRole` had appropriate permissions for S3 and Polly

2. **Task 2: Extract Bucket and File Key from Event**
   - Wrote the Lambda handler function `lambda_handler(event, context)`
   - Extracted `source_bucket` from `event["Records"][0]["s3"]["bucket"]["name"]`
   - Extracted `file_key` from `event["Records"][0]["s3"]["object"]["key"]`
   - Downloaded the S3 object using `s3_client.get_object(Bucket=source_bucket, Key=file_key)`
   - Decoded the object body as UTF-8 to produce `summary_text`
   - Printed extracted values for CloudWatch Logs verification

3. **Task 3: Convert Text to Audio with Polly**
   - Instantiated a Boto3 `polly` client inside the Lambda handler
   - Called `synthesize_speech()` with:
     - `Text`: the decoded `summary_text`
     - `OutputFormat`: `'mp3'`
     - `VoiceId`: `'Joanna'`
   - Received an `AudioStream` object in the response
   - Derived the output filename from the original file key (replacing `text/` with `audio/` and `.txt` with `.mp3`)
   - Uploaded the `AudioStream` bytes directly to S3 using `put_object()`

4. **Optional: Add Error Handling**
   - Wrapped the Polly `synthesize_speech()` call in a `try/except` block
   - Caught `polly_client.exceptions.TextLengthExceededException` and generic `Exception`
   - Logged error details to CloudWatch via `print()`
   - Tested error handling by temporarily setting `VoiceId='Frank'` (invalid voice ID)
   - Confirmed the exception was caught and logged without crashing the Lambda function

5. **End-to-End Pipeline Test**
   - Uploaded a `.txt` file containing a groomer summary to the `text/` prefix in S3
   - Observed Lambda invocation triggered automatically by the S3 event
   - Monitored CloudWatch Logs for successful execution messages
   - Verified the `.mp3` output file appeared in the `audio/` prefix of the S3 bucket
   - Confirmed the audio file was playable and matched the uploaded text content

</details>

---

## **6. Findings and Results**

- **Finding 1:** The S3 event trigger with prefix `text/` and suffix `.txt` filters fired exclusively for matching objects, preventing the Lambda from being triggered by audio output files written to the `audio/` prefix — critical for avoiding infinite trigger loops
- **Finding 2:** The Lambda event payload provided the source bucket name and object key in a predictable structure (`event["Records"][0]["s3"]`), enabling reliable extraction without hardcoded values
- **Finding 3:** Amazon Polly's `synthesize_speech()` API returned an `AudioStream` as a streaming body object, which was read directly and uploaded to S3 without requiring local disk storage — fully stateless processing
- **Finding 4:** The `Joanna` voice produced natural-sounding, intelligible audio from the groomer summary text, confirming Polly's text-to-speech quality for short-form content
- **Finding 5:** Using `VoiceId='Frank'` (an invalid voice ID) raised a `ValidationException` which was successfully caught by the `try/except` block and logged to CloudWatch — the Lambda function returned gracefully without an unhandled exception
- **Finding 6:** The `audio/{filename}.mp3` output naming convention maintained a clear correspondence between source `.txt` files and their generated `.mp3` counterparts, making the S3 bucket navigable
- **Finding 7:** The complete serverless pipeline — S3 upload → S3 event → Lambda invocation → Polly synthesis → S3 audio output — executed end-to-end without any manual intervention after the initial text file upload

---

## **7. Screenshots and Evidence**

<details>
<summary>Click to expand code implementation</summary>

### Lambda Handler Function

```python
import boto3
import json

s3_client = boto3.client("s3")
polly_client = boto3.client("polly")

def lambda_handler(event, context):
    # Extract source bucket and file key from S3 event
    source_bucket = event["Records"][0]["s3"]["bucket"]["name"]
    file_key = event["Records"][0]["s3"]["object"]["key"]

    print(f"Processing: s3://{source_bucket}/{file_key}")

    # Download and decode the text file
    response = s3_client.get_object(Bucket=source_bucket, Key=file_key)
    summary_text = response["Body"].read().decode("utf-8")

    # Derive output file path
    filename = file_key.replace("text/", "").replace(".txt", "")
    output_key = f"audio/{filename}.mp3"

    try:
        # Convert text to speech using Amazon Polly
        polly_response = polly_client.synthesize_speech(
            Text=summary_text,
            OutputFormat="mp3",
            VoiceId="Joanna"
        )

        # Upload audio stream directly to S3
        s3_client.put_object(
            Bucket=source_bucket,
            Key=output_key,
            Body=polly_response["AudioStream"].read(),
            ContentType="audio/mpeg"
        )

        print(f"Audio saved to: s3://{source_bucket}/{output_key}")

    except polly_client.exceptions.TextLengthExceededException as e:
        print(f"Text too long for Polly: {e}")
    except Exception as e:
        print(f"Error during synthesis: {e}")

    return {"statusCode": 200, "body": "Audio generation complete."}
```

### Pipeline Architecture

```
Groomer uploads .txt file
         │
         ▼
  S3: appfilesbucket-/text/summary.txt
         │
         │ S3 PUT Event (prefix: text/, suffix: .txt)
         ▼
  AWS Lambda: AppLambda (Python 3.13)
         │
         ├── Get object from S3
         ├── Decode UTF-8 text
         ├── Call Polly synthesize_speech()
         └── Upload AudioStream to S3
         │
         ▼
  S3: appfilesbucket-/audio/summary.mp3
```

### S3 Trigger Configuration

| Setting | Value |
|---------|-------|
| Trigger Type | S3 Event Notification |
| Bucket | `appfilesbucket-` |
| Event Type | `s3:ObjectCreated:Put` |
| Prefix Filter | `text/` |
| Suffix Filter | `.txt` |
| Target | `AppLambda` |

### Error Handling Test Results

| Test | VoiceId | Expected Behavior | Result |
|------|---------|------------------|--------|
| Valid synthesis | `Joanna` | MP3 generated and saved to S3 | ✅ Passed |
| Invalid VoiceId | `Frank` | Exception caught, logged to CloudWatch | ✅ Caught gracefully |

</details>

---

## **8. Challenges and Solutions**

| Challenge | Solution | Outcome |
|-----------|----------|---------|
| Preventing infinite S3 trigger loop (Lambda writes to same bucket) | Configured trigger with `text/` prefix and `.txt` suffix — audio files written to `audio/` prefix do not match the trigger | Pipeline runs without recursive invocation |
| Extracting nested S3 event data | Navigated `event["Records"][0]["s3"]["bucket"]["name"]` and `["object"]["key"]` structure from the Lambda event payload | Correct bucket and key extracted on every invocation |
| Handling Polly AudioStream as a streaming object | Called `.read()` on the `AudioStream` before passing to `put_object()` | Raw bytes successfully uploaded to S3 as MP3 |
| Testing error handling without breaking the pipeline | Used an invalid `VoiceId='Frank'` to deliberately trigger an exception and verify `try/except` behavior | Exception caught and logged; Lambda returned successfully |

---

## **9. Key Takeaways and Lessons Learned**

- **Event-Driven Architecture Eliminates Manual Steps:** Once deployed, this pipeline requires zero human intervention — text file uploads automatically produce audio outputs. This automation pattern scales to thousands of files without infrastructure changes.

- **Trigger Filters Prevent Infinite Loops:** S3 event triggers on a bucket that Lambda also writes to can cause recursive invocation. Prefix and suffix filters are essential safeguards that restrict trigger scope to only the intended input objects.

- **Polly AudioStream Requires Explicit Reading:** Polly's `synthesize_speech()` returns a streaming body, not raw bytes. Calling `.read()` is required before the data can be uploaded or saved — a subtle but critical implementation detail.

- **Lambda Roles Must Be Scoped Precisely:** The `LambdaRole` needed explicit permissions for S3 `GetObject`, S3 `PutObject`, and Polly `SynthesizeSpeech`. Missing any one permission produces an `AccessDeniedException` that can be difficult to diagnose without CloudWatch Logs.

- **Error Handling in Lambda is Critical:** Lambda functions run in managed environments where unhandled exceptions cause function failures recorded in CloudWatch. Wrapping external API calls in `try/except` ensures failures are logged gracefully and the function returns a meaningful response.

---

## **10. Artifacts and Deliverables**

- **`AppLambda` Function Code:** Complete Lambda handler with S3 event parsing, Polly synthesis, and error handling
- **S3 Trigger Configuration:** Event notification settings with prefix/suffix filtering
- **Pipeline Architecture Diagram:** End-to-end flow from text upload to audio output
- **Error Handling Reference:** Try/except patterns for Polly API failures with CloudWatch logging
- **MP3 Audio Output:** Generated audio files stored in `audio/` prefix of the S3 bucket

---

## **11. Conclusion**

This lab successfully demonstrated the design and deployment of a fully **serverless, event-driven text-to-speech pipeline** using AWS Lambda, Amazon S3, and Amazon Polly. By configuring precise S3 event triggers, writing a stateless Lambda handler that processes text and synthesizes audio, and storing results back to S3, a production-ready audio generation workflow was built without managing any servers. The implementation of `try/except` error handling and deliberate invalid-VoiceId testing validated the pipeline's resilience. This architecture directly models real-world use cases including accessibility features (audio versions of documents), automated podcast generation, and multilingual audio content delivery systems — all achievable without infrastructure provisioning.

---

## **12. References**

- [Amazon Polly Documentation](https://docs.aws.amazon.com/polly/)
- [Amazon Polly synthesize_speech API](https://docs.aws.amazon.com/polly/latest/dg/API_SynthesizeSpeech.html)
- [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)
- [S3 Event Notifications](https://docs.aws.amazon.com/AmazonS3/latest/userguide/NotificationHowTo.html)
- [Boto3 Polly Client Reference](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/polly.html)
- [AWS Serverless Application Model](https://aws.amazon.com/serverless/sam/)
- [AWS Lambda Best Practices](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html)

---

**Lab Completed:** 2026
**Course:** AWS Developer Fundamentals
**Status:** Completed ✅
