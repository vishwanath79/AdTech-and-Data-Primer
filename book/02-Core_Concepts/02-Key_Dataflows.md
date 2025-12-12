# How Data Moves in AdTech

Data is what makes AdTech work. A lot of it. It flows between systems all the time, where it is matched, enriched, anonymized, and sold. These are the main kinds of data that keep this whole thing going:

## Identifiers (Who the Hell Are You?)

These are the unique IDs that let systems recognize "this person" at different points of contact. Ad targeting would be impossible without identifiers; every impression would look like a brand new person.

**Examples:** - **Cookie IDs** are the little text files that live in your browser (note these are dying a slow death because of privacy laws).
- **IDs for mobile devices** - IDFA (for iOS) and GAID (for Android) are unique numbers that identify your phone.
- **Addresses for email** - Hashed and used to match (when you log in to something)
- **IP addresses** - Not the best signal but to be used in combination with other signals

## Location Signals (Where the heck are you?)

Location data helps ads reach the right people in the right places. Sometimes it's exact, like GPS coordinates, and other times it's just a guess, like an IP address lookup.

**Examples:** - **GPS coordinates** - Exact lat/long from your phone (needs permission) - **IP address** - Can narrow it down to a city or region (not very accurate)
- **Wi-Fi signals** - Can tell if you're in a certain store or place - **Cell tower triangulation** - Location data at the carrier level - **Zip codes** - Self-reported or guessed from other signals

"Geo-fencing" is a fun fact that uses location data to show ads only to people within a certain distance. For example, you could target everyone who lives within 5 miles of your store. In theory, it works better than in practice.

## Audience Data (Who Are You?)

This is the demographic and behavioral information that makes targeting useful. It's what changes "show my ad to everyone" into "show my ad to women in Seattle who are 25 to 34 years old and interested in yoga."

**Examples:** - **Demographics** - Age, gender, income, education (often guessed, but sometimes very wrong)
- **Categories of interest** - What you talk about online
- **Signals of intent to buy** - Are you really looking for something?
- **Things that happen in life** - Just got married? Did you buy a house? Had a child? (Weird but useful)
- **People who look like you** - People who look like your best customers

## Interest Signals: What Do You Really Care About?

These are the little things you do that show how you act when you surf the web. Somewhere, every click, scroll, and pause is recorded.

**Examples:** - **Browsing history** - The sites you've been to (usually gathered through cookies or tracking pixels) - **Search history** - What you've looked for (Google *really* loves this data)
- **How to use the app** - What apps you open and how long you use them - **Interacting with content** - Watching videos, reading articles, and looking at products - **Social media activity** - Follows, likes, shares, and comments

## Content Signals (What Are You Looking At Right Now?)

These are signals about the content itself, not necessarily about the user. This is how contextual targeting works: showing car ads on car blogs, no matter who is reading them.

**Examples:** - **Page content** - Keywords, topics, and the mood of the article or page - **Content categories** - IAB taxonomy classifications, such as "Sports > Basketball"
- **Content in video** - What's going on in the video being watched - **User-generated content** - Comments, reviews, and posts on forums - **Metadata** - Tags, descriptions, and information about the author

---

All of this information moves between DSPs, SSPs, DMPs, CDPs, and a lot of other acronyms. It gets matched, combined, and improved, and then it's used to answer the question, "Should I bid on this impression?" And how much?

Data engineers don't have a hard time collecting this data; their biggest problem is making sure it moves quickly enough (RTB auctions happen in less than 100 milliseconds), follows privacy laws (GDPR, CCPA), and really helps campaigns instead of just making dashboards look good.