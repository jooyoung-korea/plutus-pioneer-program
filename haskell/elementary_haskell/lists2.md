타입 클래스 안에 타입이 있는 거구나 `Num`은 타입클래스, `Integer`는 타입!




연습문제

다음 함수들을 작성하고 시험하라. 타입 시그너쳐를 잊지 말 것.

    takeInt는 리스트의 처음 n개 항목을 반환한다. 즉 takeInt 4 [11,21,31,41,51,61]는 [11,21,31,41]을 반환한다.


    dropInt는 리스트의 처음 n개 항목을 버리고 나머지를 반환한다. 즉 dropInt 3 [11,21,31,41,51]은 [41,51]을 반환한다.


    sumInt는 리스트 내 항목들의 합을 반환한다.


    scanSum은 리스트 내 항목들을 더해 중간 합들을 담은 리스트를 반환한다. 즉 scanSum [2,3,4,5]은 [2,5,9,14]을 반환한다.


    diffs는 인접 항목들의 차이의 리스트를 반환한다. 즉 diffs [3,5,6,8]은 [2,1,2]을 반환한다. (힌트: 두 리스트를 취해 대응하는 항목들의 차를 계산하는 보조 함수를 작성한다. 또다른 방법은 인자가 적어도 둘인 리스트는 (x:y:ys) 패턴에 일치한다는 사실을 활용하는 것이다.

처음의 세 함수는 Prelude에 take, drop, sum이란 이름 하에 들어있다.

<br />

1번)

```haskell
takeInt :: Integer -> [Integer] -> [Integer]
takeInt 0 _ = []
takeInt n (ms:me) = head (ms:me) : takeInt (n-1) tail (ms:me) 
```

        takeInt 4 [11,21,31,41,51,61]

        11 : takeInt 3 [21,31,41,51,61]
        11 : 21 : takeInt 2 [31, 41, 51, 61]
        11 : 21 : 31 : takeInt 1 [41,51,61]
        11 : 21 : 31 : 41 : takeInt 0 [51, 61]
        11 : 21 : 31 : 41 : [] 
        [11, 21, 31, 41]

<br />

2번) 

```haskell
-- dropInt 3 [11,21,31,41,51]은 [41,51]
dropInt :: Integer -> [Integer] -> [Integer]
dropInt 0 (s:e) = (s:e) -- 뒤에 리스트를 그대로 반환하려고 하면 일케 하면 되나?
dropInt n (ms:me) = dropInt (n-1) tail (ms:me) -- 여기서 생각하면 좋은게 tail (ms:me) 가 바로바로 그냥 "me"라는 사실 
```

        dropInt 3 [11,21,31,41,51]

        dropInt 2 [21,31,41,41,51]
        dropInt 1 [31,41,51]
        dropInt 0 [41,51]

<br />

3번)

```haskell
-- sumInt는 리스트 내 항목들의 합을 반환한다.
sumInt [Integer] -> Integer
sumInt [] = 0
sumInt (s:e) = head (s:e) + sumInt tail (s:e) 
```
        sumInt [1,2,3] = 1 + sumInt [2,3]
        = 1 + 2 + sumInt [3]
        = 1 + 2 + 3 + sumInt []
        = 6

<br />

4번)

```haskell
-- scanSum [2,3,4,5]은 [2,5,9,14]을 반환한다.
scanSum :: [Integer] -> [Integer]
-- 흠 위에 만든 함수들을 helper function으로 사용안하고 direct한 답을 잠깐 참고해봄
scanSum [] = []
scanSum [x] = [x] -- (x:y:xs) 는 인자가 적어도 2개 이상일 경우에 쓸 수 있는 표현이다. 
scanSum (x:y:xs) = x: scanSum ((x+y):xs)  -- (괄호) 넣는 거 항상 주의
```

        scanSum [2,3,4,5] = 2: scanSum [5,4,5]
        = 2:5: scanSum [9,5]
        = 2:5:9: scanSum [14]
        = 2:5:9: 14

<br />

5번) 


```haskell  
d_helper :: [Integer] -> [Integer] -> Integer -- [3,5,6] - [5,6,8] 이런식으로 하면 될 듯한데, 
d_helper _ [] = []
d_helper (x:xs) (y:ys) = abs(x-y) : d_helper xs ys

diffs :: [Integer] -> [Integer]
diffs (x:xs) = d_helper (x:xs) xs 

```
        diffs [3,5,6,8]
        = d_helper [3,5,6,8] [5,6,8]
        = 2 : d_helper [5,6,8] [6,8]
        = 2 : 1 : d_helper [6,8] [8]
        = 2 : 1 : 2 : d_helper [8] []
        = 2 : 1 : 2 : []
        = [2, 1, 2]


