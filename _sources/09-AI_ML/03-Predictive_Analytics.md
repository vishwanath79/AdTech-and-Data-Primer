# Predictive Analytics in AdTech

> **Note:** The code examples below are complete implementations you can run. For interactive experimentation, consider creating Jupyter notebooks based on these examples.

Using data to predict the future is what predictive analytics does. You don't use crystal balls; instead, you use regression models, time series forecasting, and a lot of patterns from the past. The goal is to guess what will happen next so you can do something about it before it happens.

 In AdTech, this means predicting conversions, churn, lifetime value, budget pacing, and about a dozen other metrics that tell you if your campaigns make money or lose it.

 ## Why Prediction Is Important

 It costs a lot to do reactive advertising.  You start a campaign, wait for results, and then make changes.  Once you realize something isn't working, you've already spent too much money.

 Predictive analytics turns this around: it lets you predict performance before you launch, find problems before they ruin your results, and make things better while there's still time to fix them.

 I've seen campaigns save 30–40% of their budget by catching performance trends that are going down early instead of waiting until the end of the month to look at them.  That is real money.

## Core Predictive Models in AdTech

### 1. Conversion Probability Prediction

The most fundamental: given a user's characteristics, what's the chance they convert?

```python
import pandas as pd
import numpy as np
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report, roc_auc_score

class ConversionPredictor:
    """
    Predict likelihood of conversion for targeting and bidding
    """
    
    def __init__(self):
        self.model = GradientBoostingClassifier(
            n_estimators=100,
            learning_rate=0.1,
            max_depth=5,
            random_state=42
        )
        self.feature_names = None
    
    def prepare_features(self, user_data):
        """
        Feature engineering for conversion prediction
        """
        
        features = pd.DataFrame()
        
        # Behavioral features
        features['sessions_last_30d'] = user_data['sessions_last_30d']
        features['pages_per_session'] = user_data['total_pages'] / user_data['sessions_last_30d'].clip(lower=1)
        features['avg_session_duration'] = user_data['total_time'] / user_data['sessions_last_30d'].clip(lower=1)
        
        # Engagement features
        features['email_open_rate'] = user_data['emails_opened'] / user_data['emails_sent'].clip(lower=1)
        features['product_views'] = user_data['product_views']
        features['cart_adds'] = user_data['cart_adds']
        features['checkout_starts'] = user_data['checkout_starts']
        
        # Recency features (days since last action)
        features['days_since_visit'] = user_data['days_since_visit']
        features['days_since_product_view'] = user_data['days_since_product_view']
        
        # Derived features
        features['cart_add_rate'] = user_data['cart_adds'] / user_data['product_views'].clip(lower=1)
        features['checkout_conversion'] = user_data['checkout_starts'] / user_data['cart_adds'].clip(lower=1)
        
        # Device/channel features
        features['is_mobile'] = (user_data['device_type'] == 'mobile').astype(int)
        features['came_from_paid'] = (user_data['acquisition_channel'] == 'paid').astype(int)
        
        self.feature_names = features.columns.tolist()
        
        return features
    
    def train(self, user_data, conversions):
        """
        Train conversion prediction model
        """
        
        X = self.prepare_features(user_data)
        y = conversions
        
        X_train, X_test, y_train, y_test = train_test_split(
            X, y, test_size=0.2, random_state=42, stratify=y
        )
        
        # Train model
        self.model.fit(X_train, y_train)
        
        # Evaluate
        y_pred = self.model.predict(X_test)
        y_pred_proba = self.model.predict_proba(X_test)[:, 1]
        
        print("Model Performance:")
        print(classification_report(y_test, y_pred))
        print(f"\nROC-AUC Score: {roc_auc_score(y_test, y_pred_proba):.3f}")
        
        # Feature importance
        importance = pd.DataFrame({
            'feature': self.feature_names,
            'importance': self.model.feature_importances_
        }).sort_values('importance', ascending=False)
        
        print("\nTop Conversion Predictors:")
        print(importance.head(10))
        
        return self.model
    
    def predict_proba(self, user_data):
        """
        Predict conversion probability for new users
        """
        
        X = self.prepare_features(user_data)
        probabilities = self.model.predict_proba(X)[:, 1]
        
        return probabilities
    
    def score_users(self, user_data):
        """
        Score users from 0-100 based on conversion likelihood
        """
        
        probabilities = self.predict_proba(user_data)
        scores = (probabilities * 100).round().astype(int)
        
        return scores

# Example usage
# users = pd.read_csv('user_data.csv')
# conversions = users['converted']
# 
# predictor = ConversionPredictor()
# predictor.train(users, conversions)
# 
# # Score new users for targeting
# new_users = pd.read_csv('new_users.csv')
# scores = predictor.score_users(new_users)
# 
# # Target users with high conversion probability
# high_value_users = new_users[scores > 70]
```

