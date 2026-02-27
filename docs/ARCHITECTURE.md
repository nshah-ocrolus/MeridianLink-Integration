# Product Architecture Document

## MeridianLink Document Integration PoC

**Version:** 1.0 — Phase 1  
**Purpose:** Show how Product A would integrate with MeridianLink to receive, process, and return mortgage documents.

---

## 1. What This System Does

This Proof of Concept demonstrates a document processing pipeline between two systems:

- **Product A** — Our document processing system (the software we are building)
- **Product B (MeridianLink)** — A mortgage industry platform that stores loan documents

The flow is straightforward:

```
  MeridianLink                  Product A                    MeridianLink
  (Document Source)             (Processing Engine)          (Document Destination)
  
  ┌─────────────┐              ┌──────────────────┐         ┌─────────────┐
  │  Loan Docs   │ ──RECEIVE──▶│  Review & Process │──SEND──▶│  Processed  │
  │  (PDFs)      │              │  Documents        │  BACK   │  Documents  │
  └─────────────┘              └──────────────────┘         └─────────────┘
```

In plain English: MeridianLink has mortgage documents (loan applications, paystubs, appraisals, etc.). Product A pulls those documents, processes them (reviews, categorizes, validates), and sends the processed results back to MeridianLink.

---

## 2. The Four Steps

Every time the system runs, it follows these four steps in order:

| Step | What Happens | How It Works |
|------|-------------|-------------|
| **1. Authenticate** | Product A proves its identity to MeridianLink | Sends OAuth credentials (client ID + secret) to MeridianLink and receives an access token |
| **2. Receive** | Product A downloads documents from MeridianLink | Calls MeridianLink's document service to list all documents for a loan, then downloads each one |
| **3. Process** | Product A processes each document | In Phase 1, this is a placeholder — it adds metadata stamps. In production, this would be the real processing logic (OCR, classification, compliance checks, etc.) |
| **4. Return** | Product A sends processed documents back to MeridianLink | Uploads each processed document back to MeridianLink, attached to the original loan |

---

## 3. System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        PoC Application                              │
│                                                                     │
│   ┌───────────────┐    ┌─────────────────┐    ┌─────────────────┐  │
│   │  Web Dashboard │    │   Integration   │    │   REST API      │  │
│   │  (Visual UI)   │    │   Orchestrator  │    │   (Endpoints)   │  │
│   └───────────────┘    └────────┬────────┘    └─────────────────┘  │
│                                 │                                   │
│                    ┌────────────┼────────────┐                      │
│                    ▼                         ▼                      │
│         ┌──────────────────┐     ┌──────────────────┐              │
│         │  MeridianLink    │     │  Document         │              │
│         │  API Client      │     │  Processor         │              │
│         └────────┬─────────┘     └──────────────────┘              │
│                  │                                                  │
└──────────────────┼──────────────────────────────────────────────────┘
                   │
                   │ HTTPS / SOAP XML
                   ▼
          ┌──────────────────┐
          │   MeridianLink   │
          │   Cloud Services │
          └──────────────────┘
```

### What Each Part Does

| Component | Role | Non-Technical Explanation |
|-----------|------|-------------------------|
| **Web Dashboard** | Visual interface | A web page where you click a button to run the pipeline and watch it execute in real-time |
| **Integration Orchestrator** | Central controller | The "brain" that coordinates the four steps in the right order and tracks progress |
| **REST API** | External access points | Allows other systems (or command-line tools) to trigger and monitor the pipeline |
| **MeridianLink API Client** | Communication layer | The code that talks to MeridianLink's servers — handles authentication, sending requests, and reading responses |
| **Document Processor** | Processing engine | Where the actual document processing happens. In Phase 1, this is a placeholder. In production, this is where the real business logic lives |

---

## 4. How Authentication Works

MeridianLink uses **OAuth 2.0** — a standard security protocol used by most modern APIs. Here's how it works:

```
Product A                                         MeridianLink
    │                                                  │
    │──── "Here are my credentials" ──────────────────▶│
    │     (client_id + client_secret)                  │
    │                                                  │
    │◀─── "Here's your access token" ─────────────────│
    │     (valid for 4 hours)                          │
    │                                                  │
    │──── "List documents for Loan #123" ─────────────▶│
    │     (includes access token as proof)              │
    │                                                  │
    │◀─── "Here are the documents" ───────────────────│
```

The credentials (`client_id` and `client_secret`) are generated from MeridianLink's **Vendor Portal**. The access token expires after 4 hours and automatically refreshes.

---

## 5. MeridianLink API Services Used

| Service | What It Does | When It's Called |
|---------|-------------|-----------------|
| **OAuth Token Endpoint** | Issues access tokens for authentication | Step 1 (Authenticate) |
| **EDocsService — ListEdocsByLoanNumber** | Returns a list of all documents attached to a loan | Step 2 (Receive) |
| **EDocsService — DownloadEdocsPdfById** | Downloads a specific document as a PDF | Step 2 (Receive) |
| **EDocsService — UploadPDFDocument** | Uploads a processed document back to a loan | Step 4 (Return) |

All document operations go through MeridianLink's **EDocsService**, which uses SOAP/XML messaging over HTTPS.

---

## 6. Two Operating Modes

The PoC can run in two modes:

| Mode | When to Use | What Happens |
|------|------------|-------------|
| **Simulated (Mock)** | Before API credentials are available | The system uses realistic fake data — 5 sample mortgage documents (loan application, appraisal, credit report, title insurance, employment verification) |
| **Live** | After API credentials are provided | The system connects to MeridianLink's real servers and processes actual loan documents |

Switching between modes is a single setting in the configuration file (`USE_MOCK=true` or `USE_MOCK=false`).

---

## 7. Technology Choices

| What | Technology | Why |
|------|-----------|-----|
| Application runtime | Node.js 18+ | Lightweight, cross-platform, great for API integrations |
| Web server | Express.js | Simple, widely used, easy for developers to understand |
| MeridianLink communication | Axios + fast-xml-parser | Handles HTTP requests and reads MeridianLink's XML responses |
| Dashboard | Plain HTML, CSS, JavaScript | No build tools needed — opens instantly in any browser |
| Configuration | .env file | Industry-standard way to manage settings and credentials securely |

---

## 8. Security

- **Credentials are never stored in the code** — they live in a `.env` file that is excluded from version control
- **OAuth tokens auto-refresh** — the system requests a new token before the current one expires
- **No sensitive data is logged** — passwords and tokens are not printed to the console
- **HTTPS only** — all communication with MeridianLink happens over encrypted connections
