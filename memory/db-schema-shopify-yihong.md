---
name: db-schema-shopify-yihong
description: Full schema reference for the shopify and yihong PostgreSQL schemas in the pacificnook Neon database
type: reference
---

# Database Schema: shopify & yihong

**Database**: `pacificnook` (Neon PostgreSQL)
**Connection**: `psql "postgresql://openclaw@ep-twilight-surf-af1go1x2-pooler.c-2.us-west-2.aws.neon.tech/pacificnook?sslmode=require"`
**Notes**: No FK constraints at DB level — relationships are enforced at application level. Uses camelCase column names in `shopify` schema, snake_case in `yihong` schema.

---

## Schema: `shopify`

### `shopify."Order"`
Core Shopify order record synced from Shopify API.

| Column | Type | Nullable | Default |
|---|---|---|---|
| id | text | NO | |
| name | text | NO | | ← order display name e.g. #1234
| orderNumber | text | NO | |
| createdAt | timestamp | NO | |
| updatedAt | timestamp | NO | |
| processedAt | timestamp | YES | |
| cancelledAt | timestamp | YES | |
| cancelReason | text | YES | |
| customerId | text | YES | | ← FK → Customer.id
| email | text | YES | |
| phone | text | YES | |
| currency | text | NO | 'USD' |
| financialStatus | text | NO | |
| fulfillmentStatus | text | YES | |
| subtotalPrice | float8 | NO | |
| totalPrice | float8 | NO | |
| totalDiscounts | float8 | NO | 0 |
| totalShipping | float8 | NO | 0 |
| totalTax | float8 | NO | |
| totalOutstanding | float8 | YES | |
| totalReceived | float8 | YES | |
| totalRefunded | float8 | YES | |
| lineItems | jsonb | NO | | ← array of line items
| billingAddress | jsonb | YES | |
| shippingAddress | jsonb | YES | |
| shippingLines | jsonb | YES | |
| discountCodes | jsonb | YES | |
| tags | jsonb | YES | |
| note | text | YES | |
| events | jsonb | YES | |
| shopifyData | jsonb | YES | | ← raw Shopify API payload

### `shopify."Customer"`
Shopify customer records.

| Column | Type | Nullable | Default |
|---|---|---|---|
| id | text | NO | |
| email | text | YES | |
| phone | text | YES | |
| firstName | text | YES | |
| lastName | text | YES | |
| state | text | YES | |
| verifiedEmail | boolean | NO | false |
| acceptsMarketing | boolean | NO | false |
| ordersCount | integer | NO | 0 |
| totalSpent | float8 | NO | 0 |
| defaultAddress | jsonb | YES | |
| addresses | jsonb | YES | |
| tags | jsonb | YES | |
| note | text | YES | |
| shopifyData | jsonb | YES | |
| createdAt | timestamp | NO | |
| updatedAt | timestamp | NO | |

### `shopify."Product"`
Shopify product catalog.

| Column | Type | Nullable | Default |
|---|---|---|---|
| id | text | NO | |
| shopifyId | text | YES | |
| handle | text | NO | |
| title | text | NO | |
| status | text | NO | 'DRAFT' |
| publishedAt | timestamp | YES | |
| createdAt | timestamp | NO | CURRENT_TIMESTAMP |
| updatedAt | timestamp | NO | |
| categoryId | text | YES | | ← FK → ShopifyCategory.id
| categoryName | text | YES | |
| description | text | YES | |
| descriptionHtml | text | YES | |
| productType | text | YES | |
| vendor | text | YES | |
| lingxingSku | text | YES | | ← links to Lingxing ERP SKU
| totalInventory | integer | NO | 0 |
| priceRangeMin | float8 | YES | |
| priceRangeMax | float8 | YES | |
| compareAtPriceMin | float8 | YES | |
| compareAtPriceMax | float8 | YES | |
| hasOnlyDefaultVariant | boolean | NO | false |
| hasOutOfStockVariants | boolean | NO | false |
| isGiftCard | boolean | NO | false |
| tracksInventory | boolean | NO | true |
| featuredImageId | text | YES | |
| legacyResourceId | text | YES | |
| metafields | jsonb | YES | |
| customFields | jsonb | YES | |
| textEmbedding | vector | YES | | ← pgvector embedding
| seoTitle | text | YES | |
| seoDescription | text | YES | |

