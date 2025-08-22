# RedactedCLI
A software that automated tasks for our users on Discord and across the NFT space. We built a community of 2,600+ members throughout the whole process.


## Feature overview
- MagicEden NFT Sniper: User set a price for a specific NFT, we handled searching the backend for it, and proccessed the transaction on the blockchain it as soon as it finds the matching parameters.
- Multi-Threaded CandyMachine NFT Minter: User set a NFT mint they wanted, we processed the transaction within a second as soon as it went live.
- Discord Mass Server Joiner, Auto link clicker, and much much more


## Flow of Candy Machine v2 Mint:

Initialize & connect
- Build web3.Connection(rpc, 'confirmed'); construct wallet from base58 secret with NodeWallet.local. Log and set window title.

Load CM state
- getCandyMachineState(anchorWallet, candyMachineId, connection) loads IDL, program, and derives flags like isActive, isSoldOut, goLiveDate, whitelist, price, etc. 

Mint one
- mintOneToken(candyMachine, payerPublicKey) creates a new mint, ATAs, handles gatekeeper & whitelist accounts, supports SPL-token payments, and builds/dispatches the mint tx. Returns [txid]. 

Confirm & report
- awaitTransactionSignatureConfirmation(txid, timeout, connection, queryStatus) waits for confirmation; on custom errors (e.g., insufficient funds), log + retry; on success, increment counters. 

Retry policy
- On missing txid or failure, backoff (setTimeout) and re-enter MintInit(). Visible success/error logs persist per task.


## Flow of MagicEden Sniper:
Boot & UI
- Derive Keypair from bs58 secret; connect to mainnet RPC; set title/log context. 

Monitor listings
- HTTPUtils.tlsHttpGet → getListedNFTsByQuery (sorted by takerAmount) for collectionSymbol. If none, log & retry after delay. 

Fetch item details
- getNFTByMintAddress to capture seller, price, auction house, token ATA, and referral data. 

Price gate & buy_now
- If price ≤ WISH_PRICE, request v2/instructions/buy_now for a pre-built, partially-signed tx blob. 

Sign & broadcast
- Transaction.from(data) → partialSign(payer) → serialize({ requireAllSignatures:false, verifySignatures:false }) → sendRawTransaction. 

Confirm & retry
- Poll re-sends while awaiting awaitTransactionSignatureConfirmation; on errors/timeouts, attempt simulation for logs and schedule a retry. 


<img width="1680" alt="Screenshot 2024-08-07 at 5 32 07 PM" src="https://github.com/user-attachments/assets/efea0118-ff3a-457c-a36b-817df37c2ab1">
<img width="1680" alt="Screenshot 2024-08-07 at 5 31 49 PM" src="https://github.com/user-attachments/assets/5741637e-7fb9-41ad-98f1-88227546dbfc">