---

<br />


:: 이렇게 첨에 정의하는 걸 타입 시그니처라고 하는 구만 

함수 자체를 *인자*로도 쓸 수도 있을까?
뭔소리지... 하하하하 이게 지금 방금 "인자 생략" (point-free)랑 상관있다는 거가?

```haskell
applyToIntegers :: (Integer -> Integer) -> [Integer] -> [Integer]
applyToIntegers _ [] = []
applyToIntegers f (n:ns) = (f n) : applyToIntegers f ns 
```
그래서  `applyToIntegers`는 `f :: Integer -> Integer`을 사용하는 함수인데, f의 결과가 리스트 안에 들어간게 세번째에 있는 [Integer] 이란건가? 이상한데 이상해 뭔가 이상해 아아아아 결국 그럼 `applyToIntegers`는 `:: Integer [Integer] -> [Integer] 인 함수라는 거지! 아핫 이제 이해했어 

쉬운 예로 `(*)` 는 `Integer -> Integer` 인 함수라고 생각할 수 있겠지. 그럼 이제 리스트의 각 원소에 어떤 숫자를 곱하는 함수를 만들어보자. 

```haskell
multiplyList :: Integer -> [Integer] -> [Integer]
multiplyList m = applyTo
```


### map 함수 

<br />

```haskell
map :: (a -> b) -> [a] -> [b]
map _ [] = []
map f (x:xs) = (f x) : map f xs
```
map ((*) m)  여기서 m은 리스트인 거지?

이면 (a -> b) 에 해당하는 부분이 * 이고 주어진 m이 세번째 인자인 [a]가 되는 셈인 것이군. 어쨋든 그럼 a랑 [a], b랑 [b]는 같은 타입이어야 하는건거 겠지?

그러면 
```haskell
multiplyList :: Integer -> [Integer] -> Integer
multiplyList m = map ((*) m)
```

그러면 map이라는 함수는 리스트의 모든 원소에 어떤 연산을 적용할 때 쓰는 것이겠구만?

근데 왜 

```haskell
a = [1,2,3]
ranf x = 2 * x + 3
map ((ranf) a) -- 일케 쓰면 안돼 (ranf a) 도 물론이고 ranf가 a를 받으면 안되니까, 스크립트에 빨간 줄 쳐져 있어도 ghci에선 아래처럼 하면 된다

map ranf a 
```

옹 리스트 안의 리스트들의 heads들만 가져오는 heads함수도 

```haskell
heads :: [[a]] -> [a]  -- 옹 그러네 그러네 
heads = map head
```

옹 그렇다 

**map은 한 함수를 리스트의 각 원소에 적용하는 이상적인 범용 해법이다.**

<br />

### 연습문제

1번) 
map을 이용하여, Int의 리스트 xs가 주어졌을 때 다음을 반환하라.

- xs의 원소들을 각기 반전한 리스트
- xs의 각 원소에 대한 인수들을 포함하는, Int의 - 리스트의 리스트 xss. 인수들을 구하기 위해 다음 함수를 사용할 수 있다.

            divisors p = [ f | f <- [1..p], pmodf == 0 ]
- xss의 원소별 반전

<br />

```haskell
xs = [1,2,3,4,7,11]
rev [] = () -- NoNo
rev (x:xs) = rev xs : x : [] -- NoNo

rev :: [Integer] -> Integer
rev m = rev tail m  --호호... map을 서야하는 건가? 리스트 반전에?? 

```
아... 여기서 말하는 반전이 1 -> (-1) 이었다.... 거기다 `Prelude`의 함수 `negate`를 사용하는 것이었음. 

그럼 간단하지. 

```haskell
xs = [1,2,4,6]
rev :: [Integer] -> [Integer]
rev m = map negate m 

```

다음 `divisors`는 흠 p를 입력하면 1에서 p까지 중에 `for f in range(1,p+1_`같은 느낌일까. 그 중에 p와 나눠떨어지는 걸 리스트로 받는 함수인 듯하다만. 

```haskell

```
