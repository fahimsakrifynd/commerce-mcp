# Fynd Storefront Commerce MCP Server

AI assistants can seamlessly interact with Fynd Commerce storefronts through natural conversations using the **Model Context Protocol (MCP)**. This enables AI-powered commerce flows like product discovery, authentication, cart management, and COD order placement — all via structured MCP tools.

## Quick Start

### 1. Generate Your Bearer Token

Combine your Fynd application ID and token, then Base64-encode them:

```bash
echo -n "YOUR_APP_ID:YOUR_APP_TOKEN" | base64
```

<details>
<summary>Other languages</summary>

**Node.js:**

```js
Buffer.from("YOUR_APP_ID:YOUR_APP_TOKEN").toString("base64");
```

**Python:**

```python
import base64
base64.b64encode(b"YOUR_APP_ID:YOUR_APP_TOKEN").decode()
```

</details>

For more details, see the [authentication reference](https://docs.fynd.com/partners/commerce/sdk/latest/graphql/application/client-libraries#authentication).

### 2. Connect Your MCP Client

Pick your client and add the config. Replace `{mcp_server_domain}` with your storefront domain and `YOUR_BASE64_TOKEN` with the token from step 1.

<details>
<summary><strong>Cursor</strong></summary>

Add to `.cursor/mcp.json` (project-level) or `~/.cursor/mcp.json` (global):

```json
{
  "mcpServers": {
    "fynd-commerce": {
      "url": "https://{mcp_server_domain}/api/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_BASE64_TOKEN"
      }
    }
  }
}
```

</details>

<details>
<summary><strong>Claude Desktop</strong></summary>

Add to the config file:

- **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "fynd-commerce": {
      "url": "https://{mcp_server_domain}/api/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_BASE64_TOKEN"
      }
    }
  }
}
```

</details>

<details>
<summary><strong>Claude Code (CLI)</strong></summary>

```bash
claude mcp add fynd-commerce \
  --transport http \
  --url "https://{mcp_server_domain}/api/mcp" \
  --header "Authorization: Bearer YOUR_BASE64_TOKEN"
```

Or add manually to `~/.claude/settings.json`:

```json
{
  "mcpServers": {
    "fynd-commerce": {
      "url": "https://{mcp_server_domain}/api/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_BASE64_TOKEN"
      }
    }
  }
}
```

</details>

<details>
<summary><strong>Antigravity</strong></summary>

Add in workspace settings:

```json
{
  "mcpServers": {
    "fynd-commerce": {
      "url": "https://{mcp_server_domain}/api/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_BASE64_TOKEN"
      }
    }
  }
}
```

</details>

### 3. Start Talking to Your Storefront

Once connected, try these example prompts:

#### Browse & Discover

> "Show me the latest sneakers under 5000 rupees"

> "What collections are available on the store?"

> "Show me details of collection summer-sale with its products"

> "Sort t-shirts by popularity"

#### Login & Session

> "Log me in with mobile number 9876543210"

> "Verify OTP 1234"

> "Am I logged in?"

#### Cart & Coupons

> "Add the Nike Air Max in size 9 to my cart"

> "Show me my cart"

> "What coupons are available?"

> "Apply coupon SAVE20 to my cart"

#### Checkout & Orders (COD)

> "Show my saved addresses"

> "Add a new address: 42 MG Road, Bengaluru, Karnataka, 560001"

> "Place my order"

> "Confirm my order with code ABC123"

> "What's the status of my last order?"

> "Show me all my past orders"

#### Full Shopping Flow

> "Search for wireless headphones, add the best-rated one to cart, apply any available coupon, and place a COD order to my saved address"

---

## Available Tools (20)

### Catalog (4)

| Tool | Description |
|------|-------------|
| `list_products` | Search products with filters — query, price, stock, sort, pagination |
| `get_product` | Get detailed product info — prices, media, sizes |
| `list_collections` | Browse product collections |
| `get_collection` | Get collection details and its products |

**Supported sort options:** `latest`, `price_asc`, `price_dsc`, `popularity`, `discount_dsc`, `rating_dsc`

### Authentication (4)

| Tool | Description |
|------|-------------|
| `send_login_otp` | Send OTP to mobile (3 per mobile / 15 min rate limit) |
| `verify_login_otp` | Verify the received OTP |
| `get_session_status` | Check current session status |
| `logout` | End current session |

- Redis-backed sessions with 24h TTL
- AsyncLocalStorage-based per-request isolation

### Cart (2)

| Tool | Description |
|------|-------------|
| `add_to_cart` | Add item to cart with size and quantity |
| `get_cart` | View current cart contents |

### Coupons (3)

| Tool | Description |
|------|-------------|
| `list_coupons` | List available coupons |
| `apply_coupon` | Apply a coupon to the cart |
| `remove_coupon` | Remove applied coupon |

### Address & Checkout (3)

| Tool | Description |
|------|-------------|
| `list_addresses` | List saved addresses |
| `add_address` | Add a new address (6-digit pincode validation) |
| `place_order` | Preview order with a 10-minute confirmation code |

### Order Management (4)

| Tool | Description |
|------|-------------|
| `confirm_order` | Finalize COD order using confirmation code |
| `get_order_status` | Check status of an order |
| `cancel_order` | Cancel an order (explicit "yes" required) |
| `list_orders` | List past orders |

## Two-Step Order Confirmation

Orders follow a secure **preview → confirm** flow:

1. `place_order` — returns an order summary and a confirmation code (valid for 10 minutes)
2. `confirm_order` — finalizes COD placement using the confirmation code

> **Note:** COD orders cannot be cancelled after confirmation.

## Security & Reliability

- Bearer token authentication
- Zod schema validation on all inputs
- OTP rate limiting with retry-after support
- Image auto-base64 encoding (JPEG/PNG/WebP, 5s timeout)

## Limitations

- Supports **COD (Cash on Delivery) orders only**
- Currently supports **Indian sales channels**