This lets you focus ad spend on users most likely to convert instead of spraying budget everywhere.

### 2. Churn Prediction

Identifying customers about to leave so you can win them back before they're gone.

```python
from sklearn.ensemble import RandomForestClassifier
from datetime import datetime, timedelta

class ChurnPredictor:
    """
    Predict which customers are likely to churn
    """
    
    def __init__(self, churn_definition_days=60):
        self.churn_definition_days = churn_definition_days
        self.model = RandomForestClassifier(n_estimators=100, random_state=42)
    
    def calculate_churn_labels(self, customer_data, reference_date):
        """
        Define churn: no activity in last N days
        """
        
        churn_threshold = reference_date - timedelta(days=self.churn_definition_days)
        last_activity = pd.to_datetime(customer_data['last_activity_date'])
        
        churned = (last_activity < churn_threshold).astype(int)
        
        return churned
    
    def engineer_churn_features(self, customer_data):
        """
        Create features that indicate churn risk
        """
        
        features = pd.DataFrame()
        
        # Recency features (strong churn indicators)
        features['days_since_purchase'] = customer_data['days_since_purchase']
        features['days_since_login'] = customer_data['days_since_login']
        features['days_since_email_open'] = customer_data['days_since_email_open']
        
        # Frequency features
        features['purchases_last_90d'] = customer_data['purchases_last_90d']
        features['logins_last_30d'] = customer_data['logins_last_30d']
        features['sessions_last_30d'] = customer_data['sessions_last_30d']
        
        # Trend features (declining engagement = churn risk)
        features['purchase_trend'] = (
            customer_data['purchases_last_30d'] - customer_data['purchases_30_60d']
        )
        features['engagement_trend'] = (
            customer_data['sessions_last_30d'] - customer_data['sessions_30_60d']
        )
        
        # Monetary features
        features['ltv'] = customer_data['lifetime_value']
        features['avg_order_value'] = customer_data['avg_order_value']
        features['days_as_customer'] = customer_data['days_as_customer']
        
        # Support interactions (more support = potential issues)
        features['support_tickets_last_30d'] = customer_data['support_tickets_last_30d']
        
        return features
    
    def train(self, customer_data, reference_date=None):
        """
        Train churn prediction model
        """
        
        if reference_date is None:
            reference_date = datetime.now()
        
        # Create labels
        y = self.calculate_churn_labels(customer_data, reference_date)
        
        # Create features
        X = self.engineer_churn_features(customer_data)
        
        # Split
        X_train, X_test, y_train, y_test = train_test_split(
            X, y, test_size=0.2, random_state=42, stratify=y
        )
        
        # Train
        self.model.fit(X_train, y_train)
        
        # Evaluate
        y_pred = self.model.predict(X_test)
        y_proba = self.model.predict_proba(X_test)[:, 1]
        
        print("Churn Prediction Performance:")
        print(classification_report(y_test, y_pred))
        print(f"ROC-AUC: {roc_auc_score(y_test, y_proba):.3f}")
        
        # Identify key churn indicators
        importance = pd.DataFrame({
            'feature': X.columns,
            'importance': self.model.feature_importances_
        }).sort_values('importance', ascending=False)
        
        print("\nTop Churn Indicators:")
        print(importance.head(10))
        
        return self.model
    
    def identify_at_risk_customers(self, customer_data, risk_threshold=0.7):
        """
        Find customers at high risk of churning
        """
        
        X = self.engineer_churn_features(customer_data)
        churn_proba = self.model.predict_proba(X)[:, 1]
        
        at_risk = pd.DataFrame({
            'customer_id': customer_data['customer_id'],
            'churn_probability': churn_proba,
            'ltv': customer_data['lifetime_value'],
            'days_since_purchase': customer_data['days_since_purchase']
        })
        
        # Focus on high-value customers at risk
        at_risk = at_risk[churn_proba > risk_threshold].sort_values(
            'ltv', ascending=False
        )
        
        return at_risk

# Usage: Build a retargeting campaign for at-risk customers
# churn_model = ChurnPredictor(churn_definition_days=60)
# churn_model.train(customers)
# 
# at_risk = churn_model.identify_at_risk_customers(customers, risk_threshold=0.7)
# 
# # Target these users with win-back campaigns
# print(f"Found {len(at_risk)} high-risk customers")
# print(f"Combined LTV at risk: ${at_risk['ltv'].sum():,.2f}")
```

Use this to trigger win-back campaigns automatically when a customer's churn risk crosses a threshold. Way more effective than generic "We miss you!" emails sent randomly.

### 3. Budget Pacing Prediction

Forecast whether you'll spend your entire budget or run out before the campaign ends.

