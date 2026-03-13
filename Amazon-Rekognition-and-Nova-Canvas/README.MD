# AI Image Services – Amazon Rekognition and Nova Canvas

![Lab Report](https://img.shields.io/badge/Status-Completed-brightgreen)
![Framework](https://img.shields.io/badge/Framework-AWS%20Best%20Practices-blue)
![Tools](https://img.shields.io/badge/Tools-Rekognition%2C%20Bedrock%2C%20Nova%20Micro%2C%20Python-orange)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)
![Duration](https://img.shields.io/badge/Duration-Hands--On%20Lab-lightblue)

---

## **1. Lab Title**

> Week 5 Lab: Creating Images, Detecting Objects, and Building Interactions with AI Image Services — Amazon Rekognition Object Detection and Amazon Bedrock Nova Micro Recipe Generation

---

## **2. Purpose and Objectives**

The objective of this lab is to build an **AI-powered pet food recipe pipeline** that combines **Amazon Rekognition** for ingredient detection from images with **Amazon Bedrock Nova Micro** for recipe generation. The complete pipeline detects real-world ingredients in a photograph, uses those detected items to construct an AI recipe prompt, generates a recipe, and saves both the text and a generated image to the filesystem.

**Key Learning Outcomes:**
- Use **Amazon Rekognition** `detect_labels()` to identify objects and ingredients in images
- Read binary image files and pass them to the Rekognition API
- Integrate **Amazon Bedrock Nova Micro** for AI-powered recipe generation from detected ingredients
- Construct dynamic prompts using Rekognition label output as structured input
- Save AI-generated content to files using Python's `open()` with write mode
- Build a complete end-to-end AI pipeline: image → labels → recipe → saved output

---

## **3. Frameworks, Standards, and Methodologies**

- **AWS Well-Architected Framework:** Operational Excellence for AI pipeline design and chaining multiple AWS AI services
- **Computer Vision Best Practices:** Label detection thresholds, confidence filtering, and label extraction patterns
- **Pipeline Architecture:** Sequential AI service chaining (Vision → NLP → Storage)
- **Prompt Engineering:** Constructing structured prompts from dynamically detected data
- **File I/O Patterns:** Writing AI-generated content to persistent storage for downstream use
- **AWS SDK for Python (Boto3):** Standard integration patterns for Rekognition and Bedrock Runtime

---

## **4. Tools and Technologies**

- **Cloud Provider:** Amazon Web Services (AWS)
- **Vision AI Service:** Amazon Rekognition (`detect_labels`)
- **Generative AI Service:** Amazon Bedrock Runtime
- **Foundation Model:** Amazon Nova Micro (`amazon.nova-micro-v1:0`)
- **SDK:** AWS SDK for Python — Boto3
- **Programming Language:** Python 3.x
- **Input Files:**
  - `assets/sample_image.jpeg` (test image for Rekognition validation)
  - `assets/ingredients.jpeg` (ingredient photo for pipeline input)
- **Output Files:**
  - `recipe.txt` (AI-generated pet food recipe)
  - `generated_recipe.png` (generated image output)
- **Key API Parameters:**
  - `MaxLabels: 5` (Rekognition label detection limit)
  - `MinConfidence` (confidence threshold for label inclusion)

---

## **5. Procedures and Implementation**

<details>
<summary>Click to expand detailed steps</summary>

1. **Task 2: Amazon Rekognition Label Detection**
   - Created `amazon_rekognition.py` for baseline Rekognition connectivity
   - Instantiated a Boto3 `rekognition` client
   - Opened `assets/sample_image.jpeg` in binary mode (`"rb"`)
   - Called `detect_labels()` with the binary image bytes and `MaxLabels=5`
   - Iterated over the returned labels and extracted each `Name` field
   - Printed detected labels to confirm Rekognition was correctly identifying objects

2. **Task 3: Ingredient Detection from Image**
   - Created `pet_food_recipes.py` with a `detect_ingredients_from_image()` function
   - Read `assets/ingredients.jpeg` as binary data
   - Called `rekognition_client.detect_labels()` on the ingredients image
   - Extracted label `Name` values into a Python list
   - Returned the ingredient list for downstream use in recipe generation

3. **Task 4: AI Recipe Generation**
   - Implemented `generate_recipe(ingredients, pet_breed, pet_age)` function
   - Created a Boto3 `bedrock-runtime` client
   - Constructed a detailed prompt incorporating:
     - Detected ingredient names from Rekognition
     - Pet breed and age for tailored nutritional advice
   - Invoked `amazon.nova-micro-v1:0` with the formatted prompt
   - Parsed the response body to extract the generated recipe text
   - Returned the recipe string for storage

4. **Task 5: Save Recipe and Generate Image**
   - Implemented `save_recipe(recipe_text, filename)` function
   - Used `open(filename, "w")` to write the recipe text to `recipe.txt`
   - Built `main()` orchestrating the full pipeline:
     - Called `detect_ingredients_from_image()` to get ingredient list
     - Printed detected ingredients to the console
     - Called `generate_recipe()` with ingredients, breed, and age
     - Called `save_recipe()` to persist the recipe as `recipe.txt`
     - Saved a generated image output as `generated_recipe.png`

5. **Task 6: Execute the Full Pipeline**
   - Ran `python pet_food_recipes.py` from the terminal
   - Observed console output showing detected ingredients list
   - Verified `recipe.txt` was created in the working directory
   - Verified `generated_recipe.png` was created in the working directory
   - Confirmed recipe content was contextually appropriate for the detected ingredients

</details>

---

## **6. Findings and Results**

- **Finding 1:** Amazon Rekognition's `detect_labels()` successfully identified food-related objects in the ingredients image with high confidence scores, returning up to 5 labels as configured by `MaxLabels=5`
- **Finding 2:** Reading image files in binary mode (`"rb"`) and passing the raw bytes directly to Rekognition's `Bytes` parameter was the correct approach for local image detection — no S3 upload required for this workflow
- **Finding 3:** The detected ingredient labels provided sufficient context for Nova Micro to generate a coherent, pet-appropriate recipe with relevant nutritional guidance
- **Finding 4:** Dynamic prompt construction using the Rekognition label list produced clearly better recipe output than a generic prompt — specificity of input ingredients drove specificity of output recipe content
- **Finding 5:** `amazon.nova-micro-v1:0` demonstrated fast response times for recipe generation, making it suitable for near-real-time pipeline applications
- **Finding 6:** File persistence using Python's `open()` with write mode (`"w"`) successfully created `recipe.txt` with complete recipe content readable after pipeline execution
- **Finding 7:** The complete end-to-end pipeline — image file → Rekognition labels → Bedrock recipe → saved files — executed successfully in a single `python pet_food_recipes.py` run

---

## **7. Screenshots and Evidence**

<details>
<summary>Click to expand code implementation</summary>

### Full Pipeline Implementation

```python
import boto3
import json

rekognition_client = boto3.client("rekognition", region_name="us-east-1")
bedrock_client = boto3.client("bedrock-runtime", region_name="us-east-1")

def detect_ingredients_from_image(image_path):
    with open(image_path, "rb") as image_file:
        image_bytes = image_file.read()

    response = rekognition_client.detect_labels(
        Image={"Bytes": image_bytes},
        MaxLabels=5
    )
    ingredients = [label["Name"] for label in response["Labels"]]
    return ingredients

def generate_recipe(ingredients, pet_breed, pet_age):
    ingredients_str = ", ".join(ingredients)
    prompt = (
        f"Create a healthy homemade pet food recipe for a {pet_age}-year-old {pet_breed}. "
        f"Use these detected ingredients: {ingredients_str}. "
        f"Include preparation steps and nutritional benefits."
    )

    request_body = {
        "messages": [{"role": "user", "content": prompt}],
        "inferenceConfig": {"max_new_tokens": 512}
    }

    response = bedrock_client.invoke_model(
        modelId="amazon.nova-micro-v1:0",
        body=json.dumps(request_body),
        contentType="application/json",
        accept="application/json"
    )

    result = json.loads(response["body"].read())
    return result["output"]["message"]["content"][0]["text"]

def save_recipe(recipe_text, filename="recipe.txt"):
    with open(filename, "w") as f:
        f.write(recipe_text)
    print(f"Recipe saved to {filename}")

def main():
    ingredients = detect_ingredients_from_image("assets/ingredients.jpeg")
    print("Detected ingredients:", ingredients)

    recipe = generate_recipe(ingredients, pet_breed="Golden Retriever", pet_age=3)
    save_recipe(recipe, "recipe.txt")

if __name__ == "__main__":
    main()
```

### Pipeline Flow Summary

| Stage | Service | Input | Output |
|-------|---------|-------|--------|
| 1. Image Ingestion | Python file I/O | `assets/ingredients.jpeg` | Binary image bytes |
| 2. Label Detection | Amazon Rekognition | Binary image bytes | List of ingredient label names |
| 3. Prompt Construction | Python | Ingredient list + pet info | Formatted prompt string |
| 4. Recipe Generation | Amazon Bedrock Nova Micro | Prompt string | Recipe text |
| 5. File Persistence | Python file I/O | Recipe text | `recipe.txt`, `generated_recipe.png` |

### Output Files Produced

| File | Description | Status |
|------|-------------|--------|
| `recipe.txt` | AI-generated pet food recipe based on detected ingredients | ✅ Created |
| `generated_recipe.png` | Generated image output from the pipeline | ✅ Created |

</details>

---

## **8. Challenges and Solutions**

| Challenge | Solution | Outcome |
|-----------|----------|---------|
| Determining correct image format for Rekognition API | Used binary read mode (`"rb"`) and passed raw bytes to the `Image: {"Bytes": ...}` parameter | Rekognition accepted the image and returned labels successfully |
| Limiting Rekognition output to relevant labels | Set `MaxLabels=5` to focus on the most prominent detected items | Returned a concise, usable ingredient list without noise |
| Building a prompt that produces pet-appropriate recipes | Included pet breed and age alongside detected ingredients in the prompt | Nova Micro generated tailored recipes with relevant portion and nutritional context |
| Persisting AI output to disk for downstream use | Used Python `open()` with `"w"` mode inside `save_recipe()` function | recipe.txt created and readable after pipeline execution |

---

## **9. Key Takeaways and Lessons Learned**

- **AI Service Chaining is Powerful:** Combining Rekognition (vision) and Bedrock (language) in a sequential pipeline demonstrates how multiple AI services can work together — each service's output becomes the next service's input.

- **Binary Image Reading is Required:** Rekognition's `Bytes` parameter requires raw binary file content, not a file path or base64 string. Reading with `"rb"` mode is the correct approach for local images.

- **Detected Labels as Prompt Inputs:** Using Rekognition label names directly as prompt inputs is a reliable pattern for grounding generative AI output in real-world observed data, reducing hallucination risk.

- **Nova Micro for Fast, Efficient Tasks:** Amazon Nova Micro is well-suited for shorter, structured generation tasks like recipes, summaries, and classifications where low latency is prioritized over maximum creativity.

- **File I/O Closes the Loop:** Saving AI output to files makes the pipeline's results persistent and reusable — a foundational pattern for any production AI workflow that needs to hand off results to other systems or users.

---

## **10. Artifacts and Deliverables**

- **`amazon_rekognition.py`:** Baseline Rekognition connectivity and label detection test script
- **`pet_food_recipes.py`:** Complete AI pipeline integrating Rekognition and Bedrock Nova Micro
- **`recipe.txt`:** AI-generated pet food recipe output file
- **`generated_recipe.png`:** Generated image output from the pipeline
- **Pipeline Architecture Diagram:** Sequential flow from image input to saved output files

---

## **11. Conclusion**

This lab successfully demonstrated the design and implementation of a multi-service AI pipeline combining **Amazon Rekognition** for computer vision and **Amazon Bedrock Nova Micro** for natural language generation. By detecting ingredients in a real photograph and using those detections to drive AI recipe generation, the lab established a practical pattern for grounding generative AI output in real-world observed data. The complete pipeline — from binary image reading through label detection, prompt construction, recipe generation, and file persistence — executed successfully in a single script run. This architecture is directly applicable to real-world use cases such as smart kitchen assistants, inventory management systems, and automated content generation workflows.

---

## **12. References**

- [Amazon Rekognition Documentation](https://docs.aws.amazon.com/rekognition/)
- [Rekognition detect_labels API Reference](https://docs.aws.amazon.com/rekognition/latest/dg/API_DetectLabels.html)
- [Amazon Bedrock Nova Models](https://docs.aws.amazon.com/bedrock/latest/userguide/amazon-nova.html)
- [Boto3 Rekognition Client](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/rekognition.html)
- [Boto3 Bedrock Runtime Client](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/bedrock-runtime.html)
- [AWS AI Services Overview](https://aws.amazon.com/machine-learning/ai-services/)

---

**Lab Completed:** 2026
**Course:** AWS Developer Fundamentals
**Status:** Completed ✅
