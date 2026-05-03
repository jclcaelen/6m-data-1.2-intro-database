```dbml
// Users
Table user {
  id int [pk, increment]
  name varchar
  email varchar [unique]
}

// Users' addresses
// Split separate because a user may have multiple addresses
// Not just relational, but user could ship/deliver to different addresses
Table user_address {
  id int [pk, increment]        // Surrogate ID
  user_id int [ref: > user.id]  // FK to retrace to user
  address varchar
}

// Restaurants
Table restaurant {
  id int [pk, increment]
  name varchar
  cuisine_type varchar
  rating decimal
}

// Menu
Table menu {
  id int [pk, increment]
  restaurant_id int [ref: > restaurant.id]  // FK: each restaurant has its own menu items
  name varchar
}

// Challenge 2: History Problem / Normalisation Trap
// GPT: this table refers to the pricing system
Table item_prices {
  id int [pk, increment]
  menu_id int [ref: > menu.id]    // to reference back to the menu where item is listed
  price decimal                   // prices may change from time to time
  valid_from datetime             // tracks price validity based on time period
  valid_to datetime
}

// Couriers
Table courier {
  id int [pk, increment]
  name varchar
  vehicle varchar
}

// Orders
Table order {
  id int [pk, increment]
  date datetime
  // FKs
  customer_id int [ref: > user.id]
  courier_id int [ref: > courier.id]
  restaurant_id int [ref: > restaurant.id]
}

// Challenge 1: Many -> Many.
// Restrictions:
  // item_id cannot exist in Orders Table (buy multiple items)
  // order_id cannot exist in Items table (item sold to many)
  // junction table between orders and menu
// GPT: this table refers to the financial record/ledger -> never depend on mutable upstream data
Table order_details {
  // order may contain many menu items
  id int [pk, increment]  // suggested by GPT
  // FKs
  order_id int [ref: > order.id]
  menu_id int [ref: > menu.id]
  // price_id int [ref: > item_prices.id] -> GPT: Because item_prices table can change, and we are allowing this to happen, which is dangerous.
  quantity int
  price_at_order decimal    // order remains immutable, records auditable and no dependency on other tables
}




// Notes
// Classical textbook junction Table: Composite PK, i.e., id 1 + id 2.
// Works for simple many to many, and minimal
// "order_details" has quantity and price_at_order, therefore transactional and not just a link.
// unable to have same item twice with different prices
// harder to update specific rows
// messy FK references
// ORMs struggle
// therefore a surrogate id as PK

// added constraints above
```
