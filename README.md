# DEGO Project - Team 12
## Team Members
- Afonso Joao - 72008
- Liane Kpocheme - 73516
- Rita Cunha - 56704
- Guilherme Carvalho - 70364
- Luca Benedict Illies - 70170
## Project Description
DEGO 2606 Group Project – Credit Application Governance Analysis

## 1. Data Engineering (Luca Illies & Afonso João)

# 1.1. Objective

The data engineering phase aimed to assess and improve the quality of NovaCred's raw credit application data before it was used for bias analysis and privacy governance assessment. This involved identifying structural inconsistencies, missing values, invalid formats, duplicate identifiers, and data type mismatches, then applying cleaning and standardization steps to produce a reliable, analysis-ready dataset. Because automated credit decisions depend on accurate and consistent input data, this step was essential to support trustworthy downstream analysis and reduce the risk of misleading or unfair conclusions.

# 1.2. Data Quality Audit

A comprehensive data quality audit was conducted across multiple dimensions commonly used in data governance frameworks:
- Structural consistency → verifying that all records followed the same schema
- Completeness → identifying missing, null, or blank values
- Consistency → detecting formatting inconsistencies across categorical fields
- Uniqueness → identifying duplicate records and repeated identifiers
- Validity → ensuring values fall within reasonable and logically valid ranges
- Accuracy → checking whether values are plausible in real-world context and coherent with related fields (e.g. credit history exceeding what is possible given the applicant’s age)

The dataset initially contained 502 credit application records. The following issues were identified: 

| Issue                                           | Count |
| ----------------------------------------------- | ----- |
| Non-ISO date-of-birth formats                   | 157   |
| Inconsistent gender encodings                   | 114   |
| Annual income stored as string                  | 8     |
| Empty email fields                              | 7     |
| Duplicate SSN values                            | 6     |
| Duplicate application IDs                       | 4     |
| Invalid email format                            | 4     |
| Negative credit history                         | 2     |
| Empty string ZIP code                           | 1     |
| Debt-to-income ratio above 1.0                  | 1     |
| Negative savings balance                        | 1     |

Additionally, a schema drift check identified that annual income was recorded under two different field names (`annual_income` and `annual_salary`) across 5 records. These were merged into a single canonical field prior to analysis.
The accuracy dimension was assessed by checking whether decision fields were internally coherent (approved records carried interest rates and loan amounts, while rejected records carried rejection reasons) and whether credit history months exceeded each applicant’s legal age. No issues were detected in either check.

# 1.3. Data Cleaning and Standardization

To address the identified issues, several remediation procedures were implemented:
- Schema normalization to ensure all records follow a consistent structure
- Date standardization converting all birth dates to ISO format (YYYY-MM-DD)
- Categorical normalization mapping all gender encodings to Male, Female, or Unknown
- Variable harmonization merging the two income fields (annual_income and annual_salary) into a single canonical field
- Data type correction converting annual income to numeric format
- Duplicate removal based on repeated application IDs, removing redundant records while preserving one canonical entry per application
- Duplicate SSN flagging for manual review, as identical SSNs across different applicants may indicate fraud or data integration errors requiring human judgment
- Field validation to detect malformed or empty identifiers, with invalid values recoded as missing (NaN) for downstream handling
- Validity remediation setting impossible or anomalous numeric values (negative credit history months, debt-to-income ratio above 1.0, negative savings balance) to NaN

Nested JSON records were flattened into a tabular structure to enable easier analysis and validation. Schema normalization was also applied to ensure all records shared a consistent top-level structure.

# 1.4. Final Dataset

After applying cleaning and validation procedures, the final dataset contained:
| -------------------------- | -------- |
| Records                    | 500      |
| Attributes                 | 25       |
| Duplicate entries          | 0        |
| Structural inconsistencies | resolved |

The resulting dataset provides a clean, standardized, and validated foundation for the subsequent bias analysis and privacy governance assessment.

---

## 2. Data Science (Liane Kpocheme)

# 2.1. Objective

The bias analysis notebook focuses on detecting potential bias an discrimination in NovaCred's historical credit application data across four analytical dimensions: gender disparate impact, age-based bias, proxy discrimination, and interaction effects. All findings are supported by appropriate statistical tests (z-test, chi-square, ANOVA, & Demographic Parity Difference).

# 2.2. Gender Disparities in Credit Approval

Approval outcomes were analyzed by gender to evaluate whether male and female applicants experienced different approval rates.

| Gender | Approval Rate |
| ------ | ------------- |
| Male   | 66.0%         |
| Female | 50.6%         |

Female applicants are approved significantly less often than male
applicants

# 2.3. Age-Based Bias

The relationship between applicant age and approval decisions was examined using correlation analysis.

The 18-30 age group faces a severe approval disadvantage (41.5%)

# 2.4. Proxy Discrimination Analysis

The analysis also examined whether certain variables indirectly encode protected characteristics.

| Region      | Approval Rate |
| ----------- | ------------- |
| New York    | 64.3%         |
| Los Angeles | 51.8%         |

Female and male applicants show virtually identical financial profiles, ruling out legitimate financial differences as an explanation for the gender gap.
While the 18-30 age group shows systematically lower income, shorter credit history, and lower savings, these financial differences reflect realistic life-stage patterns rather than proxy discrimination.

# 2.5. Gender Distribution by Region

A chi-square test was conducted to evaluate whether gender distribution differed significantly between regions.

| Region      | Female | Male  |
| ----------- | ------ | ----- |
| Los Angeles | 93.4%  | 6.6%  |
| New York    | 11.2%  | 88.8% |

ZIP code is confirmed as a gender proxy, with Los Angeles being 93.4% female and New York 88.8% male.

