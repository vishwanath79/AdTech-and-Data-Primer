# Chapter 7: Privacy and Regulation

For years, AdTech operated in the shadows. We tracked everything, stored everything, and sold everything. Users had no idea, and regulators didn't understand the tech well enough to stop it.

That party is over.

GDPR, CCPA, and a wave of global privacy laws have fundamentally changed the rules of the game. Privacy is no longer a "nice to have"—it's a legal requirement with massive fines attached.

This chapter explores the collision of surveillance capitalism and privacy rights.

## What's Covered

### [Privacy Concerns](./01-Privacy_Concerns.md)

Why are people angry? The "creepiness factor" of retargeting, data breaches, and the detailed profiles companies build without consent. Understanding the user perspective is key to understanding the regulation.

### [Regulations: GDPR, CCPA, and Beyond](./02-Regulations.md)

The alphabet soup of laws. We'll break down the core principles: consent, right to access, right to be forgotten. How these laws differ (opt-in vs. opt-out) and their global impact.

### [CMPs (Consent Management Platforms)](./03-CMPs.md)

The technical solution to a legal problem. Those "Accept Cookies" popups aren't just annoying UI—they are complex systems collecting and passing consent signals (TCF strings) downstream to ad exchanges.

### [Privacy by Design](./04-Privacy_by_Design.md)

Moving from compliance to strategy. How to build systems that respect privacy by default. Differential privacy, data clean rooms, and federated learning as technical answers to privacy challenges.

## Why This Matters

You can't architect a data system today without consulting a lawyer. Ignorance of privacy laws is a liability.

If you're building models, you need to know if you actually have the *legal right* to use the data training them. If you're storing user logs, you need a mechanism to delete them upon request.

Privacy regulation isn't just a constraint; it's a market force. It favors large players (walled gardens) who have direct user relationships and hurts intermediaries. Understanding these dynamics helps you predict where the industry is going—and keeps you out of court.

