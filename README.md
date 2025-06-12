# 📡 Communication Channels

The KDS (Kitchen Display System) supports two primary communication channels:

- **Socket channel (Recommended)**
- **REST API channel (Deprecated)**

A user must log in through either the socket or REST channel, but **not both**.

The POS server may enforce its own login rules (e.g., requiring a username or ignoring it).

For added security, a POS server can either:

- Accept login requests from KDS directly, or  
- Require multi-factor confirmation from the user through the server.

KDS will wait for a response from the POS server before logging in a user.

---

## 🔐 Login via Socket Channel

- All credentials are stored within the KDS.
- All subsequent communication must occur via the same socket connection.

### Required:

- `username`
- `socket server URL`

KDS acts as a client connecting to your POS socket server. Refer to your POS documentation for supported socket protocols.

---

## ⚠️ Login via REST API Channel (Deprecated)

> **Note:** REST API login is deprecated and disabled by default.

- User credentials are stored similarly.
- All communication must occur via the same REST channel.

### Required:

- `username`
- `REST API endpoint URL`

🛑 **If your POS system only supports REST, contact us to enable it.**

---

## 🔌 Socket Communication Overview

- Persistent, real-time connection.
- Every request is sent over the open socket.

The POS server must accept connections from KDS at its **root domain**:

```text
✅ http://172.22.34.80  
❌ http://172.22.34.80/example
```

---

## 🔑 KDS Login Procedure

### ▶️ Request Format

```json
{
  "type": "userKDSLogin",
  "username": "sailab",
  "url": "http://172.27.240.1:8088"
}
```

> Do not modify the `type` field. It identifies the login request.

---

### ❌ Rejected Response

```json
{
  "success": false,
  "message": "Access Rejected - EST has blocked your request!",
  "result": false
}
```

- Once rejected, the user will be blocked from retrying.
- POS should implement IP blocking for unauthorized users.

---

### ✅ Accepted Response

```json
{
  "success": true,
  "message": "Access granted - you can now manage Kitchen Orders!",
  "result": {
    "ip": "172.27.240.1",
    "name": "Sailab Rahi",
    "key": "KEY1749752934600",
    "serverUrl": "http://172.27.240.1:8088"
  },
  "type": "userKDSLogin"
}
```

KDS will log in the user upon receiving a `success: true` response.

---

## 📥 Receiving Orders

KDS expects the following format for incoming kitchen orders:

### ✅ Example

```json
{
  "success": true,
  "message": "Recent orders!",
  "result": [ /* array of order objects */ ],
  "type": "get_orders"
}
```

### Requirements:

- `type` must be `"get_orders"`
- `success` must be `true`
- `result` must be an array of orders

---

### Required Fields Per Order

| Field        | Description                               |
|--------------|-------------------------------------------|
| `Progress`   | Order status: Pending, Processing, Ready  |
| `OrderID`    | Unique order identifier                   |
| `OrderDate`  | Timestamp of order creation               |
| `orderOption`| Delivery method (e.g. Takeaway, Dine-in)  |
| `Items`      | List of products and/or deals             |

> **Note:** All `Items` must be in valid JSON format, even if embedded as strings.

---

## 🔄 POS → KDS: Order Status Update

When an order status changes in the POS, send the following payload to the KDS:

```json
{
  "success": true,
  "message": "Order status changed on POS!",
  "result": [ /* updated orders */ ],
  "type": "get_orders"
}
```

- `type`: Must be `"get_orders"` for compatibility  
- `result`: Array of updated order objects  
- `message`: Shown to the KDS user  

---

## 🔄 KDS → POS: Order Status Change

When an order status is updated from the KDS interface:

```json
{
  "type": "change_order_status",
  "username": "Sailab Rahi",
  "url": "http://172.27.240.1:8088",
  "data": {
    "OrderID": 1749731421671,
    "progress": "Processing"
  }
}
```

### Field Descriptions:

| Field     | Description                              |
|-----------|------------------------------------------|
| `type`    | Always `"change_order_status"`           |
| `username`| User performing the action               |
| `url`     | The POS socket endpoint                  |
| `data`    | Contains `OrderID` and new progress status|

> The POS should respond with a `get_orders` message to sync with the KDS.

---

## 🆘 Support

For further assistance or integration help, contact us at:  
🌐 [www.ebmbook.com](https://www.ebmbook.com)
