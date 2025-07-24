# FortiFi Offramp API Documentation

## Overview
The Syria Offramp API provides secure cryptocurrency-to-cash conversion services specifically designed for the Syrian market. The system currently supports two payout methods: cash pickup with OTP verification and Thiqa wallet transfers, along with comprehensive KYC verification for recipients.

## Base URL
```
https://api.syria-offramp.com/v1
```

## Authentication
All API requests require authentication using an API key. Include your API key in the request headers:
```
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
```

### API Key Registration

**POST** `/auth/register`

Register for a new API key to access the Syria Offramp API.

**Request Body:**
```json
{
  "company_name": "string",
  "contact_email": "string",
  "phone_number": "string",
  "business_license": "string",
  "intended_volume": "string"
}
```

**Response:**
```json
{
  "success": true,
  "api_key": "sk_live_...",
  "message": "API key generated successfully",
  "expires_at": "2025-07-24T10:30:00Z"
}
```

## KYC & Recipient Management

### Verify Recipient

**POST** `/kyc/verify-recipient`

Verify a recipient's identity for compliance purposes before allowing payouts.

**Request Body:**
```json
{
  "recipient_id": "string",
  "full_name": "string",
  "national_id": "string",
  "phone_number": "string",
  "address": {
    "street": "string",
    "city": "string",
    "governorate": "string"
  },
  "document_type": "national_id|passport",
  "document_front": "base64_encoded_image",
  "document_back": "base64_encoded_image"
}
```

**Response:**
```json
{
  "success": true,
  "verification_id": "ver_123456",
  "status": "pending|approved|rejected",
  "estimated_processing_time": "24-48 hours"
}
```

### Check KYC Status

**GET** `/kyc/status/{verification_id}`

Check the current status of a KYC verification request.

**Response:**
```json
{
  "success": true,
  "verification_id": "ver_123456",
  "status": "pending|approved|rejected",
  "verified_at": "2024-07-24T10:30:00Z",
  "rejection_reason": "string"
}
```

## Payout Methods

### 1. Cash Pickup with OTP

#### Create Cash Payout Order

**POST** `/payouts/cash`

Create a cash pickup order that generates an OTP for secure collection.

**Request Body:**
```json
{
  "recipient_id": "string",
  "amount": 1000.00,
  "currency": "SYP",
  "crypto_amount": 0.001,
  "crypto_currency": "BTC",
  "pickup_location": {
    "location_id": "string",
    "name": "string",
    "address": "string",
    "city": "string",
    "governorate": "string"
  },
  "reference": "string"
}
```

**Response:**
```json
{
  "success": true,
  "payout_id": "payout_123456",
  "otp_code": "123456",
  "status": "pending",
  "pickup_location": {
    "name": "Damascus Exchange Center",
    "address": "Umayyad Square, Damascus",
    "phone": "+963-11-1234567",
    "hours": "9:00 AM - 6:00 PM"
  },
  "expires_at": "2024-07-25T10:30:00Z"
}
```

#### Check Cash Payout Status

**GET** `/payouts/cash/{payout_id}`

**Response:**
```json
{
  "success": true,
  "payout_id": "payout_123456",
  "status": "pending|completed|expired|cancelled",
  "amount": 1000.00,
  "currency": "SYP",
  "picked_up_at": "2024-07-24T14:30:00Z",
  "pickup_location": {
    "name": "Damascus Exchange Center",
    "address": "Umayyad Square, Damascus"
  }
}
```

### 2. Thiqa Wallet Transfer

#### Create Thiqa Wallet Payout

**POST** `/payouts/thiqa`

Transfer funds directly to a Thiqa wallet using the recipient's phone number.

**Request Body:**
```json
{
  "recipient_id": "string",
  "thiqa_phone_number": "string",
  "amount": 1000.00,
  "currency": "SYP",
  "crypto_amount": 0.001,
  "crypto_currency": "BTC",
  "reference": "string"
}
```

