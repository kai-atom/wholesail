# Building an Intelligent Data Integration Agent Based on Large Language Models

---

### 1. Executive Summary

This project aims to develop an intelligent agent based on Large Language Models (LLMs) to automate the generation of data integration configuration files for ERP systems. Currently, data integration for different ERP systems requires significant manual effort to write and debug complex YAML configuration files. This intelligent agent will analyze raw data to automatically understand and generate the necessary transformation rules. **The goal is to reduce the integration cycle from weeks to a few days**, significantly lowering labor costs, minimizing configuration errors, accelerating the customer onboarding process, and enabling maintenance entirely by business personnel without engineering involvement.

### 2. Project Goal

To build an intelligent agent capable of automatically analyzing raw data (in CSV format) exported from various ERP systems and generating `YAML` data transformation configuration files that meet Wholesail's internal standards and are ready for production deployment.

### 3. Core Problem & Solution

*   **Current Problems:**
    *   **High Manual Dependency:** Each new ERP integration requires engineers to deeply understand business logic and manually write complex YAML rules, a time-consuming and error-prone process.
    *   **Poor Scalability:** The current process is difficult to scale across a wide variety of ERP systems, creating a bottleneck for business expansion.
    *   **Knowledge Silos:** Transformation logic and business rules are scattered across individual experience and YAML configuration files, making them difficult to maintain and transfer.

*   **Solution:**
    *   Leverage the powerful pattern recognition and code generation capabilities of LLMs to have the Agent act as a "Data Integration Expert."
    *   By providing the Agent with raw data samples and the target data structure, it can automatically learn mapping relationships, infer transformation logic, and generate standardized YAML configuration files.

### 4. Key Challenges & Mitigation Strategies

1.  **Challenge: Correct Application of DSL and Generation of Complex Logic**
    *   **Description:** The transformation rules in YAML use a dedicated Java function library (DSL) provided by Wholesail engineers. The challenge is for the Agent to accurately understand business requirements and select the most appropriate functions from the library to build complex transformation logic.
    *   **Risk:** **(Medium)** The Agent might select incorrect functions or fail to combine them properly to handle complex business scenarios (e.g., nested conditional logic).
    *   **Mitigation:**
        *   Provide the complete DSL function library documentation as the Agent's core context, allowing it to work in an "open-book exam" environment.
        *   In the human-computer collaboration interface, allow business personnel to describe complex rules in natural language, which the Agent translates into DSL, followed by expert fine-tuning.

2.  **Challenge: Complex Implicit Business Rules**
    *   **Description:** Data transformation involves not just field mapping but also a significant amount of business logic (e.g., customer status transitions, address parsing).
    *   **Risk:** **(Medium)** The Agent may not be able to infer all business details from the data alone.
    *   **Mitigation:** The project requires **deep involvement from business experts** to provide and review key business rules, ensuring the Agent-generated configurations align with actual business needs.

3.  **Challenge: Validation Complexity and Data Sampling Risks**
    *   **Description:** The YAML files run as part of a large data pipeline in the production environment. Furthermore, showing business users only a small sample of successful data transformations might cause them to overlook errors or edge cases present in the full dataset.
    *   **Risk:** **(Medium)** Isolated YAML validation may not uncover integration issues. Incomplete data previews could lead to the approval of incorrect configurations.
    *   **Mitigation:**
        *   **Develop a Lightweight Testing Environment:** Create a local validation tool that simulates key features of the production data pipeline to ensure YAML files are thoroughly tested before deployment.
        *   **Implement Comprehensive Data Scanning:** In the web interface, the validation process **must process the entire uploaded sample dataset** and generate a detailed report specifying the number of successfully processed records, the number of failed records, specific error reasons, and corresponding line numbers.

### 5. Suggested Execution Plan (Phased Approach)

*   **Phase 1: Foundational Work (Est. 1-2 Weeks)**
    *   **Tasks:** Collect and organize the complete Java DSL function documentation; define a clear internal data model (Schema).
    *   **Deliverables:** A DSL function description document for the Agent's use and a standardized data model definition.

