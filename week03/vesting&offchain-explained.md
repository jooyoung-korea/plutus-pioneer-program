
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
