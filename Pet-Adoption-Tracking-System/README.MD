# Python Dictionaries – Pet Adoption Tracking System

![Lab Report](https://img.shields.io/badge/Status-Completed-brightgreen)
![Framework](https://img.shields.io/badge/Framework-Python%20Programming-blue)
![Tools](https://img.shields.io/badge/Tools-Python%2C%20Dictionaries%2C%20F--Strings-orange)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-yellow)
![Duration](https://img.shields.io/badge/Duration-Hands--On%20Activity-lightblue)

---

## **1. Lab Title**

> Week 2 Activity: Working with Python Dictionaries — Building a Pet Adoption Tracking System for AnyCompany Pet Society

---

## **2. Purpose and Objectives**

The objective of this activity is to gain hands-on experience working with the **Python dictionary data structure** by building a practical pet adoption tracking system for AnyCompany Pet Society. The activity reinforces core Python concepts including dictionary creation, data access, mutation, and conditional logic.

**Key Learning Outcomes:**
- Create and initialize a **Python dictionary** with key-value pairs during instantiation
- Use **bracket notation** to add new keys and update existing values in a dictionary
- Construct **f-strings** to produce formatted, dynamic output messages
- Apply **conditional logic** to evaluate dictionary values and produce context-appropriate responses
- Understand the practical use of dictionaries as structured data containers in real-world applications

---

## **3. Frameworks, Standards, and Methodologies**

- **Python Programming Best Practices:** Clean, readable code using Pythonic conventions for dictionary manipulation
- **PEP 8 Style Guide:** Consistent naming conventions, spacing, and code formatting standards
- **Data Structure Fundamentals:** Understanding key-value pair storage and retrieval patterns
- **Conditional Logic Design:** If-else branching based on dictionary value evaluation
- **String Formatting Standards:** Use of f-strings for dynamic, readable output generation

---

## **4. Tools and Technologies**

- **Programming Language:** Python 3.x
- **Data Structure:** Python Dictionary (`dict`)
- **Development Environment:** AnyCompany Pet Society practice environment
- **Core Python Features Used:**
  - Dictionary literals and initialization syntax
  - Bracket notation for key access, insertion, and update
  - F-strings (`f""`) for formatted string output
  - Conditional statements (`if`, `else`)
- **File Created:** `python_dicts.py`

**Technical Stack:**
- Pure Python implementation — no external libraries required
- Single-file script architecture
- Interactive output via `print()` statements
- Conditional branching for business logic enforcement

---

## **5. Procedures and Implementation**

<details>
<summary>Click to expand detailed steps</summary>

1. **Created the Python Script File**
   - Created a new file named `python_dicts.py` in the practice environment
   - Added starter code as the foundation for the pet adoption tracker

2. **Dictionary Creation with Initial Data**
   - Created a dictionary literal to store adopted pet details at initialization time
   - Populated four key-value pairs:
     - `"name": "Charlie"`
     - `"age": 1`
     - `"type": "rabbit"`
     - `"adopted_by": "Sarah"`

3. **Adding a New Key Using Bracket Notation**
   - Used bracket notation to add a new key to the existing dictionary
   - Added `"adoption_fee"` with a value of `50`
   - Demonstrated that dictionaries are mutable and can be extended after creation

4. **Updating an Existing Value**
   - Updated the `"age"` key from `1` to `2` to reflect Charlie's recent birthday
   - Used bracket notation assignment to overwrite the existing value
   - Confirmed dictionary mutation does not require re-creating the entire structure

5. **Printing a Formatted Message with F-String**
   - Constructed an f-string to output a personalized adoption message
   - Included the pet's name, age, type, and adoption fee dynamically from dictionary values
   - Printed the message to confirm all values were correctly accessible

6. **Implementing Conditional Logic on Adoption Fee**
   - Wrote an `if` statement checking whether `adoption_fee` exceeded `$40`
   - If `True`: printed a thank-you message to the adopter acknowledging the generous contribution
   - If `False`: printed an alternative message with different messaging
   - Verified the condition evaluated correctly against the stored value of `50`

</details>

---

## **6. Findings and Results**

- **Finding 1:** Python dictionaries successfully store heterogeneous data types (strings and integers) within a single structure, making them ideal for representing real-world entities like pet adoption records
- **Finding 2:** Bracket notation provided a clean and readable syntax for both inserting new keys (`pet["adoption_fee"] = 50`) and updating existing ones (`pet["age"] = 2`), with no distinction between the two operations
- **Finding 3:** F-strings allowed seamless embedding of dictionary values into output strings without concatenation, producing readable and maintainable code
- **Finding 4:** The conditional check (`if pet["adoption_fee"] > 40`) evaluated correctly against the stored integer value, confirming that dictionary values can be used directly in comparison expressions
- **Finding 5:** The adoption fee of `50` exceeded the `$40` threshold, triggering the thank-you branch of the conditional — validating correct branching logic
- **Finding 6:** Dictionary mutation (adding keys, updating values) operated in-place without requiring reassignment of the variable, demonstrating the mutable nature of Python dictionaries
- **Finding 7:** The complete script executed without errors, producing formatted output that reflected all four data operations: creation, insertion, update, and conditional evaluation

---

## **7. Screenshots and Evidence**

<details>
<summary>Click to expand code implementation</summary>

### Final Implementation: `python_dicts.py`

```python
# Create a dictionary with initial pet adoption details
pet = {
    "name": "Charlie",
    "age": 1,
    "type": "rabbit",
    "adopted_by": "Sarah"
}

# Add adoption_fee using bracket notation
pet["adoption_fee"] = 50

# Update age because Charlie's birthday just passed
pet["age"] = 2

# Print a formatted message using an f-string
print(f"Congratulations {pet['adopted_by']}! You have adopted a {pet['age']}-year-old {pet['type']} named {pet['name']}. Your adoption fee is ${pet['adoption_fee']}.")

# Conditional check on adoption fee
if pet["adoption_fee"] > 40:
    print(f"Thank you so much, {pet['adopted_by']}! Your generous adoption fee helps us care for more animals.")
else:
    print(f"Thank you, {pet['adopted_by']}! Every contribution helps us provide better care for our animals.")
```

### Expected Output

```
Congratulations Sarah! You have adopted a 2-year-old rabbit named Charlie. Your adoption fee is $50.
Thank you so much, Sarah! Your generous adoption fee helps us care for more animals.
```

### Dictionary State Summary

| Operation | Key | Value Before | Value After |
|-----------|-----|-------------|-------------|
| Initialize | `name` | — | `"Charlie"` |
| Initialize | `age` | — | `1` |
| Initialize | `type` | — | `"rabbit"` |
| Initialize | `adopted_by` | — | `"Sarah"` |
| Add (bracket) | `adoption_fee` | — | `50` |
| Update (bracket) | `age` | `1` | `2` |

</details>

---

## **8. Challenges and Solutions**

| Challenge | Solution | Outcome |
|-----------|----------|---------|
| Understanding that bracket notation serves both insertion and update | Tested both use cases — adding a new key and overwriting an existing one — and observed identical syntax | Confirmed Python handles both transparently based on key existence |
| Accessing dictionary values inside f-strings without syntax errors | Used single quotes inside the f-string expression (`pet['name']`) when the outer string used double quotes | Resolved quote conflict cleanly without escaping |
| Ensuring conditional logic evaluated integer values not strings | Confirmed `adoption_fee` was stored as integer `50` not string `"50"` during initialization | Comparison `> 40` evaluated correctly without type errors |
| Keeping code readable as dictionary grew | Maintained consistent indentation and descriptive key names | Code remained clean and self-documenting throughout |

---

## **9. Key Takeaways and Lessons Learned**

- **Dictionaries as Real-World Models:** Python dictionaries are highly effective for modeling real-world entities. Representing a pet adoption record as a dictionary with named keys is far more readable and maintainable than using a list with positional indices.

- **Mutability is Powerful:** The ability to add keys and update values in-place makes dictionaries flexible data containers that can evolve as program requirements change — a critical property for dynamic applications.

- **F-Strings Simplify Output:** F-strings dramatically improve the readability of output code compared to string concatenation. Embedding dictionary access expressions directly in f-strings (`pet['name']`) produces clean, concise print statements.

- **Conditional Logic on Dictionary Values:** Dictionary values can be used directly in conditional expressions, enabling business logic to be driven by stored data. This pattern scales naturally to more complex decision-making systems.

- **Foundation for Larger Systems:** This activity established the foundational patterns used in more complex applications — storing structured data, updating state, and driving behavior from stored values — patterns that appear in databases, APIs, and full application development.

---

## **10. Artifacts and Deliverables**

- **`python_dicts.py`:** Complete Python script implementing the pet adoption dictionary tracker
- **Dictionary State Table:** Documentation of all key-value operations performed during execution
- **Expected Output Log:** Verified console output confirming correct program behavior
- **Code Comments:** Inline documentation explaining each operation for educational reference

---

## **11. Conclusion**

This activity successfully demonstrated the core capabilities of Python dictionaries through a practical pet adoption tracking use case. By creating, populating, mutating, and querying a dictionary, the foundational skills required for working with structured data in Python were reinforced. The activity showed that dictionaries are not only simple to use but also powerful enough to represent real-world entities and drive application logic through stored values. The combination of bracket notation, f-strings, and conditional logic provided a complete mini-application that models how data is managed in production Python systems. These patterns form the building blocks for more advanced data structures and applications explored in subsequent labs.

---

## **12. References**

- [Python Documentation – Dictionaries](https://docs.python.org/3/tutorial/datastructures.html#dictionaries)
- [Python F-Strings Documentation](https://docs.python.org/3/reference/lexical_analysis.html#f-strings)
- [PEP 8 – Python Style Guide](https://pep8.org/)
- [Python Built-in Types – Mapping Types](https://docs.python.org/3/library/stdtypes.html#mapping-types-dict)
- [AWS Cloud Institute – Developer Fundamentals Course Materials](https://awscloudinstitute.skillbuilder.aws)

---

**Lab Completed:** 2026
**Course:** AWS Developer Fundamentals
**Status:** Completed ✅
