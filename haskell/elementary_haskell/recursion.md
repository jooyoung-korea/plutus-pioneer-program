# 1. 재귀

재귀함수의 *의미*와 *행동*을 구분해서 생각해보자? Generally, 함수의 정의의 일부에 그 함수 자체가 포함되면 그 함수를 "재귀적" 이라고 한다. 그리고 그 함수를 호출하지 않아 종료 조건을 정의하게 되는 *기본가정base case*이 적어도 하나 있어야 한다. 그렇지 않으면 재귀 함수는 무한 루프를 돌게 된다. 

## 수치 재귀

<br />

### 계승함수 

팩토리얼 함수를 만들어보자 

```haskell
let { factorial 0 = 1 ; factorial n = factorial (n -1) * n}

ghci> factorial 3
6
```
여기서 또 중요하게 생각해야하는 점은 가장 구체적인 조건 부터 (모든 경우를 만족하는 조건부터 점점 더 일반적인 조건들 순으로) 나열해줘야한다는 것이다. 

<br />


> 연습문제 
> 
> 1. factorial(-1) 은 뭐가 나올까?
> 2. 겹계승 (doublefactorial)
함수, 가령 8의 겹계승 8 × 6 × 4 × 2 = 384, 7의 겹계승은 7 × 5 × 3 × 1 = 105이다. 하스켈로 정의해보자.

<br />

1번) 일단 두번째 조건에 해당하니까 정의에 따르면 factorial(-1) = factorial(-2) x (-1) 인데, factorial (-2) = factorial(-3) x (-2) 이런 식이면, factorial(-1) 은 -1부터 -inf까지의 정수를 다 곱한 것일텐데, 그 값은 수렴하지 않으니까 에러가 나나? 

```haskell
ghci> factorial (-1) 
*** Exception: stack overflow
```

한참을 무한루프 도는 것 같더니 Exception 에러가 났다. 오케이 

<br />
2번) 

```haskell
let {doublefactorial 2 = 2; doublefactorial 1 = 1; doublefactorial n = doublefactorial (n-2) * n}
```
이러면 이제 doublefactorial 6 = doublefactorial 4 * 6 = doublefactorial 2 * 4 * 6 = 2 * 4 * 6 이니까 맞능가. 확인 끝!


<br />

>

> 연습문제
>
> 1. factorial 3을 가지고 한 것과 비슷하게 곱셈 5 × 4를 전개하라.
> 2. power x y가 x의 y승인 재귀 함수 power를 정의하라.
> 3. plusOne x = x + 1이란 함수가 주어졌다. 다른 (+)를 쓰지 않고 addition x y가 x와 y를 더하는 재귀 함수 addition을 정의하라.
> 4. (어려움) 인자의 정수 로그(밑이 2)를 계산하는 함수 log2를 정의하라. 즉 log2는 인자 이하의 가장 큰 2의 거듭제곱의 지수를 계산한다. 예를 들어 log2 16 = 4, log2 11 = 3, log2 1 = 0이다. (작은 힌트: 이 연습문제들 바로 전 단락의 마지막 문장을 읽을 것.)

<br />

1번) 
```haskell
mul n 0 = 0
mul n 1 = n
mul n m = mul n (m-1) + n 
```
이라고 하면 안되고 (왜 안되징?)

```haskell
ghci> let {mul n 0 = 0; mul n 1 = 1; mul n m = mul n (m-1) + n}
ghci> mul 5 4 
20
```
<br />
2번)

```haskell
ghci> let {power x 0 = 1; power x y = power x (y-1) * x}
ghci> power 3 4
81
```
<br />

3번)
아 위에서 말한 `plusOne` 이라는 함수가 주어졌을 때, 라는 가정이 있다. 흠
```haskell
plusOne x = x + 1

-- 그럼 x에다 +1 을 y 번 한다는 느낌으로 가면 
let {addition x 0 = x; addition x y = addition (plusOne x) (y-1)}
```
<br />
4번) 대망의 4번

```haskell
let {log2 1 = 0; log 2 2 = 1; log2 x = log2 (ceiling(x/2)) + 1}
```

`log2 :: (Num a) => a -> a` 이렇게 해주면 타입 관련된 에러 안날까? 옹 엄청 긴 에러가 나왔다. 세가지 에러가 나왔는데, 

```haskell
? Could not deduce (Eq a) arising from the literal '1'
? Could not deduce (RealFrac a) arising from the literal 'ceiling'
? Could not deduce (Fractional a) arising from the literal '/'
```
`Num`이 대부분 숫자 타입을 다 커버하는 건지 알았는데, 그게 아닌건가..
결국 답지를 찾아 봐버렸다.

