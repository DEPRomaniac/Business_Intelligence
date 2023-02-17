# Business Intelligence
Implementing OLAP for Analysis and Visualization of adventureworks dataset 

## Introduction
This project includes the Analysis and Visualization of Adventureworks (Shopping website) dataset for BI.
It is also consisted of 3 parts: Integration (SSIS), Analysis (SSAS) and Visualization (Power BI).

## Part I: SSIS
- **Connections**: Each connection specifies the database we access. In this project, we use 2 connections in general. The first connection we implemented is the *AdventureWorks2008R2* database that the necessary data is read from it and then save it in the second connection that is
the *WHD* (data warehouse) for further analysis.
- **Tasks**: In this project, we used 5 data flow tasks and 5 execute sql tasks.
	 1. The data flow of the **Sales_Fact** task fills the fact table **sl.Sales_Fact_Dest** according to the star schema. This table includes the following information:
   
    ![sl.Sales_Fact_Dest table](https://github.com/DEPRomaniac/Business_Intelligence/blob/main/img/1.png)  
  This table was built using a SQL Command by joining *SalesOrderDetail* and *SalesOrderHeader* columns of the *Sales* schema from the Adventureworks database.

	 2. The dataflow task *Sales_Cutomer_Dim* fills one of the dim tables, i.e. *sl.Sales_Customer_Dest*, according to the star schema. The table includes the following information:
   
    ![sl.Sales_Fact_Dest table](https://github.com/DEPRomaniac/Business_Intelligence/blob/main/img/2.png)
    
	 This table is taken directly from the Customer table of the Sales schema of the Adventureworks database.
   
	 3. The data flow task *Sales_Product_dim* fills one of the dim tables, i.e. *sl.Product_Dest*, in a star schema model. The table includes the following information:
   
    ![sl.Sales_Fact_Dest table](https://github.com/DEPRomaniac/Business_Intelligence/blob/main/img/3.png)
    
	 This table was built using a SQL Command by joining *product*, *ProductCategory* and *ProductSubCategory* columns of the *Production* schema from the Adventureworks database.
   
	 4. The dataflow task *Sales_Territory_dim* fills one of the dim tables, i.e. *sl.Sales_Territory_Dest*, in a star schema model. The table includes the following information:
   
    ![sl.Sales_Fact_Dest table](https://github.com/DEPRomaniac/Business_Intelligence/blob/main/img/4.png)
    
	 This table is taken directly from the *Territory* table of the *Sales* schema of the Adventureworks database.
   
	 5. The dataflow task *Sales_Territory_dim* fills the dim table *sl.Sales_Territory_Dest* in a star schema model. The table includes the following information:
   
    ![sl.Sales_Fact_Dest table](https://github.com/DEPRomaniac/Business_Intelligence/blob/main/img/5.png)
    
	 This table is taken directly from the *SpecialOffer* table of the *Sales* schema of the Adventureworks database.

- **The order of execution of tasks**: since the execution of dataflow tasks is not dependent on each other and each of them fills a separate table, we do not need to sequence them, and with parallel execution, the process of completing the WHD is accelerated.	 
But on the other hand, for each execution, the contents of each table of WHD are truncated. So, before executing each data flow task, the corresponding execute sql task must be executed.

## Part II: SSAS
The SSAS file includes one Fact table and four Dim tables.
- **Relationship between modules**: In the star schema, the fact table is connected to the dim tables using its foreign keys.
  - *Sales_Fact_Dest* connects to *Sales_Territory_Dest* dim table via *TerritoryID* foreign key (Many-to-1 connection).
  - *Sales_Fact_Dest* connects to *Product_Dest* dim table via *ProductID* foreign key (Many-to-1 connection).
  - *Sales_Fact_Dest* connects to *Sales_Customer_Dest* dim table via *CustomerID* foreign key (Many-to-1 connection).
  - *Sales_Fact_Dest* connects to *Sales_SpecialOffer_Dest* dim table via *SpecialOfferID* foreign key (Many-to-1 connection).

- **Measures**: 5 measures have been made in this project.
	1. *NumOfItemsSold*: This measure gives us the total number of sold items in the fact table.\
	`DAX Code: SUM(Sales_Fact_Dest[OrderQty])`
	2. *NumOfSales*: This measure gives us the total number of sales processes completed in the fact table.\
	`DAX Code: DISTINCTCOUNT(Sales_Fact_Dest[SalesOrderID])`
	3. *TotalSaleDiscount*: This measure gives us the total amount of discount given to customers.\
	`DAX Code: SUM(Sales_Fact_Dest[SaleDiscount])`
	4. *TotalSaleRevenue*: This measure gives us the net income obtained from the sale in the output fact table.\
	`DAX Code: SUM(Sales_Fact_Dest[SaleRevenue])`
	5. *TotalSales*: This measure gives us the total sales amount obtained in the output fact table.\
	`DAX Code: SUM(Sales_Fact_Dest[LineTotal])`

To get the net income of each product, we added a new column called *GrossMargin* in the *Product_Dest* table:\
`[GrossMargin] = Product_Dest[ListPrice] - Product_Dest[StandardCost]`\
Then we added the *SalesRevenue* column to the sales (fact) table and by having the gross margin of each product, we can calculate the net income of each *SaleOrderDetail*.
We also added the *SaleDiscount* column in the sales table to calculate the *DiscountPct* related to each *SpecialOfferId*:\
`SaleDiscount=Sales_Fact_Dest[LineTotal]*RELATED(Sales_SpecialOffer_Dest[DiscountPct])`

## Part III: Power BI
The powerBI file has 5 visualizations.
On the first page, you can see two graphs. The image on the left side is related to the number of products sold in each region by separating them by color.
The image on the right shows the ratio of sales of each product in a region to the total number of products sold in that region.\
On the second page, you can see three graphs. The tachometer chart shows what the net profit is compared to the total sales. We considered two intervals 0-40M assuming low income, 40M-80M for medium income and 80M-TotalSales for high income.
   The right side gauge shows the small ratio of the total amount of discounts to the total net income.
The bar graph of the lower categories shows the number of items sold in each region by categorizing them.

