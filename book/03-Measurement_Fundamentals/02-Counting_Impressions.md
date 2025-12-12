# Counting Impressions: Tags, Pixels, and Logs

It seems easy to count things. You saw an ad, which is one impression. You clicked on it, which counts as one click. It does sound simple.

However, it's really hard to count digital events. Bots make numbers bigger. Ad blockers stop them. Different systems count at different times and in different ways. Two platforms that measure the same campaign will give you different numbers, and both of them are right.


## What Actually Happens When You Count Impressions

When a person sees an ad, that counts as an impression or thats the way its usually supposed to work assuming perfect conditions.

### Counting Methods

### 1. Server Side 

The count of the time it takes for the ad server to send the creative to the publisher's page.

User requests page → Publisher requests ad → Ad server delivers ad → COUNT +1

**Problem:** The ad might run into issues and not be viewable. The user could close the tab, the page might not load or spin endlessly, or the JavaScript might have issues. Essentially we counted an impression that didn't happen.

**Advantage:** It's easy and works well for the ad server. Not easy to block.

### 2. Counting on the Client Side (The Standard)

Count when JavaScript says the ad has loaded in the browser.

```javascript 

/** Simple impression tracking pixel 

const pixel = new Image();
pixel.src = `https://adserver.com/track?` +
    `ad=${adId}&` +
    `campaign=${campaignId}&` +
    `timestamp=${Date.now()}`;

// Pixel loads = counted as an impression
// Fire when the ad is put into the DOM

document.getElementById("ad-slot").addEventListener("DOMNodeInserted", function(e) {
    if (e.target.classList.contains("ad-creative")) {
        trackImpression('ad_12345', 'campaign_67890');
    }
});

```

**Problem:** Tracking pixels are killed by ad blockers. People who close tabs quickly might not wait for the pixel to go off. Errors in JavaScript break everything.

**Advantage:** This is more accurate since the ad made it to the browser.

3. Counting Based on Viewability (The Best)

Count only when the ad is on the screen.


```javascript
// Only track impressions that can be seen
function trackViewableImpression(adElement, adId) {
    const observer = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
            if (entry.isIntersecting) {
                // The ad is at least 50% visible
                const visibilityRatio = entry.intersectionRatio;
                
                if (visibilityRatio >= 0.5) {
                    // Start the timer; we need one second of viewability
                    setTimeout(() => {
                        // Check if it's still visible after 1 second
                        if (entry.isIntersecting) {
                            fireImpressionPixel(adId);
                        }
                    }, 1000);
                }
                
                // Stop watching after counting
                observer.unobserve(adElement);
            }
        });
    }, {
        threshold: [0.5] // Trigger when 50% visible
    });
    
    observer.observe(adElement);
}

function fireImpressionPixel(adId) {
    fetch(`https://adserver.com/track`, {
        method: 'POST',
        body: JSON.stringify({
            ad_id: adId,
            viewable: true,
            timestamp: Date.now()
        }),
        keepalive: true
    });
}
```
### How to use: 


```javascript
const adElement = document.querySelector('.ad-unit');
trackViewableImpression(adElement, 'ad_12345');
```

**IAB Viewability Standard:** 

For display ads, at least 50% of the pixels must be in view for at least 1 second. For video ads, at least 50% of the pixels must be in view for at least 2 seconds.


## The Pixel That Tracks

AdTech's most famous trick is the 1x1 pixel. It's really just an image that looks like surveillance. Its notoriously used in emails to track if the email was opened.

```html
<!-- A typical tracking pixel --> <img src="https://adserver.com/pixel.gif? campaign=123& creative=456& user=abc& timestamp=1234567890" width="1" height="1" style="display:none;" />
``

When the browser loads this "image" it sends a request to the ad server with all the query parameters. The server saves the data, sends back a small transparent GIF, and you've tracked an impression.

**The modern version uses JavaScript instead:**

### How to use: 

```javascript

// More advanced tracking beacon
(function() {
    const data = {
        ad_id: 'ad_123',
        campaign_id: 'camp_456',
        user_id: getUserId(),
        page_url: window.location.href,
        referrer: document.referrer,
        screen_size: `${window.screen.width}x${window.screen.height}`,
        viewport_size: `${window.innerWidth}x${window.innerHeight}`,
        timestamp: Date.now()
    };
    
    // Send beacon (won't be blocked by page unload)
    navigator.sendBeacon(
        'https://adserver.com/track',
        JSON.stringify(data)
    );
})();
```

navigator.sendBeacon() is meant for tracking; it works every time, even if the user closes the tab right away.

Server logs are the only place to get the truth.

There is a server logging the request behind every pixel. This is where people really count.

**Basic log entry:

``` 2024-10-04 15:23:45 | ad_12345 | campaign_67890 | user_abc123 | 192.168.1.1 | Mozilla/5.0... | viewed ```

**How to process these logs:**

