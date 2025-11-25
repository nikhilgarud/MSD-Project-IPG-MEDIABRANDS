# MSD-Project-IPG-MEDIABRANDS
This project delivers a full-funnel media analytics dashboard for MSD / Merck, covering digital ads, influencers, offline media, website behaviour and chatbot engagement in a single Power BI experience.
# MSD AP Media IQ Analytics – Power BI End-to-End Media Dashboard

This repository contains an end-to-end **media analytics solution** built for MSD / Merck, branded as **MSD AP Media IQ Analytics**.

The solution connects **Amazon Redshift → Power BI**, models a unified **Fact Media Master** table, and delivers a full-funnel view across:

- Digital Paid Media
- Influencer Media
- Offline (TV, OOH, Events)
- Website (GA4)
- Chatbot

> Grain: **date × campaign × market × platform × objective** – consistent across all media sources. :contentReference[oaicite:5]{index=5}

---

## 1. Business Problem

MSD runs multi-channel HPV awareness campaigns across APAC. Data comes from:

- Paid media platforms (Meta, TikTok, DV360, Google Ads, Programmatic)
- Influencer platforms (Instagram, Facebook, TikTok)
- Offline channels (TV, Radio, OOH, Events)
- Website analytics (GA4)
- Chatbot interactions

Stakeholders needed **one consolidated view** to answer:

- Which markets, campaigns and channels drive **awareness, education, and consideration**?
- What is the **cost efficiency** (CPM, CPC, CPV, Cost per HVA) across markets and currencies?
- How do **online, offline, influencer, website and chatbot** activities work together in the funnel? :contentReference[oaicite:6]{index=6}

---

## 2. Solution Overview

I designed and built a **Power BI semantic model and interactive report** that:

- Standardises metrics across all channels into a **unified master fact**.
- Implements a **three-stage funnel**:
  - **Awareness** – Reach, Impressions, Views, Frequency
  - **Education** – Clicks, Sessions, Engagement, Time Spent
  - **Consideration** – High-Value Actions (HVA), Cost per HVA, Sentiment :contentReference[oaicite:7]{index=7}
- Supports **dynamic currency conversion** (SGD, USD + local currencies).
- Handles **monthly reach** logic and proportional reach for custom date ranges. 

---

## 3. Architecture

**Data Source:** Amazon Redshift (UAT / PROD)

- Redshift view: `analytics.fact_media_master`
- Built using a UNION of:
  - `digital_paid_media`
  - `offline_media_event`
  - `offline_media_non_event`
  - (extendable to Influencer, Website, Chatbot) :contentReference[oaicite:9]{index=9}

**ETL / Data Engineering:**

- Redshift SQL creates a **row-level unified fact** with consistent columns for:
  - Budget, Impressions, Clicks, Views, Sessions, HVA
  - Offline GRPs, Reach, Circulation, Event attendance
  - Influencer views, ER, sentiment
  - Website sessions, engaged sessions, HVA
  - Chatbot starts, drop-off, CTA clicks :contentReference[oaicite:10]{index=10}

**Semantic Layer (Power BI):**

- Fact tables:
  - `Digital_Paid_Media`
  - `Offline_Event`
  - `Offline_NonEvent`
  - `Influencer_Data`
  - `Website_GA4_Data`
  - `Chatbot_Data` :contentReference[oaicite:11]{index=11}
- Dimension tables:
  - `Dim_Date`, `Dim_Market`, `Dim_Currency`, `Dim_CampaignType`,
    `Dim_Platform`, `Dim_Objective`.

---

## 4. Data Model & Grain

- Master grain: **date + campaign + market + platform + objective**.
- Relationships are **single-direction** from dimensions → facts to avoid ambiguity. 
- Composite key logic is first applied in `Digital_Paid_Media` and shared across other sources.

Key design notes:

- No bi-directional relationships.
- Sentiment / influencer calculations pre-aggregated where possible.
- Currency conversion never hard-coded in facts – handled via `CurrencyRates` table. :contentReference[oaicite:13]{index=13}

