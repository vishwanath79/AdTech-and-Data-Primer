# Algorithmic Bidding

> **Note:** The code examples below are complete implementations you can run. For interactive experimentation, consider creating a Jupyter notebook based on these examples.

Manual bidding is dead. Not dying—dead. If you're still manually setting max CPCs in 2024, you're bringing a knife to a drone fight.

Algorithmic bidding uses machine learning to automatically adjust bids in real-time based on the likelihood of conversion. It's faster, smarter, and frankly better at math than any human. But it's also a black box that can burn your budget if you don't understand what's happening under the hood.

## Why Algorithmic Bidding Exists

RTB auctions happen in milliseconds. There's no time for a human to evaluate whether this specific user, on this specific page, at this specific time is worth $2.50 or $3.00. You need algorithms making thousands of bid decisions per second.

The goal: bid high enough to win valuable impressions, low enough to not waste money on junk traffic.

## How It Actually Works

At its core, algorithmic bidding is a prediction problem: given features about this impression, what's the probability it converts? Then bid accordingly.

### Basic Bidding Algorithm

Here's a simplified version of what platforms like Google Ads do. The implementation is broken into logical sections: class initialization, feature extraction, model training, prediction, and bid calculation.

#### Imports and Class Definition

```python
import numpy as np
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split

class BiddingAlgorithm:
    """
    Simple ML-based bidding algorithm
    """
    
    def __init__(self, target_cpa=50.0):
        self.target_cpa = target_cpa  # How much you're willing to pay per conversion
        self.model = LogisticRegression()
        self.is_trained = False
```

#### Feature Extraction

```python
    def extract_features(self, impression_data):
        """
        Extract features that predict conversion probability
        """
        
        features = []
        
        # User features
        features.append(impression_data.get('user_previous_visits', 0))
        features.append(impression_data.get('user_previous_conversions', 0))
        features.append(impression_data.get('user_cart_value', 0))
        
        # Context features
        features.append(impression_data.get('hour_of_day', 0))
        features.append(impression_data.get('day_of_week', 0))
        features.append(impression_data.get('is_mobile', 0))
        
        # Ad features
        features.append(impression_data.get('ad_relevance_score', 0))
        features.append(impression_data.get('landing_page_quality', 0))
        
        # Site features
        features.append(impression_data.get('site_category_match', 0))
        features.append(impression_data.get('site_quality_score', 0))
        
        return np.array(features).reshape(1, -1)
```

#### Model Training

```python
    def train(self, historical_impressions, conversions):
        """
        Train the conversion prediction model
        
        historical_impressions: Past impression data with features
        conversions: Binary labels (1 = converted, 0 = didn't convert)
        """
        
        # Extract features from historical data
        X = np.array([
            self.extract_features(imp).flatten()
            for imp in historical_impressions
        ])
        
        y = np.array(conversions)
        
        # Train model
        self.model.fit(X, y)
        self.is_trained = True
        
        # Evaluate
        score = self.model.score(X, y)
        print(f"Model accuracy: {score:.3f}")
        
        return self.model
```

#### Conversion Probability Prediction

```python
    def predict_conversion_probability(self, impression_data):
        """
        Predict probability this impression will convert
        """
        
        if not self.is_trained:
            # Cold start: use a default probability
            return 0.01
        
        features = self.extract_features(impression_data)
        prob = self.model.predict_proba(features)[0][1]
        
        return prob
```

#### Bid Calculation

```python
    def calculate_bid(self, impression_data):
        """
        Calculate optimal bid for this impression
        
        Formula: bid = target_cpa × conversion_probability
        
        If conversion probability is 2% and target CPA is $50,
        we should bid up to $1 (0.02 × $50)
        """
        
        conv_prob = self.predict_conversion_probability(impression_data)
        
        # Calculate expected value
        bid = self.target_cpa * conv_prob
        
        # Add some randomness for exploration (explore/exploit trade-off)
        bid_with_exploration = bid * np.random.uniform(0.9, 1.1)
        
        return round(bid_with_exploration, 2)
```

#### Bid Decision Logic

