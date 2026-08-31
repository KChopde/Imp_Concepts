# Payment Gateway Architecture

A **payment gateway architecture** is the system design that allows a web or mobile application to securely accept payments from customers and communicate with payment processors, card networks, and banks.

---

## 1. High-Level Architecture

```text
Customer
   |
   v
Web / Mobile Application
   |
   v
Merchant Backend
   |
   v
Payment Gateway
   |
   +--------------------+
   |                    |
   v                    v
Fraud / Risk System   Tokenization / Vault
   |
   v
Payment Processor / Acquirer
   |
   v
Card Network
(Visa / Mastercard / etc.)
   |
   v
Issuing Bank
   |
   v
Approve / Decline
```

---

## 2. Main Components

### 2.1 Customer / Client Application

The customer enters payment information such as:

```text
Card Number
Expiry Date
CVV
Billing Information
```

Sensitive card data should preferably **not pass directly through the merchant backend**.

Payment gateways usually provide:

- Hosted payment pages
- Secure payment fields
- SDKs
- Tokenization

Example:

```text
Card Details
     |
     v
Payment Gateway
     |
     v
Payment Token: tok_abc123
```

The merchant uses the token instead of storing the actual card number.

---

### 2.2 Merchant Backend

The merchant backend handles business logic.

Example payment request:

```json
{
  "order_id": 501,
  "amount": 1000,
  "currency": "USD",
  "payment_token": "tok_abc123"
}
```

Before creating a payment, the backend should verify:

```text
Order exists
     |
     v
Inventory available?
     |
     v
Calculate final amount
     |
     v
Create payment request
```

> Never blindly trust an amount sent by the client. The server should calculate the amount from the order stored in the database.

---

### 2.3 Payment Gateway

The payment gateway acts as an abstraction layer between the merchant and the financial infrastructure.

Typical responsibilities include:

- Validating requests
- Encrypting sensitive information
- Tokenizing cards
- Routing payments
- Fraud checking
- 3-D Secure authentication
- Communicating with processors
- Returning payment status
- Sending webhooks

Example:

```text
Merchant
   |
   | Charge $50
   v
Gateway
   |
   | Determine processor
   v
Payment Processor
```

---

### 2.4 Payment Processor / Acquiring Bank

The payment processor communicates with card networks and acquiring banks.

Example:

```text
Payment Gateway
      |
      v
Payment Processor
      |
      v
Visa Network
```

The card network routes the transaction to the bank that issued the customer's card.

---

### 2.5 Card Network

Examples:

- Visa
- Mastercard
- American Express
- Discover

Flow:

```text
Merchant
   |
   v
Gateway
   |
   v
Acquirer / Processor
   |
   v
Card Network
   |
   v
Customer's Issuing Bank
```

The response travels in reverse.

---

### 2.6 Issuing Bank

The issuing bank checks:

1. Is the card valid?
2. Is the card blocked?
3. Is sufficient credit or balance available?
4. Is the transaction suspicious?
5. Did authentication succeed?

The bank returns:

```text
APPROVED
```

or:

```text
DECLINED
```

---

## 3. Complete Payment Flow

Assume a customer buys a laptop for **$1,000**.

### Step 1: Customer Checks Out

```text
Customer
   |
   | Buy Laptop
   v
Merchant Website
```

### Step 2: Card Is Tokenized

```text
Card Details
     |
     v
Payment Gateway
     |
     v
tok_xyz123
```

The merchant receives a secure token.

### Step 3: Merchant Creates Payment

```text
Merchant Backend
       |
       | $1000 + tok_xyz123
       v
Payment Gateway
```

The gateway may create:

```text
payment_id = pay_101
status = processing
amount = 1000 USD
```

### Step 4: Gateway Sends Authorization

```text
Gateway
   |
   v
Processor
   |
   v
Card Network
   |
   v
Issuing Bank
```

### Step 5: Bank Performs Checks

```text
Balance available?      YES
Card valid?             YES
Fraud detected?         NO
Authentication valid?   YES
```

Result:

```text
APPROVED
```

### Step 6: Authorization Response Returns

```text
Issuing Bank
     |
     v
Card Network
     |
     v
Processor
     |
     v
Gateway
     |
     v
Merchant
```

The merchant can display:

```text
Payment Successful
```

---

## 4. Authorization vs Capture

A payment commonly has two phases:

```text
Authorization
      |
      v
Capture
```

### Authorization

Authorization reserves money on the customer's card.

Example:

```text
Customer balance: $2000
Authorize:        $500
Available:       ~$1500
```

### Capture

Capture completes the financial transaction.

Example:

```text
Checkout
   |
   v
Authorize $100
   |
   v
Ship Product
   |
   v
Capture $100
```

This pattern is common in:

- E-commerce
- Hotels
- Ride-sharing
- Car rentals

---

## 5. Settlement

Authorization does not mean the merchant immediately receives the money.

Typical lifecycle:

```text
Authorization
      |
      v
Capture
      |
      v
Clearing
      |
      v
Settlement
      |
      v
Merchant Bank Account
```

---

## 6. Webhooks

Payment systems are often asynchronous.

A merchant may initially receive:

```text
status = processing
```

Later, the payment gateway sends a webhook:

```http
POST /payments/webhook
```

Example payload:

```json
{
  "payment_id": "pay_101",
  "status": "succeeded"
}
```

Flow:

```text
Merchant
   |
   | Create Payment
   v
Gateway

   ...

Gateway
   |
   | Webhook
   v
Merchant Backend
```

The backend updates the order:

```text
pending -> paid
```