---

## 5. Metrics & DAX Highlights

Core measures (examples):

- **Budget Spend (dynamic currency)**  
  - Handles:
    - Multi-market → default SGD
    - Single-market + local currency
    - USD and other currencies via `RateToSGD`
  - Formats output with country symbol (S$, HK$, ₹, ₫, etc.). :contentReference[oaicite:14]{index=14}

- **CPM, CPC, CPV, CPR**  
  - Each uses converted spend and the appropriate denominator:
    - CPM = Spend / Impressions × 1,000
    - CPC = Spend / Clicks
    - CPV = Spend / Video Views
    - CPR = Spend / Monthly Reach

- **Estimated Reach**  
  - Reach is stored at **monthly level**. A custom DAX allocates reach
    proportionally by overlapping days for any date range selection. :contentReference[oaicite:15]{index=15}

- **Engagement & Funnel Measures**
  - CTR, VVR
  - Engaged Sessions %, Click-to-Session %
  - HVA Completion %, Cost per HVA
  - Sentiment % (Positive / Neutral / Negative)
  - Repeat Rate for video completion. :contentReference[oaicite:16]{index=16}



---

## 6. Report Pages & UX

The left-hand navigation provides a clear, app-like UX: :contentReference[oaicite:17]{index=17}

1. **Funnel Overview**  
   - High-level KPIs for Paid, Influencer, Offline, Website and Chatbot.
   - Split by funnel stage: Awareness, Education, Consideration.

2. **Funnel Snapshots**  
   - Campaign-level overview of spend and funnel distribution.
   - Drill-through buttons for: Campaign Overview, Awareness, Education, Consideration.

3. **Digital Paid Delivery**  
   - Four sub-views: Campaign, Channel, Audience, Creative.
   - Deep-dive into CPM, CPC, CPV, CTR by dimension.

4. **Offline Paid Channel**  
   - TV/OOH/Print view (Non-Event) with Reach, GRPs, Frequency, Circulation.
   - Event view with Registrants, Participants, Show-up rate, Cost per Participant.

5. **Influencer Performance**  
   - Organic & Paid influencer views, ER, CPV, sentiment.
   - Tier-based (Mega/Macro) breakdown.

6. **Website Performance (GA4)**  
   - Sessions, Engaged Sessions, HVA, Cost per Engaged Session.
   - Split by traffic source and landing page.

7. **Chatbot Performance**  
   - Chat starts, engagement, CTA clicks, drop-off and retention %.

8. **Glossary & Notes**  
   - Embedded KPI definitions and developer notes for future maintainers.

---

## 7. Tech Stack

- **Data Warehouse:** Amazon Redshift
- **BI Tool:** Microsoft Power BI
- **Data Connectivity:** Import mode with environment parameters (UAT / PROD) :contentReference[oaicite:18]{index=18}
- **Languages:** SQL (Redshift), DAX, Power Query (M)
- **Source Systems:** Paid Media Platforms, GA4, Influencer reports, Offline buys, Chatbot logs.

---

## 8. How to Run Locally

1. Clone this repository.
2. Open `/pbix/MSD_AP_Media_IQ_Analytics.pbix` in **Power BI Desktop**.
3. For demo mode:
   - Use the sample CSV/Excel files in `/sample-data`.
4. For full Redshift mode:
   - Update parameters `Environment`, `RedshiftServer` in Power Query as per the **Redshift Master Integration Guide**. :contentReference[oaicite:19]{index=19}
5. Refresh all tables and interact with the report.

---

## 9. Screenshots

Add a `/images` folder with:

- `01_Funnel_Overview.png`
- `02_Funnel_Snapshots.png`
- `03_Digital_Paid_Campaign.png`
- `04_Offline_Channel.png`
- `05_Influencer_Performance.png`
- `06_Website_Performance.png`
- `07_Chatbot_Performance.png`



---

## 10. Author

**Your Name – Power BI / Data Analyst**

- Focus areas: Media Analytics, Power BI, DAX, SQL, Redshift

