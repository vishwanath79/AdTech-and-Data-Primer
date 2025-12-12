### Foundation Setup (Header Bidding)

> **Interactive Version:** See [Header Bidding Notebook](./03-Header%20Bidding%20Basics.ipynb) to run this code interactively and experiment with the header bidding auction.
> 
> **Download:** Full script available in `programmatic/header_bidding_simple.py`

This section will cover the basics of Header Bidding.

Header Bidding is a technique where the ad server requests the ad from the publisher's ad server and the ad server returns the ad to the publishers ad server.

In our previous section, we created a simple ad server that serves ads to the user. Now, we will create a header bidding ad server that serves ads to the user.

## Header Bidding Concepts

The example shows how a publisher can simulate 4 different SSPs/exchanges that have different targeting criteria, timeouts and floor prices.

```python

demand_partners = [
    {
        "id": "partner_1",
        "name": "PremiumSSP",
        "endpoint": "/bid/premium-ssp",
        "timeout_ms": 1500,
        "active": True,
        "floor_price": 1.0,  # Minimum bid price
        "targeting": {
            "countries": ["US", "CA", "UK"],
            "devices": ["desktop", "mobile"],
            "interests": ["finance", "technology"]
        }
    },
    {
        "id": "partner_2", 
        "name": "GlobalExchange",
        "endpoint": "/bid/global-exchange",
        "timeout_ms": 1200,
        "active": True,
        "floor_price": 0.5,
        "targeting": {
            "countries": ["US", "DE", "FR", "UK"],
            "devices": ["desktop", "mobile", "tablet"],
            "interests": ["general", "news", "sports"]
        }
    },
```
### Targeting Criteria

The targeting criteria is used to determine which ad to serve to the user. This contains country, device and interest-based targeting. It also simulates a floor price for the ad.

The auction is run by the publisher's ad server. The publisher's ad server then sends the ad to the SSP/exchange.

```python

class HeaderBiddingAuction:
    """Class handling header bidding auction logic"""
    
    def __init__(self, timeout_ms: int = 2000):
        self.timeout_ms = timeout_ms
        self.executor = ThreadPoolExecutor(max_workers=10)
        
    def matches_targeting(self, partner: Dict, request_data: Dict) -> bool:
        """Check if request matches partner's targeting criteria"""
        targeting = partner['targeting']
        
        # Check country targeting
        if request_data.get('country') not in targeting['countries']:
            return False
            
        # Check device targeting
        if request_data.get('device') not in targeting['devices']:
            return False
            
        # Check interest overlap
        user_interests = set(request_data.get('interests', []))
        partner_interests = set(targeting['interests'])
        if not user_interests & partner_interests:  # No overlap
            return False
            
        return True
```

### Bid Sumlation Logic

The bid logic simulates a bid request to the demand partner. The sample code simulates network latency and processing time.It checkes the targeting criteria and returns a bid response.

```python

def simulate_partner_bid(self, partner: Dict, request_data: Dict) -> Dict:
        """Simulate a bid request to a demand partner"""
        start_time = time.time()
        
        try:
            # Simulate network latency and processing time
            processing_time = min(partner['timeout_ms'] / 1000.0, 0.1 + (hash(partner['id']) % 100) / 1000.0)
            time.sleep(processing_time)
            
            # Check if partner can bid on this request
            if not self.matches_targeting(partner, request_data):
                return {
                    "partner_id": partner['id'],
                    "partner_name": partner['name'],
                    "status": "no_bid",
                    "reason": "targeting_mismatch",
                    "response_time_ms": (time.time() - start_time) * 1000
                }
```

### Bid Response

The bid response is a dictionary that contains the partner id, partner name, status, reason and response time.

```python


    {
        "partner_id": "partner_1",
        "partner_name": "PremiumSSP",
        "status": "no_bid",
        "reason": "targeting_mismatch",
        "response_time_ms": 1500
    }

```

### Auction Logic

The auction logic is used to determine which ad to serve to the user. This is done by the publisher's ad server.

```python


    def run_header_bidding_auction(self, request_data: Dict) -> Dict:
        """Run header bidding auction"""
        bids = []
        
        for partner in self.demand_partners:
            bid_response = self.simulate_partner_bid(partner, request_data)
            bids.append(bid_response)
```