4번 답) 
```haskell
log2 1 = 0
log2 n = 1 + log2 (n `div` 2) -- the "`" make div into infix notation
```

<br />

### 리스트 기반 재귀 

`++`와 `length`를 배웠다. 

<br />


> 연습문제
>
> 다음의 리스트 기반 함수들을 재귀적으로 정의하라. 각각 기본가정이 어떨지를 생각한 다음에 재귀가정은 어떨 지를 생각해볼 것. (여기의 모든 함수는 Prelude에 이미 있으므로 GHCi에서 여러분의 정의를 시험할 때는 이름을 다르게 해야 한다)
> 1. `replicate :: Int -> a -> [a]`는 카운트와 한 원소를 취해 그 원소를 그만큼 반복한 리스트를 반환한다. 예를 들어 `replicate 3 'a' = "aaa"`다. (힌트: 카운트가 0일 때 무언가의 복제가 무엇이어야 하는지 생각해보라. 카운트 0이 여러분의 '기본가정'이다.)
> 2. `(!!) :: [a] -> Int -> a`는 주어진 '인덱스'의 원소를 반환한다. 첫 번째 원소는 인덱스 0에, 두 번째는 1에 있는 식이다. Note that with this function, you're recursing both numerically and down a list.
> 3. (살짝 어려움) zip :: [a] -> [b] -> [(a, b)]는 두 리스트를 취해 'zip'하여, 첫 번째 짝은 두 리스트의 처음 원소들이고, 그렇게 계속된다. 가령 zip [1,2,3] "abc" = [(1, 'a'), (2, 'b'), (3, 'c')]이다. 두 리스트의 길이가 다르면 하나라도 다 떨어지는 시점에서 멈출 수 있다. 가령 zip [1,2] "abc" = [(1, 'a'), (2, 'b')]이다.
> 4. factorial의 루프식 대체 버전처럼 보조 함수와 누적 매개변수를 이용하여 length를 정의하라.

<br />

1번) 
```haskell
ghci> let {replicate 0 a = []; replicate n a = replicate (n-1) a ++ [a]} -- 첨에 []를 ()으로 했다가 에러났었음 

ghci> replicate 3 'a'
ghci> "aaa"
ghci> replicate 3 5
ghci> [5,5,5]
```

<br />

2번) 
```haskell
let {idx_ele [a] 0 = head [a]; idx_ele [a] n = idx_ele (tail [a]) (n-1)}
```
라고 하면 
        
        Exception: Non-exhuastive patterns in function idx_ele
    
라고 한다. 당연히 a자리에 리스트가 올거라고 타입이 정의되있다고 가정하는 것 같으니 

```haskell
ghci> let {idx_ele a 0 = head a; idx_ele a n = idx_ele (tail a) (n-1)}

ghci> idx_ele [1,2,3,4] 2
ghci> 3
```
아 맞아 `tail`이나 `head` 둘 다 empty list받으면 에러나는 것도 추가해줘야 하나봐. 답지는 이런식으로 정의되어 있네.

```haskell
[]     !! _ = error "Index too large" -- An empty list has no elements
(x:_)  !! 0 = x 
(x:xs) !! n = xs !! (n-1)
```


<br />
3번) 홍홍...

```haskell
let {zzip a [] = []; zzip [] b = []; zzip a b = zzip (tail a) (tail b) ++ [(head a, head b)]}
```
옹 괜찮은데? 테스트 해보자...
```haskell
ghci> zzip [1,2,3] ['a','b']
ghci> [(2, 'b'), (1,'a')]
```
순서가 홍! 아아 `++` 순서만 바꿔주면 되겠지?

```haskell
ghci> let {zzip a [] = []; zzip [] b = []; zzip a b = [(head a, head b)] ++ zzip (tail a) (tail b) }
ghci> zzip [1,2,3] ['a','b']
ghci> [(1,'a'), (2, 'b')]
```

왕 역시 아직 난 초초보인듯.. ㅎㅎ 답지는 느낌이 다르다

```haskell
zip (x:xs) (y:ys) = (x,y) : zip xs ys
zip _       _     = []
```
<br />

4번)
해야하는데 잘 모르겠다.

`go`는 쓰임새가 다양한 것 같은데 쉽게 말하면 literally, "go" as in "go around the loop" 그래서 recursive function할 때 많이 쓴다는 거 같은데... 답지 보자!

```haskell
length xs = go 0 xs
    where
    go acc []     = acc
    go acc (_:xs) = go (acc + 1) xs
```
