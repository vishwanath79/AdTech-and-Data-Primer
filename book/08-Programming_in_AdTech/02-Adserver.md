### Setting up the foundation (Basic Ad Server)

> **Interactive Version:** See [Ad Server Notebook](./02-Adserver.ipynb) to run this code interactively and experiment with the ad server.
> 
> **Download:** Full script available in `programmatic/app_simple.py`

In this part, we'll talk about the basics of making a simple ad server and how to target ads.

We will make a simple programmatic ad server that shows ads to users based on what they want. This will use a simple first-price auction to decide which ad to show. We will use the Flask framework to serve the ads and write the script in Python.

The ad inventory is a list of dictionaries, with each dictionary holding an ad. There is metadata, targeting criteria, and performance statistics in each ad dictionary.

## Create a simple ad server

Let's create the ad inventory first. This will be a list of dictionaries, each representing an ad. Each ad dictionary contains metadata, targeting criteria, and performance statistics.

1) Page load
- The user loads your page (e.g., `/test`) which contains an ad slot.

2) Ad request
- The page makes a `GET` request to `/serve-ad` with query params such as:
  - `size` (e.g., `300x250`)
  - `country` (e.g., `US`)
  - `language` (e.g., `en`)
  - `device` (e.g., `desktop`)
  - `age` (optional; integer)
  - `interests` (optional; comma-separated, e.g., `fashion,shopping`)
  - `premium` (optional; `true`/`false`)

3) Targeting + bidding on the server
- The server filters active ads that match size and targeting rules (country, language, device, age, budget).
- It computes a bid per matched ad:
  - Base = `cpm`
  - +20% if `premium=true`
  - +10% per interest overlap

4) Winner selection + impression
- The highest bid wins.
- The server immediately records the impression and increments spend (CPM/1000).

5) Response
- The server returns ad metadata (e.g., `id`, `name`, `image_url`, `target_url`, `size`, `advertiser`), excluding targeting/stats.

6) Render + click
- The page renders the ad creative (e.g., `<img>` inside a link).
- On user click, the page first calls `GET /ad-click/<ad_id>` to record the click, then navigates to the ad’s `target_url`.

7) Reporting
- A monitoring tool (or you) can call `GET /ad-stats` to see per-ad metrics:
  - impressions, clicks, CTR, spend

## Complete Implementation

Below is the complete ad server implementation. The code is organized into logical sections: imports and setup, ad inventory, targeting logic, API endpoints, and tracking functions.

### Imports and Setup

```python
from flask import Flask, jsonify, request, render_template
from datetime import datetime
import json
from typing import Dict
import uuid

# Initialize Flask application
app = Flask(__name__)
```

### Ad Inventory

```python
# Sample ad inventory - list of dictionaries, each representing an ad
ad_inventory = [
    {
        "id": 1,
        "name": "Summer Sale Banner",
        "image_url": "https://via.placeholder.com/300x250?text=Summer+Sale",
        "target_url": "https://example.com/summer-sale",
        "size": "300x250",
        "advertiser": "Fashion Store",
        "active": True,
        "targeting": {
            "countries": ["US", "CA", "UK"],  # Countries where ad can be shown
            "languages": ["en"],              # Supported languages
            "devices": ["desktop", "mobile"], # Target devices
            "age_range": {"min": 18, "max": 35}, # Target age range
            "interests": ["fashion", "shopping"], # Target user interests
            "daily_budget": 1000.00,          # Maximum daily spend in USD
            "cpm": 2.50                       # Cost per thousand impressions
        },
        "statistics": {
            "impressions": 0,                 # Number of times ad was shown
            "clicks": 0,                      # Number of clicks received
            "spent": 0.00                     # Total amount spent
        }
    },
    # 2nd Ad with different targeting criteria
    {
        "id": 2,
        "name": "Tech Gadget Promotion",
        "image_url": "https://via.placeholder.com/728x90?text=Tech+Gadget",
        "target_url": "https://example.com/gadget-promo",
        "size": "728x90",
        "advertiser": "Tech Shop",
        "active": True,
        "targeting": {
            "countries": ["US", "DE", "FR"],
            "languages": ["en", "de", "fr"],
            "devices": ["desktop"],
            "age_range": {"min": 24, "max": 55},
            "interests": ["technology", "gadgets"],
            "daily_budget": 1500.00,
            "cpm": 3.00
        },
        "statistics": {
            "impressions": 0,
            "clicks": 0,
            "spent": 0.00
        }
    }
]
```

