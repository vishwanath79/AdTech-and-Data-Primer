# DSPs, SSPs, Ad Exchanges, and DMPs

Acronyms in programmatic advertising are an alphabet soup - DSPs, SSPs, DMPs, CDPs, AdX, and PMPs that are meant to confound you.

 Let's figure out who these actors are, what they do, and how they all fit into the programming ecology.

 ## The Main Players

 Programmatic advertising is like a stock market for ads.  You need buyers, sellers, a place to trade, and information to help you make trading judgments.

 - **Advertisers** are Buyers (want to show advertisements)
 - **Publishers** are sellers who have ad space
 - **Ad Exchanges** are like stock exchanges, which are markets where people may buy and sell things
 - **DSPs** are Buyer's broker (tools for advertisers to acquire things)
 - **SSPs** are Seller's broker (tools for publishers to sell)
 - **DMPs** - Market research company (gives you information on who to target) by providing data or an identity graph

 Let's look at each one in detail.

 ---

 ## Demand-Side Platforms (DSPs)

 DSP refers to the software that lets advertisers buy ad space automatically on many ad exchanges.

 **What they do:**
- Connect to many ad exchanges at the same time
- Check bid requests in milliseconds
- Choose which impressions to bid on and how much to pay
- Handle the pacing, finances, and optimization of campaigns
- Report on performance

**Why they exist:** Advertisers would have to connect to each ad exchange, publisher, and SSP separately if DSPs didn't exist.  DSPs put all that merchandise into one place where people can buy it.

 ### How DSPs Work

 When someone opens a page with an ad slot:

1.  **Bid request comes in** from an ad exchange with user data and page context
2.  **DSP checks** the impression against current campaigns
3.  **Filters for targeting** (location, age, interests, etc.)
4.  **Bid calculated** based on the chance of conversion and the budget
5.  **Bid sent** to the ad exchange (if the targeting requirements are met)
6.  The ad is presented to the "auction winner."

This whole setup takes about 100 milliseconds.

**Prominent SSPs:**

- **Google Ad Manager** (formerly known as DoubleClick for Publishers)
- **Magnite** (the combination of Rubicon Project and Telaria)
- **PubMatic**
- **OpenX**
- **Index Exchange**

**How they make money:**  DSPs, SSPs take 10–20% of what consumers pay.

 ## Exchanges for Ads

Digital marketplaces that let DSPs and SSPs bid on things in real time.

 **What they do:**
- Find sellers (SSPs) who want to buy from buyers (DSPs)
- Run the real auction in a few milliseconds
- Send bid requests from SSPs to DSPs
- Give winning bids back to SSPs
- Take care of billing and making sure payments are correct

**Why they exist:** A place where buyers and sellers can meet without taking sides. Think of NASDAQ as a place to buy and sell adverts.


 **How do ad exchanges work?**
The exchange is in the middle:

```
Publisher (via SSP) → Ad Exchange → DSPs → Bids → Exchange → Winner → SSP → Ad Shown
```

**Important point:** Most ad exchanges use "second-price auctions," which means that the winner pays just a little more than the second-highest bid, not the whole amount of their bid.  This makes bidders want to bid what they think is fair.

**Big Ad Exchanges:**
- **Google Ad Exchange (AdX)** - The biggest in terms of volume
- **OpenRTB exchanges** - A lot of smaller exchanges use the OpenRTB protocol.
- **App Nexus** (now part of Microsoft/Xandr)
- The Rubicon Project is now part of Magnite.


 ## Platforms for Managing Data (DMPs)

**What they do:** They collect, organize, and use audience data to target people.

**What they do:**
- Combine data from several sources, like websites, apps, CRMs, and third parties
- Make groups of people based on their hobbies, demographics, and behavior.
- Allow DSPs to target segments
- Give information about the makeup and behavior of the audience

**Why they exist:** Advertisers need information to make wise decisions about how much to bid.  DMPs make that data useful by bringing it all together.

 ### Flow of DMP Data

```
Data Sources → DMP → Creating Segments → DSP → Bidding on Targeted Ads
```

**What data sources do DMPs use?**
- **First-party data:** People who visit your website or app
- Second-party data: data from partners
- Third-party data: audience segments that were bought
- **Data that isn't online:** CRM, point of sale, call center

**Basic DMP segmentation:**

**Important DMPs:**
- **Manager of Adobe Audience**
- **Oracle BlueKai**
- **Lotame**
- **Salesforce DMP**

**Note:** DMPs are going down since third-party cookies are going away.  Customer Data Platforms (CDPs) that focus on first-party data are taking their place.

## How They All Work Together

When you load a webpage, here is the whole process:

1.  **Publisher's page loads** with an ad space
2.  **SSP gets** an ad request from a publisher
3.  **SSP contacts ad exchange** with a request for a bid
4.  **Ad exchange broadcasts** send requests to associated DSPs
5.  **DSPs ask DMPs for information on user segments**
6.  **DSPs figure out bids** based on user and targeted data.
7.  **DSPs send bids** to the ad exchange
8.  **Ad exchange holds an auction and picks a winner**
9.  **Winning bid sent** back to SSP
10.  **SSP sends** the winning ad to the publisher's page
11.  **User sees ad** (we hope)

This everything happens in about 100 milliseconds.  It's both impressive and scary at the same time.

## The Money

Everyone gets a piece:

- **DSP fee:** 10% to 20% of the money spent on ads
- **SSP fee:** 10–20% of the money spent on ads
- **Ad exchange fee:** 5% to 10% of the money spent on ads
- **Data fees:** These can change (DMPs, third-party data)

**Example:** The advertiser pays $10 per thousand views.
- DSP gets $1.50 (15%).
- The ad exchange takes $0.75 (7.5%).
- SSP costs $1.25 (12.5%).
- **What the publisher gets:** $6.50

The "ad tech tax" takes 35% of the money before the publisher gets any.  This is why publishers don't like programmatic.

## Important Points

There are a lot of middlemen in the programmatic ecosystem, and each one adds value in its own way:

- **DSPs** - Tools for advertisers to buy
- SSPs are tools that publishers can use to sell things.
- **Ad Exchanges** - Marketplaces that make it easy to hold auctions
- **DMPs** - Collecting and separating data

They're all needed right now since the ecology is too broken up for direct interactions.  But if walled gardens get bigger and privacy gets stricter, this complicated infrastructure may get smaller or maybe go away completely.

For now, it's important to know who does what in order to get around programmatic advertising.  Keep in mind that every acronym means that someone is taking a cut of the ad dollar.