```python
from sklearn.linear_model import LinearRegression
import numpy as np

class BudgetPacingPredictor:
    """
    Predict campaign spend trajectory and recommend pacing adjustments
    """
    
    def __init__(self):
        self.model = LinearRegression()
    
    def forecast_spend(self, daily_spend_history, days_remaining):
        """
        Forecast remaining spend based on historical trend
        """
        
        # Prepare data
        days = np.arange(len(daily_spend_history)).reshape(-1, 1)
        spend = np.array(daily_spend_history)
        
        # Fit trend
        self.model.fit(days, spend)
        
        # Forecast future days
        future_days = np.arange(
            len(daily_spend_history),
            len(daily_spend_history) + days_remaining
        ).reshape(-1, 1)
        
        forecasted_spend = self.model.predict(future_days)
        
        return forecasted_spend
    
    def analyze_pacing(self, campaign_stats):
        """
        Analyze campaign pacing and recommend adjustments
        """
        
        total_budget = campaign_stats['total_budget']
        spent_so_far = campaign_stats['spent_so_far']
        days_elapsed = campaign_stats['days_elapsed']
        days_remaining = campaign_stats['days_remaining']
        daily_spend = campaign_stats['daily_spend_history']
        
        # Forecast remaining spend
        forecasted_daily_spend = self.forecast_spend(daily_spend, days_remaining)
        forecasted_total_spend = spent_so_far + forecasted_daily_spend.sum()
        
        # Calculate pacing metrics
        total_days = days_elapsed + days_remaining
        expected_spend_pct = days_elapsed / total_days
        actual_spend_pct = spent_so_far / total_budget
        
        pacing_ratio = actual_spend_pct / expected_spend_pct
        
        # Forecast vs budget
        spend_gap = forecasted_total_spend - total_budget
        spend_gap_pct = (spend_gap / total_budget) * 100
        
        # Determine pacing status
        if pacing_ratio < 0.85:
            status = "UNDER-PACING"
            recommendation = "Increase bids or expand targeting to spend remaining budget"
        elif pacing_ratio > 1.15:
            status = "OVER-PACING"
            recommendation = "Reduce bids or tighten targeting to avoid budget exhaustion"
        else:
            status = "ON-PACE"
            recommendation = "Maintain current strategy"
        
        # Calculate recommended daily budget adjustment
        if days_remaining > 0:
            remaining_budget = total_budget - spent_so_far
            recommended_daily = remaining_budget / days_remaining
            current_avg_daily = spent_so_far / days_elapsed
            adjustment_pct = ((recommended_daily / current_avg_daily) - 1) * 100
        else:
            recommended_daily = 0
            adjustment_pct = 0
        
        return {
            'status': status,
            'pacing_ratio': pacing_ratio,
            'spent_pct': actual_spend_pct * 100,
            'forecasted_total_spend': forecasted_total_spend,
            'spend_gap': spend_gap,
            'spend_gap_pct': spend_gap_pct,
            'recommended_daily_budget': recommended_daily,
            'adjustment_needed_pct': adjustment_pct,
            'recommendation': recommendation
        }

# Example usage
campaign = {
    'total_budget': 10000,
    'spent_so_far': 3000,
    'days_elapsed': 10,
    'days_remaining': 20,
    'daily_spend_history': [250, 280, 310, 290, 300, 320, 295, 305, 330, 320]
}

pacer = BudgetPacingPredictor()
analysis = pacer.analyze_pacing(campaign)

print(f"Status: {analysis['status']}")
print(f"Pacing Ratio: {analysis['pacing_ratio']:.2f}")
print(f"Spent: {analysis['spent_pct']:.1f}% of budget")
print(f"Forecasted Total: ${analysis['forecasted_total_spend']:,.2f}")
print(f"Gap: ${analysis['spend_gap']:,.2f} ({analysis['spend_gap_pct']:.1f}%)")
print(f"\nRecommended Daily Budget: ${analysis['recommended_daily_budget']:.2f}")
print(f"Adjustment Needed: {analysis['adjustment_needed_pct']:+.1f}%")
print(f"\n{analysis['recommendation']}")
```

This prevents the classic mistake of spending your entire monthly budget in the first two weeks, then sitting idle for the rest of the month.

### 4. Time Series Forecasting for Seasonal Trends

Predict how campaigns will perform based on seasonal patterns.