```python
    def should_bid(self, impression_data, floor_price):
        """
        Decide whether to bid at all
        """
        
        optimal_bid = self.calculate_bid(impression_data)
        
        # Only bid if our calculated bid meets the floor price
        return optimal_bid >= floor_price
```

#### Example Usage

```python
# Example usage
bidder = BiddingAlgorithm(target_cpa=50.0)

# Simulate training on historical data
historical_data = [
    {'user_previous_visits': 5, 'user_previous_conversions': 0, 'user_cart_value': 0,
     'hour_of_day': 14, 'day_of_week': 2, 'is_mobile': 1,
     'ad_relevance_score': 7, 'landing_page_quality': 8,
     'site_category_match': 1, 'site_quality_score': 9}
    for _ in range(1000)
]
conversions = np.random.binomial(1, 0.02, 1000)  # 2% conversion rate

bidder.train(historical_data, conversions)

# Make a bid decision
new_impression = {
    'user_previous_visits': 3,
    'user_previous_conversions': 0,
    'user_cart_value': 50,
    'hour_of_day': 15,
    'day_of_week': 3,
    'is_mobile': 1,
    'ad_relevance_score': 8,
    'landing_page_quality': 7,
    'site_category_match': 1,
    'site_quality_score': 8
}

bid = bidder.calculate_bid(new_impression)
conv_prob = bidder.predict_conversion_probability(new_impression)

print(f"\nConversion probability: {conv_prob*100:.2f}%")
print(f"Calculated bid: ${bid:.2f}")
```

That's the basic logic. Real platforms use far more sophisticated models (neural networks, gradient boosting, contextual bandits), but the principle is the same: predict conversion probability, multiply by target CPA, bid accordingly.

## Bidding Strategies

Different platforms offer different algorithmic bidding strategies. Here's what they actually mean:

### Target CPA (Cost Per Acquisition)

**What it does:** Algorithm tries to get conversions at your target cost per acquisition.

**When to use:** You know what a customer is worth and want predictable acquisition costs.

**The catch:** Needs conversion data (at least 30 conversions in 30 days) to work well. Cold start sucks.

```python
def target_cpa_bidding(conv_probability, target_cpa, uncertainty_factor=1.0):
    """
    Bid to achieve target CPA
    
    uncertainty_factor: Adjust for confidence in prediction
    - Early in campaign (low confidence): factor > 1 (bid higher to explore)
    - Mature campaign (high confidence): factor = 1
    """
    
    base_bid = target_cpa * conv_probability
    adjusted_bid = base_bid * uncertainty_factor
    
    return adjusted_bid

# Example
conv_prob = 0.02  # 2% chance of conversion
target_cpa = 50   # Willing to pay $50 per conversion

bid = target_cpa_bidding(conv_prob, target_cpa, uncertainty_factor=1.2)
print(f"Bid: ${bid:.2f}")  # $1.20
```

### Target ROAS (Return on Ad Spend)

**What it does:** Optimizes for revenue, not just conversions. Bids more for high-value conversions.

**When to use:** When conversion values vary (e.g., e-commerce with different product prices).

**The catch:** Requires passing conversion value data. Many advertisers screw this up.

```python
def target_roas_bidding(conv_probability, expected_revenue, target_roas):
    """
    Bid to achieve target return on ad spend
    
    target_roas: Desired revenue/cost ratio (e.g., 4.0 = $4 revenue per $1 spent)
    
    If expected revenue is $200 and target ROAS is 4x,
    we can spend up to $50 ($200 / 4)
    """
    
    max_bid = expected_revenue / target_roas
    bid = max_bid * conv_probability
    
    return bid

# Example
conv_prob = 0.03           # 3% conversion chance
expected_revenue = 200     # Expected order value
target_roas = 4.0          # Want $4 revenue per $1 ad spend

bid = target_roas_bidding(conv_prob, expected_revenue, target_roas)
print(f"Bid: ${bid:.2f}")  # $1.50
```

### Maximize Conversions

**What it does:** Spend your entire budget trying to get as many conversions as possible.

**When to use:** You have a fixed budget and want maximum volume.

**The catch:** Can drive up costs. If volume matters more than efficiency, great. Otherwise, risky.

