# Document Storage and Text Extraction – Amazon S3 and Amazon Textract

![Lab Report](https://img.shields.io/badge/Status-Completed-brightgreen)
![Framework](https://img.shields.io/badge/Framework-AWS%20Best%20Practices-blue)
![Tools](https://img.shields.io/badge/Tools-Amazon%20S3%2C%20Amazon%20Textract%2C%20Boto3%2C%20Python-orange)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)
![Duration](https://img.shields.io/badge/Duration-Hands--On%20Lab-lightblue)

---

## **1. Lab Title**

> Week 9 Lab: Storing Documents and Extracting Data with Amazon S3 and Amazon Textract — Automated Pet Document Processing Pipeline

---

## **2. Purpose and Objectives**

The objective of this lab is to build an **automated document processing pipeline** that uses **Amazon S3** for document storage and **Amazon Textract** for intelligent text extraction. The pipeline uploads a pet immunization record image to S3, extracts all text content using Textract, formats the extracted data as JSON, and stores the results back to S3 for downstream use.

**Key Learning Outcomes:**
- Create an **Amazon S3 bucket** programmatically with region-aware configuration using Boto3
- Upload documents to S3 using the Boto3 `upload_file()` method with organized prefix structure
- Use **Amazon Textract** `detect_document_text()` to extract text from documents stored in S3
- Filter Textract response blocks by type (`LINE`) to extract clean text content
- Format extracted text as **JSON** and upload results back to S3
- Understand Textract's supported file formats and handle `UnsupportedDocumentException` gracefully

---

## **3. Frameworks, Standards, and Methodologies**

- **AWS Well-Architected Framework:** Operational Excellence and Security pillars for document storage and processing
- **Document Processing Pipeline Design:** Input storage → OCR extraction → structured output → result storage
- **S3 Bucket Naming Best Practices:** Globally unique bucket names using account ID and date suffix
- **Region-Aware S3 Configuration:** Conditional `CreateBucketConfiguration` for non-`us-east-1` regions
- **Textract Block Filtering:** Selecting `LINE` block types for clean text output vs. `WORD` or `PAGE` blocks
- **JSON Output Formatting:** Structuring extracted text as machine-readable JSON for downstream consumption

---

## **4. Tools and Technologies**

- **Cloud Provider:** Amazon Web Services (AWS)
- **Storage Service:** Amazon S3
- **OCR/Document AI Service:** Amazon Textract
- **SDK:** AWS SDK for Python — Boto3
- **Programming Language:** Python 3.x
- **AWS Services Used:**
  - Amazon S3 (bucket creation, object upload, JSON result storage)
  - Amazon Textract (`detect_document_text`)
  - AWS IAM (permissions for S3 and Textract access)
- **Input Documents:**
  - `pet-documents/pet-immunization-record.png` (PNG immunization record)
  - `testfile.py` (used to test unsupported document exception)
- **S3 Bucket Structure:**
  - `documents/` prefix — uploaded source documents
  - `extracted_text/` prefix — JSON extraction results
- **Files Created:**
  - `create_pet_documents_bucket.py`
  - `upload_pet_document.py`
  - `extract_pet_document_text.py`

---

## **5. Procedures and Implementation**

<details>
<summary>Click to expand detailed steps</summary>

1. **Task 4: Create the S3 Bucket**
   - Created `create_pet_documents_bucket.py`
   - Retrieved the AWS account ID using `boto3.client("sts").get_caller_identity()`
   - Generated a unique bucket name: `pet-documents-{account_id}-{today}` (using `datetime.date.today()`)
   - Instantiated a Boto3 S3 client
   - Applied region-aware bucket creation logic:
     - For `us-east-1`: omit `CreateBucketConfiguration` (required by AWS)
     - For all other regions: include `CreateBucketConfiguration` with `LocationConstraint`
   - Verified bucket appeared in the S3 console after creation

2. **Task 5: Upload the Pet Document**
   - Created `upload_pet_document.py`
   - Used `s3_client.upload_file()` to upload `pet-immunization-record.png`
   - Set the S3 key as `documents/pet-immunization-record.png` (organized under `documents/` prefix)
   - Confirmed the file appeared in the S3 bucket under the correct prefix

3. **Task 6: Extract Text with Textract**
   - Created `extract_pet_document_text.py`
   - Instantiated a Boto3 `textract` client
   - Called `textract_client.detect_document_text()` with an S3 object reference:
     - `Bucket`: the created bucket name
     - `Name`: `documents/pet-immunization-record.png`
   - Filtered the response `Blocks` array to include only `BlockType == "LINE"`
   - Extracted the `Text` field from each LINE block into a list
   - Formatted the extracted lines as a JSON structure
   - Uploaded the JSON result to the same S3 bucket under `extracted_text/pet-immunization-record.json`

4. **Task 7: Test Unsupported File Type**
   - Uploaded `testfile.py` to the S3 bucket
   - Attempted to run Textract on `testfile.py`
   - Observed `UnsupportedDocumentException` from the Textract API
   - Confirmed that Textract only supports: **PNG, JPEG, TIFF, and PDF** formats
   - Documented the exception as expected behavior for unsupported file types

</details>

---

## **6. Findings and Results**

- **Finding 1:** S3 bucket creation required region-aware logic — `us-east-1` does not accept a `CreateBucketConfiguration` parameter, while all other regions require it. This is a documented AWS quirk that must be handled in production code
- **Finding 2:** The bucket naming convention (`pet-documents-{account_id}-{today}`) successfully produced a globally unique bucket name, satisfying S3's uniqueness requirement across all AWS accounts
- **Finding 3:** `upload_file()` with a `documents/` prefix key successfully organized the uploaded document within the bucket, demonstrating S3's prefix-as-folder pattern
- **Finding 4:** Textract's `detect_document_text()` successfully extracted all text from the pet immunization record PNG, returning a structured list of `Block` objects at `WORD`, `LINE`, and `PAGE` granularity
- **Finding 5:** Filtering for `BlockType == "LINE"` produced clean, readable text lines that matched the visual content of the immunization record, making downstream processing straightforward
- **Finding 6:** The extracted text was successfully serialized to JSON and uploaded to the `extracted_text/` prefix in S3, completing the round-trip document processing pipeline
- **Finding 7:** Attempting Textract on `testfile.py` immediately raised `UnsupportedDocumentException`, confirming Textract enforces strict format requirements and the application must validate file types before submission

---

## **7. Screenshots and Evidence**

<details>
<summary>Click to expand code implementation</summary>

### Task 4: Create S3 Bucket

```python
import boto3
import datetime

sts_client = boto3.client("sts")
account_id = sts_client.get_caller_identity()["Account"]
today = datetime.date.today().strftime("%Y%m%d")
bucket_name = f"pet-documents-{account_id}-{today}"

s3_client = boto3.client("s3", region_name="us-east-1")

# Region-aware bucket creation
region = "us-east-1"
if region == "us-east-1":
    s3_client.create_bucket(Bucket=bucket_name)
else:
    s3_client.create_bucket(
        Bucket=bucket_name,
        CreateBucketConfiguration={"LocationConstraint": region}
    )

print(f"Bucket created: {bucket_name}")
```

### Task 5: Upload Document

```python
s3_client.upload_file(
    Filename="pet-documents/pet-immunization-record.png",
    Bucket=bucket_name,
    Key="documents/pet-immunization-record.png"
)
print("Document uploaded successfully.")
```

### Task 6: Textract Extraction and JSON Upload

```python
import json

textract_client = boto3.client("textract", region_name="us-east-1")

response = textract_client.detect_document_text(
    Document={
        "S3Object": {
            "Bucket": bucket_name,
            "Name": "documents/pet-immunization-record.png"
        }
    }
)

lines = [
    block["Text"]
    for block in response["Blocks"]
    if block["BlockType"] == "LINE"
]

extracted_json = json.dumps({"extracted_lines": lines}, indent=2)

s3_client.put_object(
    Bucket=bucket_name,
    Key="extracted_text/pet-immunization-record.json",
    Body=extracted_json,
    ContentType="application/json"
)
print("Extraction complete. Results saved to S3.")
```

### S3 Bucket Structure

```
pet-documents-{account_id}-{date}/
├── documents/
│   ├── pet-immunization-record.png    ← Source document
│   └── testfile.py                    ← Unsupported format test
└── extracted_text/
    └── pet-immunization-record.json   ← Textract output
```

### Textract Format Support

| File Format | Supported by Textract | Test Result |
|-------------|----------------------|-------------|
| PNG | ✅ Yes | Successfully extracted |
| JPEG | ✅ Yes | Supported |
| TIFF | ✅ Yes | Supported |
| PDF | ✅ Yes | Supported |
| .py (Python) | ❌ No | `UnsupportedDocumentException` |

</details>

---

## **8. Challenges and Solutions**

| Challenge | Solution | Outcome |
|-----------|----------|---------|
| S3 bucket creation failing in `us-east-1` with `CreateBucketConfiguration` | Implemented conditional logic to omit `CreateBucketConfiguration` for `us-east-1` | Bucket created successfully regardless of target region |
| Generating a globally unique S3 bucket name | Combined AWS account ID and today's date in the bucket name | Unique bucket name that avoids naming conflicts across all AWS accounts |
| Filtering Textract output to usable text | Filtered `Blocks` list for `BlockType == "LINE"` entries only | Extracted clean, readable text lines without word-level or page-level noise |
| Handling unsupported file format gracefully | Tested with `testfile.py` and observed `UnsupportedDocumentException` | Documented the expected behavior; production code should validate file extensions before Textract submission |

---

## **9. Key Takeaways and Lessons Learned**

- **Region Quirks Require Defensive Code:** The `us-east-1` S3 bucket creation quirk (no `CreateBucketConfiguration`) is a known AWS inconsistency. Defensive region-checking code is essential for building portable, multi-region applications.

- **S3 Prefixes Enable Organization:** Using `documents/` and `extracted_text/` prefixes to organize S3 objects mirrors folder-based storage and makes bucket contents navigable and manageable at scale.

- **Textract Block Types Serve Different Purposes:** `WORD` blocks provide fine-grained token-level data; `LINE` blocks provide sentence-level readable text; `PAGE` blocks aggregate everything. Choosing the right block type for the use case avoids unnecessary post-processing.

- **Validate Before Submitting to Textract:** The `UnsupportedDocumentException` demonstrates that Textract enforces strict format requirements. Production applications should validate file extensions before invoking Textract to provide better user experience and avoid API errors.

- **Round-Trip Pipelines Close the Loop:** Uploading extraction results back to S3 in JSON format completes a self-contained document processing pipeline — the input, processing, and output all live within the same AWS ecosystem, enabling easy integration with downstream consumers.

---

## **10. Artifacts and Deliverables**

- **`create_pet_documents_bucket.py`:** Script for programmatic S3 bucket creation with region-aware logic
- **`upload_pet_document.py`:** Document upload script with organized S3 prefix structure
- **`extract_pet_document_text.py`:** Textract extraction script with LINE filtering and JSON output
- **S3 Bucket Structure Reference:** Documentation of `documents/` and `extracted_text/` prefix organization
- **Textract Format Support Table:** Reference guide for supported and unsupported document formats

---

## **11. Conclusion**

This lab successfully demonstrated the implementation of an **automated document processing pipeline** using Amazon S3 and Amazon Textract. By programmatically creating an S3 bucket, uploading a pet immunization record, extracting text using Textract's `detect_document_text()` API, and storing the structured JSON results back to S3, a complete round-trip document pipeline was built. Key operational lessons — including region-aware S3 bucket creation, LINE-block filtering for clean text output, and Textract's strict format requirements — were directly validated through hands-on testing. This pattern is applicable to any document-intensive application: healthcare records, legal documents, financial statements, and any scenario requiring automated text extraction from image-based or scanned documents.

---

## **12. References**

- [Amazon S3 Documentation](https://docs.aws.amazon.com/s3/)
- [Amazon Textract Documentation](https://docs.aws.amazon.com/textract/)
- [Textract detect_document_text API](https://docs.aws.amazon.com/textract/latest/dg/API_DetectDocumentText.html)
- [Textract Supported Document Types](https://docs.aws.amazon.com/textract/latest/dg/limits.html)
- [Boto3 S3 Client Reference](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html)
- [Boto3 Textract Client Reference](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/textract.html)
- [S3 Bucket Naming Rules](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html)

---

**Lab Completed:** 2026
**Course:** AWS Developer Fundamentals
**Status:** Completed ✅
