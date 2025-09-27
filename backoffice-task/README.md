# Pharmaceutical Product Validation System

This is a legacy Node.js application for validating and processing pharmaceutical product registrations.

## Prerequisites

- Node.js (version 12 or higher)
- No external dependencies required

## Project Structure

```
your-repo-name/
├── README.md # Project overview and instructions
├── src/ # Fixed source code files
│ ├── validators.js # Validation logic for products
│ ├── product-processor.js # Product submission and batch processing
│ ├── mock-database.js # In-memory mock database
│ └── utils.js # Helper functions (formatting, logging)
├── test-data/ # Original test data files
├── test.js # Demo script
├── test-validation.js # Validation test script
├── test-performance.js # Performance test script
└── SOLUTION.md # Detailed solution documentation
```

## Project Overview

This system simulates the backend logic of a pharmaceutical product validation workflow.
This repository contains a legacy Node.js application for pharmaceutical product validation and processing. It includes fixes for input validation, cache handling, performance improvements, and an analysis of support tickets as part of the practical coding assessment.

Check `SOLUTION.md` for more details about bug fixes and support ticket assessments.

## Use Cases

- Validating regulatory IDs for pharmaceutical products before database entry
- Preventing accidental duplicate records in product databases
- Performing bulk submissions with performance measurements
- Demonstrating parallel vs sequential processing in Node.js

## How to Run

1. **Extract all files** to a folder (maintain the directory structure)
2. **Navigate to the project folder** in terminal/command prompt
3. **Run the example script** to see the system in action:

```bash
node test.js
node test-performance.js
node test-validation.js
```
