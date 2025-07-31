# GetQuote API Specification

## Background
Cross-chain bridge protocols typically expose `getQuote` endpoints that provide users with transfer cost estimates and route information. These endpoints serve as the foundation for user experience in cross-chain applications, allowing wallets and dApps to display accurate fees and timing before users commit to transactions.

Current bridge protocols assume single-chain origin, this standard enables: multi-source input optimization across chains and standardized solver integration.

## Specification

### Types

```typescript
interface AssetAmount {
  asset: string;    // ERC-7828 format: address@chainId
  amount: string;
}

interface GetQuoteRequest {
  availableInputs: {
      input: AssetAmount;
      priority?: number;
    }[];
  recipientAmount: AssetAmount[];  // Amount recipient will receive
  preference?: 'price' | 'speed' | 'input-priority';
  expiry?: number;      // Unix timestamp when request expires
}

interface GetQuoteResponse {
  quotes: {
    orders: {
      settler: string;  // Solver contract address
      data: object;     // EIP-712 typed data for signing
    };
    requiredAllowances: AssetAmount[];
    recipientAmount: AssetAmount[];  
    validUntil: number;
    eta: number;  // Unix timestamp when transfer is expected to complete
    totalFeeUsd: number;
  }[];
}
```

### Endpoint

**`/quotes`**
- **Request**: `GetQuoteRequest`
- **Response**: `GetQuoteResponse`

### Request Parameters

- **`availableInputs`**: Assets and amounts sender can spend (using interoperable addresses), with optional priority hints (1 = highest priority, 2 = second, etc.)
- **`recipientAmount`**: Amount recipient will receive (using interoperable addresses) - sender pays this amount plus fees
- **`preference`**: High-level optimization hint ('price', 'speed', or 'input-priority')
- **`expiry`**: Optional Unix timestamp when the quote request expires

### Response Fields

- **`quotes`**: Array of executable intent options
- **`orders`**: Contains `settler` address and EIP-712 `data` for signing
- **`requiredAllowances`**: Token allowances needed before execution
- **`recipientAmount`**: Confirmed amount recipient will receive (matches request)
- **`validUntil`**: Quote expiration timestamp
- **`eta`**: Unix timestamp when the cross-chain transfer is expected to complete
- **`totalFeeUsd`**: All fees sender pays in addition to recipient amount


## Implementation Flow

1. **Intent Expression**: Alice submits `GetQuoteRequest` specifying how much Bob should receive (`recipientAmount`)
2. **Solver Competition**: Multiple solvers return quotes showing total cost (recipient amount + fees)
3. **Allowance Setup**: Alice's wallet prepares required token allowances for the total amount
4. **Alice Selection**: Alice reviews quotes showing recipient amount, fees, and timing, selects preferred option
5. **Intent Signing**: Alice signs EIP-712 data authorizing the transfer
6. **Atomic Execution**: Solver executes signed intent, Bob receives exactly the specified `recipientAmount`

## Security Considerations

- Verify `orders.data` contents match quote parameters (recipient amount, fees) before signing
- Validate `settler` address is trusted
- Respect `validUntil` timestamps
- Ensure sufficient allowances are set for total amount (recipient amount + fees)
- Confirm `recipientAmount` in response matches the request to prevent manipulation

## Rationale

### Exact Recipient Amounts
Using `recipientAmount` ensures clear specification of exactly what the recipient will receive. The sender (Alice) pays this amount plus fees, creating predictable outcomes with no slippage on the recipient side.

### Simplified Preferences  
High-level preferences (`price`, `speed`, `input-priority`) avoid exposing solver-specific algorithm details while providing clear, meaningful optimization hints. This approach is simpler and clearer than numeric weights or complex parameter combinations.

### Consistent Interoperable Addresses
Using ERC-7828 interoperable address format (`address@chainId`) throughout both `availableInputs` and `recipientAmount` eliminates redundancy and ensures consistent asset identification across the entire API. The chain information is embedded in the asset identifier, making the structure cleaner and less error-prone.

### Fee Structure
Fees are **additive** to the recipient amount. If Alice wants Bob to receive exactly 1 ETH, Alice pays 1 ETH + fees (e.g., 1.001 ETH total). Bob receives exactly 1 ETH as specified in `recipientAmount`. This creates clear separation between what the recipient gets and what the sender pays.

### Multiple Recipient Outputs  
Supporting multiple `recipientAmount` entries with interoperable addresses enables critical use cases like the recipient receiving primary tokens plus gas tokens. For example, Alice can specify that Bob should receive 100 USDC + 0.005 ETH for gas on Mainnet using the clean `asset@chainId` format.

<!-- TODO: Open Design Questions
1. Should this API support async quote responses for aggregator use cases?
2. Onchain fee handling: How should fees be calculated when intents are announced onchain rather than through direct quote requests? -->