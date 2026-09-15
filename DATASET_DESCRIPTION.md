# Order Provenance Dataset: Why Each Purchase Order Was Placed in Multi-Supplier Stockrooms

## Overview

This is an original, fully synthetic dataset of 1,500 stockrooms, each run by its own buyer. Every stockroom holds 6 to 12 items bought from 2 to 4 suppliers over 700 days, with an order book, periodic stock counts and weekly sales. Every order carries a hidden cause: the item came up on its scheduled review, the item's own stock position reached its reorder point, or the item was swept into a purchase order that had been opened for another item of the same supplier. The dataset contains no real operational records, no customer data and no third-party material; every row was produced by the generation procedure described below.

The labels are the provenance of each order. When an order was swept in, the label names the specific earlier order whose purchase order it joined, so the labels form a provenance graph over the order book rather than a class per item.

## Release At A Glance

- Raw files: 11
- Stockrooms (cases): 1,500, each run by its own buyer, so every stockroom is an independent unit
- Items: 13,303, 6 to 12 per stockroom
- Suppliers: 2 to 4 per stockroom, with hidden lead times of 2 to 10 days, 0 to 3 days of jitter and a hidden consolidation window of 0 to 3 days
- Orders: 671,435, about 455 per stockroom
- Stock counts: 154,277, one per item every 45, 60 or 90 days, recorded with 15 percent multiplicative error
- Weekly sales records: 1,343,603, 101 weeks per item
- Observation window: days 0 to 699
- Provenance mix: 32.1 percent periodic_review, 37.9 percent reorder_point, 29.9 percent swept into another order
- Review cycles: 7, 14, 21 or 28 days, on about half the items
- Prepared split: 1,200 training stockrooms, 300 test stockrooms (every fifth stockroom in hashed-id order)
- Data origin: creator-generated synthetic data

## Raw File Structure

The uploaded ZIP is flat and contains exactly these eleven files at its root:

- `stockrooms.csv`: one record per stockroom: `case_id`, `n_items`, `n_suppliers`, `n_orders`.
- `items.csv`: one record per item: `case_id`, `item_id`, `supplier_id`.
- `orders.csv`: one record per order: `case_id`, `order_id`, `day`, `minute`, `item_id`, `quantity`.
- `stock_counts.csv`: one record per stock count: `case_id`, `item_id`, `day`, `count`.
- `weekly_sales.csv`: one record per item-week: `case_id`, `item_id`, `week`, `units`.
- `provenance.csv`: one creator-side record per order: `case_id`, `order_id`, `provenance`; used by `prepare.py` and never copied into public prepared data in full.
- `source_metadata.json`: provenance, scale, menus, generation policy and licence metadata.
- `LICENSE`: CC BY 4.0 notice and licence URL.
- `ATTRIBUTION.txt`: attribution text.
- `DATASET_CARD.md`: short scope and safety summary.
- `DATASET_DESCRIPTION.md`: this document.

## How The Data Was Generated

Every draw and every identifier comes from HMAC-SHA256 keyed to a withheld 256-bit secret; the generator code and the secret are not released. Each stockroom is drawn independently.

1. **Setup.** Draw 2 to 4 suppliers, each with a lead time of 2 to 10 days, jitter of 0 to 3 days and a consolidation window of 0 to 3 days. Draw 6 to 12 items, each assigned to a supplier, with a demand rate between 2 and 40 units a day, an annual seasonal swing and an occasional burst. About half the items get a review cycle of 7, 14, 21 or 28 days with an offset and a review level of 0.7, 0.85 or 1.0 times the order-up-to level. Every item gets a reorder point, an order-up-to level and a case pack; about 60 percent also get a can-order level between the reorder point and the order-up-to level.
2. **Each day.** Receive arrivals, draw Poisson demand and sell what is on hand. Then compute the stock position, which is on hand plus what is on order.
3. **Own-triggered orders.** An item on a review cycle whose review day has come and whose position is below its review level is ordered as `periodic_review`; the review is run 1 to 3 days late with a probability of 0.2, 0.35 or 0.5. Otherwise an item whose position has reached its reorder point, compared with a 15 percent buyer tolerance, is ordered as `reorder_point`, held over to the next day with a probability of 0, 0.2 or 0.4.
4. **Purchase orders and sweeping.** The first own-triggered order for a supplier with no open purchase order opens one, which stays open through that day plus the supplier's window. While a purchase order is open, items of that supplier whose position is at or below their can-order level, again with a 15 percent tolerance, are swept into it and labelled `pulled_by:<the order that opened it>`; 0, 10 or 20 percent of eligible items are missed. Later own-triggered orders for that supplier join the open purchase order and keep their own cause.
5. **Keying and quantities.** The day's orders are keyed in one session starting between 08:00 and 12:00, 1 to 12 minutes apart, with swept-in orders keyed at a random later point of the session. The quantity is the order-up-to level minus the position, jittered by 5 to 15 percent and rounded up to the case pack.
6. **Observation.** Publish only the order book, the stock counts (true on hand times a uniform factor between 0.85 and 1.15, every 45, 60 or 90 days per item) and weekly sales totals. All settings, thresholds, cycles, windows and lead times stay hidden.

## Intended Use And Limitations

- Intended use: research and benchmarking of event-level cause attribution and structure recovery from sparse, noisy operational logs.
- Out of scope: any claim about real purchasing operations, suppliers or products. The stockrooms, items and suppliers do not exist.
- No personal data, no real company data and no third-party material is included.

## Licence

Creative Commons Attribution 4.0 International (CC BY 4.0), https://creativecommons.org/licenses/by/4.0/
