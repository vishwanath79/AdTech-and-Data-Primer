# Creative A/B Testing

Ever run the same ad campaign and wonder why the carefully crafted "Buy" button got ignored while "Shop Now" crushed it? Welcome to A/B testing—advertising's way of admitting we're all terrible at predicting what actually works.

I've seen creative marketing teams spend weeks debating button colors and email template backgrounds, only to have users pick the "wrong" version every single time. That's the beauty (and humility) of A/B testing: you let real humans tell you what they'll click instead of trusting your gut.

## How It Actually Works

The concept is simple: create two (or more) versions of your ad, show them to different users, measure which one performs better. 

Here's the basic flow:

1. **Create variants** - Change the headline, CTA, image, or any element you want to test
2. **Split your audience** - Show version A to half your users, version B to the other half
3. **Let the data speak** - Track clicks, conversions, and let the numbers tell you the winner

The key is testing ONE thing at a time. Change the headline OR the CTA, not both. Otherwise you're like a chef changing five ingredients at once and wondering why the dish tastes different—you'll never know which change mattered.

## Setting Up Test Variants

Let's start with two simple creative variants for a summer sale campaign:

```python
creatives = [
    {
        "id": "creative_A",
        "name": "Summer Sale - Variant A",
        "headline": "Summer Sale!",
        "cta": "Shop Now",
        "statistics": {
            "impressions": 0,
            "clicks": 0
        }
    },
    {
        "id": "creative_B",
        "name": "Summer Sale - Variant B",
        "headline": "Limited Time Offer!",
        "cta": "Buy Now",
        "statistics": {
            "impressions": 0,
            "clicks": 0
        }
    }
]
```

Notice we're only changing the headline and CTA. Keep it focused.

## Creative Rotation Strategies

You've got two main approaches to serving these ads:

### Even Rotation (The Learning Phase)

Show all creatives equally to collect unbiased data. No favorites, no hunches—just pure democracy. This is crucial early on when you don't know what works.

```python
import random

def select_creative(creatives):
    """Select a creative randomly for even distribution"""
    return random.choice(creatives)

# Usage
selected = select_creative(creatives)
print(f"Selected: {selected['name']}")
```

Simple. Effective. Boring. But necessary.

### Performance-Based Rotation (After You Know What Works)

Once you have enough data (usually 100+ impressions per variant), favor the winner but keep testing. The 80/20 split below ensures you're capitalizing on what works while still exploring.

```python
def select_best_creative(creatives, min_impressions=100):
    """Select creative based on CTR after learning phase"""
    
    # Check if we have enough data
    learning = [c for c in creatives if c['statistics']['impressions'] < min_impressions]
    
    # If still learning, use random rotation
    if learning:
        return random.choice(creatives)
    
    # Otherwise, pick the best performer (highest CTR)
    best = max(creatives, key=lambda c: c['statistics']['clicks'] / max(c['statistics']['impressions'], 1))
    
    # 80% best, 20% random (exploration vs exploitation)
    return best if random.random() < 0.8 else random.choice(creatives)

# Usage
selected = select_best_creative(creatives)
print(f"Selected: {selected['name']}")
```

That 80/20 split is the magic. You're not putting all your eggs in one basket, but you're betting on the winner most of the time.

## Building a Simple A/B Test Manager

Here's a lightweight test framework that tracks everything you need:

```python
from datetime import datetime, timedelta

class ABTest:
    def __init__(self, name, creatives, duration_days=7):
        self.name = name
        self.creatives = creatives
        self.end_date = datetime.now() + timedelta(days=duration_days)
    
    def is_complete(self):
        """Check if test is done"""
        return datetime.now() >= self.end_date
    
    def record_impression(self, creative_id):
        """Record an impression"""
        for c in self.creatives:
            if c['id'] == creative_id:
                c['statistics']['impressions'] += 1
    
    def record_click(self, creative_id):
        """Record a click"""
        for c in self.creatives:
            if c['id'] == creative_id:
                c['statistics']['clicks'] += 1
    
    def get_results(self):
        """Get test results with CTR"""
        results = []
        for c in self.creatives:
            stats = c['statistics']
            ctr = (stats['clicks'] / stats['impressions'] * 100) if stats['impressions'] > 0 else 0
            
            results.append({
                "name": c['name'],
                "impressions": stats['impressions'],
                "clicks": stats['clicks'],
                "ctr": round(ctr, 2)
            })
        
        # Sort by CTR
        results.sort(key=lambda x: x['ctr'], reverse=True)
        
        return {
            "test_name": self.name,
            "status": "complete" if self.is_complete() else "running",
            "winner": results[0] if results else None,
            "all_results": results
        }
```

