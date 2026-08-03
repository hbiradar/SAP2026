# Specification: ibp-supplier-commit-cap

> **Guidelines**: Read [guidelines.md](../guidelines.md) and [guidelines-cap.md](../guidelines-cap.md) before executing ANY tasks below. Follow all constraints described there throughout execution.

## Basic Setup

- [ ] Read the project input (`product-requirements-document.md`, `intent.md`)
- [ ] Invoke the `cap-development` skill from `assets/ibp-supplier-commit-cap/` to set up the CAP project structure
- [ ] Install dependencies (`npm install`), validate the project starts (`cds watch`) and responds

## Data Model

- [ ] Define a `SupplierCommit` entity in `db/schema.cds` with fields:
  - `ID` (UUID, key)
  - `supplierNumber` (String, not null)
  - `productNumber` (String, not null)
  - `year` (Integer, not null)
  - `committedQuantity` (Decimal, not null)
  - `updatedAt` (Timestamp, managed)
- [ ] Add a unique constraint / validation so that the combination of `supplierNumber + productNumber + year` is treated as a single record (upsert logic in handler)

## CAP Service (Backend)

- [ ] Define `SupplierCommitService` in `srv/commit-service.cds` exposing:
  - `SupplierCommits` entity (CRUD)
  - Custom action or function: `getCommitsByFilter(supplierNumber, productNumber, year)` returning commit quantity and metadata
- [ ] Implement a custom handler in `srv/commit-service.js`:
  - On `CREATE` / `UPDATE`: check if a record with same `supplierNumber + productNumber + year` already exists; if yes, update `committedQuantity` instead of inserting a duplicate (upsert logic)
  - On `getCommitsByFilter`: query and return matching commit records filtered by the provided parameters
- [ ] Add an IBP integration endpoint — a plain REST action `GET /odata/v4/commit/IBPFeed?supplierNumber=&productNumber=&year=` that returns commit data in a format suitable for IBP key figure ingestion (array of `{ supplierNumber, productNumber, year, committedQuantity }`)
- [ ] Seed `db/data/SupplierCommits.csv` with at least 5 sample rows covering different suppliers, products, and years

## UI — Supplier Portal (React + SAP UI5 Web Components)

- [ ] Scaffold the React frontend inside `assets/ibp-supplier-commit-cap/app/supplier-portal/` using the `cap-development` skill frontend guidelines
- [ ] Implement a **Commit Entry Form** page:
  - Input fields: Supplier Number (text), Product Number (text), Year (select: current year and next year as default options)
  - Input field: Committed Quantity (number)
  - Submit button that calls the CAP OData `SupplierCommits` CREATE/UPSERT endpoint
  - Success/error message feedback after submission
- [ ] Implement a **Planner View** page (tab or route):
  - Filter bar: Supplier Number, Product Number, Year dropdowns/inputs
  - Table displaying: Supplier Number, Product Number, Year, Committed Quantity, Last Updated
  - "Refresh" button that re-fetches data from `getCommitsByFilter`
  - Table built with SAP UI5 Web Components (`ui5-table`, `ui5-table-column`, `ui5-table-row`)
- [ ] Navigation between Commit Entry and Planner View (tabs or side navigation using SAP UI5 Web Components)
- [ ] Use SAP UI5 Web Components for all UI elements (inputs, buttons, select, table, title, bar)

## Validation & Testing

- [ ] Run `cds compile srv/` to confirm CDS models compile without errors
- [ ] Write a unit test for the upsert handler logic: given an existing `supplierNumber + productNumber + year` record, a second commit with the same key must update `committedQuantity` rather than create a duplicate
- [ ] Write a test for `getCommitsByFilter`: given seeded data, querying by a specific supplier + product + year must return exactly one matching record
- [ ] Run `cds watch` and curl the OData service and IBP feed endpoint to confirm both respond correctly
- [ ] Verify the UI loads and the Commit Entry form submits without errors (manual smoke test via browser)
