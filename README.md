# Fynd Commerce MCP

A Model Context Protocol (MCP) server for the Fynd Commerce platform, enabling AI assistants to browse catalogs, authenticate users, manage carts, and place COD orders through natural conversation.

---

## Tools (20)

### Catalog (4 tools)

| Tool               | Description                                                                         |
| ------------------ | ----------------------------------------------------------------------------------- |
| `list_products`    | Search and list products with filters (query, price range, sort, stock, pagination) |
| `get_product`      | Get detailed product info by slug — prices, media, available sizes                  |
| `list_collections` | Browse product collections with pagination                                          |
| `get_collection`   | Get collection details and its products                                             |

**Sort options:** latest, price_asc, price_dsc, popularity, discount_dsc, rating_dsc

### Authentication (4 tools)

| Tool                 | Description                                                       |
| -------------------- | ----------------------------------------------------------------- |
| `send_login_otp`     | Send OTP to mobile number (rate limited: 3 per mobile per 15 min) |
| `verify_login_otp`   | Verify OTP and establish authenticated session                    |
| `get_session_status` | Check if the current user is logged in                            |
| `logout`             | Log out and clear session                                         |

### Cart (2 tools)

| Tool          | Description                                                       |
| ------------- | ----------------------------------------------------------------- |
| `add_to_cart` | Add product to cart by slug (supports size selection and quantity) |
| `get_cart`    | View cart contents, items, and price breakup                      |

### Coupons (3 tools)

| Tool            | Description                                |
| --------------- | ------------------------------------------ |
| `list_coupons`  | List available coupons for the current cart |
| `apply_coupon`  | Apply a coupon code to the cart             |
| `remove_coupon` | Remove applied coupon from the cart         |

### Address & Checkout (3 tools)

| Tool             | Description                                                                             |
| ---------------- | --------------------------------------------------------------------------------------- |
| `list_addresses` | List saved delivery addresses                                                           |
| `add_address`    | Add a new delivery address (with 6-digit pincode validation)                            |
| `place_order`    | Preview order with cart summary, totals, and COD eligibility (returns confirmation code) |

### Order Management (4 tools)

| Tool               | Description                                                                    |
| ------------------ | ------------------------------------------------------------------------------ |
| `confirm_order`    | Confirm and place the COD order using the confirmation code from `place_order` |
| `get_order_status` | Get order details, status, shipments, and timestamps                           |
| `cancel_order`     | Cancel an order (requires explicit "yes" confirmation)                         |
| `list_orders`      | List all orders with pagination                                                |

---

## Key Behaviors

- **Two-step order confirmation:** `place_order` returns a preview with a confirmation code (valid 10 minutes). Use `confirm_order` to finalize. This ensures users see totals and COD eligibility before committing.
- **OTP rate limiting:** Max 3 OTP sends per mobile number per 15-minute window. You'll get a retry-after duration if rate limited.
- **Session persistence:** Sessions last 24 hours and persist across client reconnections.
- **Image support:** Product and collection images are automatically included in responses.

---

## Client Setup

> **Note:** `http://localhost:9090` is used as a placeholder throughout these examples. Replace it with the URL of any live Fynd Commerce storefront — for example, `https://superdry.in`, `https://nexus247.in`, or any other Fynd-powered website. Each storefront has its own application ID and token; generate the Bearer token accordingly.

### Generating the Bearer Token

The `Authorization` header uses a Base64-encoded combination of your application ID and token:

```
Bearer base64(APPLICATION_ID:APPLICATION_TOKEN)
```

**Examples:**

```bash
# Using command line
echo -n "YOUR_APP_ID:YOUR_APP_TOKEN" | base64

# Example
echo -n "YOUR_APP_ID:YOUR_APP_TOKEN" | base64
# Output: YOUR_BASE64_TOKEN
```

```javascript
// In Node.js
Buffer.from("YOUR_APP_ID:YOUR_APP_TOKEN").toString("base64");
```

```python
# In Python
import base64
base64.b64encode(b"YOUR_APP_ID:YOUR_APP_TOKEN").decode()
```

The resulting header value: `Authorization: Bearer YOUR_BASE64_TOKEN`

### Cursor

Add to `.cursor/mcp.json` in your project root (or `~/.cursor/mcp.json` globally):

```json
{
  "mcpServers": {
    "fynd-commerce": {
      "url": "http://localhost:9090/api/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_BASE64_TOKEN"
      }
    }
  }
}
```

### Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "fynd-commerce": {
      "url": "http://localhost:9090/api/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_BASE64_TOKEN"
      }
    }
  }
}
```

### Claude Code (CLI)

Run the following command to add the MCP server:

```bash
claude mcp add fynd-commerce \
  --transport http \
  --url http://localhost:9090/api/mcp \
  --header "Authorization: Bearer YOUR_BASE64_TOKEN"
```

Or add manually to `~/.claude/settings.json`:

```json
{
  "mcpServers": {
    "fynd-commerce": {
      "url": "http://localhost:9090/api/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_BASE64_TOKEN"
      }
    }
  }
}
```

### Antigravity

In your Antigravity workspace settings, add the MCP server:

```json
{
  "mcpServers": {
    "fynd-commerce": {
      "url": "http://localhost:9090/api/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_BASE64_TOKEN"
      }
    }
  }
}
```

### Custom / Any MCP-Compatible Client

The server uses **MCP Streamable HTTP** transport. Any MCP-compatible client can connect using:

- **URL:** `http://localhost:9090/api/mcp`
- **Method:** `POST` to initialize and send requests, `GET` for SSE stream, `DELETE` to close session
- **Required header:**
  - `Authorization: Bearer <base64(APP_ID:APP_TOKEN)>`
- **Optional headers:**
  - `mcp-session-id` — session ID returned from the initial POST

**Example using `curl`:**

```bash
# Initialize session
curl -X POST http://localhost:9090/api/mcp \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_BASE64_TOKEN" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"custom","version":"1.0.0"}}}'
```

### Deployed / Live Storefronts

Replace `http://localhost:9090` with any live Fynd Commerce storefront URL and use the corresponding application credentials:

```json
{
  "mcpServers": {
    "superdry": {
      "url": "https://superdry.in/api/mcp",
      "headers": {
        "Authorization": "Bearer <base64(SUPERDRY_APP_ID:SUPERDRY_APP_TOKEN)>"
      }
    },
    "nexus247": {
      "url": "https://nexus247.in/api/mcp",
      "headers": {
        "Authorization": "Bearer <base64(NEXUS247_APP_ID:NEXUS247_APP_TOKEN)>"
      }
    }
  }
}
```

You can configure multiple storefronts simultaneously — each with its own credentials and Bearer token.

> **Security note:** Avoid committing credentials in config files. Use environment variables or restrict file permissions with `chmod 600`.

---

## Tool Flow

1. `send_login_otp` → `verify_login_otp` (login)
2. `list_products` / `get_product` → `add_to_cart`
3. `list_coupons` → `apply_coupon` (optional discounts)
4. `add_address` → `place_order` → show total → user confirms → `confirm_order`
5. `get_order_status` / `list_orders` → `cancel_order` (if needed)

## Limitations

- COD only
- Single sales channel per deployment