### `shopify."ProductVariant"`
Variants of a product (size, colour, etc.)

| Column | Type | Nullable | Default |
|---|---|---|---|
| id | text | NO | |
| productId | text | NO | | ← FK → Product.id
| sku | text | NO | |
| shopifyVariantId | text | YES | |
| title | text | YES | |
| displayName | text | YES | |
| position | integer | NO | 1 |
| price | float8 | NO | |
| compareAtPrice | float8 | YES | |
| inventoryQuantity | integer | NO | 0 |
| inventoryPolicy | text | NO | 'CONTINUE' |
| availableForSale | boolean | NO | true |
| barcode | text | YES | |
| weight | float8 | YES | |
| weightUnit | text | NO | 'KILOGRAMS' |
| requiresShipping | boolean | NO | true |
| taxable | boolean | NO | true |
| option1 | text | YES | |
| option2 | text | YES | |
| option3 | text | YES | |
| selectedOptions | jsonb | YES | |
| imageId | text | YES | |
| inventoryItemId | text | YES | |
| legacyResourceId | text | YES | |
| fulfillmentService | text | YES | |
| taxCode | text | YES | |
| createdAt | timestamp | NO | CURRENT_TIMESTAMP |
| updatedAt | timestamp | NO | |

### `shopify."ShopifyCategory"`
Hierarchical product category tree.

| Column | Type | Nullable |
|---|---|---|
| id | text | NO |
| name | text | NO |
| parentId | text | YES | ← self-ref parent
| level | integer | NO |
| fullPath | text | YES |
| attributes | jsonb | YES |
| createdAt | timestamp | NO |
| updatedAt | timestamp | NO |

### `shopify."ProductImageEmbedding"`
pgvector image embeddings for visual search.

| Column | Type | Nullable |
|---|---|---|
| id | text | NO |
| productId | text | NO | ← FK → Product.id
| imageUrl | text | NO |
| imageHash | text | NO |
| embedding | vector | NO |
| updatedAt | timestamp | NO |

### `shopify."AuditLog"`
Change audit trail for all entities.

| Column | Type | Nullable | Default |
|---|---|---|---|
| id | text | NO | |
| action | text | NO | |
| entityType | text | NO | |
| entityId | text | NO | |
| changes | jsonb | YES | |
| metadata | jsonb | YES | |
| source | text | NO | 'gpt' |
| createdAt | timestamp | NO | CURRENT_TIMESTAMP |

### `shopify."SyncStatus"`
Tracks sync state for each data source.

| Column | Type | Nullable | Default |
|---|---|---|---|
| id | text | NO | |
| source | text | NO | |
| lastSyncAt | timestamp | YES | |
| status | text | NO | 'pending' |
| recordsCount | integer | NO | 0 |
| errorMessage | text | YES | |
| metadata | jsonb | YES | |
| updatedAt | timestamp | NO | |

### `shopify.order_tracking`
Bridges Shopify orders to physical delivery/logistics tracking. Links to `yihong` via `yihong_reference_no`.

| Column | Type | Nullable | Default |
|---|---|---|---|
| id | text | NO | |
| shopify_order_id | text | NO | | ← FK → Order.id
| shopify_order_name | text | NO | |
| status | enum | NO | 'ORDERED' | ← OrderTrackingStatus
| delivery_type | enum | NO | 'STANDARD' | ← DeliveryType
| status_updated_at | timestamp | NO | |
| shipping_address | text | YES | |
| shipping_city | text | YES | |
| shipping_state | text | YES | |
| shipping_zip | text | YES | |
| latitude | float8 | YES | |
| longitude | float8 | YES | |
| yihong_reference_no | text | YES | | ← FK → yihong.orders.reference_no
| ship_name | text | YES | |
| vessel_voyage | text | YES | |
| eta | timestamp | YES | |
| current_location | text | YES | |
| arrived_at | timestamp | YES | |
| in_warehouse_at | timestamp | YES | |
| scheduled_delivery_date | timestamp | YES | |
| scheduled_time_slot | enum | YES | |
| delivered_at | timestamp | YES | |
| arrival_notified_at | timestamp | YES | |
| schedule_notified_at | timestamp | YES | |
| reminder_notified_at | timestamp | YES | |
| metafields_synced_at | timestamp | YES | |
| shipping_partner | text | YES | |
| shipping_partner_ref | text | YES | |
| created_at | timestamp | NO | CURRENT_TIMESTAMP |
| updated_at | timestamp | NO | |