*   **Phase 2: Agent Prototype and Collaborative Interface Development (Est. 2-3 Weeks)**
    *   **Tasks:** Design the Agent's core workflow; develop the human-computer collaborative web interface described in Appendix B.
    *   **Deliverables:** An Agent prototype that can handle core scenarios and allow business personnel to visually review and adjust configurations.

*   **Phase 3: Testing, Evaluation, and Iteration (Ongoing)**
    *   **Tasks:** Validate Agent-generated configurations using real data; establish a feedback loop to continuously optimize the model and workflow.
    *   **Deliverables:** A stable, reliable, and intelligent data integration tool that can progressively cover more types of ERPs.

### 6. Strategic Consideration: Knowledge Base Creation and Evolution

As the agent processes more ERP systems, it will encounter many repetitive or similar business rules. It is crucial to effectively capture and reuse this "experience."

*   **Option A (Recommended for Short-Term): Dynamic Knowledge Base (Retrieval-Augmented Generation - RAG)**
    *   **Description:** Store every successful configuration case confirmed by business personnel (including source data characteristics, YAML rules, and business annotations) in a vector database to form a searchable knowledge base.
    *   **How it works:** When processing new ERP data, the Agent first searches the knowledge base for similar historical cases and injects these successful examples as high-quality reference information into the current prompt, guiding it to generate more accurate configurations.
    *   **Advantages:** Low cost, fast updates, no need for model retraining, and knowledge becomes effective immediately.

*   **Option B (Long-Term Evolution): Model Fine-Tuning**
    *   **Description:** Once a sufficient number of high-quality configuration cases (typically hundreds or thousands) have been accumulated, this data can be used to periodically fine-tune the base large model.
    *   **How it works:** Fine-tuning "internalizes" this domain knowledge into the model, turning it into a true "ERP Integration Expert Model."
    *   **Advantages:** Potentially higher accuracy and a deeper understanding of complex issues.
    *   **Disadvantages:** High cost, requires a large volume of high-quality, labeled data, and involves greater technical complexity.

**Recommendation:** Start with Option A to quickly establish knowledge reuse capabilities. Evaluate the cost-benefit of pursuing Option B once a significant volume of data and cases has been accumulated.

### 7. Conclusion and Next Steps

This project is a critical step toward enhancing our data integration capabilities and achieving scalable expansion. While challenges exist, the probability of success is high with clear planning and cross-departmental collaboration.

**Recommended Immediate Actions:**
1.  **[Technical Side]** Take responsibility for organizing the DSL function library documentation and begin designing the web-based collaborative interface and backend validation logic described in Appendix B.
2.  **[Business Side]** Arrange for business analysts to liaise with the technical team to start documenting core ERP data transformation business rules, providing the initial material for the Agent's knowledge base.

---

### Appendix A: `entree_spec_example.yaml` Structure Analysis

This YAML file is the core of the data transformation process. Its well-designed structure breaks down the complex ETL process into several logical stages:

1.  **`globalDataFrameSchema` & `localFunctions` (Global Definitions & Utility Functions):**
    *   **Purpose:** Defines the global format of the data source (e.g., CSV delimiter) and reusable transformation logic (e.g., the `toWsAddress` function for parsing addresses).
    *   **Analysis:** The existence of the `toWsAddress` function indicates that address format handling is a common pain point. The LLM Agent needs to be able to identify fields requiring complex parsing and potentially generate or call similar custom functions.

2.  **`acctToShadowSpecs` (Source Data to "Shadow Model"):**
    *   **Purpose:** Maps flat data rows read from a CSV file to one or more structured "shadow objects" (e.g., `CUSTOMER`, `CONTACT`) based on their source (e.g., filename matching `V.dataFrameName.startsWith("Customers")`).
    *   **Analysis:** This is a critical step for data preprocessing and structuring. The Agent must be able to automatically identify the business entity (customer, invoice, etc.) corresponding to the data based on filenames and column names, and generate the appropriate mapping configuration.

