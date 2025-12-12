# Data Types in AdTech (First, Second, and Third-Party)

Ever been targeted for an ad that made absolutely no sense? Like, you're a 35-year-old man and suddenly you're seeing ads for maternity wear? Or you searched for running shoes once and now every website thinks you're a marathon runner?

That's third-party data doing its thing—or more accurately, failing at its thing.

(Another way to make this happen is to use techniques like deleting/regenerating the AdID  on your phone to confuse the models but thats another chapter)

Data is what makes programmatic advertising work. Without it, you're just throwing ads at random people and hoping something sticks. With good data, you can target "women aged 25-34 in Seattle who searched for running shoes in the last 7 days and have a household income above $75K." With bad data, you're showing maternity ads to dudes.

But here's the thing: not all data is created equal. The source matters more than you'd think. It determines how valuable it is, how accurate it is, and whether you'll get sued for using it.

## First-Party Data: The Data You Actually Own

This is data you collect directly from your own customers. It's yours. You own it. You don't pay anyone else for it, and you don't have to worry about whether someone else collected it ethically. Think of it like your own garden—you planted the seeds, you watered them, you know exactly what's growing there.

**What it looks like:**
- Email addresses from newsletter signups
- Purchase history from your e-commerce site
- Browsing behavior on your website (via your own analytics)
- App usage data from your mobile app
- CRM data (customer names, addresses, preferences)
- Loyalty program data
- Customer service interactionsçç

**Why it's valuable:**
You own it—no licensing fees, no third-party dependencies. It's the most accurate because it comes directly from users interacting with your brand. People who gave you their email are more engaged than random web visitors. It's privacy-compliant because users opted in å to you. And your competitors can't access it—that's a huge advantage.

**Why it's limited:**
It only covers your customers—doesn't help you reach new audiences. It requires scale—if you only have 1,000 customers, your data isn't that useful for targeting. And you need infrastructure—you have to build systems to collect, store, and activate it.

**Real example:**
You run an online bookstore. You know John bought three sci-fi novels in the last month, Sarah abandoned a cart with a cookbook, and Mike browses every week but never buys.

With first-party data, you can retarget John with new sci-fi releases, send Sarah an email reminder about her cart, and show Mike a discount offer to convert him.

This is the gold standard. If the cookie apocalypse taught us anything, it's that first-party data is the only data you can truly rely on.

## Second-Party Data: Someone Else's Garden

This is just first-party data from another company that they share or sell directly to you. It's not a different type of data—it's a different relationship. Think of it like your neighbor letting you pick tomatoes from their garden. Same quality, different owner.

**What it looks like:**
- A hotel chain shares customer data with an airline for a partnership
- A publisher sells their audience data directly to an advertiser
- A retailer shares purchase data with a brand for co-marketing

**Why it's valuable:**
Higher quality than third-party because it comes from a known, trusted source. You have a direct relationship—you know where it came from. You can get custom audiences tailored to specific use cases. And data clean rooms allow safe collaboration without exposing raw PII.

**Why it's complicated:**
Privacy concerns—users opted in to Company A, not Company B. You need legal agreements—can't just share customer data without contracts. Limited scale—only works for specific partnerships. And you're relying on another company's data practices—you have to trust them.

**Real example:**
Spotify partners with Starbucks. Spotify shares anonymized listener data (people who listen to morning playlists) with Starbucks, who targets them with coffee ads. Starbucks shares purchase data back to Spotify for measurement.

Both companies benefit. Users... might not even know this is happening.

## Third-Party Data: The Data Graveyard

This is data collected by companies whose entire business is gathering, packaging, and selling audience data. They scrape the web, buy data from publishers, track users across sites, and bundle it into segments you can buy.

Think of it like buying produce from a sketchy roadside stand—you don't know where it came from, how fresh it is, or whether it's actually what they say it is.

**What it looks like:**
- Oracle BlueKai selling "in-market for SUVs" audiences
- Experian selling "high net worth households" data
- Lotame selling "travel enthusiasts" segments
- Data brokers selling demographic and behavioral profiles

**How it's collected:**
Cookies and tracking pixels across thousands of websites, mobile app SDKs tracking location and behavior, data partnerships with publishers, public records (address, property ownership, voter registration), and inferred data (algorithmic guesses about age, income, interests).

**Why advertisers used to love it:**
Massive scale—reach hundreds of millions of users. Easy to buy—just upload a segment to your DSP. Fills gaps—adds targeting dimensions you don't have from first-party data. And someone else did the hard work—no collection required.

**Why it's dying:**
Privacy regulations—GDPR and CCPA make it legally risky. Cookie deprecation—can't track users across sites anymore. Low quality—often inaccurate (ever been targeted as the wrong gender or age?). Lack of transparency—you don't know where it came from or how it was collected. And it's expensive—data fees add up fast.

**Real example:**
You're launching a new luxury watch brand. You don't have customers yet, so no first-party data. You buy a third-party segment: "affluent males aged 35-55 interested in luxury goods."

Does it work? Maybe. The data might be 60% accurate, 40% garbage. But it's better than nothing when you're starting from zero.

## The Data Hierarchy

In terms of value and reliability, it's simple:

**1st-party > 2nd-party > 3rd-party**

First-party: You know it's accurate because you collected it yourself. Second-party: Probably accurate if the source is trustworthy. Third-party: Who knows? Could be accurate, could be completely wrong.

For privacy compliance, the ranking is the same:

**1st-party** (users opted in to you directly) > **2nd-party** (users opted in to a partner) > **3rd-party** (users might not even know they're being tracked)

## What Data Actually Looks Like

Let's compare how the same person might appear in different data sets:

**Fictional First-Party Data (from retailer.com):**
- Name: Eddie Cobain
- Email: eddiec@email.com
- Purchase history: Guitars ($1200), Microphone stand ($280), Recording software ($350)
- Browsing: Viewed "Recording gear" section 6 times in last month
- Location: Seattle, WA
- Loyalty status: Member since 2000, 15 purchases total

**Third-Party Data (from data broker):**
- Age: 28-35 (inferred)
- Gender: Male (inferred)
- Interests: "Music enthusiast" (based on cookies from 5 sites)
- Income: $50K-$75K (inferred from zip code)
- Device: iPhone user
- Location: Seattle metro area

Notice the difference? First-party data knows Eddie bought a guitar. Third-party data guesses he might like music stuff. One is fact, the other is inference.

The cookie apocalypse is making third-party data less reliable every day. If you're building an AdTech strategy now, start with first-party data. Everything else is just guessing.