### Maximize Conversion Value

**What it does:** Like maximize conversions, but prioritizes high-value conversions.

**When to use:** E-commerce where you'd rather have one $500 sale than five $50 sales.

## Multi-Armed Bandit Approach

A more sophisticated approach uses contextual bandits—balancing exploration (trying new bids to learn) with exploitation (using what you've learned to maximize performance).

### Bandit Bidder Implementation

```python
import numpy as np

class BanditBidder:
    """
    Multi-armed bandit for bid optimization
    Uses Thompson Sampling
    """
    
    def __init__(self, bid_options):
        """
        bid_options: List of possible bid amounts to test
        """
        self.bid_options = bid_options
        
        # Track successes and failures for each bid level
        self.successes = {bid: 1 for bid in bid_options}  # Start with 1 (prior)
        self.failures = {bid: 1 for bid in bid_options}
```

#### Bid Selection with Thompson Sampling

```python
    def select_bid(self):
        """
        Select a bid using Thompson Sampling
        
        Sample from beta distribution for each bid option,
        pick the one with highest sampled value
        """
        
        sampled_values = {}
        
        for bid in self.bid_options:
            # Sample from Beta(successes, failures)
            sample = np.random.beta(
                self.successes[bid],
                self.failures[bid]
            )
            sampled_values[bid] = sample
        
        # Pick bid with highest sampled conversion rate
        best_bid = max(sampled_values, key=sampled_values.get)
        
        return best_bid
```

#### Updating Beliefs

```python
    def update(self, bid, converted):
        """
        Update our belief about this bid level based on outcome
        """
        
        if converted:
            self.successes[bid] += 1
        else:
            self.failures[bid] += 1
```

#### Performance Statistics

```python
    def get_stats(self):
        """
        Get current conversion rate estimates for each bid
        """
        
        stats = {}
        
        for bid in self.bid_options:
            total = self.successes[bid] + self.failures[bid]
            conv_rate = self.successes[bid] / total
            
            stats[bid] = {
                'conversion_rate': conv_rate,
                'impressions': total - 2,  # Subtract the prior
                'conversions': self.successes[bid] - 1
            }
        
        return stats
```

#### Bandit Example Usage

```python
# Example usage
bid_levels = [1.00, 1.50, 2.00, 2.50, 3.00]
bandit = BanditBidder(bid_levels)

# Simulate 1000 auction decisions
for i in range(1000):
    # Select bid
    bid = bandit.select_bid()
    
    # Simulate outcome (higher bids win more, win rate correlates with conversion)
    win_prob = min(bid / 5.0, 0.9)  # Higher bids win more auctions
    won = np.random.random() < win_prob
    
    if won:
        # If we won, did it convert?
        conv_prob = 0.02  # 2% base conversion rate
        converted = np.random.random() < conv_prob
        bandit.update(bid, converted)

# See what we learned
print("Bid Performance:")
stats = bandit.get_stats()
for bid, data in sorted(stats.items()):
    print(f"  ${bid:.2f}: {data['conversions']} conv / {data['impressions']} impr "
          f"({data['conversion_rate']*100:.1f}%)")
```

This approach automatically finds the sweet spot between bidding too low (missing conversions) and too high (wasting money).

## Real-Time Bid Adjustments

Smart bidders don't just use a single model—they adjust bids in real-time based on multiple signals:

### Smart Bidder Implementation

```python
class SmartBidder:
    """
    Advanced bidder with real-time adjustments
    """
    
    def __init__(self, base_bidder, target_cpa):
        self.base_bidder = base_bidder
        self.target_cpa = target_cpa
    
    def calculate_bid_with_adjustments(self, impression_data, campaign_stats):
        """
        Calculate bid with real-time adjustments
        """
        
        # Get base bid from ML model
        base_bid = self.base_bidder.calculate_bid(impression_data)
        
        # Apply adjustment modifiers
        bid = base_bid
        
        # 1. Budget pacing adjustment
        bid = self.apply_budget_pacing(bid, campaign_stats)
        
        # 2. Time-of-day adjustment
        bid = self.apply_time_of_day_modifier(bid, impression_data)
        
        # 3. Audience quality adjustment
        bid = self.apply_audience_modifier(bid, impression_data)
        
        # 4. Competitive adjustment
        bid = self.apply_competitive_modifier(bid, impression_data)
        
        return round(bid, 2)
```

#### Budget Pacing Adjustment

```python
    def apply_budget_pacing(self, bid, campaign_stats):
        """
        Adjust bid based on budget pacing
        
        If we're under-pacing (spending too slowly), bid more aggressively
        If we're over-pacing (burning through budget), pull back
        """
        
        spent_pct = campaign_stats['spent'] / campaign_stats['budget']
        time_pct = campaign_stats['hours_elapsed'] / campaign_stats['total_hours']
        
        pacing_ratio = spent_pct / time_pct
        
        if pacing_ratio < 0.8:
            # Under-pacing: bid up to 20% more
            return bid * 1.2
        elif pacing_ratio > 1.2:
            # Over-pacing: bid up to 20% less
            return bid * 0.8
        else:
            return bid
```

#### Time-of-Day Adjustment

```python
    def apply_time_of_day_modifier(self, bid, impression_data):
        """
        Adjust based on time of day performance
        """
        
        hour = impression_data.get('hour_of_day', 12)
        
        # Prime hours (9-5): normal bidding
        # Off hours (11pm-6am): reduce bids by 30%
        if 6 <= hour <= 22:
            return bid
        else:
            return bid * 0.7
```

#### Audience Quality Adjustment

```python
    def apply_audience_modifier(self, bid, impression_data):
        """
        Adjust based on user quality signals
        """
        
        # High-value signals
        if impression_data.get('user_previous_conversions', 0) > 0:
            return bid * 1.5  # Past customer? Bid 50% more
        
        if impression_data.get('user_cart_value', 0) > 100:
            return bid * 1.3  # High cart value? Bid 30% more
        
        return bid
```

#### Competitive Adjustment

```python
    def apply_competitive_modifier(self, bid, impression_data):
        """
        Adjust based on competitive intensity
        """
        
        competition_level = impression_data.get('estimated_competition', 'medium')
        
        if competition_level == 'high':
            return bid * 1.1  # Bid slightly more in competitive auctions
        elif competition_level == 'low':
            return bid * 0.9  # Bid less when competition is weak
        
        return bid
```

This multi-layered approach is closer to what enterprise platforms actually do—combine ML predictions with business logic and real-time constraints.

## When Algorithmic Bidding Fails

It's not perfect. Here's where it breaks:

### 1. Cold Start Problem

New campaigns have no data. The algorithm is guessing. Expect wild swings and wasted spend for the first few days while it learns.

**Solution:** Start with manual bidding or a conservative target CPA, then switch to algorithmic once you have 30+ conversions.

### 2. Data Quality Issues

If your conversion tracking is broken (happens more than you'd think), the algorithm learns the wrong patterns. Garbage in, garbage out.

**Solution:** Audit your tracking religiously. Test conversions manually. Check for discrepancies.

### 3. Black Box Opacity

Google's Smart Bidding doesn't tell you why it bid $3 instead of $2. You're trusting the algorithm without visibility into its decisions.

**Solution:** Monitor performance closely. If results degrade, switch back to manual control and investigate.

### 4. Budget Constraints

Algorithms optimize for your goal (conversions, ROAS), but they need budget headroom. If your budget is too tight, the algorithm can't explore and learn effectively.

**Solution:** Give it enough budget to get statistically significant data (30+ conversions minimum).

## Key Takeaways

Algorithmic bidding is mandatory at scale. The math is simple—predict conversion probability, calculate expected value, bid accordingly. But the implementation is complex, with multi-armed bandits, real-time adjustments, and cold start strategies.

The platforms (Google, Facebook, Amazon) have sophisticated algorithms you can't replicate easily. Use them. But understand what they're doing so you can troubleshoot when things go sideways (and they will).

The code above shows the fundamental concepts. In production, you're adding ensemble models, contextual bandits, constrained optimization, and way more feature engineering. But the core logic stays the same: predict value, bid proportionally, adjust in real-time.

Don't fight the algorithms—learn to work with them.