3.  **`shadowToIRSpecs` (Shadow Model to "Intermediate Representation"):**
    *   **Purpose:** This is the core of the business logic. It defines how to convert structured "shadow objects" into Wholesail's internal standard data model (IR - Intermediate Representation).
    *   **Business Logic Examples:**
        *   **Conditional Mapping:** `status: 'V.customer.status.toUpperCase().equals("ACTIVE") ? "PENDING" : "SUSPENDED"'`
            *   **Interpretation:** This rule indicates that an `"ACTIVE"` status for a customer in the ERP system is interpreted as `"PENDING"` in the Wholesail system, while other statuses are `"SUSPENDED"`. This is a typical business rule that requires confirmation from a business expert.
        *   **Data Transformation:** `balanceCents: F.fromDollars(V.customer.openAr)`
            *   **Interpretation:** Converts the `openAr` field, representing dollars, into `balanceCents` (in cents) using the `fromDollars` function. The Agent needs to understand currency unit conversion.
        *   **Field Concatenation:** `rawErpPaymentTerm: 'V.customer.customerTerms + " " + V.customer.termDueDays + ...'`
            *   **Interpretation:** Concatenates multiple fields related to payment terms into a more descriptive string.

4.  **`wsPaymentExportSpecs` (Data Export Specification):**
    *   **Purpose:** Defines the rules for exporting data (e.g., payment information) from the Wholesail system back to the source ERP system.
    *   **Analysis:** This part is "reverse ETL" and the logic is equally complex. The Agent must not only understand data import but also be able to generate configurations for data export based on requirements.

**Conclusion:** The structured design of the YAML provides a clear generation target for the LLM Agent. The Agent's core task is to populate the `fieldMappingRules` within `shadowToIRSpecs`, which requires capabilities in **field mapping, data type conversion, conditional logic generation, and custom function calls**.

---

### Appendix B: Human-in-the-Loop Workflow Design

To effectively leverage the expertise of business personnel and ensure the accuracy of the generated results, we have designed a web-based "human-in-the-loop" workflow. Business users can perform complex configuration reviews and adjustments through clicks and minimal text input, without writing any code.

#### **Workflow Steps:**

1.  **Upload Data:** Business personnel upload CSV files exported from the ERP system.
2.  **Agent Auto-Generation:** The LLM Agent analyzes the data and automatically generates a first draft of the YAML configuration.
3.  **Visual Review and Adjustment:** The system presents the YAML rules in a user-friendly interface for business personnel to review.
4.  **Real-time Validation:** Every time a business user makes an adjustment, the system runs the transformation with the data in the background and immediately displays the results.
5.  **Confirmation and Deployment:** Once the business user confirms that all rules and data previews are correct, the final YAML is generated and deployed with a single click.

#### **UI Mockups:**

**1. Data Upload & Analysis Interface**

```
+-------------------------------------------------------------------+
| Wholesail Intelligent Data Integration                            |
+-------------------------------------------------------------------+
|                                                                   |
|   Please upload your ERP data files (CSV format):                 |
|                                                                   |
|   [ Customer.csv ] [ Sales.csv ] [ Payments.csv ]                 |
|                                                                   |
|   [ Drop files here or Click to Upload ]                          |
|                                                                   |
|   [ Analyze Data and Generate Configuration ] (<- Click to start) |
|                                                                   |
+-------------------------------------------------------------------+
```

**2. Core: Mapping & Transformation Rule Review Interface**

This is the most important interface, where business personnel will collaborate with the Agent.

