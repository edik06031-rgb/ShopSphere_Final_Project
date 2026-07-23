# ShopSphere_Final_Project

ShopSphere Global Marketplace Analysis

A complete analytics workflow for the ShopSphere online marketplace: from raw data to an executive dashboard and statistical analysis of an A/B experiment. Tools used: SQL (SQLite), Tableau Public, and statistical reasoning (Simpson’s paradox).
Data
	•	Source: educational dataset ShopSphere: Global Marketplace Analytics (final project for a Data Analytics course).
	•	Five CSV tables covering the years 2022–2024: customers (3,000), products (250), orders (~12,300), order_items (~26,000), and marketing (216).
	•	Products belong to seven categories (Electronics, Clothing, Beauty, Home & Kitchen, Sports, Books, Toys) across five global regions (North America, Europe, Southeast Asia, Latin America, Middle East).
	•	The database was built from CSV files using SQLite (sqliteonline.com or a local SQLite installation). Tables are linked via customer_id, order_id, and product_id.
Project Objective
Conduct a complete analytical assessment for ShopSphere management to evaluate marketing channel performance, identify the most valuable customers, measure the true profitability of product categories (beyond revenue alone), analyze regional trends, and evaluate the results of a checkout A/B experiment. All findings are supported by SQL analysis and an executive Tableau dashboard.

Project Steps
1. SQL Analysis (Block 1)
Five SQL queries using JOINs, aggregations, Common Table Expressions (CTEs), and subqueries:
	•	Revenue, number of orders, and average order value by region and year.
	•	Top 10 customers by total spending.
	•	Revenue, profit margin, and return rate by product category.
	•	Customers whose spending exceeds the average (subquery).
	•	Marketing channel ROI.
The results were exported as CSV files for Tableau.

2. Tableau Visualizations (Block 2)
Six interactive visualizations:
	•	Monthly revenue seasonality.
	•	Marketing budget vs. ROI (dual-axis chart).
	•	Sales volume vs. profitability by category (bubble chart).
	•	Regional performance over time (multi-line chart).
	•	Customer revenue contribution (Pareto chart).
	•	Creative analysis: Region × Acquisition Channel heatmap based on Customer Lifetime Value (LTV).

3. Executive Dashboard (Block 3)
A CEO dashboard including:
	•	KPI cards:
	◦	Total Revenue
	◦	Number of Orders
	◦	Average Order Value
	◦	Return Rate
	•	Four key visualizations.
	•	Two interactive filters (Region and Year).
	•	Consistent visual design.

4. Business Case Analysis (Block 4)
Three strategic business analyses with recommendations:
	•	Marketing budget optimization based on ROI and customer LTV by acquisition channel.
	•	Real profitability of product categories ("high-volume illusion" of Electronics vs. the "hidden gem" Beauty category).
	•	Discount customer behavior and retention strategy for the top 5% of customers.

5. Statistical Analysis: A/B Test (Block 5)
Evaluation of the new checkout process:
	•	Comparison of Variant A and Variant B across all orders.
	•	Separate analysis for new and returning customers.
	•	Identification of Simpson’s paradox, where the overall improvement of Variant B masks opposite effects within customer segments.
	•	Business interpretation and recommendations for management.

Dashboard
Tableau Public: (Insert the public dashboard link after publishing.)
Dashboard screenshots are available in the tableau/ folder.
Key Findings

	•	Paid Search receives approximately 45% of the total marketing budget, yet delivers the lowest ROI (1.0) and the lowest customer LTV ($648) among all acquisition channels. Influencer Marketing and Organic Search outperform Paid Search on both metrics.
	
	•	Electronics generates approximately 60% of total company revenue, but also has the lowest profit margin (12%) and the highest return rate (16%). This is a classic example of the "high-volume illusion." In contrast, Beauty produces relatively low revenue but achieves a 55% profit margin with a very low return rate, making it a "hidden gem."
	
	•	The top 5% of customers (150 out of 3,000) generate approximately 35% of total revenue. Customer concentration is significant but does not follow the traditional 80/20 Pareto principle. In this dataset, approximately 36–40% of customers generate 80% of revenue.
	
	•	The checkout A/B test reveals Simpson’s paradox. Overall, Variant B appears slightly better (+2% average order value). However, the segmented analysis shows a strong positive effect for new customers (+22.8%) and a negative effect for returning customers (-3.2%). Therefore, Variant B should be deployed selectively rather than universally.
	
Recommendations

	•	Reallocate 15–20 percentage points of the marketing budget from Paid Search to Influencer Marketing and Email Marketing, while retaining Paid Search primarily for retargeting campaigns and seasonal peaks (November–December).
	
	•	Increase investment in the Beauty category as an undervalued high-margin segment, while reviewing the Electronics strategy by investigating return reasons and optimizing the product assortment.
	
	•	Replace broad discount campaigns with a loyalty program focused on the top 5% of customers and the broader high-value customer segment.
	
	•	Deploy the new checkout (Variant B) only for new customers, while retaining the existing version for returning customers until the cause of the negative effect is identified.
	
Repository Structure

data/       Original CSV datasets (customers, products, orders, order_items, marketing)

sql/        SQL queries and exported CSV files for Tableau

tableau/    Dashboard and visualization screenshots

report/     SQL explanations, answers to the 13 business questions,
            step-by-step Tableau guide
			
README.md
Reproducing the Project
	1	Import the five CSV files into SQLite (for example, using sqliteonline.com) while preserving the relationships via customer_id, order_id, and product_id.
	2	Execute the SQL queries from sql/queries.sql (Block 1) and export the results as CSV files.
	3	Connect both the original datasets and the SQL output files to Tableau Public and build the six visualizations (Block 2).
	4	Assemble the executive dashboard with KPI cards, key charts, and interactive filters (Block 3).
	5	Answer the 13 business questions (Blocks 4–5) using the SQL results and Tableau visualizations.
	6	Publish the dashboard on Tableau Public and add the public link to this README.
