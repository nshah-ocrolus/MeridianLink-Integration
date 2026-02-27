# Product Requirements Document (PRD)

## MeridianLink Document Integration PoC

**Version:** 1.0  
**Phase:** 1 — Proof of Concept  
**Status:** Complete

---

## 1. Background

We have a product (**Product A**) that receives mortgage-related documents from another product, processes them (reviews, categorizes, validates), and sends them back.

Currently, Product A integrates with **Encompass** (a major mortgage industry platform). The goal of this project is to build a similar integration with **MeridianLink** — another mortgage platform that stores and manages loan documents.

This Phase 1 PoC demonstrates the integration approach and proves it works technically, before committing to full development.

---

## 2. Objective

Build a functional Proof of Concept that shows the complete document flow:

**MeridianLink → Product A → MeridianLink**

Specifically:
1. Product A connects to MeridianLink's API
2. Product A downloads documents for a given loan
3. Product A processes those documents
4. Product A sends the processed documents back to MeridianLink

---

## 3. What's Included in Phase 1

| Item | Description |
|------|------------|
| Connect to MeridianLink | Authenticate using OAuth 2.0 (the same security standard used by Google, Microsoft, etc.) |
| Receive documents | Download all documents attached to a loan number from MeridianLink |
| Process documents | Apply a placeholder processing step (real processing logic will come in later phases) |
| Return documents | Upload the processed documents back to MeridianLink |
| Dashboard | A visual web interface to trigger the pipeline and see it run |
| Mock mode | A simulation mode that works without live API access, using realistic sample mortgage documents |

---

## 4. What's NOT Included in Phase 1

These are intentionally left out for now and would be part of future phases:

| Item | Why It's Excluded |
|------|------------------|
| Real document processing (OCR, AI classification, compliance) | Phase 1 only needs to prove the integration path works |
| Multi-user login / access control | Not needed for a PoC |
| Database storage | Jobs are stored in memory — sufficient for demo purposes |
| Production deployment | PoC runs locally; production hosting is a later decision |
| Automated test suite | Code will be handed to a dev team who will add their own tests |
| Error retry / queuing system | Production-grade reliability features come after the approach is finalized |

---

## 5. Functional Requirements

These are the things the system must do:

| # | Requirement | Priority | Status |
|---|------------|----------|--------|
| 1 | Authenticate with MeridianLink using OAuth 2.0 | Must Have | ✅ Done |
| 2 | List all documents for a given loan number | Must Have | ✅ Done |
| 3 | Download each document as a PDF | Must Have | ✅ Done |
| 4 | Process each document (placeholder logic) | Must Have | ✅ Done |
| 5 | Upload processed documents back to MeridianLink | Must Have | ✅ Done |
| 6 | Show pipeline progress in real-time on a dashboard | Must Have | ✅ Done |
| 7 | Work in simulation mode without live API access | Must Have | ✅ Done |
| 8 | Display job history (past pipeline runs) | Should Have | ✅ Done |
| 9 | Auto-refresh the OAuth token before it expires | Should Have | ✅ Done |

---

## 6. Technical Requirements

| # | Requirement |
|---|------------|
| 1 | Runs on Node.js 18 or later |
| 2 | Works on macOS, Windows, and Linux |
| 3 | No external databases needed |
| 4 | Configuration through a simple `.env` file |
| 5 | No build step — just `npm install` and `npm start` |
| 6 | All credentials excluded from version control |

---

## 7. MeridianLink API Interactions

The PoC uses MeridianLink's document management APIs. Here's how each API is used:

### 7.1 Authentication

| Detail | Value |
|--------|-------|
| **What it does** | Gets a secure access token to use MeridianLink's services |
| **URL** | `https://secure.mortgage.meridianlink.com/oauth/token` |
| **How it works** | Sends a `client_id` and `client_secret` (provided by MeridianLink) and receives back an access token |
| **Token duration** | 4 hours (auto-refreshes before expiry) |

### 7.2 List Documents

| Detail | Value |
|--------|-------|
| **What it does** | Gets a list of all documents attached to a specific loan |
| **Service** | MeridianLink EDocsService |
| **Method** | `ListEdocsByLoanNumber` |
| **Input** | Access token + Loan number |
| **Output** | List of documents with their names, types, and IDs |

### 7.3 Download a Document

| Detail | Value |
|--------|-------|
| **What it does** | Downloads a specific document as a PDF file |
| **Service** | MeridianLink EDocsService |
| **Method** | `DownloadEdocsPdfById` |
| **Input** | Access token + Document ID |
| **Output** | The PDF file content |

### 7.4 Upload a Document

| Detail | Value |
|--------|-------|
| **What it does** | Sends a processed document back to MeridianLink, attached to the original loan |
| **Service** | MeridianLink EDocsService |
| **Method** | `UploadPDFDocument` |
| **Input** | Access token + Loan number + Document type + PDF content |
| **Output** | Confirmation that the upload succeeded |

---

## 8. API Endpoints (PoC Application)

The PoC provides these REST API endpoints for triggering and monitoring the pipeline:

| Endpoint | What It Does |
|----------|-------------|
| `GET /api/health` | Check if the server is running |
| `POST /api/auth/test` | Test the connection to MeridianLink |
| `POST /api/integration/run` | Run the full pipeline for a loan number |
| `GET /api/integration/status` | Check if a pipeline is currently running |
| `GET /api/integration/history` | View all past pipeline runs |
| `GET /api/documents/:loanNumber` | List documents for a specific loan |

---

## 9. Success Criteria

How we know Phase 1 is complete:

| Criteria | Measurement | Status |
|----------|------------|--------|
| Pipeline runs end-to-end | All 4 steps (Authenticate → Receive → Process → Return) complete without error | ✅ Verified |
| Dashboard shows real-time progress | Pipeline steps visually animate as they execute | ✅ Verified |
| Mock mode works without internet | Full flow completes using simulated data | ✅ Verified |
| Code runs on Mac | `npm install` + `npm start` launches successfully | ✅ Verified |
| Credentials are secure | No passwords or API keys in the codebase | ✅ Verified |
| Processing is swappable | The processing step can be replaced without changing the rest of the system | ✅ Verified |

---

## 10. Risks and How We Handle Them

| Risk | Impact | How We Handle It |
|------|--------|-----------------|
| MeridianLink sandbox is unavailable | Can't test with real API | Mock mode provides identical demo capability |
| MeridianLink changes their API | Integration could break | XML parsing defensive — handles unexpected fields gracefully |
| Credentials get exposed | Security breach | Credentials stored in `.env` file, which is excluded from version control |
| Scope creep | Phase 1 takes longer | This PRD clearly defines what is and isn't in scope |

---

## 11. Phase 1 Deliverables

| Deliverable | Description | Status |
|-------------|------------|--------|
| **Working PoC** | Functional application demonstrating the full Receive → Process → Return flow | ✅ Complete |
| **Product Architecture Document** | High-level overview of how the integration works | ✅ Complete |
| **Product Requirements Document** | This document — scope, requirements, API interactions | ✅ Complete |
| **Local Setup Instructions** | Step-by-step guide to run the PoC on a MacBook | ✅ Complete |

---

## 12. What Comes Next

After Phase 1 is reviewed and the approach is approved, future phases would include:

- **API Access**: Switch from mock mode to live MeridianLink API
- **Real Processing Logic**: Replace the placeholder with actual document processing
- **Production Infrastructure**: Hosting, monitoring, error handling
- **Testing Suite**: Automated tests for reliability
- **Security Hardening**: Production-grade auth, logging, auditing
