---
description: >-
  This page provides some examples of how to set up and query the Cloud Billing
  data exported to and stored in BigQuery.
---

# Billing Export

Cloud Management Platform can automatically export detailed Google Cloud billing data \(such as usage, cost estimates, and pricing data\) to a BigQuery table. Then you can access your Cloud Billing data from BigQuery for detailed analysis or use a tool like Looker to visualize your data.

### Data Availability

* Your billing export table has Cloud Billing data incurred from the date you've joined the DoiT International consolidated billing.
* BigQuery loads are ACID compliant, so if you query the BigQuery Cloud Billing export table while data is being loaded into it, you will not encounter partially loaded data.

### Project, Dataset, and Table Name

To query the Google Cloud Billing data in BigQuery, you need to specify the table name in the FROM clause. The table name is determined using three values: project.dataset.BQ\_table\_name.

* Project is always `doitintl-cmp-gcp-data`
* dataset is `gcp_billing_` concatenated with the name of your Google Billing account. Dashes in the billing account ID should be replaced with underscores. For example, if your Google Billing Account ID is 006C3F-3613C3-2A2169, the dataset name would be  `gcp_billing_006C3F_3613C3_2A2169`
* BQ\_table\_name is `gcp_billing`

### Table Configuration

Please review the table partitioning and clustering schema to allow query optimization

|  |  |
| :--- | :--- |
| Table type | Partitioned |
| Partitioned by | Day |
| Partitioned on field | export\_time |
| Partition filter | Required |
| Clustered by | project\_id, service\_description, sku\_description |

### Sample Queries

This query shows the invoice total for each month since Jan 2020, as a sum of regular costs, credits, adjustments, and rounding errors.

```text
SELECT
  DATE_TRUNC(DATE(usage_start_time, "America/Los_Angeles"), MONTH) AS month,
  (SUM(cost)) - (SUM((
      SELECT
        SUM(credit.amount)
      FROM
        UNNEST(credits) AS credit))) AS cost
FROM
  `doitintl-cmp-gcp-data.gcp_billing_006C3F_3613C3_2A2169.gcp_billing`
WHERE
  DATE(export_time) >= "2020-01-01"
GROUP BY
  month
```


