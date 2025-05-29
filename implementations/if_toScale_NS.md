# NetSuite MapReduce Script for Item Fulfillment Receiving Events

This document outlines the implementation requirements for the NetSuite MapReduce script that will process Item Fulfillments from Transfer Orders for receiving to Scale.

## 1. Saved Search Usage

Use existing saved search `customsearch_hmk_st_if_to_dc` for Item Fulfillments which filters for:
- Transaction Type: Item Fulfillment
- Created From: Transfer Order
- Sent To WMS is false
- Item Type: Inventory Item
- Status: appropriate fulfillment statuses

## 2. Field Mapping Requirements

| NetSuite Field | Source | Notes |
|----------------|--------|-------|
| Item Fulfillment Internal ID | Internal ID from search result | For reference/tracking |
| Item Fulfillment Document Number | Transaction ID (tranId) | Document reference |
| Transfer Order Document Number | createdFromJoin.tranId | Source transfer order |
| Transfer Order Internal ID | createdFromJoin internal ID | Source TO reference |
| Transfer Location Internal ID | transferLocation internal ID | Raw location ID |
| From Location Internal ID | location internal ID | Raw location ID |
| Item ID | itemJoin.itemId | Item identifier |
| Line ID | custcol_hm_linenumberstored | Line reference |
| Quantity | basic.quantity | Fulfilled quantity |
| Custom Tracking Number | custcol_dwr_tracking_number | Tracking information |
| Other standard fields | As per search results | |

## 3. Processing Requirements

1. Execute saved search for Item Fulfillments
2. For each result:
   - Extract required fields
   - Create event payload
   - Send to midtier integration endpoint

## 4. Post-Processing Flag Update

After successful processing by midtier:
- Set custom field `custcol_dwr_sent_to_wms` to `true` for processed line items
- Use fulfillment line item number + 1 for line item identification

## 5. Error Handling Requirements

- Log detailed error information for processing failures
- Include Item Fulfillment number, Transfer Order number, and item ID in error logs
- Continue processing other records when an error occurs
- Implement appropriate retry mechanism

## 6. Complete Processing Flow

1. Execute saved search
2. Transform results to event payload
3. Send to midtier integration
4. Upon successful response, flag record as processed
