# NetSuite MapReduce Script for RA Fulfillment Events

This document outlines the implementation requirements for the NetSuite MapReduce script that will process Return Authorization (RA) fulfillment to Scale.

## 1. Saved Search Usage

Use saved search `customsearch_dwr_ra_for_fulfillment` which filters for:
- Status is any of: Return Authorization:Pending Receipt, Partially Received, Pending Refund/Partially Received
- Sent To WMS is false
- Item Type is any of: Inventory Item, Kit/Package
- Item Display Name is not empty

## 2. Field Mapping Requirements

| NetSuite Field | Source | Notes |
|----------------|--------|-------|
| Transaction ID | RA transaction ID (tranId) | For reference/tracking |
| RA Internal ID | Internal ID | For reference |
| Item ID | Item ID from search result | |
| Item Internal ID | Item internal ID | For reference |
| Display Name | Display name from search | |
| Quantity | Transaction quantity | Raw quantity |
| Expected Receipt Date | Expected receipt date or Today+30 if null | |
| Line Number | Line number from search result | For reference |
| Reason Code | custcol_dwr_reason_code | Required for RA processing |
| Line Unique Key | custcol_dwr_line_unique_key | For line identification |
| Ship From Country | Ship country code | Address information |
| Email Address | Email | Contact information |
| Bill To State | Bill state | Billing address |
| Ship Method | Ship method name | Shipping information |
| Item Vendor ID | Vendor internal ID | Item details |
| Item Net Price | Average cost | Item pricing |
| Other standard fields | As per search results | |

## 3. Processing Requirements

1. Execute saved search `customsearch_dwr_ra_for_fulfillment`
2. For each result:
   - Extract required fields
   - Create event payload
   - Send to midtier integration endpoint

## 4. Post-Processing Flag Update

After successful processing by midtier:
- Set custom field `custcol_dwr_sent_to_wms` to `true` for processed line items
- Use line number for accurate line item identification

## 5. Error Handling Requirements

- Log detailed error information for processing failures
- Include RA number, line number, and item ID in error logs
- Continue processing other records when an error occurs
- Implement appropriate retry mechanism

## 6. Complete Processing Flow

1. Execute saved search
2. Transform results to event payload
3. Send to midtier integration
4. Upon successful response, flag record as processed