### `shopify.scheduled_deliveries`
Delivery scheduling slots linked to order_tracking.

| Column | Type | Nullable | Default |
|---|---|---|---|
| id | text | NO | |
| order_tracking_id | text | NO | | ← FK → order_tracking.id
| scheduled_date | date | NO | |
| time_slot | enum | NO | |
| cluster_id | text | YES | |
| cluster_zip_prefix | text | YES | |
| is_confirmed | boolean | NO | false |
| confirmed_at | timestamp | YES | |
| crew_count | integer | NO | 1 |
| crew_notes | text | YES | |
| completed_at | timestamp | YES | |
| delivery_notes | text | YES | |
| customer_signature | text | YES | |
| created_at | timestamp | NO | CURRENT_TIMESTAMP |
| updated_at | timestamp | NO | |

### `shopify.customer_availability`
Customer-provided availability windows for delivery scheduling.

| Column | Type | Nullable | Default |
|---|---|---|---|
| id | text | NO | |
| order_tracking_id | text | NO | | ← FK → order_tracking.id
| available_date | date | NO | |
| time_slot | enum | NO | |
| priority | integer | NO | 1 |
| created_at | timestamp | NO | CURRENT_TIMESTAMP |

### Lookup/Join Tables
- **`product_collections`**: productId ↔ collectionId, collectionHandle, collectionTitle
- **`product_images`**: productId → image src, altText, position, width, height, gcsPath
- **`product_options`**: productId → option name, position, values (jsonb)
- **`product_tags`**: productId → tag (text)

---

## Schema: `yihong`

Yihong is the freight/logistics partner. This schema stores shipment orders and tracking data synced from Yihong's API.

### `yihong.orders`
Core freight order from Yihong. Linked to Shopify via `shopify.order_tracking.yihong_reference_no = reference_no`.

| Column | Type | Nullable | Default |
|---|---|---|---|
| id | text | NO | |
| reference_no | text | NO | | ← cross-schema join key
| shipping_method_no | text | YES | |
| tracking_number | text | YES | |
| shipping_method_code | text | NO | | ← FK → shipping_methods.code
| order_weight | numeric | YES | |
| order_pieces | integer | YES | 1 |
| cargo_type | text | YES | |
| mail_cargo_type | text | YES | |
| currency_code | text | YES | 'USD' |
| order_info | text | YES | |
| order_status | text | NO | 'P' |
| track_status | text | YES | |
| track_status_name | text | YES | |
| consignee_name | text | NO | |
| consignee_company | text | YES | |
| consignee_country_code | text | NO | |
| consignee_province | text | YES | |
| consignee_city | text | YES | |
| consignee_district | text | YES | |
| consignee_street | text | YES | |
| consignee_postcode | text | YES | |
| consignee_telephone | text | YES | |
| consignee_mobile | text | YES | |
| consignee_email | text | YES | |
| shipper_name | text | NO | |
| shipper_company | text | YES | |
| shipper_country_code | text | NO | 'CN' |
| shipper_province | text | YES | |
| shipper_city | text | YES | |
| shipper_district | text | YES | |
| shipper_street | text | YES | |
| shipper_postcode | text | YES | |
| shipper_telephone | text | YES | |
| shipper_mobile | text | YES | |
| gross_weight | numeric | YES | |
| volume_weight | numeric | YES | |
| charge_weight | numeric | YES | |
| total_fee | numeric | YES | |
| created_at | timestamp | NO | CURRENT_TIMESTAMP |
| updated_at | timestamp | NO | |
| last_sync_at | timestamp | NO | CURRENT_TIMESTAMP |
| raw_data | jsonb | YES | | ← raw Yihong API payload

