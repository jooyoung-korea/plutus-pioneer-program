# Week03

### Learning Objective 
- Validator that actually looks at context
- Write Time sensitive logic
- Parametrised contract

<br />
<br />

In the first lecture, let's recall about EUTxO, in order to unlock the script address, the script attached to the address is run, and that script gets 3 pieces of input: the datum, the redeemer, and the context.

스크립트 어드레스를 언록한다는게 런한다는건가

And in the second lecture, we saw many examples of ~ how it actually works on Haskell. This low-level implementation where all three args are represented by **data type**, but Lars mention that, in practice, that is not used like we did, *instead of the type version*, **datum and redeemer can be custom type**,  and the third argument, **the context, is of type `ScriptContext`**, and the examples we've looked so far only look at the datum and the redeemer, but always ignored the context, and since the context is very important, in this third lecture, we start looking at the context, the `ScriptContext` type is defined in package plutus-ledger-api, which is a package we until now haven't need, now we need, so it is now included to week03 cabal file → *cabal build* 다시 해야하나? 

The context in this module [plutus.v1.ledger.contexts]([https://github.com/input-output-hk/plutus/blob/master/plutus-ledger-api/src/Plutus/V1/Ledger/Contexts.hs](https://github.com/input-output-hk/plutus/blob/master/plutus-ledger-api/src/Plutus/V1/Ledger/Contexts.hs)), the type ScriptContext, we see it is a record(?) type with two fields, one of type `Txinfo`, which is a more suggesting? one, the second field of type `ScriptPurpose` and this type is defined in the same module here, it describes for which purpose does this script is run, so the most important purpose for us will be  . **`Spending`** purpose and this is what we have talked about until now. 

In the context of the EUTxO model, this is *when script is run* in order to *validate spending input for transaction* but there are 3 other purposes. The second most important one is the **`Minting`** one, so that comes into play when you want to **define native token**, and then this purpose of the script will be to describe under which circumstances, the native token can be minted or burned, there are 2 more purposes: **`Rewarding`** and **`Certifying`** . **Rewarding** seems to be related to **staking**, and **certifying** to certificate, such as **delegation certificate**. 

First, let's concentrate on spending purpose. Now the actual context is in this field of type `TxInfo`, which is defined also in the same module here, this basically described, it's an abbrebiation for *transaction info*, so it describes the spending transaction. In the EUTxO model that Cardano uses, **the context of validation** is the **spending transaction** so the **transaction is inputs and outputs** and that so it is expressed in this `txinfo` type so there are couple of fields that's are basically global transaction and then, in particular, we have the list of all the inputs of the transaction `TxInInfo` and outputs of the transaction `TxOut` This `TxInInfo` type and `TxOut` type again have lots of fields to allow you to drill into each individual input respectively output.

`TxInfo` fields → the inputs take in and outputs the outputs, `txInfoFee` is the transaction fee which takes into fodge? is the amount of newly fodge native token, or if its negative amount newly burned, we'll come to that later lecture. Then we have a list to certificate, `txInfoDCert` sth like delegation certificate, staking withdrawals so withdrawals from rewards `txInfoWdrl`. `txInfoValidRange` denotes **time range that transaction will be valid** and we'll talk details in a minute.`txInfoSignatories` that's the list of **public keys** that have signed to this transaction. `txInfoData` is that the spending transactions that transactions spent the script outputs need to include the datum of that script outputs where producing transactions that sent money to script address have an output of the script address when we need to include the hash so this field takes **info data** is basically a dictionary **from datum hash** to data to include the full datum values belonging to given hash, also mentioned before, **spending transaction** always have to include **datum of the input that spent**, but the *producing transaction optionally can also do that*. `txInfoId` is the id of this transaction

This brings us to an interesting dilemma, I have stressed every time one of the big advantages that Cardano EUTxO model has over sth like ETH is the fact that **validation can happen in the wallet**, transaction can still fail, because a transaction can consume an input when transaction arrived on the blockchain at the node for validation has already been consumed by somebody else but in that case, the transaction simply fails without having to pay fee what can never happen on the normal circumstances is validation script runs and then fails because you can always run script under exactly the same condition, so it would fail before you ever submitted, and that is very important feature. It is not clear how to manage time in that context, because time is important because we want to be able to express validation logic that says that certain transaction is valid after only certain time has been reached.

In Auction example, 

auction → bids are only allowed until deadline has been  reached close endpoint can only be called after the deadline passed . you see the contradiction, bec time is obviously flowing, we you trying to validate a tranx that you r constructing in your wallet, that time that you do that in your wallet can of course be different to the time that the tranx arrived to node for validation. so it;s not clear how to bring both together in one hand, but on the other hand guarantee that a validation is deterministic if and only if succeed in the wallet and it will also succeed the node. the way cardano ~ that is by adding this txinfovalidrange to tranx what that specified so we will look at that type but basically it gives valid time "interval" this transaction is valid between this time and that time, and that is specified in the tranx and now when tranx sent(submitted) to the blockchain, and validated by the node and before any script is run, some general checks are done, all inputs are present, and balance enough and so on fees are included, and one of those check is that happens before validation is time range check. one of these pre check before validation is node checks the current time and compare it to time range specified in transaction. and if the current time does not fall into this time range, then validation failed immediately, without ever running the validatior script. what this also means if this pre check succeed then,we can assume that current time does fall into this interval and the validation became deterministic again this is just static piece of data attached to the tranx so the result of the validation does not depend on when it is run whether is run in the wallet before submission or in one of the nodes when validating the transaction and this makes it possible to resolve this apparant contradiction between having deterministic validation on the one hand, and takin time into account other hand. do the time check before validation is run and during the execution of the validation script. we don't have to worry about time does fall into time interval. if it wouldn't validation would nt even run and transaction failed before. 

default all transaction such as data loss? use infinite time range, last for eternity such transaction will always be valid. only exception is like auction example there's one slight complication is that 우로스 consensus protocaol powering Cardano doesn't use posix time it uses slots so it counts slot,  each slot is logery? happen does where slot is detemined and produce block and so one. and slot is native measure of time of Cardano but plutus uses real time. so some of these have to converted between real time and slot and this is no problem if slot length is fixed, now it's 1 sec. easy to go back and forth betwen real tim eand slot number. but it could be changed in future. (slot time) 

we don't know what slot len will be in 10 yrs. 

and that means that slot divert?→ interval specified in tranx must have definite upper bound, that is two find in future. it must only possible to know what slot len will be and that happens to be like 36 hrs. we know what the slot len will be in 36 hrs. so that is sth to keep in mind,  iyou can't sepnt arbitarry time rnage, it must only be at most 36 hrs in advance, or it can be indefinite. ?? 15분쯤

그냥 우로보스랑 타임 그게 달라 복잡해 보일 수 있다는 거임 real time쓰는게 편할 텐데 but in order to translate, you have to be careful when you submit the transaction, sconstruct (15분) transaction and said the slot interval or ime interval

let's look at the `POSIXTimeRange` it's defined in plutus.v1.ledger.time and it's just a time synonym for interval posix time let's look at the interval time. it's type parameter and specified by lower bound and upper bound, lower bound Construct (Extended a) Closure let's first look at closure → whether a bound is included in interval or not, let's look at the extended, Neg Inf, Pos Inf, 

Let's get to know couple of convenient functions 

`member :: Ord a => a-> Interval a -> boo`

Finite a Given a is included in interval a, provided a has ord constraints, its instance of the ord class, you can compare one is greater, less or equal to the  others, POSIX is instance of ord as slots

`interval :: a -> a -> Interval a`  gives us interval with lower(a) and upper bound(a) included 

and we have from and to to construct interval that starts at *a* and lasts until eternity (from) and "to" to interval stops at *a* 

`from :: a -> Interval a`

`to :: a -> Interval a`

always is interval that is used as default, that includes all time

`always :: Interval a`

never is the empty interval 

`never :: Interval a`

singleton only contains one a 

`singleton :: a -> Interval a`

`hull :: Ord a => Interval a -> Interval a -> Interval a`

'hull a b' is the smallest interval containing a and b (2 given intervals_

`intersection :: Ord a => Interval a -> Interval a -> Interval a`

'Intersection a b' is the largest interval that is contained in a and in b, if it exists (intersection literal)

`overlaps :: (Enum a, Ord a) => Interval a -> Interval a -> Bool`

Check whether two intervals have an element in common (overlap, that is, whether there is a value that is a member of both intervals.)

`contains :: Ord a => Interval a -> Interval a -> Bool`

checks whether one interval is contained in other (second one must be included in the first one)

a contains b `is true` if the Interval b is entirely contained in a. That is, a contains b if for every entry s, if member s b then member s a

`isEmpty :: (Enum a, Ord a) => Interval a -> Bool`

Check if an Interval is empty

`before :: Ord a => a -> Interval a -> Bool`

check if a value is earlier than the beginning of an Interval 

whether the given time is before everything in this interval after is the opposite, 

`after :: Ord a => a -> Interval a -> Bool`

check if a value is earlier than the beginning of an Interval 

These are the conv functions to construct strict lower and upper bound, let's play for a bit. 

we can play interval a bit 

so let's start a repl

![Untitled](https://user-images.githubusercontent.com/60770121/126346234-8ae18be5-e6c3-4449-ac73-77b26880d122.png)

let's import Plutus.V1.Ledger.Interval 

and for example, we can use this interval smart constructor, and as a lets just use integer for simplicity, to for example the, interval between 10 and 20, you can see that it says lowerbound is finite 10, 10 is included and upper bound is finite 20, 20 is included, so for example we can now check with `member`  with a 9 is member of this interval? but 10 should be so should be 12 and 20, but 21 shouldn't. 

![Untitled (1)](https://user-images.githubusercontent.com/60770121/126346485-aa3faf91-668a-4700-9689-b1ffff3a6771.png)

let's use the `from` so we have `from (30 :: Integer)` 21 is not included in this, 30 should be, everything larger than 30 should also be, let's try `to` so `member 30 $ to (30 :: Integer)` 32 shouldn't be and lower than 30 as well (should be)

let's try one more, `intersection` this one, we can get intersection from 10 to 20 and from 18 to 30, and as expected, we can get from 18 to 20. let's also try `contains` a $ b → b is fully contained in interval a, and `overlaps`

because for exa, 40 is contained interval a and b 

![Untitled (1)](https://user-images.githubusercontent.com/60770121/126346485-aa3faf91-668a-4700-9689-b1ffff3a6771.png)

- What does $ mean in Haskell?

`$` is infix "application" and it is defined as 

```jsx
($) :: (a -> b) -> (a -> b)
f $ x = f x

```

[http://snakelemma.blogspot.com/2009/12/dollar-operator-in-haskell.html](http://snakelemma.blogspot.com/2009/12/dollar-operator-in-haskell.html)

![Untitled (3)](https://user-images.githubusercontent.com/60770121/126346844-5ea97b33-fd70-479d-9df2-82d88d7f2458.png)

so now let's look at an example using this time range

one of lars' colleauge at io bought some ADA for his nephew and nieces and of course lars don't know about their details, imagine you wantna make gift of ADA to child so you want the child to own the ADa but you only want the child to have access to ADA when he or she turns 18 or 21 or whatever, and using plutus this is very easy to implement a scheme like that, vesting scheme like that, 

he copied the code from IsData contract and made new module called vesting . as our first example of our validator that actually uses the context, he implemented vesting idea so you put money into a script only a dedicated person can retreive it  ,but only once a certain deadline has been reached, so the first step is to think about the types of datum and redemmer and datum  in this case it makes sense to have exactly two pieces of information: beneficiary and the deadline

so let's define this type

let's call it vesting datum, director?- type with two fields, beneficiary of type how do we identify the beneficiary by his or her pub key or pricisely pubkeyhash,  and than we need the deadline which type should be posix type derving Show instance might be useful in debugging → ? didn't get it

PlutusTx. unstableMakeISData → 왓for? 기억안남 

```haskell
data VestingDatum = VestingDatum
    { beneficiary :: PubKeyHash
    , deadline    :: POSIXTime
    } deriving Show

PlutusTx.unstableMakeIsData ''VestingDatum
```

so the first argument is the datum so let's now offer vestingdatum

so for redeemer do we need any information in redeemer? what do we need in order to check whether transa is allowed to spend this script output we need to know it's signed by the the beneficiary and we must know that it's only submitted afte the deadline. but those two pieces of information are both contained in the tranx itself so we don't need anything in addition to that  in redeemer, 

```haskell
{-# INLINABLE mkValidator #-}
mkValidator :: VestingDatum -> () -> ScriptContext -> Bool
mkValidator dat () ctx = traceIfFalse "beneficiary's signature missing" signedByBeneficiary &&
                         traceIfFalse "deadline not reached" deadlineReached
```

so we just use () unit in redeemer, so first arg now we needed, "dat" for data and redeemer as unit, and ignore the context , just put it with name, now we need to check two conditions: that the tranx signed by the beneficiary and that it submitted current time is after the deadline, so let's delegate the hyperfunction just write structure , "beneficiary's signature missing" would be error message, let's call this condition, sth by signedByBeneficiary and the second condition, deadline be reached, of course this two signedByBeneficiary and deadlineReached should be defined, and both we will need TxInfo, and that here in the repl,

![Untitled (4)](https://user-images.githubusercontent.com/60770121/126346952-10a96274-3614-4e2d-8daa-adfd293e58ae.png)
 imported Ledger and asked for the information of script context again, in order to get the txinfo from the script context we must use this field name txinfo `scripContextTxInfo` of our ctx (context) now we can define two conditions, let's first do signedByBeneficiary signed to Bool, how do we check that? While we see repl again,  there's a function called txsignedby `:t txSignedBy` it takes txinfo which we now have and PubKeyHash and wether  that tranx that txinfo refers to has been signed by this pulibc key hash, 

![Untitled (5)](https://user-images.githubusercontent.com/60770121/126347023-49056b25-0bb9-4c5f-ba0e-d822a8e38b2e.png)
```haskell
where
    info :: TxInfo
    info = scriptContextTxInfo ctx

    signedByBeneficiary :: Bool
    signedByBeneficiary = txSignedBy info $ beneficiary dat

    deadlineReached :: Bool
    deadlineReached = contains (from $ deadline dat) $ txInfoValidRange info // we can get deadline from dat, txInfoValidRandge of info
```

txSignedBy info we just defined, now we need the pub key hash of the beneficiary, and that we have we have the datum "dat" and this is of type vestingdatum and we have the beneficiary, that's our first condition, 

now we also have deadline condition, tranx validity interval before the validator script is run, other checks are made, including the time check, so teh node check , whether current time falls into the valid range of the tranx. now we're in the validator current time lies somewhere in the validity interval, so the above one can be fine or not fine if the current time can be before / after the deadline, so it's not good for us, we must declare the tranx to be valid if this is validity interval, below one is just fine. we must check these interval that all interval is right to the deadline, we can use `contains` function here. if contains (from $ deadline dat → we can get from datum) and now we need the validity interval of the tranx, when we look at the txinfo again, there's txinfovalid range(above)  of the info

let's look at this dummy type that wraps up datum and redeemer type
Typed is not a good name. Let's call it Vesting, Datum type is now on Vesting Data type, and Redeemer type is just a unit


```haskell
data Vesting
instance Scripts.ValidatorTypes Vesting where
    type instance DatumType Vesting = VestingDatum
    type instance RedeemerType Vesting = ()

typedValidator :: Scripts.TypedValidator Vesting
typedValidator = Scripts.mkTypedValidator @Vesting
    $$(PlutusTx.compile [|| mkValidator ||])
    $$(PlutusTx.compile [|| wrap ||])
where 
  wrap = Scrips.warpValidator @VestingDatum @()

validator :: Validator
validator = Scrips.validatorScript typedValidator

valHash :: Ledger.ValidatorHash
valHash :: Scripts.validatorHash typedValidator

scrAddress :: Ledger.Address
scrAddress = scriptAddress validator
```

Now, we also have to change off-chain part, but as before, because the focus of this lecture is on-chain validation. 
We just briefly go through it. At the top of the module, added some additional GHC extensions, etc 


(32m) - off-chain part
`VestingSchema` difines the endpoints that we want to expose to users. So "give" is for the person that wants to set up this vesting contract, 
and then "grab" for the beneficiary to collect. So what parameters do we need?

For "give", what this endpoint will do is, it will create a UTxO at the vesting address with the specified amount and the correct datum. 
And if you recall our vesting datum contains the beneficiary and the deadline. So this "give" endpoint must know those,

```haskell

data GiveParams = GiveParams
    { gpBeneficiary :: !PubKeyHash
    , gpDeadline    :: !POSIXTime
    , gpAmount      :: !Integer
    } deriving (Generic, ToJSON, FromJSON, ToSchema)

```

The "grab" endpoint on the other hand, doesn't need any parameters because the beneficiary will just look for UTxOs sitting at the vesting address and then 
can check whether he is beneficary, and whether the deadline passed, and then pick those UTxOs and then consume them.

(35m) - grab off-chain
the producing transaction, so in this case, the one that has been produced by the **"give"** endpoint, doesn't necessarily contain the datum, 
it can **only contain the datum Hash?**, but if that was the case, then we would have a problem here in "grab", We wouldn't be able to find the datum, that corresponds to the UTxO that we find. 


(50m) 
the wallet code in a way that only submits a transaction, if it can actually grab the UTxO. If you want to actually test the validator, you would 
have to modify the wallet code. -> Option homework! So remove these checks in the wallet and just try to grab everything, and then it should fail if you are not the beneficiary or if the deadline has not yet been reached, and then it should fail you in validation, not already just in the wallet, not even submitting the transaction. 

We made use of context for the first time, and where we made use of this valid time interval in the trasnsaction to implement deadline. 

### Parametrized

So the idea of parametrized scripts is that you can have a parameter. So depending on the value of the parameter you get different values of typed validator.
So instead of just defining one script, define a family of scripts that are parametrized by a given parameter.

Since all the information we needed until now in the datum is now contained in this param `VestingParam`
We can simply use () type as datum


(55m) - mkValidator p -> it's not at compile time so...
if we make the type of P in instance of a type class called lift, that does work at runtime. So using life code, we can compile P at runtime to plutus core...

briefly look at `lift` class

`lifeCode` is exactly what we need. 

instead of using `mkValidator p` -> apply plutustx.lifecode ~ 



### Homework

HW1
you make a gift to beneficiary one, but beneficiary one only has time till the deadline to retrieve it, otherwise you can "get it back" if you self are beneficiary too. 
So your task is just simply write the correct logic. I need two diff tranx bc the validation interval would be diff. 

HW2
revisit the vesting, the original vesting, and remember we had two versions, the unparametrized version where the datum carried both beneficiary and deadline, and then a parametrized version, both turned to unit and ~ , so for this hw, you to do another parametrized version where we split beneficiary and deadline between parameter of the contract and datum. So the signature of the validator, make validator function will be like .. Split like `PubKeyHash -> POSIXTime -> () -> ctx -> Bool`: grab doesn't need any param, 
