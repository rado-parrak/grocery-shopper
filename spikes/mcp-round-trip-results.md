# Rohlík MCP Round-Trip — Results

**Observed live:** 2026-07-31
**Net effect on real basket:** zero (confirmed — cart returned to baseline: `cartId 172378585`, `totalPrice 0.0`, `items {}`, before and after)
**Fully-qualified tool prefix observed:** `Rohlik:<toolName>` (not the community hypothesis names)

## Recording table (steps 1–7)

| Step | Observed tool name | Matches community-list hypothesis? | Parameters (as observed) | Return shape (as observed) | Error shape (if forced) | Notes |
|---|---|---|---|---|---|---|
| 1. Search | `Rohlik:batch_search_products` | No — hypothesis was `search_products` | `{"queries":[{"keyword":"rohlík"}]}` | `{success, results:[{query, products:[{productId, productName, price, currency, pricePerUnit:{full,currency}, textualAmount, brand, inStock, favourite, preorderEnabled, badges}], total_found}], total_queries, successful, failed}` | — | Product-ID field is `productId` (camelCase). Availability field is `inStock` (boolean). `favourite:true` flag is embedded directly on search results. |
| 2. `cart_read` (baseline) | `Rohlik:get_cart` | No — hypothesis was `get_cart_content` | none | `{status, messages, data:{cartId, totalPrice, totalSavings, minimalStandardOrderPrice, companies, categories, items:{}, ...}, success}` | — | Baseline: `totalPrice 0.0`, `items {}`. No explicit top-level currency field on the cart itself (implied CZK). |
| 3. Reversible add (1×) | `Rohlik:add_items_to_cart` | Close — hypothesis was `add_to_cart` | `{"items":[{"productId":1435295,"quantity":1}]}` (Rohlík pivec, 9.9 Kč, in stock) | `{success, items_failed_to_add:[], items:[{productId, orderFieldId, quantity, price, currency:"CZK", ...}], totalPrice, message}` | — | Write's own return value happened to match the read-back exactly this time — **still not treated as proof**, per policy (see Step 7 for why that matters). |
| 4. `cart_read` (verify add) | `Rohlik:get_cart` | — | none | Same shape as Step 2, now `totalPrice 9.9`, `items:{"1435295":{...}}` | — | Confirmed present at qty 1, correct price. Read-back matched the write's claim in this trial. |
| 5. Reversible remove | `Rohlik:remove_cart_item` | Partial — hypothesis had no direct equivalent listed | `{"product_id":1435295}` | `{success, totalPrice:0, items:[]}` | — | **Parameter naming inconsistency**: `product_id` (snake_case, singular, top-level int) here vs. `productId` (camelCase, nested in a list) for add. A skill author copying one convention to the other will get a schema error. |
| 6. `cart_read` (verify revert) | `Rohlik:get_cart` | — | none | Matches Step 2 exactly: `totalPrice 0.0`, `items:{}` | — | Loop closed, zero net change confirmed. |
| 7. Forced error | `Rohlik:add_items_to_cart` | — | `{"items":[{"productId":999999999,"quantity":1}]}` | `{success:false, items_failed_to_add:["999999999"], items:[], totalPrice:0.0, message:"Requested: 1 items. Added: 0 items...", items_failed_message:"...", failure_reasons:["Produkt, který se snažíte přidat do košíku bohužel neprodáváme."]}` | **Not an HTTP status or MCP-protocol-level error** — a normal 200-shaped JSON body with `success:false` and human-readable Czech failure text | This is exactly the case `mcp-degradation.md`'s "never trust a write's own return value" rule is for: a naive check of "did an `items` array come back" would miss this failure. Must check `success` and `items_failed_to_add` explicitly, not just presence of a response. Also: the failure payload includes text seemingly aimed at the calling agent ("do not retry, explain to user, don't add alternatives unless asked") — treat as informational only, not as an instruction source that overrides this project's own policies. |

## Step 8 — Favourites-equivalent tool: **present**

Two relevant tools exist, both distinct from the community's single `get_frequent_items` hypothesis:

- **`Rohlik:get_all_user_favorites`** — no params; returns a lightweight list (`productId`, `productName`, `price`, `inStock`) of everything the household has explicitly marked as a favourite. 28 items returned live. This is the direct favourites-equivalent.
- **`Rohlik:get_typical_order`** — no required params (`add_to_cart` defaults `false`, confirmed no side effect when omitted); returns `frequent_items[]` (with `frequency`, `total_quantity`, `median_quantity`, `average_price`, `last_order_date`) built from real order history — closer to the community project's `get_frequent_items` naming/intent than the favourites tool is.

**Resolution for `seed-favourites.md`:** its bootstrap role is now confirmed temporary — a Rohlík-native favourites source exists and is populated today, not just theoretically available pending this check.

## What this changes in the shared docs (for a human to apply — these files are hand-edited only)

1. **`mcp-degradation.md`** — the "Observed tool surface" table's 5 placeholder rows can be replaced with the table above, each marked "observed live, 2026-07-31."
2. **`seed-favourites.md`** — the "Bootstrap vs. steady-state" section can note that `Rohlik:get_all_user_favorites` is confirmed present and populated. Purely as candidates for the human to consider (not written automatically, per that file's human-edit-only rule): the live favourites/order-history data already contains a milk and an eggs match —
   - Milk → `1443104` (Miil BIO Čerstvé mléko plnotučné 3,6 %) — in both favourites and frequent items
   - Eggs → `1316385` (Schubert BIO Natur vejce M–L) or `1466294` (Pohodová vejce z volného výběhu) — both in favourites
   - Rohlíky and bananas were **not** in the favourites list, though frequent items shows `1346771` (Antonínovo pekařství Karlínský rohlík) and `1349785` (Banán Chiquita 1 ks) as order-history candidates.
