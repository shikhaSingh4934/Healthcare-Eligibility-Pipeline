# Multi-Partner Eligibility Ingestion Pipeline

## 📌 Project Overview
This project implements a production-grade ETL (Extract, Transform, Load) pipeline designed to ingest healthcare eligibility data from multiple third-party partners. The system is architected to handle inconsistent file formats (CSV, Pipe, Semicolon), malformed dates, and missing identifiers through a resilient, automated workflow.



---

## 🚀 Key Features

* **Configuration-Driven Design:** Support for new vendors is added via metadata updates in a JSON-like config—requiring zero changes to the core execution engine.
* **Fault-Tolerant Ingestion:** Implemented `try-except` logic within the ingestion loop. If one file is corrupt or unrecognized, the system logs the error and continues processing the remaining files to ensure maximum uptime.
* **Data Quality Gate (DLQ):** A validation layer that partitions clean records into a "Gold" dataset while diverting non-compliant records to a **Dead Letter Queue (DLQ)** for remediation.
* **Automated Standardization:**
    * **Names:** Standardized to Title Case.
    * **Phone Numbers:** Cleaned and formatted to `XXX-XXX-XXXX` via Regex.
    * **Dates:** Resilient coercion of multiple formats (MDY, YMD, DMY) to ISO-8601 (`YYYY-MM-DD`).
* **Lineage & Observability:** Every record is "stamped" with its source filename and timestamp. The system also maintains a **Processing Ledger** for file-level auditing.



---

## 🛠️ How to Run the Pipeline

1.  **Environment:** Ensure Python 3.8+ is installed with `pandas` and `numpy`.
2.  **Setup:** Run the `setup_demo_environment()` cell to generate the simulated landing zone and sample partner files.
3.  **Execute:** Run the `EligibilityEngine` class and the execution cells.
4.  **Review Audit:** Check **Cell 6** for the **File Processing Ledger** (System Health) and the **Record Quality Summary** (Data Health).

---

## ➕ How to Add a New Partner

The pipeline uses the **Strategy Design Pattern**. To add a new partner (e.g., "GlobalHealth"):

1.  **Update Config:** Add a new entry to the `config` dictionary with the partner's unique `file_identifier`, `delimiter`, `column_mapping`, and `date_format`.
2.  **Ingest:** Drop the file into the landing zone. The engine will automatically discover and process it based on the new metadata.

```python
"globalhealth": {
    "file_identifier": "global",
    "delimiter": ",",
    "partner_code": "GH_04",
    "column_mapping": {"Member_ID": "external_id", "FName": "first_name", "LNAME": "last_name", "DOB": "dob"},
    "date_format": "%Y-%m-%d"
}
```


## 🕵️ Operational Excellence (ABC)

The pipeline follows the industry-standard **Audit, Balance, and Control (ABC)** framework to ensure data integrity and system reliability:

* **Audit:** All file-level attempts (**Success**, **Failed**, or **Skipped**) are recorded in a persistent Ingestion Ledger with timestamps and error details.
* **Balance:** The system tracks record counts from the raw input stage through to the final partitioned output, ensuring no data is "lost" during transformation.
* **Control:** A robust validation layer acts as a safety gate. Malformed data is trapped in the **Dead Letter Queue (DLQ)**, preventing "poison pills" from causing downstream system failures.



---

## 🔮 Future Roadmap

To move this pipeline into a full enterprise production environment, the following enhancements are proposed:

1.  **Automated Alerting:** Integrate with **AWS SNS** or **Slack Webhooks** to notify on-call engineers immediately upon any file-level failure or high DLQ volume.
2.  **Schema Contracts:** Implement **Great Expectations** or **Pydantic** to enforce strict data contracts (checking for column types, null percentages, and value ranges) before transformation begins.
3.  **Partner Feedback Loop:** Automate the export of DLQ (Quarantine) reports back to partner SFTP folders, allowing for self-service data correction and reducing manual engineering support.**
