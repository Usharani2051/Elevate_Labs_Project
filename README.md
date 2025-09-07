# Elevate_Labs_Project

📊 E-commerce Return Rate Reduction Analysis

📌 Project Overview

Product returns are a major challenge in e-commerce, reducing profitability and impacting customer satisfaction.
This project analyzes return behavior using the Online Marketplace dataset (Kaggle) and builds a Power BI Dashboard with DAX measures to:

- Identify why customers return products.

- Measure return rates across categories, products, sellers, and regions.

- Develop a Return Risk Score to flag high-risk products.

- Provide actionable insights to reduce returns and recover lost revenue.

📂 Dataset

Source: Online Marketplace Dataset (Kaggle)

Files Used:

- Orders → OrderID, OrderDate, DeliveryDate, CustomerID, ProductID, SellerID, Price

- Returns → OrderID, ProductID, SellerID, ReturnDate

- Customers → CustomerID, CustomerName, State, City, ZipCodePrefix

- Products → ProductID, ProductName, ProductCategory

- Sellers → SellerID, SellerName

🏗️ Schema Design

The model follows a Star Schema:

Orders (Fact Table) → PK: OrderID | FK: CustomerID, ProductID, SellerID

Returns (Fact Table) → PK: (OrderID + ProductID + SellerID composite) | FK: OrderID

Customers (Dimension) → PK: CustomerID

Products (Dimension) → PK: ProductID

Sellers (Dimension) → PK: SellerID

📌 Relationships:

Orders → Customers (CustomerID)

Orders → Products (ProductID)

Orders → Sellers (SellerID)

Returns → Orders (OrderID)

📐 DAX Measures
-- Orders & Revenue
Total Orders = COUNTROWS(Orders)

Sales Revenue = SUM(Orders[Price])

Returned Orders = COUNTROWS(Returns)

Return Rate % = DIVIDE(
    CALCULATE(COUNTROWS(Returns)),
    CALCULATE(COUNTROWS(Orders)),
    0
)

Returned Revenue = SUMX(
    Returns,
    COALESCE(
        LOOKUPVALUE(
            Orders[Price],
            Orders[OrderID], Returns[OrderID],
            Orders[ProductID], Returns[ProductID],
            Orders[SellerID], Returns[SellerID]
        ),
        0
    )
)

Net Revenue = [Sales Revenue] - [Returned Revenue]

-- Time Metrics
Avg Lead Days = AVERAGEX(
    Orders,
    DATEDIFF(Orders[OrderDate], Orders[DeliveryDate], DAY)
)

Avg Return Days = AVERAGEX(
    Returns,
    VAR deliv = LOOKUPVALUE(
        Orders[DeliveryDate],
        Orders[OrderID], Returns[OrderID],
        Orders[ProductID], Returns[ProductID],
        Orders[SellerID], Returns[SellerID]
    )
    RETURN DATEDIFF(deliv, Returns[ReturnDate], DAY)
)

Previous Month Revenue = CALCULATE(
    [Sales Revenue],
    DATEADD(Orders[OrderDate], -1, MONTH)
)

Previous Month Orders = CALCULATE(
    [Total Orders],
    DATEADD(Orders[OrderDate], -1, MONTH)
)

Previous Month Return = CALCULATE(
    [Returned Orders],
    DATEADD(Orders[OrderDate], -1, MONTH)
)

-- Customer Metrics
Total Customers = DISTINCTCOUNT(Customer[CustomerID])

Customers with Returns = 
VAR ReturnOrderIDs = DISTINCT(Returns[OrderID])
RETURN CALCULATE(
    DISTINCTCOUNT(Orders[CustomerID]),
    TREATAS(ReturnOrderIDs, Orders[OrderID])
)

Avg Returns per Customer = DIVIDE(
    [Returned Orders],
    [Customers with Returns],
    0
)

Average Revenue per Customer = DIVIDE(
    [Sales Revenue],
    [Total Customers]
)

-- Risk Metrics
Product Return % = DIVIDE(
    CALCULATE(
        [Returned Orders],
        FILTER(Orders, Orders[ProductID] = MAX(Product[ProductID]))
    ),
    CALCULATE(
        [Total Orders],
        FILTER(Orders, Orders[ProductID] = MAX(Product[ProductID]))
    ),
    0
)

Seller Return % = DIVIDE(
    CALCULATE(
        [Returned Orders],
        FILTER(Orders, Orders[SellerID] = MAX(Seller[SellerID]))
    ),
    CALCULATE(
        [Total Orders],
        FILTER(Orders, Orders[SellerID] = MAX(Seller[SellerID]))
    ),
    0
)

Return Risk Score = 
VAR p = [Product Return %]
VAR s = [Seller Return %]
VAR l = DIVIDE([Avg Lead Days], CALCULATE([Avg Lead Days], ALL(Product)))
RETURN 100 * (0.6 * p + 0.3 * s + 0.1 * l)

Products with High Risk = COUNTROWS (
    FILTER (
        ADDCOLUMNS (
            VALUES ( Product[ProductID] ),
            "RiskScore", [Return Risk Score]
        ),
        [RiskScore] > 0.8
    )
)

⚙️ Implementation Steps
1. Import Data into Power BI

Open Power BI Desktop.

Go to Home → Get Data → Excel.

Import the following files:

Orders.xlsx

Returns.xlsx

Customers.xlsx

Products.xlsx

Sellers.xlsx

Load them into the Data Model.

2. Build Relationships (Schema)

Open Model View.

Connect:

Customers[CustomerID] → Orders[CustomerID]

Products[ProductID] → Orders[ProductID]

Sellers[SellerID] → Orders[SellerID]

Orders[OrderID] → Returns[OrderID]

Ensure 1-to-Many relationships are correctly set (1 on dimension side, * on fact side).

3. Create DAX Measures

Add a Measure Table in Power BI.

Paste all formulas listed above.

Organize KPIs into groups: Sales, Returns, Customers, Risk, Trends.

4. Build Dashboards

Create pages with different focus areas:

Overview Page: KPIs, Revenue Trend, Top 10 Products, Orders by Category.

Product & Category Page: Product return % trends, category-level return revenue, top return products.

Customer & Seller Page: Customer return behavior, seller return % and return days.

Risk Analysis Page: Risk score calculation, scatter of lead vs return days, top risk products.

Geographical Page: Map visualization of returns by region.

5. Apply Drill-Through & Filters

Add drill-through from Top 10 Products in Overview → Product & Category Page.

Apply filters for product category, seller, and geography.

6. Finalize Dashboard

Apply consistent color themes (green for positive, red for returns).

Add narrative text boxes with summaries.

Validate results with sample data before publishing.

📊 Dashboard Pages

Overview Page: KPIs, trend analysis, top 10 products.

Product & Category Page: Product-level return % and return revenue by category.

Customer & Seller Page: Top return customers & sellers, seller return % and return days.

Risk Analysis Page: Risk scores for products & categories, scatter insights.

Geographical Page: Map of return hotspots (Brazil, North America, Europe, India).

🔑 Key Insights

Top 10 products = 60% of sales, 40% of returns.

Electronics & Fashion categories = 2× higher return rate.

5% of customers = 25% of returns.

8% of sellers = 30% of returns.

$200K revenue lost due to returns.