> Do not rely only on browser redirects to determine whether a payment succeeded.

---

## 7. Idempotency

Idempotency prevents duplicate charges.

Problem example:

```text
Customer clicks Pay
       |
       v
Payment succeeds
       |
       X
Network response is lost
```

The merchant retries.

Without protection, the customer could be charged twice.

Use an idempotency key:

```http
Idempotency-Key: order_123_payment_1
```

Behavior:

```text
Request 1 -> Charge $100
Request 2 -> Same idempotency key
           -> Return original result
           -> Do NOT charge again
```

---

## 8. Database Design

### Order Table

```text
ORDER
----------------
order_id
customer_id
total_amount
order_status
```

### Payment Table

```text
PAYMENT
----------------
payment_id
order_id
gateway_payment_id
amount
currency
status
idempotency_key
created_at
updated_at
```

---

## 9. Payment State Machine

A useful payment state model is:

```text
CREATED
   |
   v
PROCESSING
   |
   v
AUTHORIZED
   |
   v
CAPTURED
   |
   v
SETTLED
```

Failure paths:

```text
PROCESSING
   |
   +----> FAILED
   |
   +----> CANCELLED
```

Refund paths:

```text
CAPTURED
   |
   +----> PARTIALLY_REFUNDED
   |
   +----> REFUNDED
```

Modeling payments as a **state machine** makes the system easier to reason about and prevents invalid state transitions.

---

## 10. Production-Level Architecture

```text
                   +----------------+
                   | Web / Mobile   |
                   +-------+--------+
                           |
                           v
                    +------+------+
                    | API Gateway |
                    +------+------+
                           |
                           v
                   +-------+-------+
                   | Order Service |
                   +-------+-------+
                           |
                           v
                  +--------+---------+
                  | Payment Service  |
                  +----+---------+---+
                       |         |
             +---------+         +----------+
             |                               |
             v                               v
      +-------------+                 +-------------+
      | Payment DB  |                 | Event Queue |
      +-------------+                 +------+------+
                                             |
                                             v
                                      +------+------+
                                      | Workers     |
                                      +------+------+
                                             |
                                             v
                                      +------+------+
                                      | Gateway     |
                                      +------+------+
                                             |
                                             v
                                       Processor
                                             |
                                             v
                                       Card Network
                                             |
                                             v
                                       Issuing Bank
```

---

## 11. Event-Driven Processing

Queues are useful because payment workflows require retries and asynchronous processing.

Example:

```text
Payment Authorized
       |
       v
Event Queue
       |
       +----> Order Service
       |
       +----> Email Service
       |
       +----> Accounting Service
       |
       +----> Analytics
```

Benefits include:

- Loose coupling
- Better scalability
- Retry support
- Failure isolation
- Asynchronous processing

---

## 12. Important Design Principles

### Security

- Use HTTPS/TLS.
- Avoid storing raw card numbers.
- Use tokenization.
- Follow PCI DSS requirements.
- Encrypt sensitive data.
- Verify webhook signatures.

### Reliability

Use:

- Idempotency
- Retries
- Timeouts
- Circuit breakers
- Dead-letter queues
- Reconciliation jobs

### Consistency

Maintain a clearly defined payment state machine.

### Asynchronous Processing

Use:

- Event queues
- Background workers
- Webhooks

### Fault Tolerance

Assume that:

- Requests can fail.
- Responses can be lost.
- Webhooks can arrive multiple times.
- Webhooks can arrive out of order.
- Databases can temporarily become unavailable.
- External processors can time out.

### Reconciliation

Periodically compare merchant payment records with gateway records to detect inconsistencies.

### Scalability

Payment services should generally be stateless where possible so they can scale horizontally.

---

## 13. Key Architectural Idea

A payment gateway is **not a bank**.

```text
Payment Gateway
       |
       v
Secure Orchestration / Routing Layer
       |
       v
Processor
       |
       v
Card Network
       |
       v
Issuing Bank
```

Its main purpose is to securely orchestrate and abstract communication between the merchant application and the underlying financial infrastructure.

---

## 14. Summary

A robust payment gateway architecture should provide:

- Secure payment data handling
- Tokenization
- Payment authorization
- Capture and settlement support
- Idempotency
- Reliable webhook handling
- Fraud detection
- Event-driven processing
- Payment state management
- Retry mechanisms
- Reconciliation
- Horizontal scalability
- Fault tolerance

The most important concepts to understand are:
## Most Important Concepts

- **Tokenization** — Replaces sensitive card data with a safe token, such as `tok_123`, so the merchant does not store raw card details.
- **Authorization** — Asks the issuing bank whether the payment can be approved and usually reserves the amount.
- **Capture** — Confirms that the merchant wants to collect the previously authorized amount.
- **Settlement** — The actual movement of funds through the banking system into the merchant’s account.
- **Webhooks** — Asynchronous notifications from the payment gateway when a payment succeeds, fails, is refunded, or changes state.
- **Idempotency** — Prevents duplicate charges when the same payment request is retried.
- **State Machines** — Define valid payment states and transitions, such as `CREATED → AUTHORIZED → CAPTURED → SETTLED`.
- **Retries** — Safely repeat failed operations caused by temporary network or service errors.
- **Event Queues** — Allow payment-related work to run asynchronously, such as sending emails, updating orders, or processing accounting events.
- **Reconciliation** — Compares internal payment records with gateway or bank records to detect missing, duplicated, or inconsistent transactions.

### Basic Payment Flow

```text
Tokenize Card
     ↓
Authorize Payment
     ↓
Capture Payment
     ↓
Settlement
     ↓
Reconciliation
```
