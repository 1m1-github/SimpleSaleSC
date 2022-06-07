# SimpleSaleSC

## Algorand Smart Contract to sell tokens at a fixed price

We have an owner of some tokens of some asset.
The owner wants to sell these tokens at a fixed price.

The owner will create a Smart Contract (SC) with a fixed price and the asset id set.
The owner will then send the asset tokens to the SC. Once in the SC, the tokens cannot be retrieved. The tokens can only be bought at the fixed price.

For simplicity (and as a 1st version), the currency to pay for the tokens is fixed to ALGO. The price is set in microALGO per token.

When a buyer sends ALGOs to the SC, the SC will send the buyer the correct # of tokens and the ALGOs to the owner.

To avoid minimum balance issues, it is best for both the owner and the buyer to already hold 1 ALGO in their account.

The buyer will need to opt-in to the asset and the SC if they have not yet.

## How to run

The code is run using the `goal` cli (https://developer.algorand.org/docs/get-started/devenv/). The below syntax is for *nix systems.

Step 0 - Set env vars
```
export ALGORAND_DATA="$HOME/node/testnetdata"
export WALLET=2i2i
export CODE_DIR=./code
export TXNS_DIR=./txns
export SYSTEM_APPROVAL_FILE=$CODE_DIR/state_approval_program.teal
export SYSTEM_CLEAR_FILE=$CODE_DIR/state_clear_program.teal
export OWNER=2I2IXTP67KSNJ5FQXHUJP5WZBX2JTFYEBVTBYFF3UUJ3SQKXSZ3QHZNNPY
```

Step 1 - Create dApp and set SYSTEM_ID and SYSTEM_ACCOUNT
```
goal app create --creator $OWNER --approval-prog $SYSTEM_APPROVAL_FILE --clear-prog $SYSTEM_CLEAR_FILE --global-byteslices 0 --global-ints 2 --local-byteslices 0 --local-ints 0 --on-completion OptIn -w $WALLET
export SYSTEM_ID=
goal app info --app-id $SYSTEM_ID
export SYSTEM_ACCOUNT=
```

Step 2 - Create asset and set ASA
```
goal asset create --creator $OWNER --total $AMOUNT --unitname NOVALUE --decimals 0 -w $WALLET
export ASA=
```

Step 3 - Fund SC with 0.201 ALGO (for asset opt-in)
```
goal clerk send --from=$OWNER --to=$SYSTEM_ACCOUNT --amount=201000 -w $WALLET
```

Step 4 - Init SC with PRICE and AMOUNT tokens of asset (PRICE in microALGOs)
```
export PRICE=
export AMOUNT=
goal app call --from=$OWNER --app-id=$SYSTEM_ID --foreign-asset $ASA --app-arg="str:init" --app-arg="int:$PRICE" --out=$TXNS_DIR/init_call.stxn -w $WALLET
goal asset send --assetid=$ASA --from=$OWNER --to=$SYSTEM_ACCOUNT --amount=$AMOUNT --out=$TXNS_DIR/init_asa_send.stxn -w $WALLET
cat $TXNS_DIR/init_call.stxn $TXNS_DIR/init_asa_send.stxn > $TXNS_DIR/combinedtransactions.txn
goal clerk group --infile $TXNS_DIR/combinedtransactions.txn --outfile $TXNS_DIR/groupedtransactions.txn
goal clerk sign --infile $TXNS_DIR/groupedtransactions.txn --outfile $TXNS_DIR/init.stxn -w $WALLET
goal clerk rawsend --filename $TXNS_DIR/init.stxn
```

Step 5 - Opt-in BUYER into SC and asset
```
export BUYER=
goal asset send --amount 0 --assetid $ASA --from $BUYER --to $BUYER -w $WALLET --out=$TXNS_DIR/buyer_asa_optin.stxn
goal app optin --app-id $SYSTEM_ID --from $BUYER -w $WALLET --out=$TXNS_DIR/buyer_sc_optin.stxn
cat $TXNS_DIR/buyer_asa_optin.stxn $TXNS_DIR/buyer_sc_optin.stxn > $TXNS_DIR/combinedtransactions.txn
goal clerk group --infile $TXNS_DIR/combinedtransactions.txn --outfile $TXNS_DIR/groupedtransactions.txn
goal clerk sign --infile $TXNS_DIR/groupedtransactions.txn --outfile $TXNS_DIR/buyer_optin.stxn -w $WALLET
goal clerk rawsend --filename $TXNS_DIR/buyer_optin.stxn
```

Step 6 - Buy
```
export BUYER_AMOUNT_TOKENS=
goal clerk send --from=$BUYER --to=$SYSTEM_ACCOUNT --amount=$(($BUYER_AMOUNT_TOKENS*$PRICE+2000)) --out=$TXNS_DIR/buy_send.stxn -w $WALLET
goal app call --from=$BUYER --app-id=$SYSTEM_ID --app-account $OWNER --foreign-asset $ASA --app-arg="str:buy" --out=$TXNS_DIR/buy_call.stxn -w $WALLET
cat $TXNS_DIR/buy_send.stxn $TXNS_DIR/buy_call.stxn > $TXNS_DIR/combinedtransactions.txn
goal clerk group --infile $TXNS_DIR/combinedtransactions.txn --outfile $TXNS_DIR/groupedtransactions.txn
goal clerk sign --infile $TXNS_DIR/groupedtransactions.txn --outfile $TXNS_DIR/buy.stxn -w $WALLET
goal clerk rawsend --filename $TXNS_DIR/buy.stxn
```