### `yihong.shipments`
Sea/air vessel shipment info for an order.

| Column | Type | Nullable |
|---|---|---|
| id | text | NO |
| order_id | text | NO | ← FK → orders.id
| vessel_name | text | YES |
| voyage_no | text | YES |
| vessel_imo | text | YES |
| vessel_mmsi | text | YES |
| departure_date | timestamp | YES |
| estimated_arrival | timestamp | YES |
| actual_arrival | timestamp | YES |
| departure_port | text | YES |
| arrival_port | text | YES |
| container_no | text | YES |
| extracted_from | text | YES |
| extracted_at | timestamp | YES |
| created_at | timestamp | NO |
| updated_at | timestamp | NO |

### `yihong.tracking_details`
Granular tracking events for an order.

| Column | Type | Nullable |
|---|---|---|
| id | text | NO |
| order_id | text | NO | ← FK → orders.id
| track_occur_date | timestamp | NO |
| track_location | text | YES |
| track_description | text | NO |
| created_at | timestamp | NO |

### `yihong.order_fees`
Fee line items for an order.

| Column | Type | Nullable | Default |
|---|---|---|---|
| id | text | NO | |
| order_id | text | NO | | ← FK → orders.id
| fee_kind_code | text | NO | |
| fee_kind_name | text | NO | |
| currency_code | text | NO | 'RMB' |
| currency_name | text | YES | |
| currency_rate | numeric | NO | 1.0 |
| currency_amount | numeric | NO | |
| amount | numeric | NO | | ← RMB amount
| create_date | timestamp | YES | |
| occur_date | timestamp | YES | |
| bill_date | timestamp | YES | |
| note | text | YES | |

### `yihong.order_volumes`
Box dimension records for an order.

| Column | Type | Nullable |
|---|---|---|
| id | text | NO |
| order_id | text | NO | ← FK → orders.id
| box_number | text | YES |
| length | numeric | NO |
| width | numeric | NO |
| height | numeric | NO |
| gross_weight | numeric | YES |

### `yihong.invoice_items`
Customs invoice line items.

| Column | Type | Nullable | Default |
|---|---|---|---|
| id | text | NO | |
| order_id | text | NO | | ← FK → orders.id
| invoice_en_name | text | NO | |
| invoice_cn_name | text | YES | |
| invoice_quantity | integer | NO | 1 |
| unit_code | text | YES | 'PCE' |
| invoice_unit_charge | numeric | NO | |
| net_weight | numeric | YES | |
| invoice_note | text | YES | |
| hs_code | text | YES | |
| origin_country | text | YES | |

### Reference/Lookup Tables
- **`yihong.shipping_methods`**: code, cn_name, en_name, note
- **`yihong.countries`**: code, cn_name, en_name, note
- **`yihong.extra_services`**: code, cn_name, en_name, note
- **`yihong.platforms`**: platform_id, platform_code, platform_name

---

## Cross-Schema Relationship

```
shopify."Order".id
  └── shopify.order_tracking.shopify_order_id
        └── shopify.order_tracking.yihong_reference_no
              └── yihong.orders.reference_no
                    ├── yihong.shipments.order_id
                    ├── yihong.tracking_details.order_id
                    ├── yihong.order_fees.order_id
                    ├── yihong.order_volumes.order_id
                    └── yihong.invoice_items.order_id
```

**Join to get Shopify order with Yihong freight details:**
```sql
SELECT o.name, o."orderNumber", ot.status, ot.eta, ot.delivered_at,
       yo.reference_no, yo.track_status_name, yo.total_fee
FROM shopify."Order" o
JOIN shopify.order_tracking ot ON ot.shopify_order_id = o.id
JOIN yihong.orders yo ON yo.reference_no = ot.yihong_reference_no
WHERE o.id = '<order_id>';
```
