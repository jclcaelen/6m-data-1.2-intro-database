```dbml
// =========================
// USERS
// =========================
Table user {
  id int [pk, increment]
  name varchar [not null]
  email varchar [not null, unique]
}

// A user can have multiple addresses (1-to-many)
Table user_address {
  id int [pk, increment]
  user_id int [not null, ref: > user.id]
  address varchar [not null]
}


// =========================
// RESTAURANTS & MENU
// =========================
Table restaurant {
  id int [pk, increment]
  name varchar [not null]
  cuisine_type varchar
  rating decimal(2,1)  // e.g. 4.5
}

// Each restaurant owns multiple menu items
Table menu {
  id int [pk, increment]
  restaurant_id int [not null, ref: > restaurant.id]
  name varchar [not null]
}


// =========================
// PRICING SYSTEM (HISTORICAL TABLE)
// =========================
// Tracks price changes over time (NOT used directly in orders)
// valid_to = NULL → currently active price
Table item_prices {
  id int [pk, increment]
  menu_id int [not null, ref: > menu.id]
  price decimal [not null]
  valid_from datetime [not null]
  valid_to datetime

  // Logical constraint (conceptual):
  // 1. (menu_id, valid_from) should be unique
  // 2. No overlapping validity periods per menu_id
}


// =========================
// COURIERS
// =========================
Table courier {
  id int [pk, increment]
  name varchar [not null]
  vehicle varchar
}


// =========================
// ORDERS
// =========================
// NOTE: renamed from "order" → "orders" (ORDER is SQL keyword)
Table orders {
  id int [pk, increment]
  date datetime [not null]

  customer_id int [not null, ref: > user.id]
  courier_id int [not null, ref: > courier.id]

  // Optional design choice:
  // Keeping restaurant_id improves query speed,
  // but must ensure consistency with order_details → menu → restaurant
  restaurant_id int [not null, ref: > restaurant.id]
}


// =========================
// ORDER DETAILS (TRANSACTIONAL TABLE)
// =========================
// NOT a pure junction table:
// Stores business data → quantity + price snapshot
// Represents financial record (immutable)
Table order_details {
  id int [pk, increment]

  order_id int [not null, ref: > orders.id]
  menu_id int [not null, ref: > menu.id]

  quantity int [not null]  // must be > 0 (logical constraint)

  // Snapshot to preserve historical accuracy
  // DO NOT depend on item_prices (mutable)
  price_at_order decimal [not null]

  // Optional (advanced):
  // Store item name snapshot in case menu changes
  item_name_at_order varchar
}


// =========================
// DESIGN NOTES
// =========================

// 1. Pure Junction vs Transactional Table:
// - Pure junction: only FKs, composite PK (e.g. student_course)
// - order_details is NOT pure (contains quantity + price)

// 2. Pricing Strategy:
// - item_prices = pricing system (can change over time)
// - order_details = financial ledger (must remain immutable)
// - Hence, store price_at_order instead of referencing price_id

// 3. Temporal Design:
// - item_prices tracks validity using (valid_from, valid_to)
// - valid_to = NULL → active price

// 4. Data Integrity:
// - NOT NULL used to prevent incomplete records
// - Surrogate PKs used for flexibility and ORM compatibility

// 5. Trade-off (orders.restaurant_id):
// - Kept for performance (fewer joins)
// - Must ensure consistency with menu.restaurant_id

```
