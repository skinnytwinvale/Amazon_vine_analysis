# Overview of the analysis of the Vine program:
## Background
Since your work with Jennifer on the SellBy project was so successful, you’ve been tasked with another, larger project: analyzing Amazon reviews written by members of the paid Amazon Vine program. The Amazon Vine program is a service that allows manufacturers and publishers to receive reviews for their products. Companies like SellBy pay a small fee to Amazon and provide products to Amazon Vine members, who are then required to publish a review.

In this project, you’ll have access to approximately 50 datasets. Each one contains reviews of a specific product, from clothing apparel to wireless products. You’ll need to pick one of these datasets and use PySpark to perform the ETL process to extract the dataset, transform the data, connect to an AWS RDS instance, and load the transformed data into pgAdmin. Next, you’ll use PySpark, Pandas, or SQL to determine if there is any bias toward favorable reviews from Vine members in your dataset. Then, you’ll write a summary of the analysis for Jennifer to submit to the SellBy stakeholders.

## What You're Creating
This new assignment consists of two technical analysis deliverables and a written report. You will submit the following:

Deliverable 1: Perform ETL on Amazon Product Reviews
Deliverable 2: Determine Bias of Vine Reviews
Deliverable 3: A Written Report on the Analysis

# Results
The dataset is so big we need to filter the recorded 3 million reviews to focus on the reviews that are considered to be more helpful. We need to filter is to count the Total Votes equal or greater than 20 and set the percent of Helpful Votes to Total Votes equal or greater than 50%.

**1. How many Vine reviews and non-Vine reviews were there?**

![VineNonVineTotal](https://github.com/amylio/Amazon_Vine_Analysis/blob/main/Images/VineNonVineTotal.png)

Paid Vine Program

-33 total reviews
-15 5-star reviews
-45.5% of vine reviews were 5-star
-Unpaid reviews

-45,388 total reviews
-23,733 5-star reviews
-52.3% of unpaid reviews were 5-star

**2. How many Vine reviews were 5 stars? How many non-Vine reviews were 5 stars?**

- Vine members gave 454 out of 1,080 reviews a 5 star rating.
- Non-Vine members gave 23,034 out of 49,659 reviews a 5 star rating.

**3. What percentage of Vine reviews were 5 stars? What percentage of non-Vine reviews were 5 stars?**

- 42% of the reviews for Vine members were rated 5 stars.
- 46.4% of the reviews for Non-Vine members were rated 5 stars.

# Summary
Based on the results of my data, Non-vine members the 5 star ratings were about 10 percent less then the Vine members were, so its shows that the members were also no being bias because of the difference in ratings. The members take the ratings critically based on their results. To support my assumption we could include all of the data instead of filtering it to a percentage of helpful vs. total votes as we did for this analysis. In addition, running the same analysis using datasets from different product categories can provide us with the whole picture of whether reviews made by Vine members are bias.

