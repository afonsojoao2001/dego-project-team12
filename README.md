# DEGO Project - Team 12
## Team Members
- Afonso Joao
- Liane Kpocheme
- Rita Cunha
- Guilherme Carvalho
- Luca Benedict Illies
## Project Description

## 1. Data Engineering (Luca Illies & Afonso João)

The data engineering phase focused on preparing the raw credit application dataset for analysis. The dataset initially contained 502 records and several data quality issues that required remediation.
Data quality checks identified multiple inconsistencies, including non-standard date formats, inconsistent gender coding, incorrect data types, and duplicate records.

Key findings included:
- 157 records with non-ISO date-of-birth formats
- 114 records with inconsistent gender coding
- 8 cases where annual income was stored as a string instead of a numeric value
- 7 empty email fields
- 4 duplicate application IDs
- 6 duplicate SSN values

After applying remediation procedures and validation checks, the cleaned dataset contained **500 records and 25 columns**, which was then exported as a cleaned dataset for further analysis.

---

## 2. Data Science (Liane Kpocheme)

The data science analysis focused on detecting potential bias in credit approval decisions.

Statistical tests were conducted to evaluate whether demographic characteristics influenced credit outcomes. In particular, a chi-square test was used to examine the interaction between gender and age group in loan approval decisions.

The results showed:
- Chi-square statistic: 23.5764  
- P-value: 0.0014  

Since the p-value is below the significance threshold (p < 0.05), the analysis indicates a statistically significant interaction between gender and age group in approval decisions.
This suggests the potential presence of bias patterns within the credit decision process, highlighting the importance of fairness monitoring in automated lending systems.

---

## 3. Data Privacy / Governance (Rita Cunha)

The data privacy and governance analysis focused on identifying sensitive personal data within the dataset and implementing governance mechanisms to mitigate privacy risks.

Several governance gaps were initially identified, including:
- Sensitive personally identifiable information stored without protection
- Absence of consent tracking mechanisms
- Missing data retention policies
- Lack of audit trails for credit decisions
- Missing documentation for human oversight in automated decisions

To address these issues, the project implemented multiple governance controls, including pseudonymization techniques, schema validation rules, and audit logging mechanisms.
The final governance report shows that all identified privacy and governance risks were successfully mitigated.

---

## 4. Final Recommendations (Guilherme Carvalho)

Based on the analysis, several governance recommendations are proposed:
1. Implement automated data quality validation to detect inconsistencies, duplicates, and incorrect data formats early in the pipeline.
2. Introduce fairness monitoring mechanisms to regularly evaluate credit approval decisions and detect potential discrimination.
3. Apply strong privacy protection mechanisms such as pseudonymization, hashing, and secure storage of sensitive personal data.
4. Establish governance frameworks that include audit trails, data retention policies, and human oversight for automated decision systems.
5. Ensure continuous compliance with regulatory frameworks such as GDPR and the EU AI Act when deploying automated credit decision systems.

## Structure
- `data/` – Dataset files
- `notebooks/` – Jupyter analysis notebooks
- `src/` – Python source code
- `reports/` – Final deliverables