```
+----------------------------------------------------------------------------------------------------------+
| YAML Configuration Review: Customer -> IR_CUSTOMER                                                       |
+----------------------------------------------------------------------------------------------------------+
| Wholesail Target Field | Transformation Rule (Editable)                         | ERP Source Field  | Data Preview (Source -> Target) | Status |
+------------------------+--------------------------------------------------------+-------------------+-------------------------------+------+
| customerName           | V.customer.companyName                                 | [companyName]     | "A&T Foods" -> "A&T Foods"    |  ✅  |
+------------------------+--------------------------------------------------------+-------------------+-------------------------------+------+
| status                 | V.customer.status.toUpperCase().equals("ACTIVE") ?     | [status]          | "active" -> "PENDING"         |  ⚠️  |
|                        | "PENDING" : "SUSPENDED"                                |                   |                               |      |
|                        | [ Edit ] [ Please confirm this status mapping ]        |                   |                               |      |
+------------------------+--------------------------------------------------------+-------------------+-------------------------------+------+
| shippingAddress        | L.toWsAddress.call(V.customer.shippingAddress)         | [shippingAddress] | "7400 Scout..." -> { structured } |  ✅  |
+------------------------+--------------------------------------------------------+-------------------+-------------------------------+------+
| balanceCents           | F.fromDollars(V.customer.openAr)                       | [openAr]          | "19331.00" -> 1933100        |  ✅  |
+------------------------+--------------------------------------------------------+-------------------+-------------------------------+------+
| ...                    | ...                                                    | ...               | ...                           | ...  |
+----------------------------------------------------------------------------------------------------------+
| [ Save and Re-validate ]                                  [ Complete and Export YAML ]                   |
+----------------------------------------------------------------------------------------------------------+
```

*   **How Business Personnel Participate:**
    *   **Confirm:** For simple direct mappings (like `customerName`), the user just confirms the ✅ status.
    *   **Review & Edit:** For fields with complex logic (like `status`), the system highlights them with a ⚠️ and provides a prompt. The user can click `[ Edit ]` to describe the rule in more natural language (e.g., "If status is active, set to PENDING, otherwise set to SUSPENDED") or modify the code directly. The Agent will regenerate the DSL based on the new input.
    *   **Feedback Loop:** After clicking `[ Save and Re-validate ]`, the Agent updates the YAML, re-runs the transformation with the full dataset, and the `Data Preview` column updates instantly, creating a rapid feedback loop.

**3. Validation Results & Report Interface**

```
+-------------------------------------------------------------------+
| Comprehensive Validation Report                                   |
+-------------------------------------------------------------------+
| **Summary:**                                                      |
| - Customer.csv: Processed 1,258 rows.                             |
|   - ✅ Success: 1,250 rows (99.36%)                                 |
|   - ❌ Failed: 8 rows (0.64%)                                     |
|                                                                   |
| - Sales.csv: Processed 5,430 rows.                                |
|   - ✅ Success: 5,430 rows (100%)                                   |
+-------------------------------------------------------------------+
| **Failure Details (Customer.csv):**                               |
|                                                                   |
| 1. **Error Type: Unknown Status Mapping (3 cases)**               |
|    - Row 52: `status` field value is "ON_HOLD", not defined.      |
|    - Row 118: `status` field value is "CREDIT_ISSUE", not defined.|
|    - Row 345: `status` field value is "ON_HOLD", not defined.     |
|    [ Add new rule for this error type ]                           |
|                                                                   |
| 2. **Error Type: Invalid Amount Format (5 cases)**                |
|    - Row 28: `openAr` field value is "N/A", cannot be a number.   |
|    - Row 99: `openAr` field value is "", violates not-null rule.  |
|    - ... (click to expand 3 more cases)                           |
|    [ Add default value or cleanup rule for this field ]           |
|                                                                   |
+-------------------------------------------------------------------+
| [ Back to Edit Rules ]                       [ Export YAML File ] |
+-------------------------------------------------------------------+
```

**Comprehensive Scan:** The report clearly states that **all** uploaded sample data was processed.

**Quantitative Results:** Business users can clearly see the success rate and the number of failed records, giving them a macroscopic view of data quality and rule coverage.

**Error Categorization:** The system automatically categorizes errors, helping users quickly identify whether the issue is with logic (e.g., undefined status) or data quality (e.g., format error).

