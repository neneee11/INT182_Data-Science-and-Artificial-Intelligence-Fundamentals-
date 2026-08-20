# Retail Intelligence: Pricing & Demand Signals Dataset

A retail pricing and demand dataset designed for analyzing how price changes, promotions, seasonality, and inventory levels influence customer demand.

---

## What this dataset contains

This dataset includes structured product-level demand signals across multiple regions, channels, categories, and time periods.

It is designed for:

- pricing analytics
- demand forecasting
- promotion analysis
- inventory optimization
- revenue analysis
- retail AI experimentation

---

## Files Included

### retail_pricing_demand_100k.csv

This file contains retail pricing and demand records with product, pricing, promotion, inventory, and sales-related attributes.

Each row represents a product-level demand observation for a specific date, region, and channel.

---

## Column Descriptions

date: Date of the pricing and demand observation  
product_id: Unique identifier for the product  
category: Product category  
brand: Product brand  
region: Geographic market or region  
channel: Sales channel such as web, mobile, or app  
season: Seasonal period associated with the date  
base_price: Original product price before promotion or discount  
current_price: Effective selling price after discount  
price_change_pct: Percentage change from base price  
discount_pct: Discount percentage applied to the product  
promotion_type: Type of promotion applied  
units_sold: Number of units sold during the observation period  
revenue: Total revenue generated from units sold  
inventory_level: Available inventory after sales activity  
stockout_flag: Indicates whether inventory is critically low  
demand_index: Normalized demand score for comparison across products  

---

## High-signal analysis ideas

Analysis | What to explore
--- | ---
Price sensitivity | How do discounts affect units sold?
Promotion impact | Which promotions drive the highest revenue?
Demand forecasting | Can future units sold be predicted?
Inventory risk | Which products are likely to stock out?
Seasonality | Which categories perform better by season?
Channel performance | Which channel generates stronger demand?

---

## Recommended use cases

- Demand forecasting models
- Price elasticity analysis
- Promotion optimization
- Inventory planning
- Retail analytics dashboards
- Revenue prediction
- Retail AI portfolio projects

---

## Data quality & compliance

This dataset contains structured retail pricing and demand signals for analytics and machine learning experimentation.
No personally identifiable information (PII) is included.

---

## Author

Noopur Bhatt
