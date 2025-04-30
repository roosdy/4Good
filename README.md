# 4Good - Social Impact Platform

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Status: Development](https://img.shields.io/badge/status-development-orange.svg)](-)

**A real-time, multi-network crisis dashboard that surfaces verified, on-the-ground media and routes one-click micro-donations to vetted NGOs—complete with impact receipts and community challenges.**

---

## Table of Contents

1.  [Value Proposition](#value-proposition)
2.  [Key Features (MVP)](#key-features-mvp)
3.  [Technical Architecture](#technical-architecture)
4.  [Technology Stack](#technology-stack)
5.  [Getting Started (Conceptual)](#getting-started-conceptual)
6.  [Configuration](#configuration)
7.  [API Limits (Initial Non-Production/Development)](#api-limits-initial-non-productiondevelopment)
8.  [Business & Compliance Considerations](#business--compliance-considerations)
9.  [Project Assets](#project-assets)
10. [Contributing](#contributing)
11. [License](#license)

---

## 1. Value Proposition

4Good aims to provide a more robust and engaging platform for social impact than simple social media curation.

*   **Differentiation:** Moves beyond reliance on single platforms like Instagram by diversifying content sources (Instagram, X/Twitter, direct uploads) and adding unique impact tracking features. This makes the platform more resilient to API changes.
*   **Retention Loop:** Encourages sustained user engagement through gamification elements like impact receipts, badges, streaks, and community leaderboards, providing positive reinforcement beyond the initial donation.
*   **B-corp / ESG Alignment:** Offers corporations authentic social impact stories and turnkey sponsorship opportunities, aligning with growing ESG (Environmental, Social, and Governance) mandates.

---

## 2. Key Features (MVP)

The Minimum Viable Product focuses on core functionality:

*   **Discover:**
    *   Multi-source crisis feed (initially Instagram Hashtags, X/Twitter).
    *   Search by hashtag or keyword.
    *   Shareable URLs for specific events or posts.
*   **Donate:**
    *   One-click micro-donations via Stripe Payment Links.
    *   Profile pages for vetted NGOs.
*   **Engage:**
    *   Impact receipts confirming donation details (amount, NGO).
    *   Personal user dashboard tracking impact.
*   **Moderate:**
    *   Automated content moderation using Amazon Rekognition (violence/adult content).
    *   Manual review queue for flagged content.

*(Future versions may include AI summaries, geospatial views, expanded payment options, recurring donations, community challenges, and more sophisticated moderation.)*

---

## 3. Technical Architecture

4Good leverages a serverless architecture on AWS for cost-efficiency and scalability.

*   **Frontend:** Static site built with Next.js/React, hosted on AWS S3 and distributed globally via CloudFront.
*   **Backend API:** AWS AppSync (GraphQL) or AWS Lambda functions fronted by API Gateway handle API requests.
*   **Data Storage:**
    *   DynamoDB stores feed metadata and user data (suitable for NoSQL access patterns).
    *   S3 stores user-generated media (images, videos).
    *   Aurora Serverless v2 is an option if complex relational data queries are needed.
*   **Data Ingestion:**
    *   Retrieves user-generated content (UGC) via Instagram Graph API (Hashtag Search) and X/Twitter Search v2 API, filtered for crisis-related keywords.
    *   Includes mechanisms for direct uploads from partner NGOs as a fallback/alternative.
*   **Content Moderation:** Incoming media is processed by Amazon Rekognition, potentially routed through AWS Step Functions for workflow management before landing in a manual review queue if necessary.
*   **Payments & Impact:**
    *   Stripe Payment Links handle donations, minimizing PCI compliance scope.
    *   Successful payments trigger Stripe webhooks.
    *   An AWS Lambda function processes the webhook, records the transaction (impact receipt), updates the user's dashboard (via WebSocket or polling), and contributes to a public Impact Ledger page.
    *   Optional Stripe Climate integration allows users to contribute a percentage to carbon removal.

---

## 4. Technology Stack

*   **Frontend:** Next.js, React
*   **Backend:** Node.js (for Lambda)
*   **API:** AWS AppSync (GraphQL) / AWS Lambda + API Gateway (REST)
*   **Database:** AWS DynamoDB, AWS S3, (Optional: AWS Aurora Serverless v2)
*   **Hosting:** AWS S3, AWS CloudFront
*   **Compute:** AWS Lambda
*   **Payments:** Stripe (Payment Links, Webhooks, Optional: Climate)
*   **Content Moderation:** Amazon Rekognition, AWS Step Functions
*   **Data Ingestion APIs:** Instagram Graph API, X/Twitter API v2
*   **NGO Vetting APIs:** ACNC API (Australia), Guidestar API (Global)
*   **Deployment (Demo):** AWS Amplify

---

## 5. Getting Started (Conceptual)

As this project evolves, setup will involve:

1.  **Clone the repository:** `git clone https://github.com/roosdy/4good.git`
2.  **Install dependencies:** `cd 4good && npm install` (or `yarn install`)
3.  **Configure Environment Variables:** Set up a `.env` file with necessary API keys and configuration secrets (see [Configuration](#configuration)).
4.  **Deploy Infrastructure:** Use AWS CDK, Serverless Framework, or Terraform to provision the necessary AWS resources.
5.  **Run Locally:** `npm run dev` (or `yarn dev`) for local development.
6.  **Deploy:** Deploy the Next.js frontend (e.g., to S3/CloudFront or Amplify) and backend components (Lambda, AppSync).

---

## 6. Configuration

The application requires various API keys and settings, typically managed via environment variables (`.env` file locally, or secrets management in deployment):

*   `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`: For AWS SDK interactions.
*   `INSTAGRAM_APP_ID`, `INSTAGRAM_APP_SECRET`, `INSTAGRAM_ACCESS_TOKEN`: For Instagram Graph API access (requires Business/Creator account and App Review).
*   `TWITTER_API_KEY`, `TWITTER_API_SECRET_KEY`, `TWITTER_BEARER_TOKEN`: For X/Twitter API v2 access (requires developer account approval).
*   `STRIPE_SECRET_KEY`, `STRIPE_PUBLISHABLE_KEY`: For Stripe payments.
*   `STRIPE_WEBHOOK_SECRET`: To verify incoming Stripe webhooks.
*   `REKOGNITION_COLLECTION_ID` (if applicable): For Amazon Rekognition setup.
*   `ACNC_API_KEY` (if applicable): For Australian charity lookups.
*   `GUIDESTAR_API_KEY` (if applicable): For global charity lookups.
*   Database connection details (DynamoDB table names, Aurora endpoint if used).
*   API Gateway / AppSync endpoint URL.

---

## 7. API Limits (Initial Non-Production/Development)

For initial development, testing, and low-volume non-profit use, leveraging free tiers and basic access levels is crucial. Note that production use will likely exceed these limits and require paid tiers or higher access levels.

*   **Instagram Graph API (Hashtag Search):**
    *   Requires a Facebook Developer account, an associated App, and a linked Instagram Business/Creator Account.
    *   Subject to Business Use Case Rate Limiting. Typically allows **~200 API calls per hour per user token**, but Hashtag Search itself is more restricted.
    *   Public Hashtag Search is limited to **30 unique hashtags within a 7-day period** per Business Account. [1]
    *   Requires periodic App Review (every 60-90 days) to maintain permissions. [5]
    *   Media cannot be cached for > 24 hours without explicit permission. [5]
*   **X/Twitter API v2:**
    *   **Free Tier:** Provides **1,500 tweet posting requests per month** (app-level), **50 requests per 24 hours** for media uploads, and access to endpoints like Recent Search (limited results) and Filtered Stream (1 rule). Read-only access is often sufficient for ingestion. [2] Look into the Basic tier ($100/month) for more robust access if needed. [2]
    *   Requires a Developer Account and project setup.
*   **AWS Free Tier (Generally for the first 12 months):**
    *   **Lambda:** 1 million free requests per month and 400,000 GB-seconds of compute time per month. [3]
    *   **API Gateway:** 1 million HTTP API calls received per month. [3]
    *   **AppSync:** 250,000 query/mutation operations & 250,000 real-time updates per month. [3]
    *   **DynamoDB:** 25 GB of storage, 25 provisioned Write Capacity Units (WCUs), 25 provisioned Read Capacity Units (RCUs). [3]
    *   **S3:** 5 GB of Standard Storage, 20,000 Get Requests, 2,000 Put Requests. [3]
    *   **CloudFront:** 1 TB of data transfer out, 10,000,000 HTTP/S requests. [3]
    *   **Amazon Rekognition:** Free tier includes **5,000 images processed per month** for features like moderation labels for the first 12 months. [3]
    *   **Step Functions:** 4,000 state transitions per month. [3]
*   **Stripe:**
    *   No specific API rate limit publicly listed for the free tier, but generally high and designed for production traffic. Limits are often around **100 read operations/sec and 100 write operations/sec** in live mode (test mode may be lower/less strict). [4] Exceeding limits results in `429` errors. Transactions incur standard processing fees.
*   **ACNC API / Guidestar API:**
    *   Limits vary. ACNC API usage often requires registration. Guidestar typically requires partnership or specific paid plans for API access beyond basic lookups. Check their respective documentation for non-profit access details.

**Note:** These limits are estimates based on publicly available information (as of late 2024) and are subject to change. Always consult the official documentation for each service. Production scaling will require moving beyond free tiers for most services.

---

## 8. Business & Compliance Considerations

*   **Entity Setup (Australia):** Requires registration as a company. If holding funds directly, obtain an ACNC charitable fundraising license. Alternatively, partner with a payment facilitator forwarding directly to DGR-endorsed organisations.
*   **Privacy:** Adhere to Australian Privacy Principles (similar to GDPR). Implement consent mechanisms, data export, and deletion capabilities.
*   **Platform Terms:** Comply with Meta (Instagram) and X/Twitter API terms, including app reviews and data caching restrictions. Note the legacy Instagram API endpoint migration deadline (Apr 21, 2025). [5]
*   **NGO Vetting:** Utilize official APIs like ACNC (Australia) and Guidestar (Global) to verify the legitimacy of listed NGOs.

---

## 9. Project Assets

As part of showcasing this project, the following assets are planned:

*   **One-Pager PDF:** Summary including mock-ups, problem/solution, tech stack, and role.
*   **Live Demo:** Read-only version deployed (e.g., on AWS Amplify free tier) with a mocked donation flow (Stripe test mode).
*   **Public GitHub Repository:** This repository, featuring a well-structured mono-repo, MIT license, and detailed READMEs (including Architecture Decision Records).
*   **Slide Deck (5 slides):** Concise overview for presentations, including market context and estimated operational costs.
*   **Blog Post:** Narrative describing the project's development and technical choices.

---

## 10. Contributing

Contributions are welcome! Please follow standard Fork and Pull Request workflows. Ensure code is well-tested and adheres to existing style guides. (Consider adding a `CONTRIBUTING.md` file for more detail).

---

## 11. License

This project is licensed under the MIT License. See the `LICENSE` file for details.
