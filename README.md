# Online Food Ordering Platform

## Architecture

Customer → Nginx → Order API → PostgreSQL

## Services

- nginx — Reverse proxy
- order-api — Flask Order API
- db — PostgreSQL database

## API Endpoints

### Health Check

GET /health

### Create Order

POST /orders

Example request:

```json
{
  "customer_name": "Veerendra",
  "food_item": "Pizza",
  "quantity": 2
}