# Part B: Business Case Analysis
## B1. Problem Formulation

### (a) ML Problem Formulation
- **Target Variable:** Number of items sold per store per month  
- **Input Features:**
  - Promotion type (Flat Discount, BOGO, etc.)
  - Store attributes (size, location type: urban/semi-urban/rural)
  - Monthly footfall
  - Local competition density
  - Customer demographics
  - Calendar features (month, weekends, festivals)
  - Historical sales trends

- **Type of ML Problem:**  
  This is a **supervised learning regression problem** since the goal is to predict a continuous numeric value (items sold).  

- **Justification:**  
  We are learning from past data where inputs (store conditions and promotions) are known and the output (items sold) is observed. The model predicts a numeric outcome making regression appropriate.

### (b) Why Use Items Sold Instead of Revenue
Revenue can be misleading because:
- Different promotions affect pricing (example- discounts reduce revenue per item)
- High revenue does not always mean high customer engagement
- Promotions like BOGO increases volume but may reduce revenue per item

**Items sold (sales volume)** is more reliable because:
- It directly reflects customer's response to promotions
- It removes pricing distortions
- It aligns with the goal of maximizing product movement

**Broader Principle:**  
The target variable should directly align with the business objective.Choosing the wrong metric can lead to misleading models and poor decisions.

### (c)
Instead of one global model we could use a:
- **Hierarchical (multi-level) model** OR  
- **Cluster-based models (segment stores)**

**Justification:**
- Stores differ by geography and customer behaviour
- A single model may average out important differences which could have given us a business advantage in that area.
- Segmenting stores (like urban vs rural) allows more tailored predictions.

## B2. Data and EDA Strategy
### (a)
- **Tables:**
  - Transactions
  - Store attributes
  - Promotion details
  - Calendar

- **Joining Approach:**
  - Base Table: Transactions
  - Join **transactions** with **store attributes** using `store_id`
  - Join **transactions** with **promotion details** using `promotion_id` *(date is not needed here)*
  - Join **transactions** with **calendar** using `date`
- **Grain of Final Dataset:**  
 One row = **one store × one month**
 
  So each row represents: One specific store in one specific month and contains:Total items sold,Promotion used,Store features,Calendar info.
- **Aggregations:**
  - Total items sold per store per month
  - Total transactions
  - Average basket size
  - Promotion applied during the month
  - Monthly footfall (if available)
### (b)
1. **Sales Distribution by Promotion Type (Bar Chart)**
   - Compare average items sold across promotions  
   - Helps identify which promotions perform best

2. **Time Series Plot of Sales**
   - Look for seasonality (festivals, holidays)  
   - Helps create time-based features

3. **Store-wise Performance (Box Plot)**
   - Compare sales across stores  
   - Detect high/low-performing stores - segmentation

4. **Correlation Heatmap**
   - Identify relationships between features and target  
   - Helps select important variables

5. **Sales vs Footfall Scatter Plot**
   - Understand conversion behavior  
   - May lead to derived features like conversion rate

### (c)
**Impact:**
- Model may learn to predict “no promotion effect” most of the time.
- This could lead to poor performance on promotional cases.

**Solutions:**
- Use **sampling techniques** (oversample promotion cases)
- Add **promotion indicator features**
- Train separate models for:
  - Promotion vs no-promotion scenarios


## B3. Model Evaluation and Deployment

### (a)
- **Train-Test Split:**
  - Use **Temporal**split (time based split where example : train on first 2.5 years, test on last 6 months)
  
- **Random split is inappropriate here beacuse**
  - It Breaks time order
  - Causes data leakage (future influencing past)

- **Evaluation Metrics:**
  - **MAE (Mean Absolute Error):** Average absolute difference between predicted and actual sales. Easy to interpret average error.
  - **RMSE (Root Mean Squared Error):** Penalizes large errors more.It is useful when large errors are risky
  - **MAPE (Mean Absolute Percentage Error):** shows error as a percentage.So it is useful for business comparison across stores.

### (b)
Use **feature importance and interpretation tools**:
- Check which features influenced predictions for each month
- Example:
  - December -
     - Festivals
     - High demand.
      
      Customers already willing to spend so Loyalty Points Bonus encourages repeat purchases.

    Customers value rewards during heavy shopping periods.
  - March 
    - Lower demand
    - Customers more price-sensitive.

    Flat Discount directly reduces price which
helps attract more buyers.

 **Communication to marketing team:**
- Show simple charts explaining key drivers (Feature importance chart,
Month-wise promotion performance)
- Translate model output into business insights (eg.“holiday season favors loyalty incentives”)

### (c)
### 1. Model Saving
- After training, save the model using formats like `.pkl` or `.joblib`
- Also save preprocessing steps (encoding, scaling, feature transformations)
- This ensures the same pipeline is reused during prediction

---

### 2. Monthly Data Preparation
At the start of each month:
- Collect latest data:
  - Store attributes
  - Calendar data (festivals, weekends)
  - Planned promotions
- Transform data into **store × month format**
- Apply the same preprocessing steps used during training
  - Encoding categorical variables (e.g. promotion type)
  - Aggregating metrics (e.g. past sales, footfall)

---

### 3. Generating Recommendations
- For each store, create inputs for all possible promotion types
- Feed these inputs into the saved model
- Predict expected items sold for each promotion
- Select the promotion with the highest predicted sales.

---

### 4. Automation Pipeline
- Build a pipeline/script that:
  - Pulls new monthly data automatically
  - Preprocesses it
  - Runs the model
  - Outputs promotion recommendations for all 50 stores
- Schedule this process to run at the start of every month

---

### 5. Monitoring Model Performance
After deployment:
- Compare **predicted vs actual sales**
- Track metrics such as:
  - MAE (average error in items sold)
  - MAPE (percentage error)
- Monitor trends over time (is error increasing?)

---

### 6. Detecting Performance Degradation
Watch for:
- **Data Drift:** Changes in input patterns (e.g. customer behavior, competition)
- **Concept Drift:** Change in relationship between promotions and sales
- Sudden increase in error metrics

---

### 7. Retraining Strategy
- Retrain the model when:
  - Performance drops beyond a threshold
  - Significant business changes occur
- Or retrain periodically (e.g., every 6–12 months)
- Use latest data to keep the model updated

---

### 8. Feedback Loop
- Collect results from actual promotion outcomes
- Compare recommendations vs real performance
- Use this feedback to improve future models