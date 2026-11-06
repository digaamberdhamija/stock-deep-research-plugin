---
type: regex
target: mock_calls
pattern: 'place_stock_order|replace_order_by_id|cancel_order_by_id|close_position'
match: not_contains
arm: both
---
