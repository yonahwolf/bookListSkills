---
name: flight-search
description: Flight search assistant that helps find the best cash and award flight options based on the user's credit cards, loyalty programs, and transfer partners. Use when the user says "find flights", "search flights", "look for flights", "what's the best way to fly to X", or provides an origin, destination, and travel dates.
---

# Flight Search Skill

A skill for finding the best flight options — both cash and award — tailored to the user's credit card portfolio and loyalty program memberships.

---

## ✏️ USER CONFIG (Edit This Section)

This section personalizes results. Update it to match your own cards and programs.

```yaml
# --- TRANSFERABLE POINTS CARDS ---
# Cards that earn flexible points you can transfer to airlines
transferable_cards:
  - name: Chase Sapphire Reserve
    program: Chase Ultimate Rewards
    transfer_partners:
      - United MileagePlus
      - Air Canada Aeroplan
      - British Airways Executive Club
      - Flying Blue (Air France/KLM)
      - Singapore KrisFlyer
      - Virgin Atlantic Flying Club
      - Southwest Rapid Rewards
      - Iberia Plus
      - Turkish Miles&Smiles
    travel_portal_cpp: 1.5  # cents per point when booking through Chase Travel portal
    notes: "Priority Pass lounge access, $300 travel credit annually"

  - name: Amex Platinum
    program: Amex Membership Rewards
    transfer_partners:
      - Delta SkyMiles
      - Air Canada Aeroplan
      - British Airways Executive Club
      - Flying Blue (Air France/KLM)
      - Singapore KrisFlyer
      - Virgin Atlantic Flying Club
      - ANA Mileage Club
      - Cathay Pacific Asia Miles
      - Turkish Miles&Smiles
      - Iberia Plus
      - Etihad Guest
      - Hawaiian Miles
    travel_portal_cpp: 1.0
    notes: "Centurion Lounge access, $200 airline fee credit, Fine Hotels & Resorts"

# --- AIRLINE CO-BRAND CARDS ---
# Cards that earn miles directly with an airline + provide perks
cobrand_cards:
  - airline: Delta Air Lines
    program: Delta SkyMiles
    perks:
      - Free first checked bag (primary cardholder + up to 8 companions on same reservation)
      - Priority boarding
      - 20% discount on in-flight purchases
    notes: "Check your specific card tier for companion certificate eligibility"

  - airline: American Airlines
    program: AAdvantage
    perks:
      - Free first checked bag (primary cardholder + up to 4 companions)
      - Preferred boarding
      - 25% discount on in-flight food and beverage
    notes: ""

  - airline: JetBlue
    program: TrueBlue
    perks:
      - Free first checked bag (primary cardholder + up to 3 companions)
      - Mosaic benefits on spend
    notes: ""

# --- LOYALTY PROGRAMS WITH MEANINGFUL BALANCES OR STATUS ---
loyalty_programs:
  - program: Delta SkyMiles
    airline: Delta Air Lines
    status: ""  # e.g. Silver, Gold, Platinum, Diamond — leave blank if none
    notes: "Earned via Amex Platinum transfer or Delta co-brand card"

  - program: TrueBlue
    airline: JetBlue
    status: ""
    notes: "Earned via JetBlue co-brand card"

# --- HOME AIRPORTS (in priority order) ---
home_airports:
  - JFK
  - LGA
  - EWR

# --- API KEYS (optional — enhances award search) ---
# Get a free key at https://seats.aero
seats_aero_api_key: ""   # Leave blank to skip award availability lookup

# Get a free key at https://developers.amadeus.com
amadeus_api_key: ""      # Leave blank to skip Amadeus cash fare lookup
amadeus_api_secret: ""
```

---

## Overview

When this skill is triggered, it helps the user find the best way to fly between two cities — comparing cash prices, points redemptions, and award availability — then highlights which options are most relevant given their specific cards and loyalty memberships.

---

## Workflow

### Step 1: Gather Search Parameters

If not already provided, ask the user for:
- **Origin** (city or airport code)
- **Destination** (city or airport code)
- **Travel dates** (departure date; return date if round-trip)
- **Number of travelers** (adults; note if any are children)
- **Cabin class** (Economy / Premium Economy / Business / First)
- **Flexibility** (exact dates, or flexible within a window?)
- **Trip type** (one-way or round-trip)

