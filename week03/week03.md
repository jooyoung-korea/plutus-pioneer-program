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

So now let's look at an example using this time range!

# A Vesting Example

<br />

Let's say one of our friends wants to buy some ADA for his nephew and nieces, and imagine you want the child to own the ADA but you only want the child to have access to ADA when he or she turns 18 or 21 or whatever. Using plutus this is very easy to implement a Vesting scheme like that.

Let's copy the `IsData` contract, and make new module called Vesting. This will be the first example of validator that actually uses the context! Let's try to implement our vesting idea. 

너가 이제 돈을 스크립트에 넣어두면, 너가 딱 지정해놓은 사람만 그 돈을 인출할 수 있게 되는데, 어떤 데드라인이 지나야만 인출할 수 있게 해야하는거야. 첫 단계로는 데이텀과 리디머의 타입에 대해 생각해보자. 



You put money into a script and only a dedicated person can retreive it, but only once a certain deadline has been reached. The first step is to think about the types of datum and redeemer, and datum in this case it makes sense to have exactly two pieces of information: *the beneficiary and the deadline*

데이텀은 말했듯이 두 가지 정보: 그 돈을 받을 사람(수혜자)와 아까 말한 데드라인 를 가지고 있으면 아주 말이 되는 거 같지. 그럼 이 타입을 정의를 해보자. 이쯤에서 읽어봐야했을 법한... [learnyouahaskell](http://learnyouahaskell.com/making-our-own-types-and-typeclasses#record-syntax)

Let's define this type then.

Let's call it `VestingDatum` datum, and make it a record type with those two fields. First, how do we identify the beneficiary? By his or her public key or to be more precise, public key hash! And then, we need the deadline that will be of type POSIXTime. We can derive a show instance, might be useful for debugging. (아직 이 deriving하는 거 뭔지 모름)

`unstableMakeIsData` 이 helper function하는게 뭐였는지 까먹음

<br />

```haskell
data VestingDatum = VestingDatum
    { beneficiary :: PubKeyHash
    , deadline    :: POSIXTime
    } deriving Show

PlutusTx.unstableMakeIsData ''VestingDatum
```
<br />

So the first argument is the datum, so that's now of type `VestingDatum`.

Do we need any information in redeemer? What do we need in order to check whether transaction is allowed to spent this script output. 이 트랜잭션이 올바른지 확인하기 위해(스크립트 아웃풋이 True인가?) 확인해야할 일이 뭐가 있지, 즉 리디머에는 어떤 정보가 들어있어야 할까. We need to know it's signed by the the beneficiary and we must know that it's only submitted after the deadline. But those two pieces of information are both contained in the transaction itself(datum에 들어있어서 괜찮다는 건가? 데이텀이 곧 스크립트 안에 있는 정보인건가 앞 강의부터 열심히 들을걸 ㅎㅎ) So we don't need anything in addition to that in redeemer, 어쨋든 스크립트에 필요한 정보 이미 다 있으니까 리디머에 들어갈 정보는 딱히 없으니까 유닛 해주자는 거지

<br />

```haskell
{-# INLINABLE mkValidator #-}
mkValidator :: VestingDatum -> () -> ScriptContext -> Bool
mkValidator dat () ctx = traceIfFalse "beneficiary's signature missing" signedByBeneficiary &&
                         traceIfFalse "deadline not reached" deadlineReached
```

<br />


So we just use () unit in redeemer, and now we need first argument, let's call it "dat" for datum, and the redeemer is just unit, and now we can't any longer ignore the context. So let's give it a name "ctx". 

We need to check two conditions: that the transaction is **signed by the beneficiary** and that it has been submitted or that the current time is **after the deadline**. So let's delegate the helper functions, let's just write the general structure. Let's say "beneficiary's signature missing" So this is the error message we get in case if it's false and let's call it by `SignedByBeneficiary`. The second condition, that the deadline has been reached, let's call it by `deadlineReached` and if deadline not reached, the error message would be "deadline not reached". And now of course this two `signedByBeneficiary` and `deadlineReached` should be defined, and for both, we will need `TxInfo`,

Let's first define that of type `TxInfo`

<br />

```haskell

mkValidator dat () ctx = traceIfFalse "beneficiary's signature missing" signedByBeneficiary &&
                         traceIfFalse "deadline not reached" deadlineReached

    where
        info :: TxInfo
        info = scriptContextTxInfo ctx


```


<br />

Let's import `Ledger` and ask for the information of `ScriptContext`, in order to get the `TxInfo` from the `ScriptContext`, we must use this field name txinfo `scripContextTxInfo`, so that `scripContextTxInfo` of our context, now we can define two conditions. Let's first do the `signedByBeneficiary` of type `Bool`, how do we check that? While we see repl again (below),  there's a function called `txsignedby`: it takes a `TxInfo` which we now have and `PubKeyHash` and then says whether that the transaction that `TxInfo` has been signed by this public key hash

<br />

![Untitled (5)](https://user-images.githubusercontent.com/60770121/126347023-49056b25-0bb9-4c5f-ba0e-d822a8e38b2e.png)

<br />

```haskell
where
    info :: TxInfo
    info = scriptContextTxInfo ctx

    signedByBeneficiary :: Bool
    signedByBeneficiary = txSignedBy info $ beneficiary dat

```
<br />

`txSignedBy` info, we just defined, now we need the public key hash of the beneficiary, and because we have the datum "dat" and this is of type `VestingDatum` and we have the beneficiary there, 그 우리가 beneficiary의 pubkeyhash를 "dat"이라고 이름 지어준 datum안에 정보가 있으니까 그걸 가져오자는 거

And the second condition, deadline reached, was of type `Bool`. So how do we check that? Let's first check diagram below.

<br />

![image](https://user-images.githubusercontent.com/60770121/127251752-dbd286fd-0556-4602-a43e-22ca433cb3b1.png)

<br />

The deadline is in the middle, and let's consider a transaction with **u**(upper) validity interval. So recall what that means before the validator script is run. 실제로 밸리데이터 스크립트가 돌기 전에 이 밸리디티 인터벌이 가지는 의미에 대해 생각을 해보자. (이런 vesting example외에도 밸리디티 스크립트가 도는 것과 밸리디티 인터벌이 어떻게 관계가 있어서 실제로 블록체인 노드에 트랜젝션이 submit되기 전에 fail하는 것에 도움을 줄 수 있다는 건지 - 디폴트는 to eternity인데) 

Other checks are made including the time check, so the node checks whether the current time falls into the valid range of the transaction and only then it's the validator run. 트랜젝션이 노드에 도착했을때, 노드가 현재 시간이 트랜젝션에서 정의?된 밸리드 인터벌 안에 들어가면, 벨리데이터가 돈다는 거지? 근데 이 벨리드 인터벌이 무한 시간이면 이게 의미가 없다는 거지? So we know now that we are in the validator, the current timelives somewhere in the validity interval. The first one **u** would be the validity interval. And we must not declare the transaction to be valid, because if the current time is after the deadline, it would be fine, but it could be before the deadline which would not be fine. This validity interval is not good for us. The **l**(lower) validity interval, on the other hand, is fine. We still don't know where the current time exactly is, it can be anywhere in this interval, but no matter where it is, it's after the deadline. 밑에 인터벌의 경우엔, 현재 시각을 정확히 몰라도 괜찮다 어차피 모든 구간이 데드라인에 대해 오른쪽에 위치하기 때문에 항상 성공할 것이기 때문임. So we must check that we have this situation that the whole interval is to the right of the deadline. 그래서 트랜젝션이 정의하는 밸리디티 인터벌이 데드라인 이후로 되어있는지 확인해야한다는 것이다. 그래서 근데 데드라인이 없는 스크립트에 대해선 밸리디티 인터벌이 먼 소용이란건지 아직 이해하지 못했음. And one way to do is to use `contains` function that we played earlier. So now we can define second condition, `deadlineReached`. We need the validity interval of the transaction and that if we look at the information of the `TxInfo`, then this was called `txInfoValidRange` of the info.

<br />

```haskell
where
    info :: TxInfo
    info = scriptContextTxInfo ctx

    signedByBeneficiary :: Bool
    signedByBeneficiary = txSignedBy info $ beneficiary dat

    deadlineReached :: Bool
    deadlineReached = contains (from $ deadline dat) $ txInfoValidRange info

```

<br />

Next let's look at below dummy type that wraps up datum and redeemer type (dummy 타입이 뭐지?)
Typed is not a good name. Let's call it Vesting, and the datum type is now our `VestingDatum` type, and Redeemer type is just a unit.

<br />

```haskell
data Vesting
instance Scripts.ValidatorTypes Vesting where
    type instance DatumType Vesting = VestingDatum
    type instance RedeemerType Vesting = ()

```
<br />

And this is just boiler plate.

<br />

```haskell
typedValidator :: Scripts.TypedValidator Vesting
typedValidator = Scripts.mkTypedValidator @Vesting
        $$(PlutusTx.compile [|| mkValidator ||])
        $$(PlutusTx.compile [|| wrap ||])
    where 
        wrap = Scrips.wrapValidator @VestingDatum @()

validator :: Validator
validator = Scripts.validatorScript typedValidator

valHash :: Ledger.ValidatorHash
valHash = Scripts.validatorHash typedValidator

scrAddress :: Ledger.Address
scrAddress = scriptAddress validator

```
<br />

Now, we also have to change off-chain part, but because the focus of this lecture is on-chain validation. 
We just briefly go through it. 

<br />

### Off-Chain part

`VestingSchema` difines the endpoints that we want to expose to users. So "give" is for the person that wants to set up this vesting contract, and then "grab" for the beneficiary to collect. 

<br />


```haskell
type VestingScheme = 
            Endpoint "give" GiveParmas
        .\/ Endpoint "grab" ()

```
<br />

So what parameters do we need?

For **"give"**, what this endpoint will do is, it will **create a UTxO** at the vesting address with the specified amount and the correct datum. 
And if you recall our vesting datum contains the beneficiary and the deadline. So this "give" endpoint must know those, also have to specify how much money we want to put at this UTxO, so we need the third field `gpAmount`

<br />

```haskell

data GiveParams = GiveParams
    { gpBeneficiary :: !PubKeyHash
    , gpDeadline    :: !POSIXTime
    , gpAmount      :: !Integer
    } deriving (Generic, ToJSON, FromJSON, ToSchema)

```
<br />

The "grab" endpoint on the other hand, doesn't need any parameters because the beneficiary will just look for UTxOs sitting at the vesting address and then can check whether he is beneficary, and whether the deadline passed, and then pick those UTxOs and then consume them.

<br />

Let's briefly look at the first endpoint: **give**
<br />

```haskell
give :: AsContractError e => GiveParams -> Contract w s e ()

```
It takes the give parameters `GiveParams` and first I compute the datum I want to use. So recall I need the beneficiary and the deadline, then I can get those simply from reading it from the give parameters now for the transaction. 

```haskell
give :: AsContractError e => GiveParams -> Contract w s e ()
give gp = do
    let dat = VestingDatum
                { beneficiary = gpBeneficiary gp
                , deadline = gpDeadline gp
                }
        tx = mustPayToTheScript dat $ Ada.lovelaceValueOf $ gpAmount gp

```
I need as constraints that I want to create a transaction that has an output at this script address. 이 스크립트 어드레스에 아웃풋을 주는? 먼소리야 트랜젝션을 만들고 싶다는 컨스트레인을 추가해보자. That's why I use this `mustPayToTheScript` with the data, my justified, and then I must provide the value. 그 어마운트를 정의해줘야한다는 얘기하는건가? 얼마줄지 양을 러브레이스로 지정해줘야 한다는 건가 그래서 mustPayToTheScript이라는 함수는 그 제약을 걸어주는 건가? 지정안해주면 에러나고 지정해주면 트랜젝션에 추가되는? So I take the value, the amount from the give parameters, and then use `Ada.lovelaveValueOf` to convert that to a value in lovelace. So I mention the amount, the beneficiary, and the deadline.

<br />

Now for the grab, that's a bit more involved because there can be many UTxOs at this vesting address. And some of them might not be suitable for me, either because I am not the beneficiary or because the deadline has not yet passed. 아니 데드라인이나 밸리드 인터벌은 시간이 지남에 따라 바뀌는건가? So I grab the current time, 

```haskell
grab :: forall w s e. AsContractError e => Contract w s e ()
grab = do
    now <- currenttime
```
and I lookup my own pulic key and computes hash, 
```haskell
    pkh <- pubKeyHash <$> ownPubKey
```
and then I look at all UTxOs sitting at the script address, but I filter those and the filter is defined below. 스크립트 어드레스에서 펍키해시가 이 vesting스킴을 셋업한 사람이 지정해놓은 것과 일치하는 utxo를 찾겠다는 거지?
```haskell
    utxos <- Map.filter (isSuitable pkh now) <$> utxoAt scrAddress
```
So I get such a UTxO and then I first check the datum hash and if I find it this H, then I try to lookup the correspoinding datum and recall what I said several times, before the producing transaction

```haskell
    where 
        isSuitable :: PubKeyHash -> POSIXTime -> TxOutTx -> Bool
        isSuitable pkh now o = case txOutDatumHash $ txOutTxOut o of 
            Nothing -> False
            Just h -> case Map.lookup h $ txData $ txOutTxTx o of Nothing 

```

the producing transaction, so in this case, the one that has been produced by the **"give"** endpoint, doesn't necessarily contain the datum, it can **only contain the datum hash?**, but if that was the case, then we would have a problem here in "grab", We wouldn't be able to find the datum, that corresponds to the UTxO that we find. 블럭이 utxo를 컨슘 하고 이제 새로운 utxo들을 produce하는데, 이 예제에 대해서 우리는 give 엔드포인트에서 데이텀 말고 데이텀 해시만 가지고 있어도 되는거냐, 근데 만약 그렇게 되면, grab할 때 우리는 데이텀을 찾을 수 없을 거란 말인데 But by using this `mustPayToTheScript` constraint, I make use of the option to include the datum into the transaction. So this transaction will optionally contain the datum. off-chain 코드에 이 mustpay 제한을 줌으로써, 직접적으로 트랜젝션이 데이텀을 옵셔널하게 가질수도 있게끔 랄스가 구현해놓았다는 말 

So if all of this succeeds, then I can check whether the beneficiary of the datum is myself and whether the deadline doesn't lie in the future. So at this point, the UTxOs are only UTxOs sitting at this vesting address where I am the beneficiary and where the deadline has been reached. And if I find no such UTxOs, I will just get a log message that nothing is availabe and don't do nothing. 

(-)

Recall, if a transaction wants to spend a UTxO sitting at the script address, the script must be provided by this spending transaction. 트랜젝션이 script address에 있는 utxo를 사용하고 싶으면, 그 스크립트는 spending transaction에 의해서 제공되야한다. So this `lookups` there takes care of that. Always, the time range it starts and genesis and goes on forever. `mustValidateIn (from now)`. So if I don't do anything, I would use that. And that wouldn't be good because then validation would fail, because recall validation checks that the deadline has passed. And it only knows that the current time lives in this validity interval. But it doesn't matter ~ . And here I picked `(from now)`, it's just important that everything interval that I provide here is after the deadline. But so I could, for example, just use the Singleton interval now, but then I would have a problem if the transaction doesn't get there in the same slot to the node for validation. So if it only reaches their slot later, then the current time at that point would already not be in this interval anymore, which only contains now. So the best is to make it as large as possible. So I started now, but then get it last into eternity and the rest is same as before, and then I just bundled them up in this endpoints contract and some boiler plate for the playground.

<br />

## Playground

<br />

What is the public key hash of wallet 2 and of wallet 3? There's a way to find out! Let's start repl, 
```haskell
cabal repl
```

load our module, 
```haskell
:l src/Week03/Vesting.hs
```
and import  `Ledger` and import `Wallet.Emulator`.
```haskell
import Ledger
import Wallet.Emulator

```
Now, first of all, we have a wallet type and we see wallet it's just a newtype wrap around integer. 

        :i Wallet 
        type Wallet :: *
        newtype Wallet = Wallet (getWallet :: Integer)

Secondly, we have something called wallet pub key, given a wallet gives us a pub key

        :t walletPubKey
        walletPubKey :: Wallet -> PubKey

And finally, we have pub key hash, given a pub key gives us the pub key hash.

        :t pubKeyHash
        pubKeyHash :: PubKey -> PubKeyHash


So, putting all of these together, we can do pub key hash of wallet pub key of wallet two.

```haskell
pubKeyHash $ walletPubKey $ Wallet 2
```
So let's grab the pub key hash of wallet two and put it in the first two gives, and grab the pub key hash of wallet three and put it in the third give. Next problem is the deadline. In last lecture, I showed you how to convert between slots and deadlines or POSIXTimes. But unfortunately, this has changed in the meantime. If you go back to the repl and import ledger time slot, then there's a function slot to begin, begin POSIX time.

        import Ledger.TimeSlot

        :t slotToBeginPOSIXTime
        slotToBeginPOSIXTime :: SlotConfig -> Slot -> POSIXTime


And this now takes a `SlotConfig` and the `Slot` to give us a `POSIXTime`, by the way, there are versions with begin and end. This is because a slot has, it's not just a point in time, it's a duration in time. So it has a beginning and an end, so these two are for that, So what's `SlotConfig`?

        :i SlotConfig

That contains the slot length and the zero slot time so that POSIXTime at which zero starts, so now we have to find out what slot config to use for the playground. Luckily it's the default slot config. What's the default slot config? For that we need to import yet another module, `Data.Default`. And that defines a class called default, lots of instances.

If you check `SlotConfig` again, then now that we imported `Data.Default`, we see indeed it is an instance of default. So by using `def` and telling the compiler what type we're interested in, we get the default `SlotConfig`. So we see slot length is 1000. So that means POSIXTime is obvs in milliseconds. So the 1000 is one second and we see when in the playground slot zero starts. So now we can use the slot to begin POSIX time and can use `def` as our `SlotConfig`. And now we can, for example, see what POSIX time slot 10 begins and slot 20.

        def :: SlotConfig
        
        slotToBeginPOSIXTime def 10 
        slotToBeginPOSIXTime def 20


<br />

(50m - to be continued...) 
the wallet code in a way that only submits a transaction, if it can actually grab the UTxO. If you want to actually test the validator, you would 
have to modify the wallet code. -> Option homework! So remove these checks in the wallet and just try to grab everything, and then it should fail if you are not the beneficiary or if the deadline has not yet been reached, and then it should fail you in validation, not already just in the wallet, not even submitting the transaction. 

We made use of context for the first time, and where we made use of this valid time interval in the trasnsaction to implement deadline. 

### Parametrized

Parametrised Contract

We can write the contract where the Script itself already contains this information: beneficiary and deadline


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
