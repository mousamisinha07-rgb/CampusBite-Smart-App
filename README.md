# CampusBite — Production Deployment Package

This package is the single deployment candidate for CampusBite.

## Architecture

- Customer and staff clients use the same Node/Express API.
- PostgreSQL is the source of truth for customers, wallets, transactions, orders, menu availability, and staff status changes.
- Browser localStorage is used only for the login token/session and the selected-shop UI preference. Orders, wallet balances, expenditure, transactions, and auto-pay are NOT stored in localStorage.
- Customer clients poll the API every 3 seconds for account/order/menu updates.
- Staff clients poll the API every 3 seconds for shop orders and daily summary data.
- Staff status transitions are enforced server-side: Received -> Preparing -> Ready -> Completed.
- Staff access is restricted to the authenticated staff member's assigned shop.
- Daily revenue/order counts use the Asia/Kolkata calendar date and are recalculated on every staff refresh/poll, so they roll over automatically when the date changes.

## Render

The included `render.yaml` provisions:
- one Node web service
- one PostgreSQL database
- `DATABASE_URL` from the database connection string
- a generated `JWT_SECRET`
- `NODE_ENV=production`

Build command: `npm install`
Start command: `npm start`

## Important

Do not deploy the standalone HTML preview. Deploy the contents of this package as the Render/Git repository root.

The server initializes the database schema and seeds the demo customer/staff/menu records on startup. Existing database rows are preserved; menu prices are synchronized with the application seed values.

## Live verification after deployment

1. Open the live site on Device A and log in as a customer.
2. Place an order.
3. Open the live site on Device B and log in as the appropriate shop staff.
4. Confirm the order appears there.
5. Advance it through Preparing, Ready, and Completed.
6. Confirm Device A receives the updated status within the polling interval.
7. Confirm wallet balance, transaction history, and total expenditure persist after logout/login.
8. Enable auto-pay and verify the server-side refill when the threshold condition is met.
9. Confirm staff Today's Orders/Revenue are based on the current India date.


## Demo customer accounts

- Aarav Sharma — Register: `CB2026001` — Password: `Campus@123`
- Diya Nair — Register: `CB2026002` — Password: `Campus@456`

These are fictional demo accounts. Do not use real university credentials in the prototype.

## Customer pickup time

Checkout supports ASAP or scheduled pickup from 3:00 PM through 9:00 PM in 30-minute increments. The server validates the selected pickup slot.

## Smart menu labels

Menu cards show indicative tags such as Vegetarian, Non-veg, Contains gluten, Contains dairy, Spicy, Gluten-free*, Popular, Low stock, and Best value. `*Gluten-free` is ingredient-based; customers should confirm cross-contact with the canteen.


## Database initialization
On startup, the server automatically applies `schema.sql` before running demo-account
and menu seeding. This allows a fresh PostgreSQL database (including a new Render or
Supabase database) to start without manually creating `menu_items` first.
