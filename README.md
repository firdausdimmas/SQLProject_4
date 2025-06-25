# Impact Analysis of GoodThought NGO Initiatives 

### Project Overview

This project focuses on supporting GoodThought NGO, an organization dedicated to driving positive change through initiatives in education, healthcare, and sustainable development. The goal is to leverage data-driven insights to help GoodThought optimize its impact and resource allocation.

Using the GoodThought PostgreSQL database, which contains detailed records of assignments, funding, impacts, and donor activities from 2010 to 2023, the project aims to analyze trends, evaluate program effectiveness, and uncover opportunities for improvement. This hands-on data analysis empowers the NGO to make more informed decisions, maximize its reach, and strengthen its humanitarian efforts worldwide.

### Data Sources

GoodThought has provided comprehensive dataset includes:
- **`Assignments`:** Details about each project, including its name, duration (start and end dates), budget, geographical region, and the impact score.
- **`Donations`:** Records of financial contributions, linked to specific donors and assignments, highlighting how financial support is allocated and utilized.
- **`Donors`:** Information on individuals and organizations that fund GoodThought’s projects, including donor types.

Refer to the below ERD diagram for a visual representation of the relationships between these data tables:



### Exploratory Data Analysis (EDA)

EDA involved exploring the datasets to answer key questions, such as:
- Which assignments received the highest total donations overall?
- Which donor type contributes the most to each region or assignment category?
- Which assignments have the highest impact scores across different regions?


### Data Analysis

Including some interesting code/features worked with

```sql
-- highest_donation_assignments
WITH donation_details AS
(
SELECT d.assignment_id,
       ROUND(SUM(d.amount),2) AS rounded_total_donation_amount,
       dn.donor_type
FROM donations AS d
JOIN donors AS dn
  ON d.donor_id = dn.donor_id
GROUP BY d.assignment_id, dn.donor_type
)

SELECT a.assignment_name, 
       a.region, 
       dd.rounded_total_donation_amount, 
       dd.donor_type
FROM assignments AS a
JOIN donation_details AS dd
  ON a.assignment_id = dd.assignment_id
ORDER BY rounded_total_donation_amount DESC
LIMIT 5;
```

```sql
-- top_regional_impact_assignments
WITH donation_counts AS
(
SELECT assignment_id,
       COUNT(donation_id) AS num_total_donations
FROM donations
GROUP BY assignment_id	
),

ranked_assignments AS
(
SELECT a.assignment_name,
       a.region,
       a.impact_score,
       dc.num_total_donations,
       ROW_NUMBER() OVER (PARTITION BY a.region ORDER BY a.impact_score DESC) AS rank_in_region
FROM assignments AS a
JOIN donation_counts AS dc
  ON a.assignment_id = dc.assignment_id
WHERE dc.num_total_donations > 0
)

SELECT assignment_name,
       region,
       impact_score,
       num_total_donations
FROM ranked_assignments
WHERE rank_in_region = 1
ORDER BY region ASC;
```

### Results/Findings

The analysis results are summarized as follows:
- Assignment_3033 and Assignment_300 are among the top-funded assignments.
- Individual donors contribute heavily to East region, while Organization donors dominate in North and West region projects.
- Assignment_316 holds the highest impact in East region, while Assignment_3547 leads in South region.

### Recommendations

Based on the analysis, we recommend the following actions:
- Use the top-funded assignments as case studies or models to replicate best practices for underfunded projects.
- Focus on personalized, community-driven campaigns to engage individual donors in the East region.
- Prioritize corporate partnerships and institutional fundraising for the North and West regions where organization donors dominate.
- Promote high-impact assignments like Assignment_316 (East) and Assignment_3547 (South) in donor communications to showcase effectiveness.
- Apply the successful practices from these high-impact projects to other regions with lower performance.