**Response:**
```json
{
  "success": true,
  "payout_id": "payout_789012",
  "status": "processing",
  "thiqa_transaction_id": "thiqa_tx_123",
  "estimated_completion": "5-10 minutes"
}
```

#### Check Thiqa Payout Status

**GET** `/payouts/thiqa/{payout_id}`

**Response:**
```json
{
  "success": true,
  "payout_id": "payout_789012",
  "status": "processing|completed|failed",
  "thiqa_transaction_id": "thiqa_tx_123",
  "completed_at": "2024-07-24T10:35:00Z",
  "failure_reason": "string"
}
```

## Utility Endpoints

### Get Available Pickup Locations

**GET** `/locations/cash-pickup`

Get a list of available cash pickup locations.

**Query Parameters:**
- `governorate` (optional): Filter by governorate
- `city` (optional): Filter by city

**Response:**
```json
{
  "success": true,
  "locations": [
    {
      "location_id": "loc_001",
      "name": "Damascus Exchange Center",
      "address": "Umayyad Square, Damascus",
      "city": "Damascus",
      "governorate": "Damascus",
      "phone": "+963-11-1234567",
      "hours": "9:00 AM - 6:00 PM",
      "status": "active"
    }
  ]
}
```

### Get Exchange Rates
**GET** `/rates`

Returns current exchange rates for supported cryptocurrencies to SYP.

**Response:**
```json
{
  "success": true,
  "rates": {
    "BTC_SYP": 45000000.00,
    "ETH_SYP": 2800000.00,
    "USDT_SYP": 12500.00,
    "USDC_SYP": 12480.00
  },
  "updated_at": "2024-07-24T10:30:00Z"
}
```

## Error Handling

All API responses include a `success` field. Failed requests return:

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message",
    "details": "Additional error information"
  }
}
```

### Common Error Codes

- `INVALID_API_KEY`: API key is missing or invalid
- `INSUFFICIENT_KYC`: Recipient KYC verification required
- `INVALID_AMOUNT`: Amount is below minimum or above maximum limits
- `LOCATION_UNAVAILABLE`: Selected pickup location is temporarily unavailable
- `THIQA_ACCOUNT_NOT_FOUND`: Thiqa phone number not registered
- `RATE_LIMIT_EXCEEDED`: Too many requests in a short period
- `RECIPIENT_NOT_VERIFIED`: Recipient must complete KYC verification first

## Payout Status Flow

### Cash Pickup Status Flow
1. **pending** - Order created, waiting for pickup
2. **completed** - Cash collected with valid OTP
3. **expired** - Order expired (24 hours)
4. **cancelled** - Order cancelled by user or system

### Thiqa Wallet Status Flow
1. **processing** - Transfer initiated to Thiqa wallet
2. **completed** - Funds successfully transferred
3. **failed** - Transfer failed (insufficient balance, invalid account, etc.)

## Rate Limits
- 100 requests per minute per API key
- 1000 requests per hour per API key

## Webhook Notifications

Configure webhook URLs to receive real-time updates on payout status changes.

**POST** `/webhooks/configure`

**Request Body:**
```json
{
  "url": "https://your-server.com/webhook",
  "events": ["payout.completed", "payout.failed", "kyc.approved", "kyc.rejected"]
}
```

### Webhook Payload Examples

**Payout Completed:**
```json
{
  "event": "payout.completed",
  "payout_id": "payout_123456",
  "timestamp": "2024-07-24T14:30:00Z",
  "data": {
    "amount": 1000.00,
    "currency": "SYP",
    "method": "cash",
    "recipient_id": "recipient_789"
  }
}
```

**KYC Status Update:**
```json
{
  "event": "kyc.approved",
  "verification_id": "ver_123456",
  "recipient_id": "recipient_789",
  "timestamp": "2024-07-24T10:30:00Z"
}
```

## Security & Compliance

- All recipients must complete KYC verification before first payout
- OTP codes expire after 24 hours for cash pickups
- Document images are encrypted and stored securely
- All API calls must use HTTPS
- Webhook signatures should be verified using HMAC-SHA256