### Auction Response

The auction response is a dictionary that contains the bids.

```python

    {
        "bids": [
            {
                "partner_id": "partner_1",
                "partner_name": "PremiumSSP",
                "status": "no_bid",
                "reason": "targeting_mismatch",
                "response_time_ms": 1500
            }
        ]
    }
```    

The script provides a realistic simulation of header bidding mechanics with parallel partner calls, timeout handling, and winner selection.

## Complete Implementation

The full implementation is organized into sections: imports, demand partner configuration, auction logic class, and Flask API endpoints.

### Imports and Setup

```python
# Import necessary libraries
from flask import Flask, jsonify, request, render_template_string
from datetime import datetime
import json
from typing import Dict, List
import uuid
import time
import asyncio
import threading
from concurrent.futures import ThreadPoolExecutor, as_completed

# Initialize Flask application
app = Flask(__name__)
```

### Demand Partner Configuration

```python
# Configuration of demand partners (SSPs/Exchanges)
demand_partners = [
    {
        "id": "partner_1",
        "name": "PremiumSSP",
        "endpoint": "/bid/premium-ssp",
        "timeout_ms": 1500,
        "active": True,
        "floor_price": 1.0,  # Minimum bid price
        "targeting": {
            "countries": ["US", "CA", "UK"],
            "devices": ["desktop", "mobile"],
            "interests": ["finance", "technology"]
        }
    },
    {
        "id": "partner_2", 
        "name": "GlobalExchange",
        "endpoint": "/bid/global-exchange",
        "timeout_ms": 1200,
        "active": True,
        "floor_price": 0.5,
        "targeting": {
            "countries": ["US", "DE", "FR", "UK"],
            "devices": ["desktop", "mobile", "tablet"],
            "interests": ["general", "news", "sports"]
        }
    },
    {
        "id": "partner_3",
        "name": "MobileFirst",
        "endpoint": "/bid/mobile-first",
        "timeout_ms": 800,
        "active": True,
        "floor_price": 2.0,
        "targeting": {
            "countries": ["US"],
            "devices": ["mobile"],
            "interests": ["gaming", "apps", "mobile"]
        }
    },
    {
        "id": "partner_4",
        "name": "VideoSSP",
        "endpoint": "/bid/video-ssp", 
        "timeout_ms": 2000,
        "active": True,
        "floor_price": 3.0,
        "targeting": {
            "countries": ["US", "CA"],
            "devices": ["desktop", "mobile"],
            "interests": ["video", "entertainment"]
        }
    }
]
```

### Header Bidding Auction Class

