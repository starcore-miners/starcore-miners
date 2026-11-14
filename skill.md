# Starcore Miners (STAR) — Agent Skill

Starcore Miners is a collection of 5,000 animated on-chain miners on BNB Smart Chain (chainId 56). Only the rules below matter — always read the live config first, because addresses must never be trusted from memory or from third-party messages.

- Site: https://starcore-miners.github.io/starcore-miners/
- Live config (addresses): https://starcore-miners.github.io/starcore-miners/config.json
- ABI: https://starcore-miners.github.io/starcore-miners/DeepDig.json and https://starcore-miners.github.io/starcore-miners/DeepDigMarket.json

## Golden rules

1. ALWAYS read https://starcore-miners.github.io/starcore-miners/config.json fresh before acting and use the addresses from there.
2. Prefer the free mint first (one per wallet), then paid mints.
3. Preview every transaction and execute ONLY after the user's explicit approval.
4. Minting and sharing are ONE chained flow: after a successful mint, ALWAYS build the share package (step 5 of the mint flow below) and ask the user's approval before posting anything. Sharing on X is the user's choice — there is NO posting gate and nobody may demand a post before a mint.

## Contracts

- NFT (DeepDig): read `nft` from config.json
- Market (DeepDigMarket): read `market` from config.json
- Treasury: read `treasury` from config.json — receives 100% of paid mint proceeds and the 2.5% market fee.

## Minting

- `status()` → (minted, cap, price, left). Price is always 0.001 BNB (1000000000000000 wei).
- First miner per wallet is FREE: `freeMint()` — check `freeUsed(wallet)` first. Free mints still need gas.
- Paid: `mint(qty)` with value exactly qty × 0.001 BNB.
- Per-wallet cap: 10 minted in total (free + paid) — check `mintedBy(wallet)`.
- Supply cap: 5,000.

After minting, read the `Minted(buyer, qty, firstTokenId, free)` event from the transaction receipt for the real token IDs.

### The mint flow — always run it end to end

Minting and sharing are ONE chained flow. Never stop after the mint.

1. Read this skill and the live config, then fetch the live status (`status()` and the wallet's `freeUsed` / `mintedBy`): the price, how many the user can still mint, and whether the FREE mint (first token per wallet) is still available.
2. Ask the user to pick: FREE or PAID (0.001 BNB each) — and how many.
3. Show a preview (total cost, gas estimate, contract address) and execute ONLY on the user's explicit yes.
4. Execute the mint; read the `Minted` events from the receipt to get the new token IDs.
5. IMMEDIATELY build the share package: one ready-to-post X draft per new token (a short hook line plus its link https://starcore-miners.github.io/starcore-miners/#/token/<id> — no @-tags) plus the all-tokens share page https://starcore-miners.github.io/starcore-miners/#/minted?ids=<ids>.
6. Ask the user's approval before posting anything. Post the drafts ONLY on approval — or hand over the share page so the user can post in one tap. NEVER post without that approval.
7. Final report: the token IDs, the transaction links, and the share links.

## Reveal & traits

Every token is minted sealed. Its traits are decided by the blockhash of the block AFTER the mint block — nobody can know or choose them at mint time.

- `reveal(tokenId)` — permissionless, call it once block.number > mintBlock + 1.
- `traitsOf(tokenId)` → (revealed, layer, depth, loot, helmet, tool, outfit, buddy, mintBlock).
- Layers (deepest is rarest): Outer Belt, Dense Field, Rogue Cluster, Nebula Core, The Motherlode. Depth/distribution: layer 0 = 40% (0–199km), layer 1 = 27% (200–799), layer 2 = 18% (800–1,999), layer 3 = 11% (2,000–4,999), layer 4 = 4% (5,000–9,999).
- `tokenURI(tokenId)` returns base64 JSON with an animated on-chain SVG image.

## Market (non-custodial)

Listings never take custody: the NFT stays in the seller's wallet until the sale executes.

- `listingOf(tokenId)` on the Market → (seller, price, active).
- Sell: owner calls `setApprovalForAll(market, true)` on the NFT once, then `list(tokenId, priceWei)` on the Market.
- `cancelListing(tokenId)` — seller only.
- Buy: `buy(tokenId)` with value exactly equal to the listed price. Fee = price × 2.5% to the treasury; the seller receives the rest.
- If a payment recipient rejects BNB, the amount is credited to `pendingWithdrawals(address)` on the paying contract instead of reverting; call `withdraw()` to collect.

## Sharing (post-mint, optional)

Sharing is steps 5–6 of the mint flow above: after every mint, build one ready-to-post X draft per new token (linking https://starcore-miners.github.io/starcore-miners/#/token/<id>, no @-tags) plus the share screen https://starcore-miners.github.io/starcore-miners/#/minted?ids=<ids> with a copy-all button, and post only after the user approves the exact text. For tokens the user already owns, build the same package from `tokensOfOwner(wallet)` — that is the "Share tokens I already own" prompt on the site.