### Targeting and Bidding Logic

```python
# Class to handle ad targeting and bid calculations
class AdTargeting:
    """Class handling ad targeting logic and bid calculations"""
    
    @staticmethod
    def match_targeting_criteria(ad: Dict, request_data: Dict) -> bool:
        """
        Check if an ad matches the targeting criteria based on request data
        Args:
            ad: Dictionary containing ad information and targeting rules
            request_data: Dictionary containing request parameters
        Returns:
            bool: True if ad matches all targeting criteria, False otherwise
        """
        targeting = ad['targeting']
        
        # Verify country match
        if request_data.get('country') not in targeting['countries']:
            return False
            
        # Verify language match
        if request_data.get('language') not in targeting['languages']:
            return False
            
        # Verify device type match
        if request_data.get('device') not in targeting['devices']:
            return False
            
        # Verify age requirements if provided
        user_age = request_data.get('age')
        if user_age and user_age.isdigit():
            user_age = int(user_age)
            if not (targeting['age_range']['min'] <= user_age <= targeting['age_range']['max']):
                return False
                
        # Check if daily budget has been exceeded
        if ad['statistics']['spent'] >= targeting['daily_budget']:
            return False
            
        return True

    @staticmethod
    def calculate_bid_value(ad: Dict, request_data: Dict) -> float:
        """
        Calculate the bid value for an ad based on targeting match quality
        Args:
            ad: Dictionary containing ad information
            request_data: Dictionary containing request parameters
        Returns:
            float: Calculated bid value
        """
        base_bid = ad['targeting']['cpm']
        
        # Apply premium inventory multiplier
        if request_data.get('premium'):
            base_bid *= 1.2
            
        # Calculate interest match multiplier
        user_interests = request_data.get('interests', [])
        if isinstance(user_interests, str):
            user_interests = [i.strip() for i in user_interests.split(',') if i.strip()]
        interest_match = len(set(user_interests) & set(ad['targeting']['interests']))
        interest_multiplier = 1 + (0.1 * interest_match)
        
        return base_bid * interest_multiplier
```

### API Endpoints

#### Serve Ad Endpoint

```python
@app.route('/serve-ad', methods=['GET'])
def serve_ad():
    """
    API endpoint to serve an ad based on targeting criteria
    Query parameters:
        size: Ad size (e.g., '300x250')
        country: Target country code
        language: Target language code
        device: Device type
        age: User age
        interests: Comma-separated list of interests
        premium: Boolean indicating premium inventory
    Returns:
        JSON response with matched ad or error message
    """
    # Parse and normalize request parameters
    request_data = {
        'size': request.args.get('size', '300x250'),
        'country': request.args.get('country', 'US'),
        'language': request.args.get('language', 'en'),
        'device': request.args.get('device', 'desktop'),
        'age': int(request.args.get('age', 0)) if request.args.get('age') else None,
        'interests': request.args.get('interests', '').split(',') if request.args.get('interests') else [],
        'premium': request.args.get('premium', '').lower() == 'true'
    }
    
    # Find matching ads and calculate their bid values
    matching_ads = []
    for ad in ad_inventory:
        if (ad['size'] == request_data['size'] and 
            ad['active'] and 
            AdTargeting.match_targeting_criteria(ad, request_data)):
            
            bid_value = AdTargeting.calculate_bid_value(ad, request_data)
            matching_ads.append((ad, bid_value))
    
    # Return error if no matching ads found
    if not matching_ads:
        return jsonify({
            "error": "No matching ads found",
            "requested_criteria": request_data
        }), 404
    
    # Select highest bidding ad
    matching_ads.sort(key=lambda x: x[1], reverse=True)
    selected_ad, winning_bid = matching_ads[0]
    
    # Record impression and update statistics
    track_impression(selected_ad, winning_bid)
    
    # Remove sensitive targeting and statistics data
    ad_response = selected_ad.copy()
    ad_response.pop('targeting')
    ad_response.pop('statistics')
    
    return jsonify(ad_response)
```

### Tracking Functions

```python
def track_impression(ad: Dict, bid_value: float):
    """
    Track ad impression and update statistics
    Args:
        ad: Dictionary containing ad information
        bid_value: Winning bid value for the impression
    Returns:
        str: Unique impression ID
    """
    impression_id = str(uuid.uuid4())
    impression = {
        "id": impression_id,
        "ad_id": ad['id'],
        "timestamp": datetime.utcnow().isoformat(),
        "ip": request.remote_addr,
        "bid_value": bid_value,
        "user_agent": request.headers.get('User-Agent')
    }
    
    # Update impression count and spent amount
    ad['statistics']['impressions'] += 1
    ad['statistics']['spent'] += bid_value / 1000  # Convert CPM to actual cost

    return impression_id
```

