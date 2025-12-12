# Appendix: Resources, Tools & Further Reading

This appendix contains resources I've found useful over the years—documentation that's actually readable, tools that don't require a PhD to use, and learning materials that cut through the marketing fluff.

I've skipped the obvious vendor marketing sites and focused on genuinely helpful resources. Some are technical, some are strategic, all are practical.

---

## Essential Documentation & Specs

### OpenRTB Protocol
**What:** Standard protocol for programmatic advertising communication between DSPs and SSPs.  
**URL:** https://www.iab.com/guidelines/openrtb/  
**Why it matters:** If you're building anything programmatic, this is your bible. Defines bid requests, bid responses, and all the data structures.

### IAB Standards & Guidelines
**What:** Industry standards for ad formats, measurement, viewability, and more.  
**URL:** https://www.iab.com/guidelines/  
**Key specs:**
- Ad formats and sizes
- Viewability measurement
- Video ad serving (VAST/VPAID)
- Native advertising specs

**Why it matters:** These standards determine how the entire industry operates. Know them.

### Google Ad Manager API Documentation
**What:** Comprehensive API docs for programmatic ad serving.  
**URL:** https://developers.google.com/ad-manager  
**Why it matters:** Even if you don't use Google Ad Manager, their docs are surprisingly good for understanding how ad servers work.

### Facebook Marketing API
**What:** Documentation for Facebook/Meta advertising APIs.  
**URL:** https://developers.facebook.com/docs/marketing-apis/  
**Why it matters:** If you're working with social advertising data, this is required reading.

---

## Tools & Platforms

### Development & Testing

**Postman**  
**What:** API testing tool  
**URL:** https://www.postman.com/  
**Use case:** Testing RTB endpoints, ad server APIs, tracking pixels. Build collections for OpenRTB bid requests to test your bidder.

**Charles Proxy / Fiddler**  
**What:** HTTP debugging proxy  
**URLs:** https://www.charlesproxy.com/ | https://www.telerik.com/fiddler  
**Use case:** Intercept and inspect ad requests, tracking calls, and API traffic. Essential for debugging why ads aren't showing or tracking isn't firing.

**Chrome DevTools**  
**What:** Built-in browser developer tools  
**Use case:** Network tab shows all ad requests, tracking pixels, and JavaScript errors. Console shows client-side issues. Priceless for debugging.

**Selenium / Playwright**  
**What:** Browser automation frameworks  
**URLs:** https://www.selenium.dev/ | https://playwright.dev/  
**Use case:** Automated testing of ad implementations, A/B tests, and creative rendering across browsers.

### Ad Tech Specific Tools

**Prebid.js**  
**What:** Open-source header bidding wrapper  
**URL:** https://prebid.org/  
**Use case:** Industry-standard header bidding implementation. If you're a publisher, you'll use this. If you're a vendor, you need to integrate with it.

**Google Tag Manager**  
**What:** Tag management system  
**URL:** https://tagmanager.google.com/  
**Use case:** Deploy tracking pixels, analytics tags, and ad tags without touching code. Also has server-side container option.

**VAST Validator**  
**What:** Video ad format validator  
**URL:** https://vast.iabtechlab.com/validator  
**Use case:** Validate your VAST XML for video ads. Broken VAST = no video ad revenue.

### Analytics & Measurement

**Google Analytics 4**  
**What:** Web and app analytics platform  
**URL:** https://analytics.google.com/  
**Use case:** Track user behavior, measure ad campaign effectiveness, attribution modeling. Free tier is generous.

**Looker Studio (formerly Data Studio)**  
**What:** Data visualization and dashboards  
**URL:** https://lookerstudio.google.com/  
**Use case:** Build custom dashboards for ad performance. Connects to Google Analytics, Google Ads, BigQuery, and more. Also free.

**BigQuery**  
**What:** Google's data warehouse  
**URL:** https://cloud.google.com/bigquery  
**Use case:** Store and analyze massive ad logs, impression data, conversion data. SQL interface, ML capabilities built-in.

### Privacy & Consent

**OneTrust**  
**What:** Consent management platform  
**URL:** https://www.onetrust.com/  
**Use case:** GDPR/CCPA compliance, cookie consent banners, privacy policy management. Enterprise-grade but also has SMB options.

**Cookiebot**  
**What:** Cookie consent solution  
**URL:** https://www.cookiebot.com/  
**Use case:** Simpler, cheaper alternative to OneTrust for basic consent management.

---

## Learning Resources

### Books Worth Reading

**"Programmatic Advertising" by Oliver Busch**  
Deep dive into programmatic ecosystems. Academic but thorough. Good for understanding the economics and game theory of ad auctions.

**"Data-Driven Marketing" by Mark Jeffery**  
Explains marketing analytics and attribution modeling. Less technical, more strategic. Good for understanding the business side.

**"Trustworthy Online Controlled Experiments" by Kohavi, Tang, and Xu**  
The definitive book on A/B testing. Microsoft research. If you're running experiments (and you should be), read this.

**"Designing Data-Intensive Applications" by Martin Kleppmann**  
Not AdTech-specific, but essential for understanding the distributed systems that power AdTech platforms. Best technical book I've read in years.

### Blogs & Publications

**AdExchanger**  
**URL:** https://www.adexchanger.com/  
**What:** Daily news on programmatic advertising, ad tech M&A, privacy regulations  
**Worth following?** Yes. Best source for industry news. Their podcasts are good too.

**Digiday**  
**URL:** https://digiday.com/  
**What:** Media and marketing news  
**Worth following?** For understanding publisher perspectives and broader industry trends.

**MarTech Today / MarTech**  
**URL:** https://martech.org/  
**What:** Marketing technology news and analysis  
**Worth following?** Good for staying on top of marketing tech trends beyond just AdTech.

**The Programmatic Advisory**  
**URL:** Various LinkedIn posts and industry analysis  
**What:** Deep analysis of programmatic advertising trends  
**Worth following?** If you're deep in programmatic, yes.

---

## Tools I Use (Personal Recommendations)

These are tools I actually use, not just ones I've heard of:

**VS Code** - Code editor with great Python/JavaScript support  
**Jupyter Notebooks** - For exploratory data analysis and ML prototyping  
**DBeaver** - Universal database tool (connects to BigQuery, Postgres, MySQL, etc.)  
**Postman** - API testing (mentioned earlier but worth repeating)  
**Docker** - For containerizing ad tech services  
**Apache Airflow** - Workflow orchestration for data pipelines  
**DBT** - Data transformation tool, great for cleaning ad data  

---