# 2.6. Intersectional Bias

The analysis also examined whether bias becomes stronger when multiple demographic attributes interact.

A chi-square test was used to evaluate the interaction between gender and age group in credit approval decisions.

| Group          | Approval Rate |
| -------------- | ------------- |
| Female (18–30) | 34.1%         |
| Male (18–30)   | 50.0%         |


The results show a statistically significant interaction effect, indicating that young female applicants (18-30) face the most severe compounding disadvantage of any subgroup, with an approval rate of only 34.1%.

# 2.7. Key Findings

The bias analysis revealed several important fairness concerns:
- Remove ZIP code as a model input to eliminate confirmed gender proxy discrimination
- Audit the approval decision process for gender bias, as financial profiles alone cannot explain the observed gap
- Introduce alternative creditworthiness metrics for the 18-30 age group to avoid systematic exclusion from credit access
- Implement continuous bias monitoring with automatic alerts when DI ratios approach the 0.8 threshold

These findings highlight the importance of implementing fairness monitoring and bias mitigation strategies in automated credit decision systems.

---

## 3. Data Privacy / Governance (Rita Cunha)

# 3.1. Objective

This stage evaluates privacy risks within the credit application dataset and implements governance mechanisms to ensure responsible data handling and compliance with GDPR and principles aligned with the EU AI Act.

Because the dataset contains sensitive personal information used in financial decision-making, improper handling could lead to identity exposure, unauthorized processing, or regulatory violations. The objective was therefore to identify sensitive attributes and implement technical controls to mitigate privacy risks.

# 3.2. Identification of Sensitive Data

The dataset contains several categories of personally identifiable information (PII):

| Data Category          | Attributes                                        |
| ---------------------- | ------------------------------------------------- |
| **Direct Identifiers** | Name, Social Security Number (SSN), Email address |
| **Online Identifiers** | IP address                                        |
| **Quasi-identifiers**  | Date of birth, Gender, ZIP code                   |

The combination of these attributes increases the risk of re-identification, requiring privacy protection measures before analysis or storage.

# 3.3. Privacy Protection Measures

# 3.3.1. Pseudonymization

Personal names were replaced with synthetic identities generated using the Faker library, allowing the dataset to remain analytically useful while removing direct identifiers.

# 3.3.2. Hashing

Sensitive attributes were protected using SHA-256 hashing, including:
- Social Security Number
- Email address
- IP address
- Date of birth

Hashing ensures values cannot be reversed while preserving uniqueness for analytical tasks.

 # 3.4. Governance Controls

Several governance mechanisms were implemented using MongoDB:

- Data Retention Policy
A TTL index automatically deletes records after five years, supporting GDPR storage limitation requirements.

- Consent Management
A dedicated consent collection records user consent status, timestamps, and processing purposes.

- Audit Logging
All credit decision updates are logged with document ID, modified fields, previous values, new values, timestamps, and operation type.

- Schema Validation
A MongoDB validation schema enforces document structure and prevents invalid records from being stored.

# 3.5. Regulatory Alignment

The governance framework supports compliance with key regulatory requirements,

# 3.5.1. GDPR
- data minimization through pseudonymization and hashing
- storage limitation through automated retention policies
- accountability via audit logging
- lawful processing through consent tracking

# 3.5.2. EU AI Act
- Credit scoring systems are considered High-Risk AI Systems, requiring:
- dataset quality monitoring
- traceability of automated decisions
- risk mitigation measures
- human oversight mechanisms

# 3.6. Key Outcome

The implemented privacy and governance framework significantly reduces re-identification risks, improves transparency and accountability, and enables safer use of sensitive data while maintaining compliance with modern data protection regulations.

---

## 4. Product Leader Recomendations (Guilherme Carvalho)

# 4.1. Objective

Based on the findings from the data engineering, bias analysis, and privacy governance assessments, this section outlines strategic recommendations to improve the reliability, fairness, and regulatory compliance of NovaCred’s credit decision system.

The goal is to translate technical insights into practical governance actions that product leadership can implement.

# 4.2. Key Recommendations

| Area             | Action                                                                           |
| ---------------- | -------------------------------------------------------------------------------- |
| **Data Quality** | Add automated checks for duplicates, missing values, and inconsistent formats.   |
| **Fairness**     | Monitor approval outcomes using fairness metrics (e.g., Disparate Impact Ratio). |
| **Model Inputs** | Review variables like ZIP code that may act as bias proxies.                     |
| **Privacy**      | Protect sensitive data through pseudonymization and hashing.                     |
| **Governance**   | Maintain audit logs, retention policies, and human oversight.                    |

# 4.3. Strategic Impact

Implementing these recommendations would allow NovaCred to:
- improve data reliability and decision accuracy
- reduce the risk of algorithmic discrimination
- strengthen data privacy protections
- demonstrate compliance with GDPR and EU AI Act requirements

Together, these measures help ensure that automated credit systems are transparent, fair, and accountable.

# 4.4. Conclusion

The analysis conducted across data engineering, bias detection, and privacy governance highlights the importance of strong oversight in automated credit decision systems. Data quality issues, fairness concerns, and privacy risks can significantly impact the reliability and legitimacy of algorithmic decision-making. By implementing improved data validation processes, continuous fairness monitoring, and robust privacy protections, NovaCred can strengthen both the accuracy and accountability of its credit approval system. These measures not only mitigate operational and regulatory risks but also support the development of transparent, fair, and compliant financial technologies.

## Structure
- `data/` – Dataset files
- `notebooks/` – Jupyter analysis notebooks
- `src/` – Python source code
- `reports/` – Final deliverables
