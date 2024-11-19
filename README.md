# Building an Ordinals NFT Marketplace on Bitcoin Network

## Technical Definition of Bitcoin NFTs
Bitcoin NFTs, implemented through the Ordinals protocol, are created by inscribing data directly into satoshis (the smallest unit of bitcoin, 1/100,000,000 of a BTC). Unlike Ethereum NFTs that use smart contracts, Bitcoin NFTs leverage the network's native features:

### Core Components of Bitcoin NFTs
1. **Content Storage**
   - Digital content (images, text, etc.) stored directly on Bitcoin blockchain
   - Stored in transaction witness data using Taproot technology
   - Supports standard file formats (JPEG, PNG, etc.)

2. **Ordinal Numbers**
   - Every satoshi (0.00000001 BTC) gets a unique serial number
   - Numbers assigned in order when Bitcoin is mined
   - These numbers let us track specific satoshis like unique tokens

3. **Technical Structure**
```
witness: <content_data>
scriptPubKey: <taproot_address>
```

4. **Storage Method**
   - Uses SegWit (Segregated Witness) for efficient storage
   - Data permanently stored on Bitcoin blockchain
   - Utilizes Taproot for improved scalability

## Bitcoin Script Limitations
Bitcoin uses a simple stack-based scripting language called Script. While it supports basic operations, it's intentionally restricted compared to platforms like Ethereum. A typical Bitcoin script looks like:

```
OP_DUP OP_HASH160 <pubKeyHash> OP_EQUALVERIFY OP_CHECKSIG
```

This limitation means we can't implement traditional NFT marketplace logic directly in Bitcoin's script. Instead, we need an alternative approach using PSBTs.

## PSBT-Based Architecture

### Overview
The marketplace architecture relies on three main components:
1. PSBT generation and storage
2. Off-chain order matching
3. Transaction finalization

### PSBT Generation
When a user lists an Ordinal for sale:
1. Create a PSBT with the Ordinal UTXO as input
2. Sign the seller's input partially
3. Store the PSBT in the marketplace database
4. Create an order entry linking to the PSBT

```javascript
// Pseudocode for PSBT creation
const psbt = new Bitcoin.Psbt()
psbt.addInput({
    hash: ordinalTxId,
    index: ordinalVout,
    witnessUtxo: ordinalUtxo
})
psbt.addOutput({
    address: escrowAddress,
    value: ordinalValue
})
psbt.signInput(0, sellerPrivateKey)
```

### Database Schema
```sql
CREATE TABLE listings (
    id UUID PRIMARY KEY,
    ordinal_id TEXT,
    price_sats BIGINT,
    psbt_hex TEXT,
    seller_address TEXT,
    status TEXT
);
```

## Order Matching & Execution

### Buyer Flow
1. Buyer selects an Ordinal listing
2. System retrieves stored PSBT
3. Buyer adds payment input and output
4. Buyer signs their inputs
5. System finalizes transaction

```javascript
// Pseudocode for purchase execution
const storedPsbt = Bitcoin.Psbt.fromHex(listing.psbt_hex)
storedPsbt.addInput({
    hash: buyerUtxoTxId,
    index: buyerUtxoVout,
    witnessUtxo: buyerUtxo
})
storedPsbt.addOutput({
    address: sellerAddress,
    value: listing.price_sats
})
storedPsbt.signInput(1, buyerPrivateKey)
const finalTx = storedPsbt.finalizeAllInputs().extractTransaction()
```

## Security Considerations

### PSBT Validation
- Verify input UTXOs exist and are unspent
- Validate signature for Ordinal input
- Check output addresses match listing parameters
- Ensure no additional inputs/outputs were maliciously added

### Transaction Monitoring
- Monitor mempool for competing transactions
- Track transaction confirmation status
- Implement automatic refunds for failed transactions

## Error Handling
1. PSBT Creation Failures
   - Invalid inputs
   - Signing errors
   - Insufficient funds

2. Transaction Broadcasting
   - Network issues
   - Fee estimation errors
   - Double-spend attempts

## Best Practices

### Performance Optimization
- Cache UTXO data
- Batch PSBT operations
- Implement connection pooling for database operations

### User Experience
- Provide transaction status updates
- Implement webhook notifications
- Display estimated confirmation times

## Conclusion
Building an Ordinals NFT marketplace requires working within Bitcoin's constraints while ensuring security and usability. The PSBT-based approach provides a robust solution that maintains Bitcoin's security model while enabling complex marketplace functionality.

Remember that this architecture requires careful testing and monitoring, especially around transaction finalization and error handling. Regular security audits and thorough testing of PSBT handling are essential for a production deployment.
