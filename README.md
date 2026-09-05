# Procurement MCP Server for n8n

An importable n8n workflow that exposes procurement tools through the Model
Context Protocol (MCP). It stores RFQs and supplier data in Google Sheets, then
ranks suppliers with a transparent weighted scoring model.

## Included tools

| Tool | Action |
| --- | --- |
| `create_rfq` | Append a request for quotation to the `RFQs` sheet |
| `list_rfqs` | Read all logged RFQs |
| `find_suppliers` | Find suppliers with an exact category match |
| `recommend_supplier` | Score candidates and return a recommendation |

Typical flow:

```text
create_rfq -> find_suppliers -> recommend_supplier
```

## Supplier scoring

| Dimension | Weight | Preferred value |
| --- | ---: | --- |
| Unit price | 40% | Lower |
| Lead time | 25% | Lower |
| On-time delivery | 15% | Higher |
| Rating | 20% | Higher |

The workflow also flags suppliers whose minimum order exceeds the requested
quantity and suppliers with an on-time rate below 90%.

## Requirements

- An n8n instance with MCP and Google Sheets tool nodes
- A Google Sheet with `Suppliers` and `RFQs` tabs
- A Google Sheets OAuth credential configured inside n8n
- An MCP-compatible client

## Sheet schema

`Suppliers`:

```text
supplier_id, name, category, country, unit_price, currency,
lead_time_days, min_order_qty, rating, on_time_rate, contact
```

`RFQs`:

```text
rfq_id, created_at, requester, item, category, quantity, unit,
needed_by, budget_max, notes, status
```

Category matching is exact, so keep casing and whitespace consistent.

## Setup

1. Create the two sheet tabs and header rows shown above.
2. Import [`procurement-mcp-workflow.json`](procurement-mcp-workflow.json) into n8n.
3. Reconnect every Google Sheets node to your own OAuth credential.
4. Replace `YOUR_GOOGLE_SHEET_ID` with your sheet.
5. Configure authentication on the MCP Server Trigger.
6. Publish the workflow and connect your MCP client to:

   ```text
   https://your-n8n-host/mcp/procurement
   ```

## Security

The template does not configure authentication on the MCP trigger. Add bearer
or header authentication before exposing the endpoint; otherwise anyone with
the URL could read supplier data or create RFQs.

Use n8n Credentials for OAuth tokens and secrets. Do not commit raw exports from
an active instance without removing credential IDs, sheet IDs, execution data,
and pinned data.

## Limitations

- RFQ IDs use timestamps to the second and can collide under simultaneous use.
- Categories are described with a hard-coded list that can drift from the sheet.
- Min-max normalization may exaggerate small differences within a narrow range.
- Quantity and unit are stored separately without cross-field validation.

This is a working prototype intended for learning and internal experimentation,
not an unattended production procurement system.