```python
from statsmodels.tsa.holtwinters import ExponentialSmoothing
import pandas as pd

class SeasonalForecaster:
    """
    Forecast campaign performance accounting for seasonality
    """
    
    def __init__(self, seasonal_period=7):
        """
        seasonal_period: 7 for weekly patterns, 30 for monthly, etc.
        """
        self.seasonal_period = seasonal_period
        self.model = None
    
    def fit(self, historical_data):
        """
        Fit model to historical performance data
        
        historical_data: Time series of daily metrics (conversions, spend, etc.)
        """
        
        # Use Holt-Winters for trend + seasonality
        self.model = ExponentialSmoothing(
            historical_data,
            seasonal_periods=self.seasonal_period,
            trend='add',
            seasonal='add'
        ).fit()
        
        return self.model
    
    def forecast(self, periods):
        """
        Forecast next N periods
        """
        
        forecast = self.model.forecast(periods)
        
        return forecast
    
    def forecast_with_confidence(self, periods, confidence=0.95):
        """
        Forecast with confidence intervals
        """
        
        forecast = self.model.forecast(periods)
        
        # Calculate prediction intervals (simplified)
        std_error = self.model.resid.std()
        z_score = 1.96 if confidence == 0.95 else 2.58  # 95% or 99%
        
        margin = z_score * std_error
        
        forecast_df = pd.DataFrame({
            'forecast': forecast,
            'lower_bound': forecast - margin,
            'upper_bound': forecast + margin
        })
        
        return forecast_df

# Example usage
# Generate sample data with weekly seasonality
dates = pd.date_range('2024-01-01', periods=90, freq='D')
base_conversions = 100
trend = np.linspace(0, 20, 90)
seasonality = 30 * np.sin(np.arange(90) * 2 * np.pi / 7)  # Weekly pattern
noise = np.random.normal(0, 10, 90)

conversions = base_conversions + trend + seasonality + noise
conversion_series = pd.Series(conversions, index=dates)

# Fit and forecast
forecaster = SeasonalForecaster(seasonal_period=7)
forecaster.fit(conversion_series)

# Forecast next 14 days
forecast = forecaster.forecast_with_confidence(14)

print("14-Day Forecast:")
print(forecast)
print(f"\nExpected conversions: {forecast['forecast'].sum():.0f}")
print(f"Range: {forecast['lower_bound'].sum():.0f} - {forecast['upper_bound'].sum():.0f}")
```

Use this to plan budget allocation—if you know Black Friday will drive 3x normal conversions, you allocate budget accordingly instead of spreading it evenly.

## Lookalike Modeling

Find new customers that resemble your best existing customers.

```python
from sklearn.neighbors import NearestNeighbors
from sklearn.preprocessing import StandardScaler

class LookalikeModeler:
    """
    Find users similar to your high-value customers
    """
    
    def __init__(self, n_neighbors=100):
        self.n_neighbors = n_neighbors
        self.model = NearestNeighbors(n_neighbors=n_neighbors, metric='cosine')
        self.scaler = StandardScaler()
    
    def fit_seed_audience(self, seed_users, feature_columns):
        """
        Define your seed audience (high-value customers)
        """
        
        self.feature_columns = feature_columns
        X = seed_users[feature_columns]
        
        # Normalize features
        X_scaled = self.scaler.fit_transform(X)
        
        # Fit model
        self.model.fit(X_scaled)
        
        return self
    
    def find_lookalikes(self, potential_users):
        """
        Find users similar to seed audience from potential user pool
        """
        
        X = potential_users[self.feature_columns]
        X_scaled = self.scaler.transform(X)
        
        # Find nearest neighbors
        distances, indices = self.model.kneighbors(X_scaled)
        
        # Calculate similarity scores (inverse of distance)
        similarity_scores = 1 / (1 + distances.mean(axis=1))
        
        lookalikes = potential_users.copy()
        lookalikes['similarity_score'] = similarity_scores
        lookalikes = lookalikes.sort_values('similarity_score', ascending=False)
        
        return lookalikes

# Example usage
# high_value_customers = customers[customers['ltv'] > 1000]
# 
# modeler = LookalikeModeler(n_neighbors=50)
# modeler.fit_seed_audience(
#     high_value_customers,
#     feature_columns=['sessions', 'pages_viewed', 'email_opens', 'product_categories']
# )
# 
# # Find similar users in prospect database
# lookalikes = modeler.find_lookalikes(prospects)
# 
# # Target top 10% most similar users
# top_lookalikes = lookalikes.head(int(len(lookalikes) * 0.1))
```

Facebook's Lookalike Audiences use this concept—upload your customer list, they find similar users to target. You can build your own version.

## Key Takeaways

Predictive analytics in AdTech is about seeing around corners—forecasting conversions, identifying churn before it happens, optimizing budget pacing, finding high-value prospects.

The models aren't complex—mostly regression, classification, and time series. The value is in applying them systematically: predict churn every week, adjust bids based on conversion probability daily, forecast pacing in real-time.

The code above gives you working implementations. In production, you'd add more sophisticated feature engineering, ensemble methods, and automated retraining. But the core logic stays the same: use past patterns to predict future outcomes, then act on those predictions.

The companies winning in AdTech aren't the ones with the fanciest models—they're the ones consistently using predictions to make better decisions faster than the competition.

