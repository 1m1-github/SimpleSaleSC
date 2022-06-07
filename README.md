# SimpleSaleSC
Algorand Smart Contract to sell tokens at a fixed price

We have an owner of some tokens of some asset.
The owner wants to sell these tokens at a fixed price.

The owner will create a Smart Contract (SC) with a fixed price and the asset id set.
The owner will then send the asset tokens to the SC. Once in the SC, the tokens cannot be retrieved. The tokens can only be bought at the fixed price.

For simplicity (and as a 1st version), the currency to pay for the tokens is fixed to ALGO. The price is set in microALGO per token.

When a buyer sends ALGOs to the SC, the SC will send the buyer the correct # of tokens and the ALGOs to the owner.

Total fees for the buyer is 0.004 ALGO, 0.001 each to
- call the SC
- send ALGOs to the SC
- compensate the SC to send tokens to the buyer
- compensate the SC to send ALGOs to the owner

The owner has to pay 0.001 ALGO to create the SC. The owner also has to keep x ALGO reserved in the account for the SC state.

The buyer will need to opt-in to the token if they have not yet. Cost 0.001 ALGO.
