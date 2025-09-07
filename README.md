# Elevate_Labs_Project

📊 **E-commerce Return Rate Reduction Analysis**

📌**Project Overview**

Product returns are a major challenge in e-commerce, reducing profitability and impacting customer satisfaction.
This project analyzes return behavior using the Online Marketplace dataset (Kaggle) and builds a Power BI Dashboard with DAX measures to:

- Identify why customers return products.

- Measure return rates across categories, products, sellers, and regions.

- Develop a Return Risk Score to flag high-risk products.

- Provide actionable insights to reduce returns and recover lost revenue.

📂 **Dataset**

Source: Online Marketplace Dataset (Kaggle)

Files Used:

- Orders → OrderID, OrderDate, DeliveryDate, CustomerID, ProductID, SellerID, Price

- Returns → OrderID, ProductID, SellerID, ReturnDate

- Customers → CustomerID, CustomerName, State, City, ZipCodePrefix

- Products → ProductID, ProductName, ProductCategory

- Sellers → SellerID, SellerName

🏗️ **Schema Design**

The model follows a Star Schema:

- Orders (Fact Table) → FK: OrderID | FK: CustomerID, ProductID, SellerID

- Returns (Fact Table) → PK: OrderID 

- Customers (Dimension) → PK: CustomerID

- Products (Dimension) → PK: ProductID

- Sellers (Dimension) → PK: SellerID

📌 **Relationships:**

- Orders → Customers (CustomerID)

- Orders → Products (ProductID)

- Orders → Sellers (SellerID)

- Returns → Orders (OrderID)

📐 **DAX Measures**

-- **Orders & Revenue**

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


-- **Time Metrics**

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


-- **Customer Metrics**

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

-- **Risk Metrics**
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



⚙️ **Implementation Steps**

**1. Import Data into Power BI**

- Open Power BI Desktop.

- Go to Home → Get Data → Excel.

- Import the following files:

    - Orders

    - Returns

    - Customers

   -  Products

    - Sellers

- Load them into the Data Model.

**2. Build Relationships (Schema)**

- Open Model View.

- Connect:

    - Customers[CustomerID] → Orders[CustomerID]

    - Products[ProductID] → Orders[ProductID]

    - Sellers[SellerID] → Orders[SellerID]

    - Orders[OrderID] → Returns[OrderID]

- Ensure 1-to-Many relationships are correctly set (1 on dimension side, * on fact side).

**3. Create DAX Measures**

- Add a Measure Table in Power BI.

- Paste all formulas listed above.

- Organize KPIs into groups: Sales, Returns, Customers, Risk, Trends.

**4. Build Dashboards**

Create pages with different focus areas:

- Overview Page: KPIs, Revenue Trend, Top 10 Products, Orders by Category.

- Product & Category Page: Product return % trends, category-level return revenue, top return products.

- Customer & Seller Page: Customer return behavior, seller return % and return days.

- Risk Analysis Page: Risk score calculation, scatter of lead vs return days, top risk products.

- Geographical Page: Map visualization of returns by region.

**5. Apply Drill-Through & Filters**

- Add drill-through from Top 10 Products in Overview → Product & Category Page.

- Apply filters for product category, seller, and geography.

**6. Finalize Dashboard**

- Apply consistent color themes (green for positive, red for returns).

- Add narrative text boxes with summaries.

- Validate results with sample data before publishing.

📊 **Dashboard Pages**

🔹** 1. Overview Page**

**Purpose:** Provides a high-level summary of the e-commerce performance with key KPIs, revenue trends, category performance, and top 10 products.

**Key Components:**

KPI Cards:

- Revenue: $7.76M

- Profit (Net Revenue): $6.21M

- Orders: 50K

- Return Orders: 10K

- Avg Lead Days: 6.01

- Return Rate %: 20%

- Revenue Trend Line Chart: Shows daily revenue movement from April to May 2025.

- Insights: Revenue increased 11.79% between Apr 2 – May 21, 2025. A sharp growth of $21K (+14.32%) occurred in mid-May.

- Orders by Category (Bar Chart): Bags & Accessories lead with 8,346 orders, which is 40% higher than Clothing (5,959).

- Monthly Revenue Comparison: Cards showing current vs previous month, with % change.

- Top 10 Products (Table): Lists order count, return orders, return rate, and revenue for the top-selling products.

    - Example: Paper Ream 891 had 205 orders, 47 returns, and a 23% return rate despite contributing $32K revenue.

