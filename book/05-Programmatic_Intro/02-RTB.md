# The Millisecond Auction: Bidding in Real Time

 Real-time bidding (RTB) is what happens when programmatic advertising goes mad.  We're talking about **auctions that happen in less than 100 milliseconds**—faster than you can blink—to determine which ad you see on a webpage.

 Every time you open a page with ads, there is an auction going on behind the scenes. A lot of advertisers are bidding. A program on a computer that picks the winner. People giving each other money before the page has fully loaded.

 It's like eBay, however the auction takes place in real time instead than over days.

### How RTB really works

 This is what occurs when you click on a link and load a webpage:

 1. You click a link

 While you're browsing, you click on a news article. The page is beginning to load.

 2. The ad server of the publisher finds an open ad spot.

 There are ads on the page, such as a banner at the top and ads in the sidebar. The publisher's ad server, which is linked to an SSP, knows that there is inventory to sell.
 
 3. The SSP sends a **bid request** that includes: 
 
 - **Ad slot details**  - Size (300x250), place (above the fold), and kind (display)
 - **Page's context**  - URL and kind of content (news, sports, entertainment)
 - **Information about users**  - Cookie ID, type of device,ocation, and interests (depending on what you've been looking at)
 
 4. SSP sends in a request for a bid. This request includes:
 
 - **Price on the floor**  - The lowest bid the publisher will take (if there is one)

 This request for bids goes out to a lot of ad exchanges at once.

 5. The ad exchange sends broadcasts to DSPs

 The ad exchange sends the bid request to many or maybe hundreds of DSPs that could be interested.

 Each DSP has a few milliseconds to decide if they want this impression.  How much will we give?

 6. DSPs look at the bids and make their own

 Each DSP has its own set of algorithms:
 - Does this user meet any of our targeting criteria?
 - Have we showed this user advertising in the past few days?  (frequency capping)
 - How much do you think this impression is worth?
 - Do we still have money in the budget?
 - How much can we bid?

 If the DSP wants the impression, it sends a **bid response** that says:
- The price of the bid (for example, $3.50 CPM)
 - Creative to show (picture, video, or HTML for an ad)

 If not interested, it sends no response.

 7. Auction resolves

 The ad exchange takes all the bids and performs an auction.  In a second-price auction, the winner pays a little bit more than the second-highest bid, not their whole amount.

 **Example:**
 - DSP A offers $5.00 - DSP B offers $3.50 - DSP C offers $2.00

 The winner is DSP A, but they only have to pay $3.51, which is $0.01 less than the second price.

8. The commercial that wins is shown

 The exchange tells the SSP who won.The SSP tells the ad server to serve DSP A's creative. The ad appears on your screen.

9. You notice the ad!

 All of this happens in **50-100 milliseconds**—before the rest of the page even finished loading.

 You clicked on a link, the page loaded, and an ad showed up.  You have no idea that a whole auction just took place to choose which ad you saw.

 The Speed Issue (And How to Fix It)

 100 milliseconds sounds rapid, but think about what needs to happen:
 1. Created a bid request (10ms)
 2. Sent to the exchange (20ms network delay)
 3. Send to many DSPs at once (10ms)
 4. DSPs look at and answer (30ms)
 5. Auction ends (5ms)
 6. Winning creative sent back (20ms)
 7. The ad shows up on the site (5ms)

 If everything goes perfectly, that's 100ms.  Did you miss the deadline?  Your offer is disregarded.

 This is why RTB infrastructure is **crazy optimized**:

 - DSPs run on powerful servers that can answer database queries in less than a millisecond.
 - Ad exchanges employ networks with minimal latency
 - Bidding methods are already set up (not calculated on the fly)
 - CDNs around the world store cached copies of creatives

 A 50ms delay in your bidding logic can lose you millions of views.

 ## RTB vs. Other Programmatic Deals

 RTB is just one kind of programmatic buying, and it's the open auction kind.

 | Type of Deal | Auction? | Price | Access | 

| Type of Deal | Auction? | Price | Access |
|---------------|----------|-------|--------|
| **RTB (Open Auction)** | Yes | Real-time bidding | Anyone |
| **Private Marketplace** | Yes | Real-time bidding above floor | Invited only |
| **Programmatic Guaranteed** | No | Fixed price | One buyer |

 RTB has the most common, highest-volume, and lowest-quality inventory. But it's also the easiest to get to and can grow forever.

 ## Who Uses RTB?

 **Performance advertisers** run direct response ads that are optimized for clicks, conversions, or app installs.  They require a lot of content and don't care about high-end publishers.

 **Retargeters** are businesses who show adverts to those who have already been to their site. RTB lets them reach millions of websites.

 **Programmatic agencies** buy a lot of things for customers, run campaigns for them, and make sure they get the best deal possible.

 **Walled gardens**—Google, Facebook, and Amazon all have their own RTB-like systems that work within their own ecosystems.

 ### The Business Side of RTB

 RTB costs less than direct deals.  This is why:

 - **Open auction CPMs:** $0.50 to $3.00; 
 - **Private marketplace CPMs:** $5 to $15; 
 - **Programmatic guaranteed CPMs:** $15 to $30+

 What's the difference?

 **Quality.**
 - Open auctions have questionable websites, bots that send traffic, and stuff that doesn't get much attention.
 - PMPs are publishers that have been handpicked for their excellence.
 - PG agreements are high-end publishers that promise to supply

 You get what you paid for.  A $1 CPM ad on a random blog is not the same as a $20 CPM ad on the homepage of The New York Times.

 ### The Bad Side of RTB

 RTB makes things bigger, but it also makes things worse:

 - **Ad fraud**: Bots create phony impressions, and advertisers pay for traffic that isn't real. 
 - **Brand safety**: Your ad shows up next to material that isn't acceptable (like a luxury brand on a conspiracy theory site). 
 - **Viewability**  - Ads that load below the fold and never get viewed still count as "impressions." 
 - **Arbitrage** is when middlemen resell inventory and take cuts without contributing value. 
 - **Lack of transparency** means advertisers don't always know where their ads ran.

 This is why verification companies like IAS and DoubleVerify exist and why many advertisers use PMPs or direct deals for brand campaigns.

 ## RTB in Action (Real Numbers)

 You are in charge of a campaign for an online store that sells sneakers.

 - **Budget:** $10,000 
 - **Goal:** Get more people to visit the website and buy things 
 - **Target:** People who like sneakers, running, and sports

 **Campaign setup:**
 - Bid target: $2.50 per thousand views
 - Daily spending limit: $500
 - Target audience: people in the US who are 18 to 45 years old who love sports and fitness

 **What happens:**
 - DSP bids on millions of impressions per day and wins about 200,000 impressions (4 million over the course of the campaign).
 - Effective CPM: $2.50
 - Clicks: 8,000 (0.2% CTR)
 - Purchases: 160 (2% conversion rate)
 - Cost per purchase: $62.50

 If you're selling $100 sneakers with $40 margin, you made $6,400 gross profit on $10,000 spend.  Not great, but if you optimize targeting and bidding, you can improve that over time.

 That's RTB in action: buying impressions at scale, tracking performance, optimizing continuously.

 ---

 ## The Bottom Line

 Real-Time Bidding turned digital advertising into a financial market.  Impressions became a commodity traded in millisecond auctions.  Algorithms replaced salespeople.  Scale exploded.

 It's efficient, fast, and data-driven.  It's also imperfect, sometimes fraudulent, and often lacks transparency.

 But it's not going anywhere.  RTB is how the open web funds itself—by selling your attention to the highest bidder, one impression at a time.

 Next up: We'll dive into the data that powers all this targeting.