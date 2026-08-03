# IBP Supplier Commit Quantity Portal

CAP application integrating a supplier-facing commit portal with SAP IBP planning data.

## Business challenge

Suppliers need a simple web portal to enter their committed supply quantities (by supplier number, product number, and year) in annual buckets. The planning team needs to fetch and display this supplier commit data within the IBP planning context to align demand and supply plans.

## Business Architecture (RBA)

### End-to-End Process

Source to Pay (generic)

### Process Hierarchy

```
Source to Pay (generic)
└── Manage Suppliers and Collaboration (generic)
    └── Manage suppliers and networked collaboration (BPS-332)
        └── Manage supplier collaboration
└── Plan to Optimize Fulfillment (generic)
    └── Align demand, supply and financial plans (BPS-327)
        └── Perform sales and operations planning
```

### Summary

The solution maps to two sub-processes: supplier collaboration for commit data entry (BPS-332) and demand/supply alignment in IBP (BPS-327), bridged by a custom CAP application acting as the commit data hub.

## Fit Gap Analysis

| Requirement (business) | Standard asset(s) found | API ORD ID | MCP Server ORD ID | MCP Server Version | Gap? | Notes / assumptions |
| ---------------------- | ----------------------- | ---------- | ----------------- | ------------------ | ---- | ------------------- |
| Supplier commit quantity entry (by supplier, product, year) | No standard SAP product covers a simple annual-bucket commit portal | — | — | — | Yes | Custom CAP portal required |
| Fetch & display supplier commits in IBP planning view | SAP IBP – S&OP Demand and Supply Balancing (SC2996) | — | — | — | Maybe | IBP key figure integration API exists but has no ORD ID; CAP acts as integration layer |
| Supplier confirmation data from S/4HANA | Supplier Confirmation OData API | `sap.s4:apiResource:CE_SUPPLIERCONFIRMATION_0001:v1` | — | — | No | Available; no MCP server found |

### Key findings
- No standard SAP product provides a dedicated annual-bucket supplier commit entry portal — custom CAP development is required.
- SAP IBP covers demand/supply balancing but data must be pushed via the Key Figure Integration API.
- The S/4HANA Supplier Confirmation OData API is available as a reference/enrichment source.
- The CAP backend will serve as the commit data store and integration hub between the portal and IBP.
- React frontend with SAP UI5 Web Components will provide the supplier-facing entry form.
- Authentication and supplier number scoping must be handled at the CAP layer.

## Recommendations

### IBP Supplier Commit CAP Application

#### Executive Summary

Custom CAP app with supplier portal and IBP data integration layer.

#### Recommended Solution

Build a CAP Node.js backend with a React/UI5 Web Components frontend. The supplier portal page allows suppliers to select their supplier number, product number, and year, then enter committed quantities in annual buckets. The CAP service persists commit records and exposes an endpoint for IBP to fetch supplier commit quantities. The IBP integration reads committed quantities and displays them as a supply planning key figure.

#### Recommended solution category

CAP App

#### Intent fit
88%
