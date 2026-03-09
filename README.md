# DEGO Project - Team 12
## Team Members
- Afonso Joao
- Liane Kpocheme
- Rita Cunha
- Guilherme Carvalho
- Luca Benedict Illies
## Project Description

## 1. Data Engineering (Luca Illies & Afonso João)

# 1.1. Objective

The data engineering phase aimed to evaluate and improve the quality, consistency, and reliability of the raw credit application dataset before performing statistical and governance analyses. Because automated decision systems depend heavily on high-quality input data, identifying and resolving data quality issues is a critical step to ensure accurate and fair outcomes.

# 1.2. Data Quality Audit

A comprehensive data quality audit was conducted across multiple dimensions commonly used in data governance frameworks:
- Structural consistency → verifying that all records follow the same schema
- Completeness → identifying missing or blank values
- Consistency → detecting formatting inconsistencies across categorical fields
- Uniqueness → identifying duplicate records and identifiers
- Validity → ensuring values fall within reasonable and logically valid ranges

The dataset initially contained 502 credit application records with several structural and formatting inconsistencies.

| Issue                                           | Count |
| ----------------------------------------------- | ----- |
| Non-ISO date-of-birth formats                   | 157   |
| Inconsistent gender encodings                   | 114   |
| Annual income stored as text instead of numeric | 8     |
| Empty email fields                              | 7     |
| Duplicate application IDs                       | 4     |
| Duplicate SSN values                            | 6     |

Additional validation checks also identified formatting problems in fields such as ZIP codes, IP addresses, and Social Security Numbers, as well as inconsistencies in financial variables and optional attributes.

# 1.3. Data Cleaning and Standardization

To address the identified issues, several remediation procedures were implemented:
- Schema normalization to ensure all records follow a consistent structure
- Date standardization converting all birth dates to ISO format
- Categorical normalization standardizing gender encoding
- Data type correction converting financial variables to numeric format
- Duplicate removal based on application IDs and SSN values
- Field validation for identifiers such as email, ZIP code, IP address, and SSN
- Variable harmonization merging inconsistent financial attributes into a canonical income variable

Nested JSON records were flattened into a tabular structure to enable easier analysis and validation.

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

The bias analysis aimed to evaluate whether the credit approval decisions exhibit systematic disparities across demographic groups, particularly with respect to gender and age. Detecting such disparities is essential in financial decision systems, as discriminatory outcomes can violate fairness principles and regulatory requirements.
The analysis applied several statistical techniques to assess potential bias, including approval rate comparisons, disparate impact metrics, hypothesis testing, and proxy variable detection.

# 2.2. Gender Disparities in Credit Approval

Approval outcomes were analyzed by gender to evaluate whether male and female applicants experienced different approval rates.

| Gender | Approval Rate |
| ------ | ------------- |
| Male   | 66.0%         |
| Female | 50.6%         |

To quantify potential discrimination, the Disparate Impact (DI) ratio was calculated:
DI = 0.767

According to the widely used four-fifths rule, a DI value below 0.80 indicates potential discrimination. The observed value therefore suggests a significant disparity in approval outcomes.

A two-proportion z-test was conducted to evaluate statistical significance:
p-value = 0.0002

Since the p-value is well below the significance threshold (p < 0.05), the difference in approval rates is statistically significant, indicating that female applicants were significantly less likely to receive loan approval.

# 2.3. Age-Based Bias

The relationship between applicant age and approval decisions was examined using correlation analysis.

Correlation coefficient:
r = 0.122

This indicates a weak positive relationship, suggesting that older applicants are slightly more likely to be approved. Conversely, younger applicants—particularly those aged 18–30—experienced lower approval rates.

Although the effect size is small, the results indicate that age may influence credit approval outcomes.

# 2.4. Proxy Discrimination Analysis

The analysis also examined whether certain variables indirectly encode protected characteristics.

ZIP codes were analyzed as potential proxy variables, since geographic data can sometimes correlate strongly with demographic attributes.

ZIP codes were grouped into regional clusters based on their prefixes.

| Region      | Approval Rate |
| ----------- | ------------- |
| New York    | 64.3%         |
| Los Angeles | 51.8%         |

The calculated disparate impact ratio was:
DI = 0.805

Although slightly above the four-fifths threshold, the difference suggested possible indirect discrimination and warranted further analysis.

# 2.5. Gender Distribution by Region

A chi-square test was conducted to evaluate whether gender distribution differed significantly between regions.

| Region      | Female | Male  |
| ----------- | ------ | ----- |
| Los Angeles | 93.4%  | 6.6%  |
| New York    | 11.2%  | 88.8% |

Chi-square test results:

χ² = 318.25
p < 0.0001

These results indicate a very strong association between ZIP code and gender, meaning that geographic variables can function as proxy variables for protected attributes. If ZIP codes were used in a credit scoring model, the model could unintentionally discriminate based on gender.

# 2.6. Intersectional Bias

The analysis also examined whether bias becomes stronger when multiple demographic attributes interact.

A chi-square test was used to evaluate the interaction between gender and age group in credit approval decisions.

| Group          | Approval Rate |
| -------------- | ------------- |
| Female (18–30) | 34.1%         |
| Male (18–30)   | 50.0%         |

Test results:
χ² = 23.5764
p = 0.0014

The results show a statistically significant interaction effect, indicating that young female applicants experience the lowest approval rates.

# 2.7. Key Findings

The bias analysis revealed several important fairness concerns:
- Female applicants experienced significantly lower approval rates than male applicants.
- Younger applicants showed slightly lower approval probabilities.
- ZIP codes act as proxy variables for gender, introducing indirect discrimination risks.
- Intersectional bias was observed, with young female applicants being the most disadvantaged group.

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