**Recommendation:** The interface provides quick actions like "Add new rule" or "Set default value" to guide users to resolve issues directly, forming an efficient "**Identify Problem -> Adjust Rule -> Re-validate**" loop.

---

### Appendix C: RAG Workflow

RAG (Retrieval-Augmented Generation) is widely known for its excellent performance in Q&A and conversational systems. In those scenarios, its workflow is:
*   **User Query:** "What is our company's expense reimbursement policy?"
*   **Retrieval:** The system finds the most relevant text snippets about "reimbursement policy" from an internal document library (knowledge base).
*   **Augmented Generation:** The LLM uses these retrieved snippets as core reference material to generate an accurate, well-founded answer, rather than fabricating one.

However, the core idea of RAG is much more general. Its essence is: before the LLM performs a task, it first retrieves the most relevant and authoritative information from an external knowledge source and injects this information as context into the prompt to "guide" the LLM to produce higher-quality results.

**Applying RAG to Our YAML Generation Project**

Now, let's apply this general idea to our project:

*   **What is the user's "question"?**
    *   The user's "question" is not a question but a task instruction. For example: "Here is a `customer_data.csv` file exported from a new ERP. Its columns are `['ID', 'Company', 'Addr', 'Status']`. Please generate the YAML mapping rules for `IR_CUSTOMER`."

*   **What "knowledge" needs to be retrieved?**
    *   The knowledge base doesn't store Q&A pairs or policy documents, but rather our past successful, business-verified YAML configuration snippets and related business rules. Each piece of knowledge can be a structured record containing:
        *   **Source Data Features:** ERP name, filename pattern, source column names (`['CUST_ID', 'NAME', 'ADDRESS']`).
        *   **Successful YAML Rule:** The corresponding `fieldMappingRules` code snippet.
        *   **Business Metadata:** "Business expert confirmed: in the status field, 1 represents ACTIVE, and 0 represents INACTIVE."

*   **How does RAG work?**
    1.  **Task Analysis (Query Formulation):** When a business user uploads the new `customer_data.csv`, the Agent first analyzes its column names `['ID', 'Company', 'Addr', 'Status']`.
    2.  **Retrieval:** The Agent uses these column names (and other features) as keywords for a semantic search in the "historical success cases" knowledge base. It might find:
        *   A historical case where the source columns were `['CUST_ID', 'C_NAME', 'Address', 'C_STATUS']`, which is very similar to the current task.
        *   Another case where the rule for the `Status` field was `V.customer.status == 1 ? "ACTIVE" : "INACTIVE"`, along with a business annotation.
    3.  **Augmented Generation:** The Agent constructs a highly enriched prompt:
       > “Task: Generate YAML rules for `['ID', 'Company', 'Addr', 'Status']`.
       >
       > Reference Material: Here is a very similar historical success case. Its source columns were `['CUST_ID', ...]` and the corresponding YAML rule was... Note especially that when handling the `Status` field, we often have a business rule like: 1 means ACTIVE... Please refer to these high-quality examples to complete the new task.”

Through this process, the LLM is no longer "blindly" guessing but creating based on past successful experiences, which will exponentially improve the accuracy and consistency of its output.

**Knowledge Base Creation and Maintenance**

*   **How to build the knowledge base?**
    *   We don't need to start from scratch. Every YAML configuration that is finally approved by a business user and successfully deployed is a high-quality piece of knowledge. We can develop a simple script to automatically parse these validated YAML files, along with their corresponding source data features and business annotations, and store them in a vector database (like Pinecone, ChromaDB, etc.). This process can be fully automated.

*   **Do we need to train the model?**
    *   Not at all. This is one of the biggest advantages of RAG. We enhance a general-purpose base model with an external knowledge base, rather than "stuffing" knowledge into the model through expensive and time-consuming fine-tuning. This allows our knowledge base to be updated in real-time—as soon as a new integration configuration is approved, it can immediately become a reference for the next task. The entire system becomes "smarter" with use.