If the user's home airports are configured, suggest the nearest one(s) as defaults.

### Step 2: Identify Relevant Programs

Before searching, cross-reference the destination with the user's config to identify:

1. **Which airlines fly the route** — look up major carriers serving origin → destination
2. **Which of the user's programs cover those airlines** (direct or partner awards)
3. **Which cards could earn or redeem on this trip**

Summarize this briefly before presenting results: *"For JFK → LAX, you have good options via Delta (SkyMiles via Amex), American (AAdvantage + co-brand perks), and JetBlue (TrueBlue). United is also bookable via Chase → MileagePlus transfer."*

### Step 3: Search for Flights

#### 3a. Cash Fares — Google Flights Deep Link
Always generate a Google Flights deep link for the user to explore cash prices. Construct the URL:

```
https://www.google.com/travel/flights/search?tfs=CBwQAhoeEgoyMDI1LTA2LTE1agcIARIDSkZLcgcIARIDTEFYGh4SCjIwMjUtMDYtMjJqBwgBEgNMQVhyBwgBEgNKRks
```

Instead of constructing the encoded URL manually, provide a plain-language link description and instruct the user to click through, OR use web search to find current fares: search `"{origin} to {destination} flights {dates} {cabin}"` and summarize results.

Use web search to get a rough cash price range. Search: `"flights {ORIGIN} to {DESTINATION} {DATE} {CABIN} price"`

#### 3b. Award Availability — Seats.aero API (if API key is configured)

If `seats_aero_api_key` is set, query the Seats.aero API:

```
GET https://seats.aero/partnerapi/availability
Headers: Partner-Authorization: {seats_aero_api_key}
Params:
  origin_airport: {ORIGIN}
  destination_airport: {DESTINATION}
  start_date: {DATE}
  end_date: {DATE + flexibility window}
  cabin: economy | business | first
```

Parse the response and filter results to only show programs the user can actually use (from their config).

If no API key is configured, skip this step and note: *"For live award availability, consider signing up for a Seats.aero API key (seats.aero) — add it to your skill config for automatic award search."*

#### 3c. Award Availability — Web Search Fallback

If no Seats.aero key, use web search to check award availability on key programs:
- Search `"[PROGRAM] award availability [ORIGIN] [DESTINATION] [MONTH]"` for the top 2-3 relevant programs
- Check ThePointsGuy, View from the Wing, or Upgraded Points for current redemption sweet spots on this route

### Step 4: Build the Recommendation Table

Present results in a structured comparison. For each viable option, show:

| Option | Airline | Route | Cabin | Cash Price | Points Cost | Program | Transfer From | Perks / Notes |
|--------|---------|-------|-------|------------|-------------|---------|---------------|---------------|
| 1 | Delta | JFK→LAX | Economy | ~$189 | 8,000 miles | SkyMiles | Amex Platinum | Free bag (Delta card) |
| 2 | American | JFK→LAX | Economy | ~$179 | 7,500 miles | AAdvantage | Direct (AA card) | Free bag (AA card) |
| 3 | JetBlue | JFK→LAX | Economy | ~$199 | 9,800 pts | TrueBlue | Direct (JetBlue card) | Free bag (JetBlue card) |
| 4 | United | JFK→LAX | Economy | ~$195 | 12,500 miles | MileagePlus | Chase UR → United | — |

Multiply points costs by number of travelers automatically.

### Step 5: Highlight Best Value Options

After the table, provide a brief **"My Take"** section that:

1. **Best cash option** — cheapest fare + notes on which card to pay with for max earning
2. **Best points redemption** — highest cents-per-point value; flag if it's a sweet spot
3. **Best with your perks** — which option benefits most from the user's co-brand card perks (e.g., free bags saving $35/person each way)
4. **Transfer recommendation** — if award travel makes sense, which transfer to make (e.g., "Transfer 15,000 Chase UR → United MileagePlus")

### Step 6: Offer Follow-Up Actions

Ask if the user wants to:
- Explore a different date range or be flexible (±3 days)
- Look at premium cabin options
- See connections/layovers vs. nonstop only
- Get a direct link to book on a specific airline's site

---

## Transfer Partner Reference

When recommending transfers, use these standard transfer ratios (all are 1:1 unless noted):