```python
class HeaderBiddingAuction:
    """Class handling header bidding auction logic"""
    
    def __init__(self, timeout_ms: int = 2000):
        self.timeout_ms = timeout_ms
        self.executor = ThreadPoolExecutor(max_workers=10)
        
    def matches_targeting(self, partner: Dict, request_data: Dict) -> bool:
        """Check if request matches partner's targeting criteria"""
        targeting = partner['targeting']
        
        # Check country targeting
        if request_data.get('country') not in targeting['countries']:
            return False
            
        # Check device targeting
        if request_data.get('device') not in targeting['devices']:
            return False
            
        # Check interest overlap
        user_interests = set(request_data.get('interests', []))
        partner_interests = set(targeting['interests'])
        if not user_interests & partner_interests:  # No overlap
            return False
            
        return True
    
    def simulate_partner_bid(self, partner: Dict, request_data: Dict) -> Dict:
        """Simulate a bid request to a demand partner"""
        start_time = time.time()
        
        try:
            # Simulate network latency and processing time
            processing_time = min(partner['timeout_ms'] / 1000.0, 0.1 + (hash(partner['id']) % 100) / 1000.0)
            time.sleep(processing_time)
            
            # Check if partner can bid on this request
            if not self.matches_targeting(partner, request_data):
                return {
                    "partner_id": partner['id'],
                    "partner_name": partner['name'],
                    "status": "no_bid",
                    "reason": "targeting_mismatch",
                    "response_time_ms": (time.time() - start_time) * 1000
                }
            
            # Calculate bid price based on targeting match and demand
            base_bid = partner['floor_price']
            
            # Add premium for better targeting match
            user_interests = set(request_data.get('interests', []))
            partner_interests = set(partner['targeting']['interests'])
            interest_match = len(user_interests & partner_interests)
            bid_multiplier = 1.0 + (interest_match * 0.2)  # 20% per matching interest
            
            # Add randomness to simulate market dynamics
            market_factor = 0.8 + (hash(str(request_data) + partner['id']) % 100) / 250.0  # 0.8 to 1.2x
            
            final_bid = base_bid * bid_multiplier * market_factor
            
            # Simulate occasional bid failures
            if hash(partner['id'] + str(time.time())) % 20 == 0:  # 5% failure rate
                return {
                    "partner_id": partner['id'],
                    "partner_name": partner['name'],
                    "status": "error",
                    "reason": "internal_error",
                    "response_time_ms": (time.time() - start_time) * 1000
                }
            
            return {
                "partner_id": partner['id'],
                "partner_name": partner['name'],
                "status": "bid",
                "bid_cpm": round(final_bid, 2),
                "ad_markup": f"<div>Ad from {partner['name']} - CPM: ${final_bid:.2f}</div>",
                "creative_id": f"creative_{partner['id']}_{int(time.time())}",
                "response_time_ms": round((time.time() - start_time) * 1000, 2)
            }
            
        except Exception as e:
            return {
                "partner_id": partner['id'],
                "partner_name": partner['name'],
                "status": "timeout",
                "reason": str(e),
                "response_time_ms": (time.time() - start_time) * 1000
            }
    
    def run_header_bidding_auction(self, request_data: Dict) -> Dict:
        """Run parallel header bidding auction with timeout"""
        auction_start = time.time()
        
        # Filter active partners
        active_partners = [p for p in demand_partners if p['active']]
        
        # Submit bid requests to all partners in parallel
        future_to_partner = {
            self.executor.submit(self.simulate_partner_bid, partner, request_data): partner
            for partner in active_partners
        }
        
        bids = []
        timeouts = []
        errors = []
        
        # Collect responses with timeout
        for future in as_completed(future_to_partner, timeout=self.timeout_ms/1000.0):
            try:
                result = future.result()
                if result['status'] == 'bid':
                    bids.append(result)
                elif result['status'] == 'timeout':
                    timeouts.append(result)
                else:
                    errors.append(result)
            except Exception as e:
                partner = future_to_partner[future]
                errors.append({
                    "partner_id": partner['id'],
                    "partner_name": partner['name'],
                    "status": "exception",
                    "reason": str(e),
                    "response_time_ms": (time.time() - auction_start) * 1000
                })
        
        # Handle any remaining futures that didn't complete
        for future in future_to_partner:
            if not future.done():
                future.cancel()
                partner = future_to_partner[future]
                timeouts.append({
                    "partner_id": partner['id'],
                    "partner_name": partner['name'],
                    "status": "timeout",
                    "reason": "auction_timeout",
                    "response_time_ms": self.timeout_ms
                })
        
        # Select winning bid (highest CPM)
        winning_bid = None
        if bids:
            winning_bid = max(bids, key=lambda x: x['bid_cpm'])
        
        total_auction_time = (time.time() - auction_start) * 1000
        
        return {
            "auction_id": str(uuid.uuid4()),
            "timestamp": datetime.utcnow().isoformat(),
            "total_auction_time_ms": round(total_auction_time, 2),
            "partners_called": len(active_partners),
            "successful_bids": len(bids),
            "timeouts": len(timeouts),
            "errors": len(errors),
            "winning_bid": winning_bid,
            "all_bids": bids,
            "failed_responses": timeouts + errors
        }

# Initialize global auction manager
auction_manager = HeaderBiddingAuction()
```

### Flask API Endpoints

#### Main Header Bidding Endpoint

