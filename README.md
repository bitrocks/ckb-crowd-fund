# RFC: CKB Crowdfund(WIP)

## Intro

Crowdfund is a on-chain application where particiapnts can deposit tokens toward a target raising amount before the expiration time, and withdraw their tokens if the goal is not reached. It can be building block for many other useful on-chain applications, like various ICO events.

## Workflow

1. Alice lauch a fund raising event, set the target amount, expiration time and beneficiary; 

2. Bob deposit certain amount of tokenA into the contract, and generate a receipt; other donors depost into the contract, too;

3. After expiration time, if the target amount is reached, the beneficiary can withdraw the tokens in contact, and bob can claim it's reward(like tokenB); if not, bob can withdraw his tokenA;

## Crowdfund lock script

1. parameters
    - Beneficiary's public key hash
    - Target amount fo tokens
    - Expiration time(block height or epoch)
    - optional: minimum deposit amount, token exchange rate
2. deposit conditions 
    - current block time &lt;= expiration time
    - deposit amount &gt;= minimum deposit amount, if the minimum deposit parameter is set
3. unlock conditions
    - for beneficiary, raised amount >= target amount, and the beneficiary should withdraw all the funds once for simplicity reason(continously withdraw should also be possible);
    - for donors, current block time &lt;= expiration time, the capacity of output crowd-fund cell should be bigger than the input crowd-fund cell, which means before the expiration time, donors can continuously deposit into the crowd-fund cell;
    - for donors, raised amount &lt; target amount && current block time &gt; expiration time, but the amount should be the same as they deposited into the pool;

## Crowdfund recipit type script

For every donation deposited into the crowd-fund cell, it should generate a recipit cell keeping the amount of the donation.

The recipit type script should validate:
Todo

## Transaction Examples

1. LauchCrowdfund

Anyone can lauch a crowdfund event, create a crowdfund cell and a recipit cell.

```
Inputs:
    Normal Cell:
        Capacity: 1100 CKB
        Lock: 
            code_hash: secp256k1_blake2b lock
            args: <PubKeyHash A>

Outputs:
    Crowdfund Cell:
        Capacity:  1000 CKB
        Lock: 
            code_hash: crowd-fund lock
            args: <PubKeyHash B> <CKB Target amount: 10000> <Expiraion block height: 3000000>
    Recipit Cell:
        Capacity: 99.99 CKB
        Lock:
            code_hash: secp256k1_blake2b lock
            args: <PubKeyHash A>
        Type:
            code_hash: crowd-fund-recipit 
            args:
        data: <deposit CKB amount: 1000>
        

Witnesses: 
    <signature for PubKeyHashA>
```

For ICO-like funding, it might contain a token exchange rate parameter in Crowdfund Cell:

```
// simple crowdfund event
args: <PubKeyHash B> <CKB Target amount: 10000> <Expiraion block height: 3000000> [<CKB minimum: 20>]

// ICO-like crowdfund event
args: <PubKeyHash B> <CKB Target amount: 10000> <Expiraion block height: 3000000> [<CKB minimum: 20> <Token exchange rate: 10>]
```

2. Deposit

Anyone can deposit to a crowd fund cell before expiration(if it's not a capped fund). After expiration, all deposit would be rejected no matter whether the target is meet or not.

```
Inputs:
    Normal Cell:
        Capacity: 200 CKB
        Lock:
            code_hash: secp256k1_blake2b lock
            args: <PubKeyHash C>
    Crowdfund Cell:
        Capacity: 1000 CKB
        Lock: 
            code_hash: crowd-fund lock
            args: <PubKeyHash B>
    
Outputs:
    Crowdfund Cell: 
        Capacity: 1100 CKB 
        Lock:
            code_hash: crowd-fund lock
            args: <PubKeyHash B> <CKB Target amount: 10000> <Expiraion block height: 3000000>
    Recipit Cell:
        Capacity: 99.99 CKB
        Lock:
            code_hash: secp256k1_blake2b lock
            args: <PubKeyHash C>
        Type:
            code_hash: crowd-fund-recipit type
            args: 
        data: <deposit CKB amount: 100>

Witnesses:
    <signature for PubKeyHashC>
```
3. Beneficiary withdraw

If the crowdfund target is meet before expiration, the beneficiary can withdraw all CKB from the crowdfund cell once.

``` 
Inputs:
    Crowdfund Cell:
        Capacity: 15000 CKB
        Lock:
            code_hash: crowd-fund lock
            args: <PubKeyHash B> <CKB Target amount: 10000> <Expiraion block height: 3000000>

Outputs:
    Normal Cell:
       Capacity: 14999.99 CKB 
       Lock:
            code_hash: crowd-fund lock
            args: <PubKeyHash B> <CKB Target amount: 10000> <Expiraion block height: 3000000>

Witnesses:
    <signature for PubKeyHashB>
```

4. Refund

If the crowdfund target is not meet after expiration, donors can perform a refund event, withdrawing their donations by submitting the recipit cell.

```
Inputs:
    Crowdfund Cell:
        Capacity: 9000 CKB
        Lock:
            code_hash: crowd-fund lock
            args: <PubKeyHash B> <CKB Target amount: 10000> <Expiraion block height: 3000000>
    Recipit Cell:
        Capacity: 99.99 CKB
        Lock:
            code_hash: secp256k1_blake2b lock
            args: <PubKeyHash C>
        Type:
            code_hash: crowd-fund-recipit type
            args: 
        data: <deposit CKB amount: 100>

Outputs:
    Crowdfund Cell:
        Capacity: 8900 CKB
        Lock:
            code_hash: crowd-fund lock
            args: <PubKeyHash B> <CKB Target amount: 10000> <Expiraion block height: 3000000>
    Normal Cell:
        Capcity: 199.98
        Lock: code_hash: secp256k1_blake2b lock
        args: <PubKeyHash C>

Witnesses:
    <signature for PubKeyHashC>
```

5. optional: ClaimToken

For ICO-like crowdfund event, participants would claim the token after the target amount is meet.

