### Deal IDs & Private Marketplace (PMP)

Ever notice how the ads on The New York Times or Wall Street Journal never seem to be for sketchy weight-loss pills or "one weird trick" scams? That's Private Marketplace deals at work—the VIP lounge of programmatic advertising where publishers get to pick who shows up to the party.

Unlike the open exchange (which is basically the digital equivalent of selling ad space on Craigslist), PMP lets premium publishers say "you, you, and you can bid—but the rest of you tech bros, not today."

## What's a Private Marketplace, Really?

Think of it like Costco, but for ads. You need a membership to get in, and once you're inside, you get access to the good stuff at prices that were negotiated upfront. No surprise $47 CPMs at 2 AM.

Publishers love PMPs because they:
- **Control who sees their inventory** (no sketchy advertisers next to their Pulitzer-winning journalism)
- **Lock in better prices** ($5-$20 CPMs vs the $0.50-$2 wasteland of open exchanges)
- **Guarantee brand safety** (Nike doesn't want to appear next to conspiracy theories or adult content)
- **Build actual relationships** with advertisers instead of treating everyone like a random bidder on eBay

### PMP vs Open Exchange (The Difference)

| Feature | Open Exchange | Private Marketplace |
|---------|--------------|---------------------|
| Access | Anyone with a credit card | Invitation only |
| Pricing | Whatever chaos dictates | Pre-negotiated |
| Inventory | The leftover pizza | The good stuff |
| Transparency | "Trust us" | Actual contracts |
| Brand Safety | Roll the dice | Guaranteed |
| Floor Price | $0.50-$2 (yikes) | $5-$20+ (worth it) |

## Three Flavors of PMP Deals

### 1. Preferred Deals (First Dibs, No Promises)

This is like being first in line at a yard sale. You get to see the inventory before it goes to auction, at a fixed price. But if you don't want it? No problem—publisher moves on to the next buyer.

```python
preferred_deal = {
    "deal_id": "PREF-12345",
    "deal_type": "preferred",
    "publisher": "Premium News Publisher",
    "buyer_seat_id": "DSP-BRAND-001",
    "price": 8.50,  # Fixed CPM - take it or leave it
    "currency": "USD",
    "guaranteed": False,  # Not guaranteed to fill
    "priority": 1,  # You get first look
    "inventory": {
        "site": "premium-news.com",
        "sections": ["/business", "/technology"],
        "ad_sizes": ["728x90", "300x250", "970x250"],
        "position": "above_fold"  # The primo real estate
    },
    "terms": {
        "start_date": "2025-01-01",
        "end_date": "2025-12-31",
        "viewability_guarantee": 70,  # 70% of impressions must be viewable
        "brand_safety": True
    }
}
```

### 2. Programmatic Guaranteed (Reserved Seats)

This is the airline's first-class section—you paid for it, you're getting it. Publisher guarantees you a specific number of impressions at a fixed price. Like a traditional insertion order (IO) but without the endless email chains and PDF attachments.

```python
programmatic_guaranteed = {
    "deal_id": "PG-67890",
    "deal_type": "programmatic_guaranteed",
    "publisher": "Premium Lifestyle Magazine",
    "buyer_seat_id": "AGENCY-ATD-001",
    "price": 12.00,  # Premium inventory = premium price
    "currency": "USD",
    "guaranteed": True,  # You WILL get these impressions
    "guaranteed_impressions": 1000000,  # 1M impressions or bust
    "daily_cap": 50000,
    "inventory": {
        "site": "lifestyle-magazine.com",
        "sections": ["/homepage"],
        "ad_sizes": ["970x250"],  # Homepage takeover
        "format": "display"
    },
    "terms": {
        "start_date": "2025-06-01",
        "end_date": "2025-08-31",
        "billing": "CPM",
        "makegoods": True,  # If they underdeliver, they comp you
        "cancellation_notice": "30_days"
    },
    "delivered_impressions": 0  # Track actual delivery
}
```

### 3. Private Auction (Closed Bidding War)

Multiple invited buyers compete, but it's not a free-for-all. Publisher sets a floor price and only lets in the brands they trust. Think silent auction at a charity gala, not a mob scene at a Black Friday sale.

```python
private_auction = {
    "deal_id": "PA-11223",
    "deal_type": "private_auction",
    "publisher": "Sports Network",
    "invited_buyers": [
        "DSP-001",  # Only these four get to bid
        "DSP-002",
        "DSP-003",
        "DSP-004"
    ],
    "floor_price": 5.00,  # Don't even think about bidding lower
    "currency": "USD",
    "auction_type": "second_price",  # Winner pays second-highest bid
    "inventory": {
        "site": "sports-network.com",
        "categories": ["sports", "fitness"],
        "ad_sizes": ["300x600", "300x250"],
        "device_types": ["desktop", "mobile"]
    },
    "terms": {
        "start_date": "2025-03-01",
        "end_date": "2025-05-31",
        "viewability_guarantee": 65
    }
}
```

## How Deal IDs Travel (The Technical Bit)

Deal IDs live inside the bid request in a `pmp` object. It's like a secret handshake—if you've got the right deal ID, you get access to the premium inventory. No deal ID? Enjoy the open auction with everyone else.

Here's what it looks like in OpenRTB 2.5:

```python
# Bid request with PMP deal
bid_request_with_pmp = {
    "id": "abc123",
    "imp": [{
        "id": "1",
        "banner": {
            "w": 728,
            "h": 90,
            "pos": 1  # Above the fold = more expensive
        },
        "pmp": {
            "private_auction": 1,  # 1 = deals only, no open auction fallback
            "deals": [
                {
                    "id": "PREF-12345",  # The magic ticket
                    "bidfloor": 8.50,     # Minimum bid for this deal
                    "bidfloorcur": "USD",
                    "at": 2,              # 1=first price, 2=second price
                    "wseat": ["DSP-BRAND-001"],  # Who's allowed to bid
                    "wadomain": ["brand.com"],   # Advertiser whitelist
                    "ext": {
                        "priority": 1,
                        "deal_type": "preferred"
                    }
                }
            ]
        },
        "bidfloor": 2.00,  # Open auction floor (way cheaper)
        "bidfloorcur": "USD"
    }],
    "site": {
        "id": "site123",
        "name": "Premium News Publisher",
        "domain": "premium-news.com",
        "cat": ["IAB12"]  # News category
    },
    "user": {
        "id": "user456"
    },
    "device": {
        "ua": "Mozilla/5.0...",
        "ip": "192.168.1.1"
    }
}
```

## Managing PMP Deals (The Code)

Here's how to match deals to buyers and track which ones are eligible:

```python
from datetime import datetime

class PMPDealManager:
    """Manage your PMP deals without losing your mind"""
    
    def __init__(self):
        self.deals = []
    
    def add_deal(self, deal):
        """Add a new deal to the system"""
        # Basic validation - don't skip this or you'll regret it later
        required_fields = ['deal_id', 'deal_type', 'price', 'buyer_seat_id']
        for field in required_fields:
            if field not in deal:
                raise ValueError(f"Missing required field: {field}")
        
        self.deals.append(deal)
        print(f"Added deal: {deal['deal_id']}")
    
    def find_eligible_deals(self, bid_request, buyer_seat_id):
        """
        Find which deals this buyer can actually access
        Returns the good stuff first (guaranteed > preferred > private auction)
        """
        eligible = []
        imp = bid_request['imp'][0]
        site = bid_request.get('site', {})
        
        for deal in self.deals:
            # Active date range check
            if not self._is_active(deal):
                continue
            
            # Is this buyer even invited?
            if not self._is_buyer_eligible(deal, buyer_seat_id):
                continue
            
            # Does the inventory match what they're looking for?
            if not self._matches_inventory(deal, imp, site):
                continue
            
            # For guaranteed deals, check if we still have capacity
            if deal['deal_type'] == 'programmatic_guaranteed':
                if not self._has_capacity(deal):
                    continue
            
            eligible.append(deal)
        
        # Sort by what matters: guaranteed first, then by price
        eligible.sort(key=lambda d: (
            d['deal_type'] == 'programmatic_guaranteed',
            d.get('priority', 99),
            -d['price']
        ))
        
        return eligible
    
    def _is_active(self, deal):
        """Check if deal is still valid (not expired)"""
        if 'terms' not in deal:
            return True
        
        terms = deal['terms']
        now = datetime.now()
        
        start = datetime.fromisoformat(terms.get('start_date', '2020-01-01'))
        end = datetime.fromisoformat(terms.get('end_date', '2099-12-31'))
        
        return start <= now <= end
    
    def _is_buyer_eligible(self, deal, buyer_seat_id):
        """Check if this buyer is on the list"""
        if deal['deal_type'] == 'private_auction':
            return buyer_seat_id in deal.get('invited_buyers', [])
        else:
            return deal.get('buyer_seat_id') == buyer_seat_id
    
    def _matches_inventory(self, deal, imp, site):
        """Check if the ad size and site match the deal terms"""
        inventory = deal.get('inventory', {})
        
        # Site domain must match
        if 'site' in inventory:
            if site.get('domain') != inventory['site']:
                return False
        
        # Ad size must match
        if 'banner' in imp and 'ad_sizes' in inventory:
            size = f"{imp['banner']['w']}x{imp['banner']['h']}"
            if size not in inventory['ad_sizes']:
                return False
        
        return True
    
    def _has_capacity(self, deal):
        """Check if guaranteed deal hasn't hit its cap yet"""
        if 'guaranteed_impressions' not in deal:
            return True
        
        delivered = deal.get('delivered_impressions', 0)
        total = deal['guaranteed_impressions']
        
        return delivered < total
    
    def record_win(self, deal_id):
        """Track impression delivery (important for guaranteed deals)"""
        for deal in self.deals:
            if deal['deal_id'] == deal_id:
                if 'delivered_impressions' not in deal:
                    deal['delivered_impressions'] = 0
                deal['delivered_impressions'] += 1
                return True
        return False

# Usage example
pmp_manager = PMPDealManager()

# Add your deals
pmp_manager.add_deal(preferred_deal)
pmp_manager.add_deal(programmatic_guaranteed)
pmp_manager.add_deal(private_auction)

# Find what this buyer can access
eligible = pmp_manager.find_eligible_deals(
    bid_request_with_pmp,
    buyer_seat_id="DSP-BRAND-001"
)

print(f"\nFound {len(eligible)} eligible deals:")
for deal in eligible:
    print(f"  - {deal['deal_id']}: {deal['deal_type']} @ ${deal['price']} CPM")
```

## DSP Bidding Logic (With Deals)

DSPs prioritize deals over open auctions—better inventory, better performance, happier clients.

```python
class DSPWithDeals:
    """DSP that knows how to play the PMP game"""
    
    def __init__(self, seat_id, budget):
        self.seat_id = seat_id
        self.budget = budget
        self.spent = 0.0
    
    def create_bid(self, bid_request, eligible_deals):
        """
        Bid on deals first, fall back to open auction if necessary
        """
        imp = bid_request['imp'][0]
        
        # Got deals? Use 'em
        if eligible_deals:
            return self._bid_on_deal(bid_request, eligible_deals[0])
        
        # Check if open auction is even allowed
        pmp = imp.get('pmp', {})
        if pmp.get('private_auction') == 1:
            # Nope, deals only
            return None
        
        return self._bid_open_auction(bid_request)
    
    def _bid_on_deal(self, bid_request, deal):
        """Bid on a PMP deal at the agreed price"""
        imp = bid_request['imp'][0]
        bid_price = deal['price']
        
        # Budget check (important!)
        if self.spent + (bid_price / 1000) > self.budget:
            return None
        
        # Create bid response with the deal ID
        return {
            "id": bid_request['id'],
            "seatbid": [{
                "seat": self.seat_id,
                "bid": [{
                    "id": f"bid_{deal['deal_id']}",
                    "impid": imp['id'],
                    "price": bid_price,
                    "dealid": deal['deal_id'],  # Critical: include this
                    "adm": f"<div>Premium ad for {deal['deal_id']}</div>",
                    "crid": "creative_premium_001",
                    "w": imp['banner']['w'],
                    "h": imp['banner']['h']
                }]
            }],
            "cur": "USD"
        }
    
    def _bid_open_auction(self, bid_request):
        """Fall back to open auction if no deals available"""
        imp = bid_request['imp'][0]
        floor = imp.get('bidfloor', 1.0)
        bid_price = floor * 1.2  # Bid slightly above floor
        
        if self.spent + (bid_price / 1000) > self.budget:
            return None
        
        return {
            "id": bid_request['id'],
            "seatbid": [{
                "seat": self.seat_id,
                "bid": [{
                    "id": "bid_open_001",
                    "impid": imp['id'],
                    "price": bid_price,
                    # No dealid for open auction
                    "adm": "<div>Standard ad</div>",
                    "crid": "creative_standard_001"
                }]
            }],
            "cur": "USD"
        }

# Example: Bid on available deals
dsp = DSPWithDeals(seat_id="DSP-BRAND-001", budget=10000.0)

eligible = pmp_manager.find_eligible_deals(
    bid_request_with_pmp,
    buyer_seat_id="DSP-BRAND-001"
)

bid_response = dsp.create_bid(bid_request_with_pmp, eligible)

if bid_response:
    bid = bid_response['seatbid'][0]['bid'][0]
    deal_id = bid.get('dealid', 'Open Auction')
    print(f"\nBid created:")
    print(f"  Deal ID: {deal_id}")
    print(f"  Price: ${bid['price']} CPM")
```

## Tracking Deal Performance

You need to know if your PMP deals are actually worth the premium prices. Spoiler: they usually are, but you should verify.

```python
class DealPerformanceTracker:
    """Track which deals are pulling their weight"""
    
    def __init__(self):
        self.stats = {}
    
    def record_impression(self, deal_id, price):
        """Track impression delivery"""
        if deal_id not in self.stats:
            self.stats[deal_id] = {
                "impressions": 0,
                "clicks": 0,
                "conversions": 0,
                "spend": 0.0,
                "revenue": 0.0
            }
        
        self.stats[deal_id]['impressions'] += 1
        self.stats[deal_id]['spend'] += price / 1000
    
    def record_click(self, deal_id):
        """Track clicks"""
        if deal_id in self.stats:
            self.stats[deal_id]['clicks'] += 1
    
    def record_conversion(self, deal_id, revenue=0.0):
        """Track conversions and revenue"""
        if deal_id in self.stats:
            self.stats[deal_id]['conversions'] += 1
            self.stats[deal_id]['revenue'] += revenue
    
    def get_report(self):
        """Generate performance report (sorted by ROAS)"""
        report = []
        
        for deal_id, stats in self.stats.items():
            impr = stats['impressions']
            clicks = stats['clicks']
            conv = stats['conversions']
            spend = stats['spend']
            revenue = stats['revenue']
            
            ctr = (clicks / impr * 100) if impr > 0 else 0
            cvr = (conv / clicks * 100) if clicks > 0 else 0
            cpa = (spend / conv) if conv > 0 else 0
            roas = (revenue / spend) if spend > 0 else 0
            
            report.append({
                "deal_id": deal_id,
                "impressions": impr,
                "clicks": clicks,
                "conversions": conv,
                "ctr": round(ctr, 2),
                "cvr": round(cvr, 2),
                "cpa": round(cpa, 2),
                "spend": round(spend, 2),
                "revenue": round(revenue, 2),
                "roas": round(roas, 2)
            })
        
        # Best ROAS first
        report.sort(key=lambda x: x['roas'], reverse=True)
        return report

# Example: Compare deal performance vs open exchange
tracker = DealPerformanceTracker()
import random

# Simulate preferred deal traffic
for _ in range(1000):
    tracker.record_impression("PREF-12345", price=8.50)
    if random.random() < 0.025:  # 2.5% CTR
        tracker.record_click("PREF-12345")
        if random.random() < 0.05:  # 5% CVR
            tracker.record_conversion("PREF-12345", revenue=30.0)

# Simulate programmatic guaranteed
for _ in range(500):
    tracker.record_impression("PG-67890", price=12.00)
    if random.random() < 0.032:  # 3.2% CTR (better quality)
        tracker.record_click("PG-67890")
        if random.random() < 0.06:  # 6% CVR
            tracker.record_conversion("PG-67890", revenue=40.0)

# Simulate open exchange (for comparison)
for _ in range(2000):
    tracker.record_impression("Open Exchange", price=2.50)
    if random.random() < 0.015:  # 1.5% CTR (worse quality)
        tracker.record_click("Open Exchange")
        if random.random() < 0.03:  # 3% CVR
            tracker.record_conversion("Open Exchange", revenue=20.0)

# See the results
print("\n=== Deal Performance Report ===")
report = tracker.get_report()

for r in report:
    print(f"\n{r['deal_id']}:")
    print(f"  Impressions: {r['impressions']:,}")
    print(f"  CTR: {r['ctr']}%")
    print(f"  Conversions: {r['conversions']}")
    print(f"  CPA: ${r['cpa']}")
    print(f"  Spend: ${r['spend']:,.2f}")
    print(f"  Revenue: ${r['revenue']:,.2f}")
    print(f"  ROAS: {r['roas']}x")
```

## The Bottom Line

Private Marketplace deals are how premium publishers survive in programmatic advertising. Instead of racing to the bottom on price (looking at you, open exchanges), they lock in relationships with quality advertisers at fair prices.

**Why PMPs matter:**
- Better inventory = better performance
- Pre-negotiated prices = no surprise bills
- Brand safety guarantees = sleep better at night
- Direct relationships = actual humans you can call when things break

**Three deal types to remember:**
1. **Preferred**: First look, fixed price, non-guaranteed
2. **Programmatic Guaranteed**: Reserved inventory, contracted volume
3. **Private Auction**: Multiple invited buyers, competitive but controlled

**Technical implementation:**
- Deal IDs travel in the `pmp.deals[]` array in OpenRTB
- Match deals to buyers using seat IDs
- Always prioritize deals over open auction
- Track performance by deal ID to prove ROI

In my experience working with both sides, PMPs are usually worth the premium. Higher CPMs, sure, but the quality difference is like comparing a Tesla to a '92 Civic—you get what you pay for.