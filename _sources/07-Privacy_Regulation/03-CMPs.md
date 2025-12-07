# CMPs (Platforms for Managing Consent)

Do you remember when websites would just set cookies without asking? Things were easier back then. Not good times legally, but easier.

With GDPR, CCPA, and a dozen other privacy laws in place, every website must now ask for permission before tracking users. Enter **Consent Management Platforms (CMPs)**—the programs that run those cookie banners you see all over the place.

Some CMPs are just annoying ways to get people to click "Accept All," but good ones actually help businesses follow privacy laws while still respecting users' choices.

 ### What is a CMP?

A **Consent Management Platform** is a program that
1. Shows users a modal or consent banner
2. Tells you what data will be gathered and why
3. Lets users choose, reject, or change their choices
4. Keeps track of those choices
5. Enforces those choices all over the site by blocking tags and pixels if the user says no.
6. Gives regulators proof that the rules are being followed

 You could think of it as the person who checks people at the door of your website's tracking system. The CMP stops all tracking scripts from running if the user says "no tracking."

 ### Do It Yourself (Custom Build)

 Some businesses make their own CMPs. This is hard:
 - Must keep up with changing laws
 - Have to deal with edge cases
 - Needs to be checked by a lawyer
 - Simple to mess up and get a fine

 Not a good idea unless you have privacy engineering resources set aside.

 ### IAB Transparency and Consent Framework (TCF)

 The **Interactive Advertising Bureau (IAB)** made a standard called **TCF 2.0** for how CMPs should get permission in the AdTech world.

 ### How TCF Works

1. **CMP gets consent** and turns it into a string (TC String).
2. **TC String is sent to ad tech companies in bid requests.**
3. **Vendors check the string** to make sure they have permission to handle data.
4. **Vendors only process data if they have permission.**

**Sample TC String:**
```
CPYBBYBPYBBYBABAMBFRACBsAP_AAH_AACiQHJNf...
```

 This encodes:
 - Which vendors have given their permission
 - For what reasons (personalized ads, analytics, etc.)
 - When permission was given
 - TCF version

 ### Why It Matters

 Without TCF, each vendor would have to get permission on their own. With TCF, consent is only collected once and then shared throughout the programmatic supply chain.

 Publishers who use TCF-certified CMPs can make more money from GDPR-compliant traffic because they can show ad exchanges and DSPs that they have consent.

 ### The Catch

 People have taken TCF to court. Some people say it's too complicated and doesn't give real consent.

 Some regulators say that TCF-style consent bundling (agreeing to work with 200 or more vendors at once) goes against GDPR's "specific consent" rule.

 It's changing. Stay tuned.
