# Device IDs vs. People IDs: Who Are We Even Following?

This is a question that keeps coming up in AdTech: Are you keeping an eye on *people* or *devices*?

More than you might imagine, the response counts.

The Difference (And Why It's Messy)

**Device-based tracking:** You pick a phone, tablet, or computer and keep an eye on what happens on it.

**Tracking people:** You find a person and keep track of them on all of their gadgets.

You would follow people in a perfect world. But individuals own more than one gadget and share devices with family members, so tracking devices is what you usually receive by default.

**The issues:
- Track devices: Treat three devices as three different people.
- Tracking people means matching signals between devices, which is hard and occasionally wrong.

For example, you and your partner both use the same iPad. Tracking devices think you're the same individual. If done right, people tracking knows you're not the same person. It still thinks you're one person if you do it wrong.

## Device Identifiers (The Strings of Letters and Numbers)

Every phone, tablet, and computer has a unique code called a device ID. They don't expire like cookies do (no 30-day refresh).
- **Are different for each device, not for each browser**
- **Work on different apps** (at least on mobile)
- **Give greater tracking than cookies ever did

### Different Kinds of Device IDs

**IDs for Mobile Devices:**

**IDFA (Identifier for Advertisers)** - Apple's ID for devices that run iOS
- Used to keep an eye on how users act in different apps
- Made optional in iOS 14.5 (App Tracking Transparency)
- The opt-in rate is about 25% in the US and substantially lower in Europe.

**GAID (Google Advertising ID)** - Google's ID for Android devices
- Same purpose as IDFA, Android version
- Still available but facing similar privacy pressures
- Google is moving toward Privacy Sandbox for Android

**IDFV (Identifier for Vendor)** - Apple's per-publisher ID
- Different for each app publisher
- Can't be used for cross-app tracking
- Survives even when IDFA is denied

**Desktop/Web:**
- No real equivalent to mobile device IDs
- Cookies were the closest thing
- Device fingerprinting tries to fill the gap (we'll get to that)

## Device Fingerprinting (The Sneaky Alternative)

When you can't get a device ID or cookie, some companies try to *infer* a unique identifier by combining device signals:

**Used signals:**
- Screen resolution
- Browser type and version
- Operating system
- Installed fonts
- Language settings
- Timezone
- IP address
- GPU details
- Plugins and extensions

Combine enough of these and you can create a "fingerprint" that's unique to a specific device—even without cookies or IDs.

**Is it effective?** Sometimes. But it's:
- Less reliable than real IDs
- Doesn't work well on mobile
- Breaks when users update browsers or change settings
- Ethically questionable (tracking without consent)
- Banned or restricted in some jurisdictions

## The Privacy Apocalypse (For Device IDs)

Device IDs had a good run, but privacy regulations and platform policies are shutting them down.

### Apple's App Tracking Transparency (ATT)

In iOS 14.5 (2021), Apple made IDFA **opt-in** instead of default.

**The prompt users see:**
> "Allow [App Name] to track your activity across other companies' apps and websites?"

Guess what most people click? **"Ask App Not to Track."**

**Results:**
- IDFA availability dropped from ~90% to ~25% overnight
- Mobile app attribution got way harder
- Facebook publicly complained about losing $10 billion in revenue
- Advertisers scrambled for alternatives

### Google's Privacy Sandbox (For Android)

Google is doing a softer version:
- Deprecating GAID over time
- Replacing with Privacy Sandbox APIs (similar to web). As of 2025, this may also be canned.
- Trying to balance privacy with not breaking their ad business
- Moving slower than Apple because ads are Google's main revenue

**As of 2025, GAID still works**, but expect it to look more like IDFA (limited availability) in the near future.

## Measurement Based on Devices vs. People

You still have the multi-device problem, even with device IDs.

**Device-based measurement:** I view an ad on my phone while I'm eating lunch, I study on my laptop at work, and I buy something on my tablet at home. The attribution system says, "That's three different people!"

**Measurement based on people:
- The platform tries to link those three devices to one person (me) via email logins, IP addresses, behavioral signals, and device graphs.
- Attribution system: "That's one person on three devices"

**Which is more accurate?** People-based. **Which is tougher to build?** People-based, too.

**Who does it well?**
- **Google** - If you're logged into Google on all your devices, they know it's you
- **Facebook/Meta** - Same deal with Facebook login
- **Apple** - Knows your Apple ID across devices
- **Amazon** - Tracks you via login across phone, tablet, Fire TV, Alexa

**Who struggles?**
- Open web publishers without login systems
- Small advertisers without first-party data
- Anyone trying to track users across different ecosystems

---

## The Reality Check

We're moving from:
- **"Track everyone, everywhere, all the time"** (the cookie era)

To:
- **"Track logged-in users on your own properties"** (the first-party data era)

Device IDs were the mobile equivalent of third-party cookies—powerful, useful, and increasingly restricted by privacy regulations.

The future? Identity will be:
- **Fragmented** - Some tracking here, none there, maybe a little over there
- **Logged-in** - If users don't sign in, you won't know who they are
- **Modeled** - Machine learning will try to fill the gaps (with varying success)
- **Contextual** - When you can't identify people, target content instead

Is it better for users? Probably. Is it better for advertisers? Depends on whether you invested in building direct relationships or just relied on platform tracking.

Either way, the wild west of device tracking is ending. Time to adapt.