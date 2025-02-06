# Codebasics Challenge4 Provide Insights to Management in Consumer Goods Domain Using SQL (Atliq Hardware)
This repository contains the solution to the SQL challenge for Atliq Hardwares, a leading computer hardware producer. The task involved answering 10 ad-hoc business requests using SQL queries based on the provided dataset and metadata. The goal was to generate valuable insights for top-level management and create an engaging presentation showcasing these insights. The solution includes SQL scripts for each request asked in the **ad-hoc-requests.pdf**

## 10 Ad-hoc Requests
1. Provide the list of markets in which customer "Atliq Exclusive" operates its business in the APAC region. <br>

    Select distinct(market) as Market From dim_customer Where customer = "Atliq Exclusive" AND region = "APAC"
   
2. What is the percentage of unique product increase in 2021 vs. 2020? The final output contains these fields, unique_products_2020 unique_products_2021 percentage_chg <br>

    with cte as(
select 
     count(distinct(case when fiscal_year=2020 then product_code end)) as unique_products_2020,
  count(distinct(case when fiscal_year=2021 then product_code end)) as unique_products_2021 
from fact_sales_monthly)

    select unique_products_2020, unique_products_2021, concat(round(((unique_products_2021 - unique_products_2020) /unique_products_2020)*100,2),'%') as percentage_chg from cte;

3. Provide a report with all the unique product counts for each segment and sort them in descending order of product counts. The final output contains 2 fields, segment product_count <br>

   Select segment, count(product_code) as product_count From dim_product Group By segment Order By count(product_code) Desc

4. Follow-up: Which segment had the most increase in unique products in 2021 vs 2020? The final output contains these fields, segment product_count_2020 product_count_2021 difference <br>

   with cte as 
(
select 
       d.segment,
       count(distinct(case when fs.fiscal_year=2020 then d.product_code end)) as product_count_2020,
       count(distinct(case when fs.fiscal_year=2021 then d.product_code end)) as product_count_2021
        from dim_product d 
        join fact_sales_monthly fs on d.product_code=fs.product_code
        group by d.segment
        )
        select 
               segment, 
               product_count_2020, 
               product_count_2021, 
               (product_count_2021-product_count_2020) as difference
        from cte
        order by difference desc;
   
5. Get the products that have the highest and lowest manufacturing costs. The final output should contain these fields product_code product manufacturing_cost <br>

    select fm.product_code, d.product, round(fm.manufacturing_cost,2) as manufacturing_cost from fact_manufacturing_cost fm 
    join dim_product as d on fm.product_code = d.product_code 
    where manufacturing_cost = (select min(manufacturing_cost) from fact_manufacturing_cost) or
       manufacturing_cost = (select max(manufacturing_cost) from fact_manufacturing_cost);
   
6. Generate a report which contains the top 5 customers who received an average high pre_invoice_discount_pct for the fiscal year 2021 and in the Indian market. The final output contains these fields, customer_code customer average_discount_percentage <br>

    select f.customer_code, c.customer, avg(pre_invoice_discount_pct) as average From fact_pre_invoice_deductions f
    join dim_customer c On f.customer_code = c.customer_code
    Where fiscal_year = 2021 AND c.market='India' Group By f.customer_code, c.customer Order By average Desc Limit 5;
   
7. Get the complete report of the Gross sales amount for the customer “Atliq Exclusive” for each month. This analysis helps to get an idea of low and high-performing months and take strategic decisions. The final report contains these columns: Month Year Gross sales Amount <br>

    Select year(f.date) as Years, Monthname(f.date) as Months,  
      concat(round(sum(f.sold_quantity*p.gross_price)/1000000,2),'m') as 'Gross sales Amount' 
      From fact_sales_monthly f 
      join fact_gross_price p on f.product_code = p.product_code
      join dim_customer c on f.customer_code = c.customer_code 
      Where c.customer = "Atliq Exclusive" 
      Group By Months, Years;
   
8. In which quarter of 2020, got the maximum total_sold_quantity? The final output contains these fields sorted by the total_sold_quantity, Quarter total_sold_quantity <br>

    SELECT CONCAT('Q', QUARTER(date)) AS Quarter, SUM(sold_quantity) AS total_sold_quantityFROM fact_sales_monthly
    WHERE YEAR(date) = 2020  GROUP BY Quarter ORDER BY total_sold_quantity DESC;
   
9. Which channel helped to bring more gross sales in the fiscal year 2021 and the percentage of contribution? The final output contains these fields, channel gross_sales_mln percentage <br>

      with cte as 
       (
          select 
                  c.channel, 
                  sum(p.gross_price*f.sold_quantity)/1000000 as gross_sales_mln  
          from fact_sales_monthly f 
          inner join fact_gross_price p on f.product_code=p.product_code
          inner join dim_customer c on f.customer_code=c.customer_code 
          where f.fiscal_year=2021
          group by channel
       ) , 
      cte2 as 
       (
          select 
                 sum(p.gross_price*f.sold_quantity)/1000000 as total  
       from fact_sales_monthly f 
          inner join fact_gross_price p on f.product_code=p.product_code
       inner join dim_customer c on f.customer_code=c.customer_code 
          where f.fiscal_year=2021
       )
      select 
             cte.channel, 
             round(cte.gross_sales_mln,2) as gross_sales_mln, 
             concat(round((cte.gross_sales_mln/cte2.total)*100,2),'%') as percentage
      from cte 
      cross join cte2
      order by percentage desc;
   
10. Get the Top 3 products in each division that have a high total_sold_quantity in the fiscal_year 2021? The final output contains these fields, division product_code product total_sold_quantity rank_order <br>

    with cte as(
      Select p.division, p.product_code, p.product, sum(sold_quantity) as total_sold_quantity,
      dense_rank() OVER (partition by p.division order by sum(s.sold_quantity) Desc) as rank_order
      From dim_product p join fact_sales_monthly s on p.product_code = s.product_code
      where s.fiscal_year=2021
      group by  p.division, p.product_code, p.product)
      
      Select * From cte Where rank_order <4;

## Conclusion
The tasks presented require in-depth analysis of sales, product growth, customer behavior, and performance metrics for Atliq Exclusive. Python notebook file with SQL magic has been uploaded. It allows you to run SQL queries directly within the notebook, making the process of fetching and analyzing data streamlined.

**Credit:-**
This project was created as part of the CodeBasics Resume Project Challenge, specifically Challenge #4: Provide Insights to Management in the Consumer Goods Domain. For more details, visit CodeBasics Challenge.
[CodeBasics Challenge](https://codebasics.io/challenge/codebasics-resume-project-challenge)

**Note:-** 
Download the Python Notebook File 
[click here to download](https://github.com/Prian04/codebasics_challenge4_atliq_sql/blob/main/atliq_hardware_db.ipynb)
