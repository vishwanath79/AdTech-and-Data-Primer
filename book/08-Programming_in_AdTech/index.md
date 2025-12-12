# Chapter 8: Programming in AdTech

Theory is great. Code is better. 

You can read about RTB, header bidding, and creative optimization all day, but until you've actually built something that bids, serves ads, or optimizes creatives, it stays abstract. This chapter changes that.

We're building practical AdTech systems from scratch—ad servers, bidding algorithms, A/B testing frameworks, and more. Not production-grade enterprise systems (those are massive), but working implementations that demonstrate the core concepts.

If you're a data engineer or developer working in AdTech, these examples will give you the foundation to understand what the platforms are actually doing under the hood. If you're building your own AdTech solutions, this is your starting point.

## What's Covered

### [Introduction to AdTech Programming](./01-Introduction.md)

Setting the stage: what we're building, what tools we'll use, and how these examples relate to real-world AdTech systems.

### [Building a Simple Ad Server](./02-Adserver.md) | [Notebook](./02-Adserver.ipynb)

Ad servers are the backbone of digital advertising. We'll build one from scratch—handling ad requests, targeting logic, creative selection, and basic analytics. Python/Flask implementation.

### [Header Bidding Basics](./03-Header%20Bidding%20Basics.md) | [Notebook](./03-Header%20Bidding%20Basics.ipynb)

Before the page loads its ad server, header bidding lets multiple demand sources compete. We implement the client-side wrapper and server-side auction logic.

### [Real-Time Bidding Implementation](./04-Real%20Time%20Bidding.md) | [Notebook](./04-Real%20Time%20Bidding.ipynb)

Millisecond auctions at scale. We build a simplified RTB bidder that evaluates bid requests, applies targeting, calculates bids, and submits responses in real-time.

### [Creative A/B Testing](./05-Creative%20AB%20Testing.md)

Testing which ad creative performs better. We implement even rotation, performance-based selection, and a complete A/B testing framework including DCO (Dynamic Creative Optimization).

### [Deal IDs & Private Marketplace (PMP)](./06-Deal%20IDs%20%26%20Private%20Marketplace%20(PMP).md)

Private marketplaces and programmatic guaranteed deals. Building deal management, inventory reservation, and priority bidding systems.

### [Client-Side vs Server-Side Tagging](./07-Client%20%26%20Server%20Side%20Tagging.md)

How tracking actually works. We implement both client-side JavaScript tracking and server-side tagging approaches, comparing benefits and trade-offs.

## Why This Matters

Reading about AdTech gives you knowledge. Building AdTech gives you understanding.

When you've written the code that:
- Selects which ad to show based on targeting criteria
- Calculates optimal bids in real-time
- Runs A/B tests and picks winners
- Tracks impressions and conversions

...you understand the system at a level that reading alone can't provide. You know what's possible, what's hard, and where the edge cases hide.

These implementations are simplified—real AdTech platforms handle billions of requests per day, use distributed systems, and have sophisticated fraud detection and optimization. But the core logic? That's what we're building here.

## Technical Prerequisites

You should be comfortable with:
- **Python** - All examples use Python (some Flask for web servers)
- **Basic HTTP/REST APIs** - Ad systems talk via HTTP
- **JSON** - Data format for bid requests/responses
- **JavaScript basics** - For client-side implementations

If you're rusty on any of these, you'll still follow along—the code is heavily commented and explained.

## How to Use This Chapter

**Option 1: Read and Learn**
Read through the implementations in the markdown files to understand how things work. You'll see the actual logic platforms use (simplified, but representative). The code is organized into logical sections with clear explanations.

**Option 2: Interactive Learning with Notebooks**
Several chapters include Jupyter notebooks (`.ipynb` files) that you can run interactively:
- [Ad Server Notebook](./02-Adserver.ipynb) - Run the ad server and experiment with targeting
- [Header Bidding Notebook](./03-Header%20Bidding%20Basics.ipynb) - Test header bidding auctions
- [Real-Time Bidding Notebook](./04-Real%20Time%20Bidding.ipynb) - Experiment with RTB exchanges

Notebooks let you modify code, see outputs, and experiment in real-time. Perfect for hands-on learning.

**Option 3: Download and Run**
Full standalone scripts are available in the `programmatic/` directory. Clone the repository and run them locally to see the systems in action.

**Option 4: Adapt for Your Needs**
Take these implementations as starting points for your own projects. Need an internal ad server? Start with Chapter 8.2. Building a bidding system? Chapter 8.4 has your foundation.

### Understanding Code Examples

Each chapter provides code in multiple formats:
- **Markdown files** - Complete code with explanations, organized into logical sections. Best for reading and reference.
- **Jupyter notebooks** - Interactive, executable versions. Best for experimentation and learning by doing.
- **Standalone scripts** - Ready-to-run Python files in `programmatic/`. Best for quick testing or integration.

All formats contain the same core logic—choose what works best for your learning style.

## A Note on Production Systems

These are teaching implementations, not production systems. Real AdTech platforms need:
- **Scale**: Handling millions of requests per second
- **Low latency**: Sub-100ms response times
- **Fault tolerance**: No single points of failure
- **Security**: Protection against fraud and attacks
- **Compliance**: Privacy regulations, data retention policies

We're ignoring most of that to focus on the core logic. Don't deploy these to production without serious hardening. But do use them to understand how the pieces fit together.

Let's build some AdTech.

