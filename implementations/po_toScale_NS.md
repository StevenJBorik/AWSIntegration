# NetSuite MapReduce Script for PO Fulfillment Events

This document outlines the implementation requirements for the NetSuite MapReduce script that will process Purchase Orders for fulfillment to Scale.

## 1. Saved Search Usage

Use existing saved search `customsearchdwr_pos_for_fulfillment` which filters for:
- Status is any of: Purchase Order:Pending Receipt, Partially Received, Pending Billing/Partially Received, Pending Bill
- Sent To WMS is false
- Main Line is false
- Purchase Order Type is not Dropship
- Item Type is Inventory Item
- Quantity is greater than 0
- Location is any of specified warehouses

## 2. Field Mapping Requirements

| NetSuite Field | Source | Notes |
|----------------|--------|-------|
| Transaction ID | From search result | For reference/tracking |
| Line ID | Line unique key | For line item identification |
| Item ID | Item ID from search | |
| Quantity | Quantity from line | |
| Location | Location from line | |
| Expected Receipt Date | Expected receipt date or Today+30 if null | |
| PO Internal ID | Internal ID | For reference |
| Other standard fields | As per search results | |

## 3. Processing Requirements

1. Execute saved search `customsearchdwr_pos_for_fulfillment`
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
- Include PO number, line number, and item ID in error logs
- Continue processing other records when an error occurs
- Implement appropriate retry mechanism

## 6. Complete Processing Flow

1. Execute saved search
2. Transform results to event payload
3. Send to midtier integration
4. Upon successful response, flag record as processed