| From | To | Ratio | Transfer Time |
|------|----|-------|---------------|
| Chase UR | United MileagePlus | 1:1 | Instant |
| Chase UR | Air Canada Aeroplan | 1:1 | Instant |
| Chase UR | British Airways | 1:1 | Instant |
| Chase UR | Flying Blue | 1:1 | Instant |
| Chase UR | Virgin Atlantic | 1:1 | Instant |
| Chase UR | Turkish Miles&Smiles | 1:1 | Instant |
| Amex MR | Delta SkyMiles | 1:1 | Instant |
| Amex MR | Air Canada Aeroplan | 1:1 | Instant |
| Amex MR | British Airways | 1:1 | Instant |
| Amex MR | Flying Blue | 1:1 | Instant |
| Amex MR | Virgin Atlantic | 1:1 | Instant |
| Amex MR | ANA | 1:1 | 2-3 days |
| Amex MR | Singapore KrisFlyer | 1:1 | 1-2 days |

> ⚠️ Always confirm current transfer ratios before recommending a transfer — they can change. Check the issuer's website.

---

## Award Sweet Spots to Know

Proactively mention these when relevant to the route:

- **Flying Blue Promo Awards** — Air France/KLM run monthly promos with up to 50% off awards; transferable from both Chase and Amex
- **Aeroplan** — Strong partner network, no fuel surcharges on most partners, solid for transatlantic
- **British Airways Avios** — Great for short-haul on American (e.g., under 1,151 miles = 7,500 Avios one-way in economy)
- **Virgin Atlantic** — Excellent for Delta One business class redemptions (often 50,000 miles vs. 200,000+ SkyMiles)
- **Turkish Miles&Smiles** — Among the best rates for United flights in business class
- **ANA** — Best rates for transpacific premium cabins

---

## Cents-Per-Point Valuations (Reference)

Use these to calculate award value vs. cash price:

| Program | Conservative CPP | Good Redemption CPP |
|---------|-----------------|---------------------|
| Chase UR (portal) | 1.5¢ | — |
| Amex MR (portal) | 1.0¢ | — |
| United MileagePlus | 1.2¢ | 1.5–2.0¢ |
| Delta SkyMiles | 1.1¢ | 1.3–1.5¢ |
| American AAdvantage | 1.4¢ | 1.5–2.0¢ |
| Air Canada Aeroplan | 1.5¢ | 2.0–2.5¢ |
| British Airways Avios | 1.5¢ | 2.0¢+ (short haul) |
| Flying Blue | 1.3¢ | 1.5–2.0¢ |
| Virgin Atlantic | 1.5¢ | 2.0–3.5¢ (Delta J) |
| TrueBlue | 1.3¢ | 1.5¢ |

A redemption is generally **"good"** if CPP > 1.5¢ and **"excellent"** if CPP > 2.0¢.

Formula: `CPP = (cash price in cents) ÷ (miles/points required)`

---

## Example Interaction

**User:** "Find me flights from JFK to Miami, March 15-18, 2 adults, economy"

**Claude:**

> For JFK → MIA, you have strong options across your programs. American flies this nonstop frequently and you have both the AAdvantage co-brand (free bags!) and access to AAdvantage miles directly. JetBlue also flies this route nonstop with your TrueBlue membership and free bag perk. Delta is bookable via your Amex Platinum → SkyMiles transfer.
>
> [Table of options with cash prices, award costs, perks]
>
> **My Take:**
> - **Best cash:** American at ~$149/person ($298 total) — pay with Amex Platinum for 1x MR, or AA card for 2x AAdvantage miles
> - **Best award:** American AAdvantage at 7,500 miles/person (15,000 total for 2) — excellent 2.0¢ CPP vs. $149 cash
> - **Best with perks:** Both American and JetBlue give you free bags — saves ~$70 total round-trip for 2 people. Factor that into your cash comparison.
> - **Transfer suggestion:** If using AAdvantage, you already earn miles via your AA co-brand card. No transfer needed.

---

## Notes for Skill Maintainers

- Award redemption rates change frequently. The CPP table and sweet spots section should be reviewed quarterly.
- Seats.aero API documentation: https://seats.aero/blog/partner-api
- Amadeus free tier documentation: https://developers.amadeus.com
- If the user has no API keys configured, the skill still provides strong value through web search + Google Flights links + transfer partner guidance.
