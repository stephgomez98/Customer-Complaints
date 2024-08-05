# Customer-Complaints
Data Walkthrough/Column Description
Company: Name of the company involved in the complaint.
Company public response: Public statement by the company regarding the complaint.
Company response to consumer: Specific response or action taken by the company for the consumer.
Complaint ID: Unique identifier for each complaint.
Consumer consent provided?: Indicates if the consumer consented to share their complaint details (yes/no).
Consumer disputed?: Indicates if the consumer disputed the company's response (yes/no).
Date Received: Date the complaint was received.
Date Submitted: Date the complaint was submitted by the consumer.
Issue: Main category of the problem reported.
Product: Type of product or service involved.
State: U.S. state of the consumer.
Sub-issue: More detailed categorization of the issue.
Sub-product: More detailed categorization of the product.
Submitted via: Medium through which the complaint was submitted.
Tags: Specific tags related to the complaint.
Timely response?: Indicates if the company responded in a timely manner (yes/no).
ZIP code: Consumer's postal ZIP code.
All Complaints (Selected): Indicates if the complaint is part of a selected subset.
Number of Complaints: Number of complaints related to the consumer, company, or product.
Target: Target resolution date or goal for handling the complaint.
Time to Receipt: Time taken from submission to receipt of the complaint.

Row Count: 75514
Complaint ID: Primary Key

70-80 percent is spend understanding the data and then cleaning it

---
Data Cleaning
EXEC sp_help complaints_data

You can check the data type to know what you need to change and for memory usagae
Through inspection can see that the date_received and date submitted return negaive numbers so will makw those dates null:
--counting how many incorrect values are in there
select count(complaint_id) as countcomplaint
from (
select complaint_id,date_received,
case when date_submitted>date_received then NULL
else date_submitted
end as date_submitted
from complaints_data) as subquery
 where date_submitted is NULL
 --counting how many incorrect values are in there
--updating the original database
UPDATE complaints_data
SET 
    date_submitted = CASE 
                        WHEN date_submitted > date_received THEN NULL
                        ELSE date_submitted
                     END,
    date_received = CASE 
                        WHEN date_submitted > date_received THEN NULL
                        ELSE date_received
                    END;

Now that we have corrected errors we can proceed


The following are the requirements client/manager is asking for:
LEVEL 1
KPI Requirements
Average Response Time

Description: The average time taken by the company to respond to consumer complaints.
Calculation: Calculate the difference between Date Received and Date Submitted for each complaint, then average these values.
Column Used: Date Received, Date Submitted

'''sql
select avg(DATEDIFF(day,date_submitted, date_received)) as days_difference_avg
from complaints_data
'''sql


Percentage of Timely Responses
Description: The percentage of complaints that received a response within a designated time frame (e.g., within 30 days).
Calculation: (Number of timely responses / Total number of complaints) * 100
Column Used: Timely response?
Timely response will be that of the same day or one day later
'''sql
WITH TotalComplaints AS (
    SELECT COUNT(*) AS total_count 
    FROM complaints_data
),
OneDayDifference AS (
    SELECT COUNT(*) AS one_day_count 
    FROM complaints_data 
      WHERE DATEDIFF(day, date_submitted, date_received) IN (0, 1)
)
SELECT 
    CAST(one_day_count AS FLOAT) / CAST(total_count AS FLOAT) * 100 AS percentage
FROM 
    TotalComplaints, OneDayDifference;
  '''sql


Consumer Dispute Rate

Description: The percentage of complaints where the consumer disputed the company's response.
Calculation: (Number of disputed complaints / Total number of complaints) * 100
Column Used: Consumer disputed?
'''sql
WITH TotalComplaintsDisputed AS (
    SELECT COUNT(*) AS total_dispute 
    FROM complaints_data 
    WHERE consumer_disputed = 'Yes'
),
Total AS (
    SELECT COUNT(*) AS total 
    FROM complaints_data
)
SELECT 
    CAST(total_dispute AS FLOAT) / CAST(total AS FLOAT) * 100 AS percentage
FROM 
    TotalComplaintsDisputed, Total;

'''sql


Complaint Volume by Product

Description: The number of complaints received for each product category.
Calculation: Count of complaints grouped by Product.
Column Used: Product
'''sql
select product, count(product) as count from complaints_data group by product order by count desc
'''sql


Complaint Volume by State

Description: The number of complaints received from each state.
Calculation: Count of complaints grouped by State.
Column Used: State
Resolution Rate
'''sql
select state, count(product) as count from complaints_data group by state order by count desc
'''sql


Description: The percentage of complaints that received a company response to the consumer.
Calculation: (Number of complaints with a company response / Total number of complaints) * 100
Column Used: Company response to consumer
Complaint Escalation Rate

