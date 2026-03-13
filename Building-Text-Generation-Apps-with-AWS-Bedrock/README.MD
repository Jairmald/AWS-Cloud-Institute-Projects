# Building Text Generation Apps with Amazon Bedrock

![Lab Report](https://img.shields.io/badge/Status-Completed-brightgreen)
![Framework](https://img.shields.io/badge/Framework-AWS%20Best%20Practices-blue)
![Tools](https://img.shields.io/badge/Tools-Amazon%20Bedrock%2C%20Boto3%2C%20Amazon%20Q-orange)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)
![Duration](https://img.shields.io/badge/Duration-Hands--On%20Lab-lightblue)

---

## **1. Lab Title**

> Week 3 Lab: Building Text Generation Applications with Amazon Bedrock — Pet Recipe Generator and Pet Description Generator using Amazon Nova Lite

---

## **2. Purpose and Objectives**

The objective of this lab is to build production-pattern text generation applications using **Amazon Bedrock** and the **Amazon Nova Lite** foundation model. Through four structured tasks, this lab progresses from basic API connectivity to building two complete Python applications: a dynamic pet recipe generator and an automated pet description generator with variable inference parameters.

**Key Learning Outcomes:**
- Set up and configure the **AWS Code Editor IDE** and sign in to **Amazon Q** for AI-assisted development
- Establish a **Boto3 connection** to Amazon Bedrock Runtime and send raw API requests
- Experiment with **temperature inference parameters** (`0.1`, `0.5`, `0.9`) and observe their effect on output
- Build a **pet recipe generator** with user input validation, dynamic prompts, and safe inference parameters
- Build a **pet description generator** that processes a list of pet dictionaries with species-based temperature tuning
- Understand and resolve **cross-region inference profile errors** when invoking Nova models

---

## **3. Frameworks, Standards, and Methodologies**

- **AWS Well-Architected Framework:** Operational Excellence pillar for AI application development patterns
- **Generative AI Best Practices:** Prompt engineering, inference parameter tuning, and output consistency strategies
- **Input Validation Design:** Enforcing allowed values for user-supplied inputs before API invocation
- **Dynamic Prompt Construction:** Building context-aware prompts from structured data inputs
- **Species-Based Temperature Strategy:** Adapting model creativity based on content type (cats vs. dogs)
- **AWS SDK for Python (Boto3):** Standard patterns for Bedrock Runtime service integration

---

## **4. Tools and Technologies**

- **Cloud Provider:** Amazon Web Services (AWS)
- **IDE:** AWS Code Editor (cloud-based development environment)
- **AI Assistant:** Amazon Q (in-IDE AI coding assistant)
- **AI Service:** Amazon Bedrock Runtime
- **Foundation Model:** Amazon Nova Lite (`amazon.nova-lite-v1:0`)
- **SDK:** AWS SDK for Python — Boto3
- **Programming Language:** Python 3.x
- **Key Inference Parameters:**
  - `temperature` (tested at `0.1`, `0.5`, `0.7`, `0.9`)
  - `max_tokens: 512`
  - `topP: 0.9`
- **Files Created:**
  - `bedrock_connect.py` (Tasks 2)
  - `pet_recipe_generator.py` (Task 3)
  - `pet_description_generator.py` (Task 4)

---

## **5. Procedures and Implementation**

<details>
<summary>Click to expand detailed steps</summary>

1. **Task 1: Set Up AWS Code Editor and Amazon Q**
   - Opened AWS Code Editor IDE from the lab environment URL
   - Signed in to Amazon Q for AI-assisted code suggestions within the IDE
   - Verified the development environment was ready for Python development
   - Confirmed AWS credentials were available for Boto3 service calls

2. **Task 2: Connect to Bedrock and Send Requests**
   - Created `bedrock_connect.py` to establish baseline Bedrock connectivity
   - Instantiated a `bedrock-runtime` Boto3 client targeting `us-east-1`
   - Built a basic text generation request payload with a test prompt
   - Used `amazon.nova-lite-v1:0` as the model ID (avoiding cross-region prefix)
   - Sent the request and successfully parsed the response
   - Experimented with temperature values `0.1`, `0.5`, and `0.9`:
     - `0.1`: Highly deterministic, repetitive output across multiple runs
     - `0.5`: Balanced, moderately varied output
     - `0.9`: Creative, highly variable output with more diverse language

3. **Task 3: Build the Pet Recipe Generator**
   - Created `pet_recipe_generator.py` with structured user input collection
   - Collected three inputs from the user:
     - Pet type (validated: must be `dog` or `cat`)
     - Available ingredients (free text)
     - Known allergies (free text)
   - Implemented input validation loop to reject invalid pet types
   - Constructed a dynamic prompt incorporating all three user inputs
   - Invoked Bedrock with `max_tokens=512`, `temperature=0.7`, `topP=0.9`
   - Displayed the generated recipe to the user

4. **Task 4: Build the Pet Description Generator**
   - Created `pet_description_generator.py` with a hardcoded list of pet dictionaries
   - Defined three pets: `Buddy` (dog), `Mittens` (cat), `Lucky` (dog)
   - Each dictionary contained: `name`, `breed`, `age`, `species`
   - Looped through the list and generated a description for each pet
   - Applied **species-based temperature logic**:
     - Cats (`species == "cat"`): `temperature = 0.9` (more creative, whimsical descriptions)
     - Dogs (`species == "dog"`): `temperature = 0.3` (more consistent, reliable descriptions)
   - Stored each generated description back into its respective dictionary
   - Displayed all descriptions after loop completion

</details>

---

## **6. Findings and Results**

- **Finding 1:** The AWS Code Editor provided a fully functional cloud IDE with Amazon Q integration, enabling AI-assisted code completion without requiring local development environment setup
- **Finding 2:** The correct model ID for Nova Lite in `us-east-1` is `amazon.nova-lite-v1:0` — using the cross-region prefix `us.amazon.nova-lite-v1:0` causes authentication errors because cross-region inference is not supported in this context
- **Finding 3:** Temperature experimentation confirmed the parameter's direct impact: `temperature=0.1` produced near-identical outputs across multiple runs, while `temperature=0.9` produced significantly varied language and structure
- **Finding 4:** Input validation for pet type successfully blocked invalid entries and re-prompted the user, demonstrating that defensive input handling is essential before constructing API payloads
- **Finding 5:** Dynamic prompt construction — embedding user-supplied ingredients and allergy information — produced contextually relevant recipes tailored to each user's specific inputs
- **Finding 6:** Species-based temperature assignment produced noticeably different description styles: cat descriptions were more imaginative and playful (`0.9`), while dog descriptions were more grounded and factual (`0.3`)
- **Finding 7:** Storing generated descriptions back into the pet dictionaries demonstrated a practical pattern for enriching structured data with AI-generated content

---

## **7. Screenshots and Evidence**

<details>
<summary>Click to expand code implementation</summary>

### Task 3: Pet Recipe Generator

```python
import boto3
import json

client = boto3.client("bedrock-runtime", region_name="us-east-1")

# Collect and validate user input
while True:
    pet_type = input("Enter pet type (dog or cat): ").strip().lower()
    if pet_type in ["dog", "cat"]:
        break
    print("Invalid pet type. Please enter 'dog' or 'cat'.")

ingredients = input("Enter available ingredients (comma-separated): ").strip()
allergies = input("Enter known allergies (or 'none'): ").strip()

# Build dynamic prompt
prompt = (
    f"Create a healthy homemade food recipe for a {pet_type}. "
    f"Use these ingredients: {ingredients}. "
    f"Avoid the following allergens: {allergies}."
)

# Invoke Bedrock
request_body = {
    "messages": [{"role": "user", "content": prompt}],
    "inferenceConfig": {
        "max_new_tokens": 512,
        "temperature": 0.7,
        "topP": 0.9
    }
}

response = client.invoke_model(
    modelId="amazon.nova-lite-v1:0",
    body=json.dumps(request_body),
    contentType="application/json",
    accept="application/json"
)

result = json.loads(response["body"].read())
print(result["output"]["message"]["content"][0]["text"])
```

### Task 4: Pet Description Generator

```python
import boto3
import json

client = boto3.client("bedrock-runtime", region_name="us-east-1")

pets = [
    {"name": "Buddy", "breed": "Golden Retriever", "age": 3, "species": "dog"},
    {"name": "Mittens", "breed": "Siamese", "age": 5, "species": "cat"},
    {"name": "Lucky", "breed": "Beagle", "age": 2, "species": "dog"}
]

for pet in pets:
    temperature = 0.9 if pet["species"] == "cat" else 0.3
    prompt = f"Write an adoption description for a {pet['age']}-year-old {pet['breed']} named {pet['name']}."

    request_body = {
        "messages": [{"role": "user", "content": prompt}],
        "inferenceConfig": {
            "max_new_tokens": 512,
            "temperature": temperature
        }
    }

    response = client.invoke_model(
        modelId="amazon.nova-lite-v1:0",
        body=json.dumps(request_body),
        contentType="application/json",
        accept="application/json"
    )

    result = json.loads(response["body"].read())
    pet["description"] = result["output"]["message"]["content"][0]["text"]
    print(f"\n{pet['name']}:\n{pet['description']}")
```

### Temperature Behavior Reference

| Temperature | Behavior | Use Case |
|-------------|----------|----------|
| `0.1` | Highly deterministic, minimal variation | Facts, structured data |
| `0.5` | Balanced creativity and consistency | General purpose |
| `0.7` | Moderately creative, good for recipes | User-facing content |
| `0.9` | Highly creative, varied language | Marketing, storytelling |

</details>

---

## **8. Challenges and Solutions**

| Challenge | Solution | Outcome |
|-----------|----------|---------|
| Cross-region model ID error when using `us.amazon.nova-lite-v1:0` | Switched to the standard model ID `amazon.nova-lite-v1:0` for direct invocation in `us-east-1` | Authentication error resolved; model invoked successfully |
| Input validation allowing infinite invalid entries | Implemented a `while True` loop with break condition on valid input | Users could not proceed past validation with invalid pet types |
| Understanding response body structure for Nova vs. Titan | Inspected the response JSON to identify `output.message.content[0].text` path for Nova models | Correctly extracted generated text from Nova's response format |
| Species-based temperature requiring conditional logic in loop | Used a ternary-style assignment: `temperature = 0.9 if species == "cat" else 0.3` | Clean, readable per-species temperature assignment within the loop |

---

## **9. Key Takeaways and Lessons Learned**

- **Model ID Precision Matters:** Using the wrong model ID prefix (`us.` vs no prefix) causes silent authentication failures in specific regions. Always verify the exact model ID for the target region in the Bedrock documentation.

- **Temperature as a Design Decision:** Choosing a temperature value is a product decision, not just a technical one. Matching temperature to content type (creative vs. factual) produces noticeably better user experiences.

- **Input Validation Before API Calls:** Validating user input before constructing prompts prevents malformed API requests and improves application reliability. The while-loop pattern is a clean, Pythonic approach to enforcing constraints.

- **Dynamic Prompts Scale Naturally:** Embedding variables into prompt strings scales from simple pet types to complex multi-field inputs without architectural changes — the same pattern works for any domain.

- **AI-Enriched Data Structures:** Storing AI-generated content back into dictionaries (the description pattern) enables downstream use of generated content in databases, APIs, or user interfaces — a real-world production pattern.

---

## **10. Artifacts and Deliverables**

- **`bedrock_connect.py`:** Baseline Bedrock connectivity script with temperature experimentation
- **`pet_recipe_generator.py`:** Complete recipe generator with input validation and dynamic prompts
- **`pet_description_generator.py`:** Batch description generator with species-based temperature tuning
- **Temperature Behavior Reference:** Documentation of temperature effects across the 0.1–0.9 range
- **Model ID Reference:** Correct Nova Lite model ID for `us-east-1` direct invocation

---

## **11. Conclusion**

This lab successfully progressed from basic Bedrock connectivity to building two complete text generation applications using Amazon Nova Lite. The pet recipe generator demonstrated user input collection, validation, dynamic prompt construction, and safe inference parameter selection. The pet description generator extended these skills with batch processing, conditional temperature assignment, and AI-enriched data structures. A critical operational lesson — using the correct model ID without cross-region prefix — was learned through direct troubleshooting. Together, these two applications established the complete pattern for Bedrock-powered Python applications: collect inputs, build prompts, invoke the model, parse the response, and deliver results to users or downstream systems.

---

## **12. References**

- [Amazon Bedrock Documentation](https://docs.aws.amazon.com/bedrock/)
- [Amazon Nova Foundation Models](https://docs.aws.amazon.com/bedrock/latest/userguide/amazon-nova.html)
- [Boto3 Bedrock Runtime Reference](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/bedrock-runtime.html)
- [Inference Parameters – Amazon Bedrock](https://docs.aws.amazon.com/bedrock/latest/userguide/inference-parameters.html)
- [Amazon Q Developer Documentation](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/what-is.html)
- [AWS Code Editor](https://docs.aws.amazon.com/cloud9/latest/user-guide/welcome.html)

---

**Lab Completed:** 2026
**Course:** AWS Developer Fundamentals
**Status:** Completed ✅