#### Click Tracking Endpoint

```python
@app.route('/ad-click/<int:ad_id>', methods=['GET'])
def track_click(ad_id):
    """
    Track ad clicks for a specific ad
    Args:
        ad_id: ID of the ad that was clicked
    Returns:
        JSON response indicating success or failure
    """
    ad = next((ad for ad in ad_inventory if ad['id'] == ad_id), None)
    if ad:
        ad['statistics']['clicks'] += 1
        return jsonify({"status": "success", "message": "Click tracked"})
    return jsonify({"status": "error", "message": "Ad not found"}), 404
```

#### Statistics Endpoint

```python
@app.route('/ad-stats', methods=['GET'])
def get_statistics():
    """
    Get performance statistics for all ads
    Returns:
        JSON response with ad statistics including impressions, clicks, CTR, and spend
    """
    stats = [{
        "id": ad['id'],
        "name": ad['name'],
        "impressions": ad['statistics']['impressions'],
        "clicks": ad['statistics']['clicks'],
        "ctr": (ad['statistics']['clicks'] / ad['statistics']['impressions'] * 100) 
               if ad['statistics']['impressions'] > 0 else 0,
        "spent": ad['statistics']['spent']
    } for ad in ad_inventory]
    
    return jsonify(stats)
```

#### Test Page Endpoint

```python
@app.route('/')
@app.route('/test')
def test_page():
    """Render the test page template"""
    return render_template('test.html')
```

### Running the Server

```python
# Run the Flask application in debug mode if executed directly
if __name__ == '__main__':
    app.run(debug=True)
```

The above code is a simple ad server that serves ads to the user based on the targeting criteria.

## Key Areas of the code



### Ad inventory

The ad inventory is a list of dictionaries each representing an ad. Each ad dictionary contains metadata, targeting criteria, and performance statistics.

```python 
ad_inventory = [
    {
        "id": 1,
        "name": "Summer Sale Banner",
        "image_url": "https://via.placeholder.com/300x250?text=Summer+Sale",
        "target_url": "https://example.com/summer-sale",
        "size": "300x250",
        "advertiser": "Fashion Store",
        "active": True,
        "targeting": {
            "countries": ["US", "CA", "UK"],  # Countries where ad can be shown
            "languages": ["en"],              # Supported languages
            "devices": ["desktop", "mobile"], # Target devices
            "age_range": {"min": 18, "max": 35}, # Target age range
            "interests": ["fashion", "shopping"], # Target user interests
            "daily_budget": 1000.00,          # Maximum daily spend in USD
            "cpm": 2.50                       # Cost per thousand impressions
        },}
]
```

### Ad targeting

This criteria will determine which ad to serve to the user.

```python
     "targeting": {
            "countries": ["US", "DE", "FR"],
            "languages": ["en", "de", "fr"],
            "devices": ["desktop"],
            "age_range": {"min": 24, "max": 55},
            "interests": ["technology", "gadgets"],
            "daily_budget": 1500.00,
            "cpm": 3.00
        },
```

### Ad bidding

The bidding logic is used to determine which ad to serve to the user.    

```python
    bid_value = AdTargeting.calculate_bid_value(ad, request_data)
    matching_ads.append((ad, bid_value))
```

### Ad serving

The ad serving logic is used to serve the ad to the user.

```python
    ad_response = selected_ad.copy()
    ad_response.pop('targeting')
    ad_response.pop('statistics')
    return jsonify(ad_response)
```

### Ad tracking       

The ad tracking logic is used to track the impressions and clicks of the ad.

```python
    ad['statistics']['impressions'] += 1
    ad['statistics']['spent'] += bid_value / 1000  # Convert CPM to actual cost
```

### Ad reporting

The ad reporting logic is used to report the performance of the ad.

```python
    stats = [{
        "id": ad['id'],
        "name": ad['name'],
        "impressions": ad['statistics']['impressions'],
        "clicks": ad['statistics']['clicks'],
        "ctr": (ad['statistics']['clicks'] / ad['statistics']['impressions'] * 100) 
               if ad['statistics']['impressions'] > 0 else 0,
        "spent": ad['statistics']['spent']
    } for ad in ad_inventory]
```

The above code is a simple ad server that serves ads to the user based on the targeting criteria.