Description: The percentage of complaints that were escalated to a public response.
Calculation: (Number of complaints with a public response / Total number of complaints) * 100
Column Used: Company public response
Average Time to Receipt

Description: The average time taken for complaints to be processed from the date submitted to the date received.
Calculation: Calculate the difference between Date Submitted and Date Received for each complaint, then average these values.
Column Used: Date Received, Date Submitted
Top Issues by Product

Description: The most common issues reported for each product category.
Calculation: Count of issues grouped by Product and Issue.
Column Used: Product, Issue
Complaints by Submission Method

Description: The distribution of complaints based on the submission method (e.g., Email, Phone, Website).
Calculation: Count of complaints grouped by Submitted via.
Column Used: Submitted via
Complaint Resolution Time

Description: The average time taken to resolve complaints.
Calculation: Calculate the difference between Date Received and the date the response was provided, then average these values.
Column Used: Date Received, Company response to consumer
Complaint Trends Over Time

Description: Analysis of complaint volumes over different time periods (e.g., monthly, quarterly).
Calculation: Count of complaints grouped by the time period (month, quarter, year).
Column Used: Date Received



Complex KPI Requirements
Sentiment Analysis of Company Responses

Description: Measure the sentiment (positive, neutral, negative) of the company’s responses to complaints.
Calculation: Use natural language processing (NLP) techniques to analyze the text in Company response to consumer and categorize the sentiment.
Column Used: Company response to consumer
Consumer Satisfaction Score

Description: Score derived from consumer feedback or follow-up surveys after complaint resolution.
Calculation: Aggregate scores from follow-up surveys on a scale (e.g., 1 to 5) and calculate the average satisfaction score.
Column Used: Feedback or survey data (requires additional data collection)
Root Cause Analysis

Description: Identify the underlying causes of recurring issues.
Calculation: Use machine learning clustering algorithms to group similar complaints based on Issue, Sub-issue, Product, and Sub-product. Analyze the clusters to identify common root causes.
Column Used: Issue, Sub-issue, Product, Sub-product
Resolution Effectiveness

Description: Measure the effectiveness of the company's responses in resolving complaints without further escalation.
Calculation: (Number of complaints resolved without dispute / Total number of complaints) * 100
Column Used: Consumer disputed?, Company response to consumer
Complaint Recurrence Rate

Description: The percentage of consumers who file multiple complaints.
Calculation: (Number of consumers with more than one complaint / Total number of consumers) * 100
Column Used: Consumer ID (requires unique consumer identifier)
Time to First Response by Issue Type

Description: Average time taken to respond to complaints categorized by issue type.
Calculation: Calculate the average difference between Date Received and Date Submitted, grouped by Issue.
Column Used: Date Received, Date Submitted, Issue
Resolution Time by Product and State

Description: Average time taken to resolve complaints, broken down by product type and consumer state.
Calculation: Calculate the average difference between Date Received and the date when the company provided a response, grouped by Product and State.
Column Used: Date Received, Company response to consumer, Product, State
Impact Analysis of Public Responses

Description: Analyze the impact of making company responses public on the resolution rate and consumer satisfaction.
Calculation: Compare resolution rates and consumer satisfaction scores before and after implementing public responses.
Column Used: Company public response, Company response to consumer, Feedback or survey data (requires additional data collection)
Consumer Demographic Analysis

Description: Analyze complaints based on consumer demographics such as age, gender, income level (requires additional demographic data).
Calculation: Aggregate and analyze complaints based on demographic data to identify trends and patterns.
Column Used: Demographic data (requires additional data collection)
Predictive Analysis for Complaint Volume

Description: Predict future complaint volumes based on historical data.
Calculation: Use time series forecasting models to predict future complaint volumes based on historical Date Received and Product.
Column Used: Date Received, Product
Resolution Quality Score

Description: Score that measures the quality of resolution based on multiple factors such as timeliness, consumer satisfaction, and recurrence rate.
Calculation: Create a weighted score combining metrics like timeliness, consumer satisfaction score, and recurrence rate.
Column Used: Timely response?, Feedback or survey data, Consumer disputed?
Cost Analysis of Complaint Handling

Description: Estimate the cost incurred by the company to handle and resolve complaints.
Calculation: Calculate based on factors such as labor hours, resources used, and any compensation provided to consumers.
Column Used: Requires additional data on resource usage and costs
Advanced Visualization Examples
Sentiment Heatmap: Visualize the sentiment analysis results by product or issue.
Sankey Diagram: Show the flow of complaints from submission to resolution and any escalations.
Predictive Dashboard: Integrate time series forecasts with current complaint trends.
Geographical Impact Map: Combine complaint volume, resolution time, and recurrence rate by state or region.
Resolution Quality Index Dashboard: Display the composite resolution quality score with drill-down capabilities.