```python 

import pandas as pd from datetime import datetime

def count_impressions(log_file): """ Count impressions from raw ad server logs """
    
    # Read the log file logs = pd.read_csv(log_file, sep='|', names=[ 'timestamp', 'ad_id', 'campaign_id', 'user_id', 'ip_address', 'user_agent', 'event_type' ])
    
    # Filter for events that show impressions (not clicks or other events)
    impressions = logs[logs['event_type'].str.strip() == 'viewed'
    
    # Remove duplicates (one impression for the same user and ad within an hour)
    impressions['timestamp'] = pd.to_datetime(impressions['timestamp'])
    impressions = impressions.sort_values('timestamp')
    
    # Group by user and ad, and only keep the first impression that happens within the time window.
    impressions['time_diff'] = impressions.groupby(['user_id', 'ad_id'])['timestamp'].diff() impressions = impressions(impressions['time_diff']isna() | |  # First impression (impressions['time_diff'] > pd.Timedelta(hours=1))  # Or more than an hour since the last
    
    # Count how many impressions each campaign got impression_counts = impressions.groupby('campaign_id').size()
    
    return impression_counts

# Usage # counts = count_impressions('ad_server_logs.csv')
# print(counts)

**Problems with counting based on logs:** 1. **Bot traffic:** Bots send HTTP requests similar to normal traffic.
2. **Duplicate requests:** Network retries can make counts go up
3. **Privacy:** User agents and IP addresses are now private information. 
4. **Scale:** Big campaigns make millions of log entries every hour.

## Why the Numbers Never Match

You run a campaign. According to Google Analytics, there were 50,000 views. Your ad server says 48,000. The publisher says 52,000. Who is right?

Everyone. And no one. 

**Why there are differences:**

1. Different ways of counting
- The ad server counts when it serves the ad. - The publisher counts when the ad shows up on their page. - The analytics counts when the tracking JavaScript loads.

**2. Blockers of Ads**
- More often than ad serving, analytics tracking gets blocked. - Some users see ads, but tracking never fires.

**3. Problems with technology**
- Tracking doesn't work because of JavaScript errors - Tracking times out because pages load slowly - Request failures happen because of network problems

4. Logic for removing duplicates
- Different systems have different rules about what makes something unique.
- The time windows are different (hourly, daily, or the whole campaign)

**5. Filtering Bots**
- Each platform has its own way of finding bots. Some filter a lot, while others don't filter at all.

**Normal range of differences:** 5–15% is usually normal. 

## Fraud: The Counting Issue 

Fake impressions waste billions of dollars on ads. Sophisticated bot networks act like people, which makes it almost impossible to find fraud on a large scale.

**Different types of impression fraud:**

**1. Bot Traffic** Scripts that run automatically and load pages, view ads, and even make mouse movements and clicks.

**2. Stacking Ads**
There are a lot of ads stacked on top of each other in the same place. People can only see the topmost ad, but all of them get counted as impressions.

**3. Stuffing Pixels
Putting ads in frames that are 1x1 pixels. "Delivered" in a technical sense, but not seen by people.

4. Domain Spoofing
Scammers say that their low-quality site is actually a premium publisher so they can charge higher CPMs.

**Basic fraud detection:**

```python 

def detect_suspicious_traffic(impression_logs): 
    
    suspicious = []
    
    for idx, row in impression_logs.iterrows(): flags = []
        
        # Too many impressions coming from one IP
        ip_count = impression_logs[impression_logs['ip_address'] == row['ip_address']].shape[0]
        if ip_count > 1000: flags.add('high_ip_frequency')
        
        # The user agent is a bot that is known
        bot_keywords = ['bot', 'crawler', 'spider', 'scraper'] if any(keyword in row['user_agent'].lower() for each keyword in bot_keywords): flags.append('bot_user_agent')
        
        # Impression duration not realistic (too fast or too slow)
        if "session_duration" is in the row and row["session_duration"] < 0.1, flags.append("impossible_speed")
        
        # Patterns of engagement (no mouse movement, no scrolling)
        if row.get('mouse_events', 0) == 0 and row.get('scroll_events', 0) == 0: flags.add('no_engagement')
        
        if there are flags, something is wrong.append({ 'impression_id': row['impression_id'], 'flags': flags, 'confidence': len(flags) / 4 # Simple confidence score })
    
    return pd.DataFrame(suspicious)
```

# How to use it
```python
suspicious_traffic = detect_suspicious_traffic(logs)
print(f"Flagged {len(suspicious_traffic)} suspicious impressions")
```

Real fraud detection is a lot more complicated (ML models, device fingerprinting, behavioral analysis), but the idea is the same: look for patterns that don't fit with how people normally act.

## The Measurement Solution Stack

 Counting impressions accurately means that several systems work together:

1. **Ad Server**: Shows ads and counts impressions on the server side. 2. **Tag Manager**: Puts tracking pixels on your site.
3. **Platform for Analytics** - Keeps track of how users act and checks that they see ads
4. **Viewability Vendor** - Checks to see if people actually saw the ads (IAS, Moat, DoubleVerify)
5. **Finding Fraud** - Stops bot traffic (White Ops, Forensiq)

Each one adds a layer of validation and takes a cut. It costs a lot to measure things correctly.

## Important Points

Counting impressions looks easy, but there are a lot of different ways to do it, like server-side, client-side, and viewability-based.
- Balancing simplicity and accuracy - Big differences between platforms (5–15% is normal)
- Always fighting fraud

There is no one-size-fits-all way to measure things. The best you can do is know what your limits are, use the same methods every time, and check your numbers often.

Always ask, "How are you counting?" when someone tells you how many impressions they have. The answer is more important than the number itself.