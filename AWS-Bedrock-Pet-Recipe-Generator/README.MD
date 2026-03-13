# Amazon Bedrock Pet Recipe Generator – Demonstration Activity

![Lab Report](https://img.shields.io/badge/Status-Completed-brightgreen)
![Framework](https://img.shields.io/badge/Framework-AWS%20Best%20Practices-blue)
![Tools](https://img.shields.io/badge/Tools-Amazon%20Bedrock%2C%20Boto3%2C%20Python-orange)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-yellow)
![Duration](https://img.shields.io/badge/Duration-Demonstration%20Activity-lightblue)

---

## **1. Lab Title**

> Week 3 Activity: Building a Pet Recipe Generator with Amazon Bedrock — Enabling Foundation Models and Connecting via Boto3

---

## **2. Purpose and Objectives**

The objective of this activity is to gain introductory hands-on experience with **Amazon Bedrock** by enabling a foundation model through the AWS Console and connecting to it programmatically using **Boto3** to generate AI-powered pet food recipes. This demonstration establishes the foundational workflow for all subsequent Bedrock-based development labs.

**Key Learning Outcomes:**
- Navigate the **Amazon Bedrock console** to discover and enable foundation models
- Enable **Amazon Titan Text G1 Express** for use in a development environment
- Establish a **Boto3 client connection** to the Bedrock Runtime service
- Construct and send a **text generation request** to a foundation model via API
- Understand the role of **inference parameters** such as `temperature` in controlling output creativity
- Interpret and display the **model response** in a Python application

---

## **3. Frameworks, Standards, and Methodologies**

- **AWS Well-Architected Framework:** Operational Excellence and reliability for AI/ML service integration
- **Generative AI Best Practices:** Prompt engineering fundamentals and inference parameter tuning
- **AWS SDK for Python (Boto3):** Standard patterns for connecting to and invoking AWS AI services
- **API-First Design:** Treating foundation models as API endpoints for modular application development
- **Responsible AI Usage:** Understanding model capabilities and appropriate use cases for pet care content generation

---

## **4. Tools and Technologies**

- **Cloud Provider:** Amazon Web Services (AWS)
- **AI Service:** Amazon Bedrock
- **Foundation Model:** Amazon Titan Text G1 Express (`amazon.titan-text-express-v1`)
- **SDK:** AWS SDK for Python — Boto3
- **Programming Language:** Python 3.x
- **Key Inference Parameters:**
  - `temperature: 0.7` (balanced creativity and coherence)
  - `maxTokenCount` (controls response length)
- **AWS Services Used:**
  - Amazon Bedrock (model hosting and inference)
  - AWS IAM (permissions for Bedrock API access)

**Technical Stack:**
- Console-based model enablement workflow
- Boto3 `bedrock-runtime` client for API invocation
- JSON request/response payload handling
- Python `print()` for output display

---

## **5. Procedures and Implementation**

<details>
<summary>Click to expand detailed steps</summary>

1. **Navigated to Amazon Bedrock in AWS Console**
   - Opened AWS Management Console and searched for Amazon Bedrock
   - Reviewed the Bedrock service overview and model catalog
   - Identified the Model Access section for enabling foundation models

2. **Enabled Amazon Titan Text G1 Express**
   - Navigated to **Model Access** within the Bedrock console
   - Located **Amazon Titan Text G1 Express** in the available models list
   - Submitted a request to enable the model for the current AWS account
   - Waited for model access status to update to **Access Granted**
   - Confirmed the model ID: `amazon.titan-text-express-v1`

3. **Set Up Python Development Environment**
   - Opened the Python development environment for the activity
   - Verified Boto3 was installed and available for import
   - Confirmed AWS credentials were configured for programmatic access

4. **Created Boto3 Bedrock Runtime Client**
   - Imported `boto3` and `json` libraries
   - Instantiated a `bedrock-runtime` client using `boto3.client()`
   - Specified the correct AWS region where Bedrock access was granted

5. **Constructed the Text Generation Request**
   - Built a prompt string requesting a pet food recipe
   - Structured the request body as a JSON object with:
     - The input prompt text
     - `temperature` set to `0.7` for balanced creativity
     - `maxTokenCount` to define response length
   - Serialized the request body with `json.dumps()`

6. **Invoked the Foundation Model**
   - Called `invoke_model()` on the Bedrock Runtime client
   - Passed `modelId`, `body`, and `contentType` parameters
   - Received the response object from the API

7. **Parsed and Displayed the Response**
   - Read and decoded the response body using `json.loads()`
   - Extracted the generated text from the response structure
   - Printed the pet recipe output to the console

</details>

---

## **6. Findings and Results**

- **Finding 1:** Amazon Bedrock model access is managed at the account level through the console — models are not available by default and must be explicitly enabled before programmatic access is possible
- **Finding 2:** The Boto3 `bedrock-runtime` client successfully connected to Amazon Titan Text G1 Express using standard AWS credential chain configuration, requiring no special authentication beyond IAM permissions
- **Finding 3:** The `temperature` parameter at `0.7` produced creative yet coherent pet food recipes, striking a balance between deterministic and exploratory output — lower values would produce more predictable results
- **Finding 4:** The model responded with a well-structured pet food recipe that included ingredients and preparation steps, demonstrating Titan's ability to follow natural language instructions without fine-tuning
- **Finding 5:** JSON serialization and deserialization of the request/response payloads followed a consistent pattern applicable to all Bedrock foundation models, establishing a reusable code template
- **Finding 6:** End-to-end latency from API invocation to response receipt was under 5 seconds for short recipe prompts, confirming Bedrock's suitability for interactive application development
- **Finding 7:** The `invoke_model()` method returned a streaming body object requiring explicit reading and decoding — a key implementation detail for correct response handling

---

## **7. Screenshots and Evidence**

<details>
<summary>Click to expand code implementation</summary>

### Core Implementation

```python
import boto3
import json

# Create Bedrock Runtime client
client = boto3.client(
    service_name="bedrock-runtime",
    region_name="us-east-1"
)

# Define the prompt
prompt = "Generate a healthy homemade food recipe for a pet rabbit named Charlie."

# Construct request body
request_body = {
    "inputText": prompt,
    "textGenerationConfig": {
        "temperature": 0.7,
        "maxTokenCount": 512
    }
}

# Invoke the model
response = client.invoke_model(
    modelId="amazon.titan-text-express-v1",
    body=json.dumps(request_body),
    contentType="application/json",
    accept="application/json"
)

# Parse and display the response
response_body = json.loads(response["body"].read())
generated_text = response_body["results"][0]["outputText"]
print(generated_text)
```

### Inference Parameter Reference

| Parameter | Value | Effect |
|-----------|-------|--------|
| `temperature` | `0.7` | Balanced creativity — varied but coherent output |
| `maxTokenCount` | `512` | Controls maximum response length |
| `modelId` | `amazon.titan-text-express-v1` | Titan Text G1 Express foundation model |

### Model Access Status

| Step | Status |
|------|--------|
| Navigate to Bedrock Console | ✅ Complete |
| Locate Titan Text G1 Express | ✅ Complete |
| Request Model Access | ✅ Complete |
| Access Status: Granted | ✅ Confirmed |
| Boto3 Client Connected | ✅ Verified |
| Recipe Generated Successfully | ✅ Verified |

</details>

---

## **8. Challenges and Solutions**

| Challenge | Solution | Outcome |
|-----------|----------|---------|
| Model not available immediately after access request | Waited for Bedrock console to reflect "Access Granted" status before invoking via API | Avoided `AccessDeniedException` errors by confirming access first |
| Understanding response body structure | Used `json.loads()` and inspected the response dict to navigate to `results[0]["outputText"]` | Successfully extracted generated text from nested response structure |
| Selecting an appropriate temperature value | Started with `0.7` as a balanced default, understood that lower values increase determinism | Produced usable recipe output on first invocation |
| Ensuring correct region configuration | Specified `region_name` explicitly in the Boto3 client to match where model access was granted | Eliminated region mismatch errors |

---

## **9. Key Takeaways and Lessons Learned**

- **Model Access is a Prerequisites Step:** In Amazon Bedrock, foundation models must be explicitly enabled at the account level before any API call can succeed. This gating step is easy to overlook but critical for a functioning development environment.

- **Boto3 Follows Consistent Patterns:** The `bedrock-runtime` client follows the same instantiation and invocation patterns as other Boto3 clients (S3, Lambda, etc.), making it approachable for developers already familiar with the AWS SDK.

- **Temperature Controls Creativity:** The `temperature` inference parameter is one of the most impactful settings for text generation. Understanding its effect — higher values increase variability, lower values increase consistency — is essential for prompt engineering.

- **JSON is the Universal Language:** Both the request and response payloads are JSON structures. Building comfort with JSON serialization and deserialization in Python is foundational for all API-driven AI development on AWS.

- **Foundation Models are API Endpoints:** This activity reframed AI models as callable services rather than complex local systems, establishing the mental model that underpins all cloud-native AI application development.

---

## **10. Artifacts and Deliverables**

- **Boto3 Bedrock Client Setup:** Reusable code pattern for connecting to Amazon Bedrock Runtime
- **Model Invocation Template:** JSON request/response structure for Titan Text G1 Express
- **Inference Parameter Reference:** Documentation of `temperature` and `maxTokenCount` effects
- **Model Access Workflow:** Step-by-step process for enabling foundation models in Bedrock console
- **Sample Recipe Output:** Example AI-generated pet food recipe from the foundation model

---

## **11. Conclusion**

This demonstration activity successfully introduced the core workflow for working with Amazon Bedrock: enabling a foundation model through the console and invoking it programmatically via Boto3. By enabling **Amazon Titan Text G1 Express** and building a simple pet recipe generator, the end-to-end pattern of prompt construction, API invocation, and response parsing was established. The `temperature` parameter demonstrated how inference settings shape model output, and the consistent Boto3 pattern confirmed that integrating Bedrock into Python applications is straightforward for developers with existing AWS SDK experience. This activity laid the groundwork for all subsequent Bedrock labs, where more advanced prompt engineering, multi-step workflows, and additional AWS services are introduced.

---

## **12. References**

- [Amazon Bedrock Documentation](https://docs.aws.amazon.com/bedrock/)
- [Amazon Titan Text Models](https://docs.aws.amazon.com/bedrock/latest/userguide/titan-text-models.html)
- [Boto3 Bedrock Runtime Client](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/bedrock-runtime.html)
- [Amazon Bedrock Model Access](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html)
- [Inference Parameters for Foundation Models](https://docs.aws.amazon.com/bedrock/latest/userguide/inference-parameters.html)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)

---

**Lab Completed:** 2026
**Course:** AWS Developer Fundamentals
**Status:** Completed ✅
