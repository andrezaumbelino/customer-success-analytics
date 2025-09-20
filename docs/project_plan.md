# Customer Success Analytics — Project Plan 

> **Overall Objective:**  
The **Customer Success Analytics Project** aims to leverage data and predictive modeling to improve **workload balance, transparency, and decision-making** in Customer Success operations.  
The project is divided into **three phases**, each addressing a specific need:  

1. **Phase 1 — Automatic Distribution of New Clients**  
   - Build predictive models to estimate client interactions (emails + activities).  
   - Use these predictions to calculate workload scores and implement an automatic distribution system that ensures fair assignment of new clients across CSs.  

2. **Phase 2 — Power BI Dashboard**  
   - Design a multi-page dashboard to visualize client predictions, portfolio load, and distribution logic.  
   - Provide managers and CSs with actionable KPIs and insights into portfolio health, client complexity, and routing outcomes.  

3. **Phase 3 — Monthly CS Performance Analysis**  
   - Develop a dataset and reporting framework to track monthly CS performance (emails, activities, and deals concluded).  
   - Deliver a Power BI view that enables continuous monitoring of productivity and identification of top/bottom performers.  

Together, these three phases create a **complete analytics solution**:  
- **Prediction** (future workload),  
- **Visualization** (portfolio dashboards), and  
- **Performance monitoring** (monthly analysis).  

This end-to-end approach ensures **data-driven portfolio management**, **equitable distribution of clients**, and **clear visibility into team performance**.


-------------------------------------------------------------------


# Phase 1: Automatic Distribution of New Clients

> **Objective:** Predict the total number of interactions per client (emails + deal activities) and use this prediction to balance CS portfolios and automatically distribute new clients based on a scoring system.

---

## 1. Overview
- **Business problem:** Uneven workload among CSs and lack of objective criteria for new client distribution.  
- **Proposed solution:**
  1. Build a unified dataset of clients and interactions.  
  2. Train a count regression model to predict interactions.  
  3. Calculate **load scores for each client and CS portfolio.  
  4. Implement routing rules for new clients based on availability and skills.  

**Interaction definition:**  
`total_interactions = emails_count + activities_count`

---

## 2. Fields & Sources (Client Information)
### Table deals_base
- `client_id`  
- `visa_type`  
- `contract_category`  
- `country`  
- `n_applicants`  
- `start_date`  
- `end_date`  
- `status` (open, completed, suspended)  
- `stage_closed`   
- `cs_owner`  
- `client_language`  

### Table `interactions`
- `client_id`  
- `emails_count`  
- `activities_count`  
- `total_interactions` = `emails_count + activities_count`  

### Table `cs_skills`
- `cs_name`  
- `languages`  
- `product_scopes`  

---

## 3. Filtering Rules
- **Include:** completed clients (for training labels).  
- **Exclude:** suspended clients.  
- **Flag:** deals closed at *New Clients* stage.  

---

## 4. Interaction Label
- **Label `y`:** total interactions.
- **Formula:** `y = emails_count + activities_count`.  

---

## 5. Dataset Preparation
- **Deduplication:** unique `client_id`, remove duplicate emails/activities.  
- **Missing Data:** 
- **Outliers:** 
- **Scaling/Encoding:** 

---

## 6. Exploratory Analysis
- Check if countries or visa types affect interactions.  

---

## 7. Modeling
**Candidate models:**  
- GLM Poisson / Negative Binomial  
- Gradient Boosting (XGBoost/LightGBM)  
- Random Forest (baseline)  

**Validation:** TimeSeriesSplit, metrics = MAE, RMSE, Poisson deviance.  
**Explainability:** SHAP values, feature importance.  

---

## 8. Portfolio Load Scoring 
### 8.1 Interaction Score
- Predict `y_pred`.  
- Classify into 1–5 bands (quintiles).  

