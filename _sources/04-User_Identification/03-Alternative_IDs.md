# Life After Cookies (Other Ways to Identify Yourself)

As third-party cookies slowly and dramatically go away, the ad tech industry has been scrambling to develop other ways to track users. Some of these work, but others are just giving the same tracking a different name. 

## Tracking by email (hashed emails)

**Hash email addresses or phone numbers** to make unique IDs that operate on all devices and websites.

**How it works:**

1. The user connects onto a website using their email address (you@example.com)
2. The site turns that email into a cryptographic string (abc123)
3. This hash, which is also known as Universal ID, now goes with ad requests.
4. If several sites have the same email address, they can match the same hash or even hash the email address using the same algorithm to add to their identity database.
5. Advertisers can now follow that "person" (actually, that hashed email) from site to site.

**Pros:**
- Works on all devices (same email equals same ID)
- Lasts longer than cookies (doesn't go bad after 30 days)
- Somewhat private (it's hashed, not raw email)
- More matches than cookies ever had

**Cons:** 
It only works for people who are logged in, and most people aren't logged in most of the time.

- Still keeping an eye on individuals, but in a different way - Users have to give you their email address 
- Privacy regulators are keeping a tight eye on this area

**Key players:** The Trade Desk (UID2.0), LiveRamp (RampID), ID5, and Unified ID Solutions

Universal IDs usually work effectively for publications whose users are logged in (such news sites, streaming services, and e-commerce sites). For surfing without giving your name? Not really.

Data Clean Rooms 

In a neutral hosted site, data clean rooms are safe places where businesses can match and analyze data without sharing it directly.

**How it works:**
1. Company A or Company B uses their provider to host a clean room.
2. Company A sends their first-party data to the server (encrypted)
3. Company B sends their first-party data to the cloud (encrypted)
4. The data is matched by a neutral third party, such Liveramp, Snowflake, or Google Ads Data Hub.
5. Businesses can learn about each other and target their ads without viewing each other's raw data.
6. All data stays encrypted and can't be moved out of the clean room.

**Examples of use:

- **Ad targeting**: Match your customer list with a publisher's audience. 
- **Frequency capping**: Make sure consumers don't see the same ad too many times across platforms. 
- **Attribution**: Find out which advertisements led to conversions without giving over PII.
- **Campaign measurement**—Find out how many people saw your ads and how many of them were on different media partners' sites.

**Pros:** 
- Should be private 
- Lets people work together without leaking data
- Works for closed-off areas like Facebook and Amazon

**Cons:**
- Somewhat difficult to set up, contacts and legal agreements required
- Costly (not for small firms)
- Both sides need to have strong first-party data 
- There are no guidelines for how each clean room functions

**Big players:** Google Ads Data Hub, Amazon Marketing Cloud, Meta Advanced Analytics, Snowflake, and LiveRamp


Contextual Targeting (The Old-School Comeback)

The idea here is if you can't trace users, focus on the content instead. This is contextual advertising, and it's coming back.

**How it works:** Instead of "show car ads to people who are interested in cars," you "show car ads on car-related content."

**For example:
- Ads for health and fitness products on health and wellness blogs 
- Ads for financial services on business news sites
- Ads for travel on travel material, no matter who is reading
- Ads for audio gear on music production podcasts

**Why it's useful:** 

- **Works everywhere** - on linear TV, in print, on podcasts, and in places cookies never went 
- **Protects privacy** - No user IDs or monitoring needed
- Doesn't need user IDs or monitoring 
- **Always worked** - This is how advertising operated for decades before cookies. Someone who reads about vehicles is likely to be interested in cars.

**The problem:** It isn't as exact as behavioral targeting. You reach vehicle lovers, but you might also attract folks who are simply reading one automotive article and don't plan to buy.

Even so, contextual targeting is making a comeback in a world where tracking users is hard. The AI-powered versions that are being made right now are very advanced. They can look at page text, sentiment, video frames, and even audio to figure out which advertising make sense.

---

What really works?

Essentially, it's a mix.

**For publishers with login walls:** Universal IDs and clean rooms 
**For walled gardens (Google, Meta, Amazon):** Their own first-party data ecosystems 
**For open web:** Contextual targeting + whatever user data you can legally collect 
**For everyone:** Build your own first-party data and stop relying on someone else's infrastructure

Advertisers didn't have to construct their own tracking infrastructure during the cookie era. That time is coming to an end. You need to put money into your own data, relationships, and ability to measure things in this new era.

It's more difficult. It costs extra. But it's also better for the environment and less spooky.