Now let's run a test with simulated traffic to see it in action:

```python
# Create test
test = ABTest("Headline Test", creatives, duration_days=7)

# Simulate traffic
for _ in range(1000):
    creative = select_creative(creatives)
    test.record_impression(creative['id'])
    
    # Simulate clicks (Variant B performs better: 3% vs 2%)
    if creative['id'] == 'creative_A' and random.random() < 0.02:
        test.record_click(creative['id'])
    elif creative['id'] == 'creative_B' and random.random() < 0.03:
        test.record_click(creative['id'])

# Get results
results = test.get_results()
print(f"\n=== {results['test_name']} ===")
print(f"Status: {results['status']}")
print(f"\nWinner: {results['winner']['name']}")
print(f"CTR: {results['winner']['ctr']}%")

print("\nAll Results:")
for r in results['all_results']:
    print(f"{r['name']}: {r['impressions']} impr, {r['clicks']} clicks, {r['ctr']}% CTR")
```

In this simulation, Variant B wins with a 3% CTR versus 2%. That 1% difference might seem small, but at scale it's huge—imagine that on a million impressions.

## Dynamic Creative Optimization (DCO)

A/B testing is great when you have two static options. But what if you could personalize the ad for each user? That's DCO—assembling custom creatives on the fly based on who's looking.

Think of it like a algorithm that sees different users and serves them different creatives. New visitor? Hit them with a discount. Returning customer? Welcome them back. Cart abandoner? Remind them what they left behind.

Here's a simple implementation:

```python
class SimpleCreativeBuilder:
    """Build personalized creatives based on user data"""
    
    def __init__(self):
        self.headlines = {
            "new_user": "Welcome! Get 20% Off",
            "returning": "Welcome Back!",
            "default": "Special Offer"
        }
        
        self.ctas = {
            "new_user": "Get Discount",
            "returning": "Shop Now",
            "default": "Learn More"
        }
    
    def build(self, user_profile):
        """Build personalized creative"""
        # Determine user type
        if user_profile.get('is_new'):
            user_type = 'new_user'
        elif user_profile.get('returning'):
            user_type = 'returning'
        else:
            user_type = 'default'
        
        # Select components
        return {
            "headline": self.headlines[user_type],
            "cta": self.ctas[user_type],
            "personalized_for": user_type
        }

# Example usage
builder = SimpleCreativeBuilder()

# New user
creative = builder.build({"is_new": True})
print(f"New User Creative: {creative['headline']} - {creative['cta']}")

# Returning user
creative = builder.build({"returning": True})
print(f"Returning User Creative: {creative['headline']} - {creative['cta']}")
```

This is a basic DCO setup, but you can expand it infinitely—different images by location, headlines by purchase history, CTAs by device type. The real implementations get complex fast (hence why platforms charge premium for this feature).

## What You Should Actually Test

Headlines and CTAs are the obvious ones, but here are some other elements worth testing:

- **Images/Videos** - Product shots vs lifestyle imagery
- **Color schemes** - That button color debate isn't totally crazy
- **Offer positioning** - "Save 20%" vs "$20 Off"
- **Length** - Short and punchy vs detailed and informative
- **Social proof** - "Join 1M users" vs no social proof

Key ideas to remember: test one thing at a time, wait for statistical significance (usually a few hundred conversions minimum), and don't call it early because you're excited about preliminary results. 

## Key Takeaways

A/B testing keeps you honest. Your opinion doesn't matter—the data does. DCO takes it further by personalizing at scale. Both are essential tools in modern AdTech, and both require patience, discipline, and a willingness to be wrong about what you thought would work.

The code above gives you the foundation. The rest is experimentation, iteration, and learning to trust the numbers over your instincts (which is harder than it sounds).