### 8.2 Family Score
- `score_family = n_applicants 

### 8.3 Deal Score
- `score_deal = w1 * score_interactions + w2 * score_family`  
  (default: `w1=0.7`, `w2=0.3`).  

### 8.4 Portfolio Score
- `score_portfolio_CS = Σ(score_deal)` for all open deals.  
- Lower score = higher availability.  

---

## 9. Automatic Distribution of New Clients
- **Eligibility:** match by language and product scope.  
- **Score for new client:**  
  - Predict `y_pred_new`.  
  - Compute `score_interactions_new + score_family_new`.  
- **Assignment:** choose eligible CS with lowest portfolio score.  
- Tie-breaker: fewest open deals or best expertise.  

## 10. Success Criteria
- Predictive model trained and validated with acceptable error (MAE, RMSE, or Poisson deviance).  
- Portfolio load scoring system (interaction score + family score) implemented and reproducible.  
- Automatic distribution logic correctly routes new clients to eligible CSs based on language, product scope, and portfolio availability.  
- Documentation and dataset published in GitHub repository with reproducible steps.  

---

> **Summary:** Phase 1 delivers the foundation of the **Customer Success Analytics project** by unifying client and interaction data, training a predictive model, and creating a scoring framework to measure workload. This enables **fair and data-driven distribution of new clients**, ensuring balanced portfolios among CSs and greater transparency in workload management. The outputs include datasets, a predictive model, scoring rules, and a dashboard for decision-making.


------------------------------------------------------------------



# Phase 2: Power BI Dashboard

> **Objective:** Build a Power BI dashboard to visualize client predictions, portfolio load, and support automatic client distribution. The dashboard will centralize KPIs for managers and CSs, making data-driven decisions easier and more transparent.

---

## 1. Overview
- **Business problem:** Lack of visibility into portfolio load, client complexity, and fair client assignment.  
- **Proposed solution:** Create a multi-page dashboard in Power BI with views for portfolio health, client analysis, and automatic distribution of new clients.  

---

## 2. Dashboard Pages & Features

### 2.1 Portfolio by CS
- **KPIs**:  
  - Total open clients  
  - Σ `score_deal` (sum of interaction + family scores)  
  - Portfolio score (availability indicator)  
- **Visuals**:  
  - Bar chart: `score_deal` per CS  
  - Line chart: portfolio score trends over time  
  - KPI cards with current workload per CS  

---

### 2.2 Client Analysis
- **KPIs**:  
  - Top 10 clients by `score_deal`  
  - Average predicted interactions by country and visa type  
  - Family size distribution and complexity impact  
- **Visuals**:  
  - Horizontal bar chart: Top 10 clients by score  
  - Heatmap: Country × Visa type vs. average interactions  
  - Boxplot: Family size vs. predicted interactions  

---

### 2.3 Automatic Distribution
- **Logic**:  
  - Predict `y_pred_new` for new clients  
  - Compute `score_interactions_new` + `score_family_new`  
  - Select CS with lowest portfolio score among eligible candidates  
- **Visuals**:  
  - Dynamic card: “**Client X → assign to [CS]**”  
  - Justification panel with:  
    - Language match  
    - Product/visa type match  
    - Client total score  
    - Current CS workload  

---

## 3. Data Requirements
For each dashboard page, the dataset must include:
- **Client-level data**: `client_id`, `visa_type`, `country`, `n_applicants`, `y_pred`, `score_interactions`, `score_family`, `score_deal`, `cs_owner`.  
- **Portfolio-level data**: Σ `score_deal` per CS, number of open clients, portfolio score trend.  
- **Distribution logic**: For new clients, `y_pred_new`, `score_total_new`, suggested CS.  

---

## 4. Deliverables
- Power BI file (`CustomerSuccessAnalytics.pbix`) with 3 pages: Portfolio by CS, Client Analysis, Automatic Distribution  
- Dataset integration with predictive model outputs  
- Documentation explaining metrics, visuals, and business rules  

---

## 5. Success Criteria
- Dashboard accurately reflects model predictions and scores  
- Managers can compare workload across CSs with clear KPIs  
- Client analysis identifies high-complexity cases by country/visa/family size  
- Automatic distribution card suggests CS assignments correctly based on business rules  

---

> **Summary:** This phase delivers a **3-page Power BI dashboard** to track portfolio load, analyze client complexity, and automatically assign new clients. It connects predictive modeling outputs with business decision-making in a visual and actionable way.


------------------------------------------------------------------



# Phase 3: Monthly CS Performance Analysis

> **Objective:** Build a dataset and reporting system to monitor monthly CS performance, focusing on activities, emails, and deals concluded. This complements the client interaction prediction model by providing visibility into team productivity and individual CS contributions.

---

## 1. Overview
- **Business problem:** Lack of structured monitoring for CS individual productivity over time.  
- **Proposed solution:** Create a dataset at the CS × period level to track key metrics and expose them in a Power BI dashboard.

---

## 2. Dataset Design — `cs_performance`
**Grain:** CS × Deal × Period (monthly)

**Fields:**
- `cs_name` — Customer Success Specialist name  
- `period_start`, `period_end` — Monthly boundaries  
- `deal_id` — Associated deal  
- `emails_sent` — Number of emails sent by this CS for the deal within the period  
- `activities_completed` — Number of activities completed by this CS for the deal within the period  
- `deals_concluded` — Flag = 1 if deal was concluded by this CS in the period, else 0  

**Aggregations (monthly, per CS):**
- Total `emails_sent`  
- Total `activities_completed`  
- Total `deals_concluded`  

---

## 3. Data Preparation
- **Deduplication:** Ensure unique `email_id` and `activity_id` (avoid double-counting).  
- **Missing data:** Treat null activity/email counts as 0.  
- **Period assignment:** Assign each event (email/activity/deal closure) to the correct calendar month.  
- **Validation:** Cross-check totals with CRM reports for accuracy.  

---

## 4. Dashboard in Power BI
### Page: CS Monthly Performance
**Metrics per CS (monthly):**
- Number of emails sent  
- Number of activities completed  
- Number of deals concluded  

**Visuals:**
- **Bar chart:** Emails/activities per CS  
- **Line chart:** Trend of activities and emails per CS over months  
- **KPI cards:** Deals concluded per CS (monthly and cumulative)  
- **Comparison table:** Ranking of CSs by total activities/emails  

---

## 5. Deliverables
- New dataset `cs_performance` with monthly metrics  
- ETL/SQL scripts to populate the dataset  
- Power BI dashboard page dedicated to CS performance  
- Documentation of data dictionary and methodology  

---

## 6. Timeline
- **Week 1:** Define schema, extract and transform raw data into `cs_performance`  
- **Week 2:** Validate data accuracy; build Power BI visuals  
- **Week 3:** Publish dashboard and document findings  

---

## 7. Success Criteria
- Accurate monthly metrics for all CSs  
- Dashboard adoption by management for performance review  
- Ability to identify top/bottom performers and productivity trends  


> **Summary:** Phase 3 delivers a dedicated dataset and dashboard for **monthly CS performance monitoring**, ensuring transparency on emails, activities, and deal closures. This empowers leadership to make data-driven decisions on team productivity and workload distribution.
