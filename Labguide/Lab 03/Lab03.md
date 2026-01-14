# Day 3

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image1.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image2.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image3.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image4.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image5.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image6.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image7.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image8.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image9.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image10.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image11.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image12.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image13.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image14.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image15.png)

![A screenshot of a computer program AI-generated content may be
incorrect.](./media/image16.png)

![A screenshot of a computer program AI-generated content may be
incorrect.](./media/image17.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image18.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image19.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image20.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image21.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image22.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image23.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image24.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image25.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image26.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image27.png)

## Knowledge of Kusto Query Language (KQL) to analyze the data.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image28.png)

### //Challenge 2.2 Increase the cost of top 3 most demanded products of all time by 15% each day

//top 3 products by demand

//summarize is similar to group by in sql

clickstream

| where event_type in ("purchase_completed", "add_to_cart",
"product_click", "checkout_initiated")

| summarize TotalDemand = count() by product_id

| top 3 by TotalDemand desc

| project TotalDemand, product_id

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image29.png)

//let\`s see the cost of those top 3 products

clickstream

| where event_type in ("purchase_completed", "add_to_cart",
"product_click", "checkout_initiated")

| summarize TotalDemand = count() by product_id

| top 3 by TotalDemand desc

| join products_silver on $left.product_id == $right.ProductId

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image30.png)

//let\`s create a copy of the table products_silver to test out the cost
increase

.create table product_copy (ProductId:string, ProductName:string,
SKU:string, Brand:string, Category:string, UnitCost:int)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image31.png)

//recommended for one time load, now our table looks like the product
table

.set-or-replace product_copy \<|

products_silver

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image32.png)

//let\`s also save the result of the TOP 3 products in another table,
because we\`ll need it in the update command, the let Delete or Append
only accepts the table name you are modifying

.create table Top3Products (ProductId:string, ProductName:string,
SKU:string, Brand:string, Category:string, UnitCost:int)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image33.png)

.set-or-replace Top3Products \<|

clickstream

| where event_type in ("purchase_completed", "add_to_cart",
"product_click", "checkout_initiated")

| summarize TotalDemand = count() by product_id

| top 3 by TotalDemand desc

| join products_silver on $left.product_id == $right.ProductId

| project ProductId, ProductName, SKU, Brand, Category, UnitCost;

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image34.png)

//to update we\`ll use a .update command that uses append and delete

.update table product_copy delete top3Products append
Top3ProductsWithUpdatedCost \<|

let top3Products = product_copy

| where ProductId in (Top3Products | project ProductId );

let Top3ProductsWithUpdatedCost = product_copy

| where ProductId in (Top3Products | project ProductId )

| extend UnitCost = toint(UnitCost + UnitCost \* 0.15);

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image35.png)

//great, now run again the query that finds out the Top 3 products, they
aren\`t the same right?

// that is why we need to have a pipe

//in the pipe we\`ll create 1 KQL acitivity where we\`ll update our top
3 products daily

.set-or-replace Top3Products \<|

clickstream

| where event_type in ("purchase_completed", "add_to_cart",
"product_click", "checkout_initiated") and ingestion_time() \> ago(1d)

| summarize TotalDemand = count() by product_id

| top 3 by TotalDemand desc

| join products_silver on $left.product_id == $right.ProductId

| project ProductId, ProductName, SKU, Brand, Category, UnitCost;

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image36.png)

//the 2nd KQL activity will update the products table with the cost
increased by 15% for the most demanded products

.update table product_copy delete top3Products append
Top3ProductsWithUpdatedCost \<|

let top3Products = product_copy

| where ProductId in (Top3Products | project ProductId );

let Top3ProductsWithUpdatedCost = product_copy

| where ProductId in (Top3Products | project ProductId )

| extend UnitCost = toint(UnitCost + UnitCost \* 0.15);

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image37.png)

## //Challenge 3.3 Create a Growth Opportunity Report

//in demand product

clickstream

| where event_type in ("purchase_completed", "add_to_cart",
"product_click", "checkout_initiated")

| summarize TotalDemand = count() by product_id

| top 1 by TotalDemand desc

| join products_silver on $left.product_id == $right.ProductId

| project ProductName

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image38.png)

//website with highest traffic

//you mean referral platform

clickstream

| where event_type in ("purchase_completed", "add_to_cart",
"product_click", "checkout_initiated") and isnotempty(referral_platform)

| summarize TotalDemand = count() by referral_platform

| top 1 by TotalDemand desc

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image39.png)

// product clicks over time

clickstream

| where event_type == "product_click"

| extend EventTime = todatetime(timestamp)

| extend ClickTime = bin(EventTime, 1h)

| summarize NumberOfClicks = count() by product_id, ClickTime

| order by ClickTime asc

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image40.png)

//product clicks over time

clickstream

| where event_type in ("product_click")

| extend ClickTime = bin(todatetime(timestamp), 1h)

| summarize NumberOfClicks = count() by product_id, ClickTime

| order by ClickTime asc

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image41.png)

//website traffic distribution

clickstream

| where isnotempty(referral_platform)

| summarize Websites = count() by referral_platform

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image42.png)

//device traffic distribution

clickstream

| where isnotempty(referral_source_type)

| summarize Traffic = count() by referral_source_type

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image43.png)

//forecast the temperature

manufacturing

| extend timestamp = todatetime(timestamp)

| where DefectProbability == "Anomaly"

// Creates a time series of average temperature values, grouped into
1-hour intervals over the past 1 day

| make-series avg_temp=avg(Temperature) on timestamp from ago(3d) to
now() step 6m

// Applies anomaly detection to the temperature series, 7h

| extend forecast = series_decompose_forecast(avg_temp,48)

| render timechart

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image44.png)

## Data Factory to create pipelines.

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image45.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image46.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image47.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image48.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image49.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image50.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image51.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image52.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image53.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image54.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image55.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image56.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image57.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image58.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image59.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image60.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image61.png)

![A screenshot of a schedule AI-generated content may be
incorrect.](./media/image62.png)

![A screenshot of a schedule AI-generated content may be
incorrect.](./media/image63.png)

![A screenshot of a schedule AI-generated content may be
incorrect.](./media/image64.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image65.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image66.png)

## Create a “Growth Opportunity” Report

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image67.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image68.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image69.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image70.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image71.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image72.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image73.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image74.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image75.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image76.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image77.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image78.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image79.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image80.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image81.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image82.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image83.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image84.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image85.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image86.png)

## Setup Data Agent with Real-Time Intelligence

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image87.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image88.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image89.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image90.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image91.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image92.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image93.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image94.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image95.png)