📌 **Takeaway:** Overall sales are strong, but return rates remain high in top-selling products, especially in categories like Bags & Accessories and Clothing.

🔹 **2. Product & Category Analysis Page**

**Purpose:** Deep dive into product-level and category-level return performance.

**Key Components:**

- Metric Selector (Toggle): Allows switching between different metrics: Orders, Profit, Return %, Returns, Revenue.

- Return % Trend (Line Chart): Shows product return % over time.

- Insights: Return % decreased 15.88% overall but spiked 4.68% in May.

- Product-Level Analysis (Bar Chart):

    - Top return products include Paper Ream 891, Jeans 195, Belt 146.

    - These products show 20–24% return rates, making them key problem items.

- Return Revenue by Category (Treemap):

    - Bags & Accessories → highest return revenue.

    - Followed by Beauty & Personal Care, Sports & Fitness.

    Summary Box: Automatically generated narrative (trend up/down, steepest change, % variations).

📌 **Takeaway:** Certain products (e.g., Paper Ream, Jeans) drive a disproportionate share of returns. Categories like Bags & Accessories and Beauty & Personal Care are highest contributors to lost revenue.

🔹**3. Customer & Seller Analysis Page**

**Purpose:** Identifies which customers and sellers are most responsible for returns, and analyzes return behavior.

**Key Components:**

KPI Cards:

- Total Customers: 2,050

- Avg Revenue/Customer: $40.16

- Avg Returns/Customer: 1.58

- Customers with Returns: 6,373 (repeat returners included).

- Customer Average Returns (Line Chart): Daily fluctuation of average returns per customer.

- Top 10 Return Customers (Table):

    - Example: Michael Larsen → 9 orders, 6 returns (67% return rate).

    - Ashley Moore → 16 orders, 6 returns ($2,359 revenue).

- Returns by Customer (Bar Chart): Highlights frequent returners visually.

- Returns by Seller (Bar Chart):

    - Johnson PLC leads with 113 returned orders.

    - Other sellers also contribute high returns.

Seller Metrics:

- Seller Return % = 19.41%

- Avg Return Days = 8.06

📌 **Takeaway:** A small group of repeat customers (5–10 people) drive many returns. Sellers like Johnson PLC have high return volumes, suggesting product quality/service issues.

🔹**4. Risk Score Analysis Page**

**Purpose:** Predicts and highlights high-risk products and categories most likely to be returned.

**Key Components:**

KPI Cards:

- Return Risk Score = 25.82 (weighted measure of return risk).

- Most Risky Product = Toaster 341.

- Top 10 Risk Products (Bar Chart): Toaster 341, Marker 435, Laptop Bag 643 have the highest scores.

- Scatter Plot (Avg Lead Days vs Avg Return Days):

    - X-axis = Delivery Time

    - Y-axis = Return Time

    - Bubble size = return volume

    - Example: Office Supplies → highest Avg Return Days (8.30).

- Risk by Category (Table):

    - Clothing Risk Score = 34.29

    - Bags & Accessories Risk Score = 33.65

- Summary Box:

    - Sports & Fitness → longest delivery times (6.06 days).

    - Toaster 341 → risk score 9.48% higher than lowest product.

📌** Takeaway:** Products with long delivery times (lead days) and short return windows (return days) pose the highest risk. Certain categories (Clothing, Bags & Accessories) are priority areas for intervention.

🔹**5. Geographical Analysis Page**

**Purpose:** Visualizes returns across regions to detect geographic hotspots.

**Key Components:**

- Returns by Geography (Map):

    - Largest return volumes in Brazil (South America).

    - Other hotspots: US, Canada, France, Spain, Germany, UK, and India.

    - Smaller clusters: Africa, Atlantic islands.

- Filter Options: Can filter map by category, seller, or customer segment.

📌 **Takeaway:** Returns are heavily concentrated in Brazil. Region-specific policies, localized quality checks, and closer return centers could help reduce costs.

🔑 **Key Insights**

- Top 10 products = 60% of sales, 40% of returns.

- Electronics & Fashion categories = 2× higher return rate.

- 5% of customers = 25% of returns.

- 8% of sellers = 30% of returns.

- $200K revenue lost due to returns.

  PowerBI Dashboard Project Link - https://app.powerbi.com/links/Sk5mLiR36F?ctid=46805c18-2ee4-4b75-970e-c9ce28de3cbf&pbi_source=linkShare
