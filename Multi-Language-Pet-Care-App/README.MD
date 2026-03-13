# Multi-language Pet Care Application – Amazon Translate

![Lab Report](https://img.shields.io/badge/Status-Completed-brightgreen)
![Framework](https://img.shields.io/badge/Framework-AWS%20Best%20Practices-blue)
![Tools](https://img.shields.io/badge/Tools-Amazon%20Translate%2C%20Boto3%2C%20Python-orange)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow)
![Duration](https://img.shields.io/badge/Duration-Hands--On%20Lab-lightblue)

---

## **1. Lab Title**

> Week 4 Lab: Building a Multi-language Application with Amazon Translate — Dynamic Pet Data Translation for International Pet Adoption

---

## **2. Purpose and Objectives**

The objective of this lab is to build a fully functional **multi-language pet care application** using **Amazon Translate** and Boto3. The application allows users to select a pet and a target language, then dynamically translates the pet's background information into their preferred language — enabling international accessibility for AnyCompany Pet Society's adoption platform.

**Key Learning Outcomes:**
- Connect to **Amazon Translate** using a Boto3 client and call `translate_text()`
- Build a **structured pet database** using nested Python dictionaries
- Implement a **language code mapping function** supporting Spanish, German, French, Italian, and English
- Create a **pet information formatter** using f-strings for clean, structured output
- Develop a **translation function** that integrates Boto3 Translate with source/target language parameters
- Build a complete **`main()` function** with user input, validation, translation, and formatted output
- Test edge cases including invalid pet IDs and unsupported languages

---

## **3. Frameworks, Standards, and Methodologies**

- **AWS Well-Architected Framework:** Reliability and operational excellence for multilingual application design
- **Internationalization (i18n) Best Practices:** Language code standardization and dynamic content translation
- **Data Architecture:** Nested dictionary design for structured relational pet data
- **Input Validation Patterns:** Defensive programming for user-supplied pet IDs and language selections
- **API-First Translation:** Treating Amazon Translate as a stateless translation microservice
- **Separation of Concerns:** Modular function design separating formatting, translation, and application logic

---

## **4. Tools and Technologies**

- **Cloud Provider:** Amazon Web Services (AWS)
- **Translation Service:** Amazon Translate
- **SDK:** AWS SDK for Python — Boto3
- **Programming Language:** Python 3.x
- **AWS Services Used:**
  - Amazon Translate (`boto3.client('translate')`)
  - AWS IAM (permissions for Translate API access)
- **Supported Languages:**
  - Spanish (`es`), German (`de`), French (`fr`), Italian (`it`), English (`en`)
- **Files Created:**
  - `amazon_translate.py` (Task 2 — baseline connectivity)
  - `pet_data_translator.py` (Tasks 3–6 — full application)

**Technical Stack:**
- Nested dictionary pet database (in-memory)
- `translate_text()` API with `SourceLanguageCode` and `TargetLanguageCode`
- Language code lookup dictionary
- F-string formatted output with Name, Breed, Age, Price, and Background fields

---

## **5. Procedures and Implementation**

<details>
<summary>Click to expand detailed steps</summary>

1. **Task 2: Amazon Translate Baseline Connectivity**
   - Created `amazon_translate.py` to verify service connectivity
   - Instantiated a Boto3 `translate` client
   - Built a `translate_text()` function call with a test English phrase
   - Tested English → Spanish translation and English → German translation
   - Verified response structure and extracted `TranslatedText` field

2. **Task 3: Define the Pet Database**
   - Created `pet_data_translator.py` with a nested `PETS_DATABASE` dictionary
   - Defined three pets with unique IDs:
     - `P001`: Max, German Shepherd dog, age 3, $200, background in English
     - `P002`: Luna, Siamese cat, age 2, $150, background in English
     - `P003`: Bella, Golden Retriever dog, age 1, $250, background in English
   - Each entry contained: `name`, `breed`, `age`, `price`, `background`
   - Built a `LANGUAGES` dictionary mapping display names to ISO language codes:
     - `"Spanish": "es"`, `"German": "de"`, `"French": "fr"`, `"Italian": "it"`, `"English": "en"`

3. **Task 4: Build the Pet Info Formatter**
   - Implemented `format_pet_info(pet, translated_background)` function
   - Used an f-string to produce structured output including:
     - Name, Breed, Age, Price, and the translated background text
   - Function returned a formatted string for display to the user

4. **Task 5: Build the Translation Function**
   - Implemented `translate_pet_info(text, target_language_code)` function
   - Created a Boto3 `translate` client inside the function
   - Called `translate_text()` with:
     - `Text`: the pet's background text
     - `SourceLanguageCode`: `"en"` (all data stored in English)
     - `TargetLanguageCode`: the user's selected language code
   - Returned the `TranslatedText` value from the response

5. **Task 6: Build the Main Application Function**
   - Implemented `main()` orchestrating the full user workflow:
     - Prompted user for a pet ID (converted to uppercase for consistency)
     - Validated the pet ID exists in `PETS_DATABASE`
     - Prompted user for a target language
     - Validated the language exists in `LANGUAGES`
     - Called `translate_pet_info()` to translate the background field
     - Called `format_pet_info()` to produce the final formatted output
     - Printed the result to the console

6. **Task 7: Test Valid and Invalid Scenarios**
   - **Valid test:** Pet ID `P002` (Luna), language `Spanish` → background translated to Spanish ✅
   - **Invalid pet ID:** Pet ID `P999` → graceful error message "Pet not found" ✅
   - **Unsupported language:** Language `Japanese` → graceful error message "Language not supported" ✅

</details>

---

## **6. Findings and Results**

- **Finding 1:** Amazon Translate's `translate_text()` API successfully translated English pet background text into Spanish, German, French, and Italian with natural, contextually appropriate phrasing
- **Finding 2:** The nested `PETS_DATABASE` dictionary provided an effective in-memory data structure for representing relational pet records, with each pet accessible by unique ID key
- **Finding 3:** The `LANGUAGES` dictionary mapping display names to ISO codes cleanly decoupled user-facing language names from the API's required language code format
- **Finding 4:** Converting user-supplied pet IDs to uppercase (`.upper()`) resolved case-sensitivity issues and prevented valid IDs from being incorrectly rejected
- **Finding 5:** Luna's (P002) background text was successfully translated into Spanish with accurate vocabulary and grammar, confirming Amazon Translate's quality for short pet description content
- **Finding 6:** Invalid pet ID `P999` triggered the expected validation branch, printing a user-friendly error without crashing the application
- **Finding 7:** Unsupported language `Japanese` triggered the language validation branch correctly, as Japanese was not included in the `LANGUAGES` dictionary
- **Finding 8:** Modular function design (separate format, translate, and main functions) allowed each component to be tested independently before integration

---

## **7. Screenshots and Evidence**

<details>
<summary>Click to expand code implementation</summary>

### Pet Database and Language Map

```python
PETS_DATABASE = {
    "P001": {
        "name": "Max", "breed": "German Shepherd", "age": 3,
        "price": 200,
        "background": "Max is a loyal and energetic dog who loves outdoor activities and playing fetch."
    },
    "P002": {
        "name": "Luna", "breed": "Siamese", "age": 2,
        "price": 150,
        "background": "Luna is an elegant and curious cat who enjoys warm sunlit spots and gentle play."
    },
    "P003": {
        "name": "Bella", "breed": "Golden Retriever", "age": 1,
        "price": 250,
        "background": "Bella is a playful and affectionate puppy who is great with children and families."
    }
}

LANGUAGES = {
    "Spanish": "es",
    "German": "de",
    "French": "fr",
    "Italian": "it",
    "English": "en"
}
```

### Core Functions

```python
import boto3

def translate_pet_info(text, target_language_code):
    translate_client = boto3.client("translate", region_name="us-east-1")
    response = translate_client.translate_text(
        Text=text,
        SourceLanguageCode="en",
        TargetLanguageCode=target_language_code
    )
    return response["TranslatedText"]

def format_pet_info(pet, translated_background):
    return (
        f"Name: {pet['name']}\n"
        f"Breed: {pet['breed']}\n"
        f"Age: {pet['age']} years\n"
        f"Price: ${pet['price']}\n"
        f"Background: {translated_background}"
    )

def main():
    pet_id = input("Enter pet ID: ").strip().upper()
    if pet_id not in PETS_DATABASE:
        print("Pet not found. Please enter a valid pet ID.")
        return

    pet = PETS_DATABASE[pet_id]
    language_name = input("Enter target language (Spanish/German/French/Italian/English): ").strip()

    if language_name not in LANGUAGES:
        print(f"Language '{language_name}' is not supported.")
        return

    target_code = LANGUAGES[language_name]
    translated_background = translate_pet_info(pet["background"], target_code)
    print(format_pet_info(pet, translated_background))
```

### Test Results Summary

| Test Case | Input | Expected Result | Actual Result |
|-----------|-------|----------------|---------------|
| Valid pet + valid language | `P002`, `Spanish` | Luna's background in Spanish | ✅ Translated correctly |
| Invalid pet ID | `P999`, `Spanish` | "Pet not found" message | ✅ Error handled gracefully |
| Unsupported language | `P001`, `Japanese` | "Language not supported" message | ✅ Error handled gracefully |

</details>

---

## **8. Challenges and Solutions**

| Challenge | Solution | Outcome |
|-----------|----------|---------|
| User entering lowercase pet IDs (e.g., `p002` instead of `P002`) | Applied `.upper()` to user input before dictionary lookup | Case-insensitive pet ID lookup without modifying the database keys |
| Decoupling user-friendly language names from ISO language codes | Created a `LANGUAGES` mapping dictionary to translate display names to API codes | Clean separation between user input and API requirements |
| Handling unsupported languages gracefully | Validated language name against `LANGUAGES` dictionary before calling Translate | Application exits cleanly with informative error message |
| Structuring multi-field pet output readably | Built `format_pet_info()` with labeled f-string fields | Output clearly displays all pet details with translated background |

---

## **9. Key Takeaways and Lessons Learned**

- **Amazon Translate is Stateless and Simple:** The `translate_text()` API requires only the text, source language, and target language — no model configuration, no warm-up time. This simplicity makes it ideal for embedding translation into any application workflow.

- **Data Architecture Drives Application Design:** The nested dictionary structure for `PETS_DATABASE` directly shaped how every other function was written. Good data architecture reduces complexity throughout the application.

- **Input Normalization Prevents Subtle Bugs:** Normalizing user input (`.upper()`, `.strip()`) before comparison catches common entry errors without burdening the user with strict format requirements.

- **Validation Before API Calls:** Checking both the pet ID and language name before invoking the Translate API prevents unnecessary API calls and provides faster, clearer feedback to the user.

- **Modular Functions Enable Testing:** Breaking the application into `translate_pet_info()`, `format_pet_info()`, and `main()` allowed each function to be tested and verified independently before integration testing.

---

## **10. Artifacts and Deliverables**

- **`amazon_translate.py`:** Baseline connectivity script for Amazon Translate with test translations
- **`pet_data_translator.py`:** Complete multi-language pet info application with database, translation, and formatting
- **`PETS_DATABASE`:** In-memory nested dictionary with three pet records
- **`LANGUAGES` Map:** Language display name to ISO code mapping reference
- **Test Results Documentation:** Verified output for valid, invalid ID, and unsupported language scenarios

---

## **11. Conclusion**

This lab successfully demonstrated how to integrate **Amazon Translate** into a Python application to deliver localized content dynamically. By building a structured pet database, implementing language code mapping, and connecting to the Translate API via Boto3, a complete multi-language application was produced. The `main()` function orchestrated user input collection, validation, translation, and formatted display in a clean, linear workflow. Edge case testing confirmed robust error handling for invalid pet IDs and unsupported languages. This lab establishes a reusable pattern for any application requiring real-time content translation — applicable to e-commerce, healthcare, education, and any customer-facing platform serving an international audience.

---

## **12. References**

- [Amazon Translate Documentation](https://docs.aws.amazon.com/translate/)
- [Amazon Translate Supported Languages](https://docs.aws.amazon.com/translate/latest/dg/what-is-languages.html)
- [Boto3 Translate Client Reference](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/translate.html)
- [Python Dictionary Documentation](https://docs.python.org/3/tutorial/datastructures.html#dictionaries)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [Internationalization Best Practices](https://www.w3.org/International/quicktips/)

---

**Lab Completed:** 2026
**Course:** AWS Developer Fundamentals
**Status:** Completed ✅
