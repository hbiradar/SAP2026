# Product Requirements Document (PRD)

**Title:** IBP Supplier Commit Quantity Portal
**Date:** 2026-08-03
**Owner:** Supply Chain Planning Team
**Solution Category:** CAP App

## Product Purpose & Value Proposition

**Elevator Pitch:**
Suppliers currently have no dedicated, simple way to communicate committed supply quantities to the planning team. This CAP application gives suppliers a lightweight web portal to enter annual commit quantities, and gives planners instant visibility of those commits inside their IBP supply planning context.

**Business Need:**
The planning team needs reliable supplier commit data integrated into IBP to align demand and supply plans. Without it, planners rely on emails or spreadsheets, leading to outdated and inconsistent data.

**Expected Value:**
- Faster supplier commit data collection — eliminating manual email/spreadsheet tracking.
- Improved supply plan accuracy in IBP by using actual supplier commits as a key figure.
- Reduced planning cycle time for S&OP.

**Product Objectives (Prioritized):**
1. Enable suppliers to self-serve commit quantity entry by supplier number, product number, and year.
2. Expose supplier commit data to IBP as a structured, queryable data set.
3. Display IBP supply planning data alongside supplier commits for planners.

## Requirements

### Must-Have Requirements

**R1: Supplier Commit Entry Form**
- **User Story:** As a supplier, I need to select my supplier number, a product number, and a year, then enter a committed quantity so that the planning team can plan supply accordingly.
- **Acceptance Criteria:**
  - Given I open the portal, when I select supplier number, product number, and year and submit a quantity, then the commit record is saved.
  - The form enforces a single annual bucket (one quantity value per supplier + product + year combination).
- **Priority Rank:** 1

**R2: Supplier Commit Data Storage**
- **User Story:** As a planner, I need all supplier commit entries stored in a reliable data store so that I can fetch and review them at any time.
- **Acceptance Criteria:**
  - CAP OData service persists commit records with supplier number, product number, year, and committed quantity.
  - Duplicate entries (same supplier + product + year) update the existing record.
- **Priority Rank:** 2

**R3: IBP Data Fetch & Display**
- **User Story:** As a planner, I need to view supplier commit quantities alongside IBP planning data so that I can assess supply coverage against the demand plan.
- **Acceptance Criteria:**
  - The planner view fetches and displays supplier commit quantities filtered by supplier number, product number, and year.
  - Data refreshes on demand.
- **Priority Rank:** 3

**R4: Supplier & Product Selector**
- **User Story:** As a supplier or planner, I need dropdowns or input fields to select supplier number, product number, and year so that I can quickly navigate to the right data.
- **Acceptance Criteria:**
  - The portal UI provides input/select controls for supplier number, product number, and year (current year ± 1 as default options).
- **Priority Rank:** 4

## Solution Architecture

**Architecture Overview:**
A CAP Node.js backend with a React/SAP UI5 Web Components frontend, deployed on SAP BTP Cloud Foundry. The CAP service acts as the data hub — storing supplier commits and serving them to both the supplier portal and the IBP integration endpoint.

**Key Components:**

- **Supplier Portal (React + SAP UI5 Web Components):** Simple single-page form for commit quantity entry and a planner read view.
- **CAP OData Service (Node.js):** Manages commit records — create, update, and query by supplier/product/year.
- **SAP HANA Cloud (persistence):** Stores commit records.
- **IBP Integration Endpoint:** A CAP OData or REST endpoint that IBP (or a middleware) can call to fetch supplier commit quantities as a key figure feed.

**Integration Points:**

- **SAP IBP:** Reads committed quantities from the CAP service endpoint to populate a supply planning key figure.
- **S/4HANA Supplier Confirmation API** (`sap.s4:apiResource:CE_SUPPLIERCONFIRMATION_0001:v1`): Optional enrichment — cross-reference commits with existing S/4HANA supplier confirmation records.

### Automation & Agent Behaviour

**Automation Level:** Rule-based

**Actions the system performs without human approval:**
- Saving and overwriting supplier commit quantity entries.
- Serving commit data to IBP fetch requests.

**Actions that require human review or approval:**
- None — data entry and retrieval are self-service.

### Configuration & Data

**Organisational & Master Data:**
- Supplier numbers and product numbers are entered by suppliers as free text or selected from a pre-loaded reference list.
- Year range: current year and the following year are the default options.
