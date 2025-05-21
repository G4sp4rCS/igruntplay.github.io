# AppSec KPIs & Metrics

- In AppSec, time is often spent implementing controls and processes, but their impact is not adequately measured. Without clear metrics, it is difficult to demonstrate value, justify decisions, or identify opportunities for improvement.
- In this article, I want to share some of the key metrics I explored within the Security Champions program, based on real experience and applied with concrete data.

## Fundamental KPIs in AppSec

1. Number of Active Critical Vulnerabilities (NACV)

- This KPI evaluates the active risk to the business based on the number of critical vulnerabilities open at a given time. It helps answer questions like: Are we improving or accumulating technical debt? How many critical vulnerabilities do we have open? How many of them are in production? How many of them are in production and lack a remediation plan? etc.

1. Time to Remedy (TTR)

- Measures the average time between the creation of a vulnerability and its remediation. It reflects the team's efficiency in responding to findings and minimizing exposure time.
- This KPI is essential to understand the team's response speed to vulnerabilities.
- It helps identify bottlenecks in the remediation process and evaluate the effectiveness of training or code reviews. A low TTR indicates an agile and proactive team, while a high TTR may point to issues in vulnerability management or lack of resources.
- The equation to measure a low TTR is as follows: **TTR = (Resolved_Date - Creation_Date).days**
  - This is an approximation; the reality depends on the business logic of each company, but generally, a low TTR is expected.

1. Number of Vulnerabilities per Team (NVT)

- This indicator helps identify which teams or platforms concentrate vulnerabilities, allowing prioritization of training, reviews, or refactoring.
- A high NVT in a specific team may indicate the need for a deeper review of its code or processes. It may also point to a lack of training or resources in that team.
- This KPI is useful for identifying patterns in the occurrence of vulnerabilities and directing training or process improvement efforts toward the teams that need it most.
- In summary, which team needs more help and what is their level of maturity in terms of security.

1. Time to Remedy per Criticality (TTRC)

- Similar to TTR but broken down by severity level. Not all vulnerabilities should have the same response time. This KPI helps understand if the most critical ones are being addressed with the appropriate urgency.
- These metrics depend heavily on the company's and team's culture, but generally, critical vulnerabilities are expected to have a much lower TTR than low-severity ones.
  - Ideal thresholds:

    - Critical: ≤ 30 days
    - High/Major: ≤ 90 days
    - Normal/Medium: ≤ 120 days
    - Low: ≤ 180 days or even a year

### Typical Implementations

- All these KPIs can be calculated from queries with Jira, Confluence, or any ticketing tool used by the company.
- Dashboards can also be implemented in tools like Grafana or Kibana to visualize these KPIs in real-time.
- For me, the best way is to extract the information into a CSV so that information from different sources can be standardized into the same format.
- With `pandas`, we can combine, transform, and filter the data.
- We can create graphs with `matplotlib` to visualize the KPIs.

### Things to Keep in Mind

- These metrics **are useless if there is not enough data** or if the data is incomplete or poorly structured.
- Therefore, it is important to define a clear process for data collection and storage to ensure that the KPIs are accurate and useful.
- Generally, in security champions and SSDLC programs, a security policy is determined, and a clear process is defined for data collection and storage to ensure that the KPIs are accurate and useful.

## Example Implementation

- [Repository](https://github.com/G4sp4rCS/AppSec-KPI-metrics)

- I made a small implementation in script format for demonstration purposes.