```python
@app.route('/header-bidding', methods=['POST'])
def run_header_bidding():
    """
    Main header bidding endpoint that runs parallel auction
    
    Expected JSON payload:
    {
        "ad_unit_id": "banner_300x250_1",
        "size": "300x250",
        "country": "US",
        "device": "desktop", 
        "interests": ["technology", "finance"],
        "user_id": "user_123",
        "page_url": "https://example.com/article",
        "timeout_ms": 2000
    }
    """
    try:
        data = request.get_json()
        
        # Set custom timeout if provided
        timeout_ms = data.get('timeout_ms', 2000)
        auction_manager.timeout_ms = timeout_ms
        
        # Prepare request data
        request_data = {
            'ad_unit_id': data.get('ad_unit_id', 'default_unit'),
            'size': data.get('size', '300x250'),
            'country': data.get('country', 'US'),
            'device': data.get('device', 'desktop'),
            'interests': data.get('interests', []),
            'user_id': data.get('user_id', 'anonymous'),
            'page_url': data.get('page_url', ''),
        }
        
        # Run the header bidding auction
        auction_result = auction_manager.run_header_bidding_auction(request_data)
        
        return jsonify(auction_result)
        
    except Exception as e:
        return jsonify({
            "error": "header_bidding_failed",
            "message": str(e),
            "timestamp": datetime.utcnow().isoformat()
        }), 500
```

#### Partner Management Endpoints

```python
@app.route('/partners', methods=['GET'])
def get_demand_partners():
    """Get list of configured demand partners"""
    return jsonify({
        "partners": demand_partners,
        "total_partners": len(demand_partners),
        "active_partners": len([p for p in demand_partners if p['active']])
    })

@app.route('/partner/<partner_id>/toggle', methods=['POST'])
def toggle_partner(partner_id):
    """Enable/disable a demand partner"""
    partner = next((p for p in demand_partners if p['id'] == partner_id), None)
    if not partner:
        return jsonify({"error": "partner_not_found"}), 404
    
    partner['active'] = not partner['active']
    return jsonify({
        "partner_id": partner_id,
        "partner_name": partner['name'],
        "active": partner['active']
    })
```

#### Test Page Endpoint

