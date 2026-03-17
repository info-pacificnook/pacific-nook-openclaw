# Long-Term Memory

## Database Schema Reference

See `memory/db-schema-shopify-yihong.md` for full schema of the `pacificnook` Neon PostgreSQL database.

**Quick facts:**
- **Connection**: `psql "postgresql://openclaw@ep-twilight-surf-af1go1x2-pooler.c-2.us-west-2.aws.neon.tech/pacificnook?sslmode=require"` (password via `~/.pgpass`, no env var needed)
- **Schemas**: `shopify` (15 tables, camelCase columns) and `yihong` (10 tables, snake_case columns)
- **shopify** — Shopify orders, customers, products, variants, categories, delivery tracking, scheduling
- **yihong** — Freight/logistics: orders, shipments, tracking events, fees, volumes, invoice items
- **Cross-schema join**: `shopify.order_tracking.yihong_reference_no = yihong.orders.reference_no`
- No DB-level FK constraints; relationships enforced in application code
- `shopify."Order"`, `shopify."Customer"`, `shopify."Product"`, `shopify."ProductVariant"` use quoted PascalCase table names

## Critical Rules

### Never ask teammates/users to run terminal commands
**Learned: 2026-03-17**

The agent was asking teammates in group chat channels (WeCom) to run `gog` and other commands in the terminal. This is wrong.

- You have the `exec` tool. Use it to run commands yourself.
- `gog` is installed at `~/.npm-global/bin/gog` — run it directly via `exec`.
- Never send a message to any channel telling a human to run something in their terminal.
- If a tool or binary is missing, either install it yourself or report the problem to the owner (not teammates).
