## Example with wNEAR on testnet

#### Init

```bash
near call $CONTRACT_ID --accountId=$OWNER_ID new '{
  "locked_token_account_id": "wrap.testnet",
  "meta": {"spec": "ft-1.0.0", "name": "Future NEAR", "symbol": "fNEAR", "decimals": 24},
  "owner_id": "'$OWNER_ID'"
}'
```

#### Wrap 20 NEAR into wNEAR (optional)

```bash
near call wrap.testnet --accountId=$OWNER_ID storage_deposit '' --amount=0.00125
near call wrap.testnet --accountId=$OWNER_ID near_deposit --amount=20
```

#### Storage deposit

```bash
near call wrap.testnet --accountId=$OWNER_ID storage_deposit '{"account_id": "'$CONTRACT_ID'"}' --amount=0.00125
```

#### Lock 10 wNEAR into fNEAR

```bash
near call wrap.testnet --accountId=$OWNER_ID --depositYocto=1 --gas=100000000000000 ft_transfer_call '{
  "receiver_id": "'$CONTRACT_ID'",
  "amount": "10000000000000000000000000",
  "msg": ""
}'
```

#### View methods

```bash
near view $CONTRACT_ID get_info
near view $CONTRACT_ID ft_metadata
near view $CONTRACT_ID ft_total_supply
near view $CONTRACT_ID ft_balance_of '{"account_id": "'$OWNER_ID'"}' 
```

#### Storage for the new account

```bash
near call $CONTRACT_ID --accountId=$ACCOUNT_ID storage_deposit '' --amount=0.00125
```

#### Transfer 1 fNEAR from the owner

```bash
near call $CONTRACT_ID --accountId=$OWNER_ID --depositYocto=1 ft_transfer '{
  "receiver_id": "'$ACCOUNT_ID'",
  "amount": "1000000000000000000000000"
}' 
```

#### Check balance

```bash
near view $CONTRACT_ID ft_balance_of '{"account_id": "'$OWNER_ID'"}' 
near view $CONTRACT_ID ft_balance_of '{"account_id": "'$ACCOUNT_ID'"}' 
```

#### Attempt to transfer back (should fail, because not whitelisted)

```bash
near call $CONTRACT_ID --accountId=$ACCOUNT_ID --depositYocto=1 ft_transfer '{
  "receiver_id": "'$OWNER_ID'",
  "amount": "1000000000000000000000000"
}' 
```

Expected error:
```
'Not whitelisted for transfers'
```

#### Attempt to unwrap (should fail, still locked)

```bash
near call $CONTRACT_ID --accountId=$ACCOUNT_ID --depositYocto=1 --gas=100000000000000 unwrap ''
```

Expected error:
```
'The token is still locked'
```

#### Whitelist transfers for the account

```bash
near call $CONTRACT_ID --accountId=$OWNER_ID --depositYocto=1 add_transfer_whitelist '{
  "account_id": "'$ACCOUNT_ID'"
}'
```

#### Transfer of 0.1 by the account to the owner

```bash
near call $CONTRACT_ID --accountId=$ACCOUNT_ID --depositYocto=1 ft_transfer '{
  "receiver_id": "'$OWNER_ID'",
  "amount": "100000000000000000000000"
}' 
```

#### Unlock the unwrapping by the owner

```bash
near call $CONTRACT_ID --accountId=$OWNER_ID --depositYocto=1 unlock ''
```

#### Attempt to unwrap the token (should fail, no wNEAR storage)

```bash
near call $CONTRACT_ID --accountId=$ACCOUNT_ID --depositYocto=1 --gas=100000000000000 unwrap ''
```

Expected error:
```
'The account is not registered'
```

#### Verify balances didn't change

```bash
near view $CONTRACT_ID ft_balance_of '{"account_id": "'$OWNER_ID'"}' 
near view $CONTRACT_ID ft_balance_of '{"account_id": "'$ACCOUNT_ID'"}' 
```

#### Storage deposit for wNEAR

```bash
near call wrap.testnet --accountId=$ACCOUNT_ID storage_deposit '' --amount=0.00125
```

#### Unwrap the token

```bash
near call $CONTRACT_ID --accountId=$ACCOUNT_ID --depositYocto=1 --gas=100000000000000 unwrap ''
```

#### Verify balances of the account for fNEAR and wNEAR

```bash
near view $CONTRACT_ID ft_balance_of '{"account_id": "'$ACCOUNT_ID'"}' 
near view wrap.testnet ft_balance_of '{"account_id": "'$ACCOUNT_ID'"}' 
```