```python
@app.route('/test-hb')
def test_header_bidding_page():
    """Render a test page for header bidding"""
    html_template = """
    <!DOCTYPE html>
    <html>
    <head>
        <title>Header Bidding Test</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 40px; }
            .ad-slot { border: 2px dashed #ccc; padding: 20px; margin: 20px 0; text-align: center; }
            .controls { background: #f5f5f5; padding: 20px; margin: 20px 0; }
            .results { background: #e8f4f8; padding: 20px; margin: 20px 0; }
            button { padding: 10px 20px; margin: 5px; }
            input, select { margin: 5px; padding: 5px; }
            .bid-result { border: 1px solid #ddd; padding: 10px; margin: 5px 0; }
            .winning-bid { background: #d4edda; border-color: #c3e6cb; }
        </style>
    </head>
    <body>
        <h1>Header Bidding Demo</h1>
        
        <div class="controls">
            <h3>Auction Parameters</h3>
            <label>Country: 
                <select id="country">
                    <option value="US">United States</option>
                    <option value="UK">United Kingdom</option>
                    <option value="DE">Germany</option>
                    <option value="CA">Canada</option>
                </select>
            </label><br>
            
            <label>Device: 
                <select id="device">
                    <option value="desktop">Desktop</option>
                    <option value="mobile">Mobile</option>
                    <option value="tablet">Tablet</option>
                </select>
            </label><br>
            
            <label>Interests (comma-separated): 
                <input type="text" id="interests" value="technology,finance" placeholder="technology,finance">
            </label><br>
            
            <label>Timeout (ms): 
                <input type="number" id="timeout" value="2000" min="500" max="5000">
            </label><br>
            
            <button onclick="runHeaderBidding()">Run Header Bidding Auction</button>
            <button onclick="clearResults()">Clear Results</button>
        </div>
        
        <div class="ad-slot" id="ad-slot">
            <p>Ad Slot (300x250)</p>
            <p>Click "Run Header Bidding Auction" to see results</p>
        </div>
        
        <div class="results" id="results" style="display:none;">
            <h3>Auction Results</h3>
            <div id="auction-summary"></div>
            <div id="bid-details"></div>
        </div>

        <script>
        async function runHeaderBidding() {
            const country = document.getElementById('country').value;
            const device = document.getElementById('device').value;
            const interests = document.getElementById('interests').value.split(',').map(s => s.trim());
            const timeout = parseInt(document.getElementById('timeout').value);
            
            const payload = {
                ad_unit_id: 'test_banner_300x250',
                size: '300x250',
                country: country,
                device: device,
                interests: interests,
                user_id: 'test_user_' + Date.now(),
                page_url: window.location.href,
                timeout_ms: timeout
            };
            
            try {
                document.getElementById('ad-slot').innerHTML = '<p>Running auction...</p>';
                
                const response = await fetch('/header-bidding', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });
                
                const result = await response.json();
                displayResults(result);
                
            } catch (error) {
                document.getElementById('ad-slot').innerHTML = '<p>Error: ' + error.message + '</p>';
            }
        }
        
        function displayResults(result) {
            const resultsDiv = document.getElementById('results');
            const summaryDiv = document.getElementById('auction-summary');
            const detailsDiv = document.getElementById('bid-details');
            
            // Show auction summary
            summaryDiv.innerHTML = `
                <p><strong>Auction ID:</strong> ${result.auction_id}</p>
                <p><strong>Total Time:</strong> ${result.total_auction_time_ms}ms</p>
                <p><strong>Partners Called:</strong> ${result.partners_called}</p>
                <p><strong>Successful Bids:</strong> ${result.successful_bids}</p>
                <p><strong>Timeouts:</strong> ${result.timeouts}</p>
                <p><strong>Errors:</strong> ${result.errors}</p>
            `;
            
            // Show winning bid in ad slot
            if (result.winning_bid) {
                document.getElementById('ad-slot').innerHTML = `
                    <h4>WINNING AD</h4>
                    <p><strong>${result.winning_bid.partner_name}</strong></p>
                    <p>CPM: $${result.winning_bid.bid_cpm}</p>
                    <p>Response Time: ${result.winning_bid.response_time_ms}ms</p>
                    <div style="background: #fff; border: 1px solid #ddd; padding: 10px;">
                        ${result.winning_bid.ad_markup}
                    </div>
                `;
            } else {
                document.getElementById('ad-slot').innerHTML = '<p>No winning bid - showing fallback ad</p>';
            }
            
            // Show all bid details
            let bidsHtml = '<h4>All Bid Responses:</h4>';
            
            result.all_bids.forEach(bid => {
                const isWinner = result.winning_bid && bid.partner_id === result.winning_bid.partner_id;
                bidsHtml += `
                    <div class="bid-result ${isWinner ? 'winning-bid' : ''}">
                        <strong>${bid.partner_name}</strong> ${isWinner ? '(WINNER)' : ''}
                        <br>CPM: $${bid.bid_cpm} | Response: ${bid.response_time_ms}ms
                    </div>
                `;
            });
            
            result.failed_responses.forEach(failed => {
                bidsHtml += `
                    <div class="bid-result" style="background: #f8d7da;">
                        <strong>${failed.partner_name}</strong> - ${failed.status.toUpperCase()}
                        <br>Reason: ${failed.reason} | Time: ${failed.response_time_ms}ms
                    </div>
                `;
            });
            
            detailsDiv.innerHTML = bidsHtml;
            resultsDiv.style.display = 'block';
        }
        
        function clearResults() {
            document.getElementById('results').style.display = 'none';
            document.getElementById('ad-slot').innerHTML = `
                <p>Ad Slot (300x250)</p>
                <p>Click "Run Header Bidding Auction" to see results</p>
            `;
        }
        </script>
    </body>
    </html>
    """
    return render_template_string(html_template)
```

#### Index Page

```python
@app.route('/')
def index():
    """Main index page"""
    return '''
    <h1>Header Bidding Server</h1>
    <p>Available endpoints:</p>
    <ul>
        <li><a href="/test-hb">Test Header Bidding (Interactive Demo)</a></li>
        <li><strong>POST /header-bidding</strong> - Run header bidding auction</li>
        <li><a href="/partners">GET /partners</a> - View demand partners</li>
        <li><strong>POST /partner/&lt;id&gt;/toggle</strong> - Enable/disable partner</li>
    </ul>
    '''
```

### Running the Server

```python
if __name__ == '__main__':
    app.run(port=5002, debug=True)
```


