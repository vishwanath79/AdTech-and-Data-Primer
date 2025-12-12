### Real Time Bidding

> **Interactive Version:** See [Real Time Bidding Notebook](./04-Real%20Time%20Bidding.ipynb) to run this code interactively and experiment with RTB auctions.
> 
> **Download:** Full script available in `programmatic/rtb_simple.py`

This section will cover the basics of Real Time Bidding.

RTB is a type of programmatic advertising where advertisers bid on the available inventory in real time.

## How does RTB work?

The process of RTB is as follows:

1. The publisher sends the available inventory to the ad exchange.
2. The ad exchange sends the inventory to the DSPs.
3. Multiple DSPs bid on the inventory.
4. The ad exchange selects the highest bidder and serves the ad to the user.


![RTB Process](../assets/images/rtb-process.png)


## Bid Response

The bid responses need to follow the Open RTB specification.

A sample bid response is as below. The bid below represent a 'Premium DSP' that has a daily budget of $10,000 and has a base cpm of $3.50. It also has a max cpm of $15.00 and a bid floor multiplier of 1.2. It also has a targeting boost of 0.3.


```python

rtb_bidders = [
    {
        "id": "dsp_premium",
        "name": "Premium DSP",
        "endpoint": "http://premium-dsp.com/bid",
        "seat_id": "premium_seat_001",
        "active": True,
        "daily_budget": 10000.0,
        "spent_today": 0.0,
        "targeting": {
            "countries": ["US", "CA", "UK", "AU"],
            "languages": ["en"],
            "devices": ["desktop", "mobile"],
            "age_range": {"min": 25, "max": 54},
            "interests": ["finance", "technology", "business"],
            "min_viewability": 70,  # Minimum viewability percentage
        },
        "bidding": {
            "base_cpm": 3.50,
            "max_cpm": 15.00,
            "bid_floor_multiplier": 1.2,  # Bid at least 20% above floor
            "targeting_boost": 0.3,  # 30% boost for good targeting match
        }
    },
]
```

## RTB Exchange

This is the exchange that handles the bid requests and auctions.
In the sample code, we have a simple RTB exchange that handles bid request and auctions.

1) It creates a unique bid request id and impression id.

This identifies unique impressions for each bid request.

```python
bid_request_id = str(uuid.uuid4())
impression_id = str(uuid.uuid4())
```

2) It builds the user object.

The user object is used to target the user based on their interests, demographics and location.

```python
user_obj = {
    "id": request_data.get('user_id', 'anonymous_' + str(int(time.time()))),
    "geo": {
        "country": request_data.get('country', 'US'),
        "region": request_data.get('region', 'CA'),
        "city": request_data.get('city', 'San Francisco'),
        "type": 2  # IP-derived location
    }
}
```

3) It builds the device object.

The device object is used to target the device based on its type, browser, operating system and language.

```python
device_obj = {
    "devicetype": self._get_device_type(request_data.get('device', 'desktop')),
    "ua": request_data.get('user_agent', 'Mozilla/5.0 (compatible; RTB-Bot/1.0)'),
    "ip": request_data.get('ip', '192.168.1.1'),
    "language": request_data.get('language', 'en'),
    "os": request_data.get('os', 'Unknown')
}
```

4) It builds the impression object.

The impression object is used to target the impression based on its size, position and format.

```python
impression_obj = {
    "id": impression_id,
    "banner": {
        "w": width,
        "h": height,
        "format": [{"w": width, "h": height}],
        "pos": request_data.get('ad_position', 1),  # Above the fold
        "topframe": 1 if request_data.get('top_frame', True) else 0
    },
    "bidfloor": request_data.get('floor_price', 0.50),
    "bidfloorcur": "USD",
    "secure": 1 if request_data.get('secure', True) else 0,
    "tagid": request_data.get('ad_unit_id', 'default_unit')
}
```

5) It builds the site object.

The site object is used to target the site based on its domain, category and page.

```python
site_obj = {
    "id": request_data.get('site_id', 'demo_site_001'),
    "name": request_data.get('site_name', 'Demo Publisher Site'),
    "domain": request_data.get('domain', 'demo-publisher.com'),
    "cat": request_data.get('site_categories', ['IAB1']),  # Arts & Entertainment
    "page": request_data.get('page_url', 'https://demo-publisher.com/article'),
    "publisher": {
        "id": self.publisher_id,
        "name": "Demo Publisher",
        "cat": ["IAB1"]
    }
}
```

6) All the above are combined to build the bid request object.

```python
bid_request = {
    "id": bid_request_id,
    "imp": [impression_obj],
    "site": site_obj,
    "user": user_obj,
    "device": device_obj,
}
return bid_request
```

### Auction Logic

The auction logic is used to determine which ad to serve to the user. This is done by the publisher's ad server.

In the sample code, we have a simple auction logic that selects the highest bidder and serves the ad to the user.

```python
def run_rtb_auction(self, request_data: Dict) -> Dict:
    """Run RTB auction"""
    bids = []
    
    for bidder in self.rtb_bidders:
        bid_response = self.simulate_bidder_bid(bidder, request_data)
        bids.append(bid_response)
```

### Auction Process

The auction process is used to determine the winning bid.

The process goes like this:
        
        1. The Ad Exchange collects valid bids from all bidders
        2. It then sorts the bids by (price * targeting_score) to find winner
        3. This determines the clearing price (second-price auction)
        4. The Ad Exchange then updates the winner's spend
        5. Finally, it stores the auction results
        
Note - second-price auction is a model where the winner pays the second-highest bid price.
        

```python

valid_bids = self.collect_bids(bid_request)
        
        if not valid_bids:
            return AuctionResult(status="no_bids")
            
        # Sort bids by effective bid value (price * targeting quality)
        sorted_bids = sorted(
            valid_bids,
            key=lambda x: x.price * x.targeting_match_score,
            reverse=True
        )
        
        winning_bid = sorted_bids[0]
        
        # In a second-price auction, winner pays the second-highest bid price
        clearing_price = sorted_bids[1].price if len(sorted_bids) > 1 else winning_bid.price
        
        # Update the winner's spent amount
        self._update_bidder_spend(winning_bid.bidder_name, clearing_price)
        
        # Store auction result for analytics
        self._store_auction_result(bid_request, winning_bid, clearing_price)
        
```

### Auction Result

The sample code uses Redis to store the auction results in a dictionary. THis data includes auction id, timestamp, bid request details, winner information, pricing information and targeting performance.
    

```python

 result = {
            "auction_id": str(uuid.uuid4()),
            "timestamp": datetime.utcnow().isoformat(),
            "bid_request_id": bid_request.id,
            "winning_bidder": winning_bid.bidder_name,
            "winning_price": winning_bid.price,
            "clearing_price": clearing_price,
            "user_country": bid_request.user.country,
            "targeting_score": winning_bid.targeting_match_score
        }
        

```

