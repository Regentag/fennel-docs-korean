# Fennel 시작하기

프로그래밍 언어는 **문법**(Syntax)과 **의미**(Sementics)로 구성됩니다. Fennel의 의미는 Lua와 아주 작은 차이만 있습니다(모두 아래에 설명). Fennel의 문법은 Lisp 계열 언어에서 유래했습니다. Lisp은 매우 균일하고 예측 가능한 문법을 가지고 있어 [코드에서 작동하는 코드 작성][1]과 [구조적 편집][2]을 더 쉽게 수행할 수 있습니다.

Lua와 Lisp에 이미 익숙하다면 Fennel이 아주 익숙하게 느껴질 것입니다. 그렇지 않더라도 Lua는 현존하는 가장 간단한 프로그래밍 언어 중 하나이므로, 이전에 프로그래밍을 해 본 경험이 있다면 큰 어려움 없이 익힐 수 있을 것입니다. 특히 클로저(Closure)를 사용하는 다른 동적 명령형 언어를 사용해 본 경험이 있다면 더욱 그렇습니다. [Lua 참조 설명서][3]에서 자세한 내용을 찾아볼 수 있지만, Fennel의 [Lua Primer][14]는 더 간략하고 핵심 내용을 다룹니다.

이미 Lua 예제 코드가 있고 Fennel에서 어떻게 보이는지 확인하고 싶다면 [Antifennel][19]에 넣어서 많은 것을 배울 수 있습니다.

## 좋아요, 그러면 어떻게 하면 되나요?

### 함수와 람다(lambda)

함수를 만들려면 `fn`을 사용합니다. 함수에 이름을 붙이는것은 선택적입니다. 이름을 붙이면 함수는 로컬 범위에서 해당 이름에 바인딩됩니다. 그렇지 않으면 단순히 익명 값입니다.

> 이름 짓기에 대한 참고사항: 식별자는 일반적으로 대시(-)로 구분된 소문자입니다(kebab-case).
> 식별자는 숫자를 포함할 수 있습니다(가장 첫 글자는 불가).
> 물음표를 사용할 수 있습니다(일반적으로 참/거짓을 반환하는 함수의 경우. 예: `at-max-velocity?`).
> 밑줄(`_`)은 사용하지 않을 변수의 이름을 지정하는 데 자주 사용됩니다.

인자 목록은 대괄호로 감싸 제공됩니다. 함수 본문의 최종 값이 반환됩니다.

(이전에 리스프를 사용해 본 적이 없다면, 가장 중요한 점은 호출되는 함수나 매크로가 괄호 *안*에 들어가야 하며, 밖이 아니라는 것입니다.)

```fennel
(fn print-and-add [a b c]
  (print a)
  (+ b c))
```
함수는 선택적으로 인자 목록 바로 뒤에 오는 문자열 형태의 docstring을 받을 수 있습니다.
일반 컴파일에서는 이 docstring이 생성된 Lua 파일에서 제거되지만, REPL에서 개발하는 동안에는 `,doc` 명령을 사용하여 docstring과 함수 사용법을 확인할 수 있습니다.

```fennel
(fn print-sep [sep ...]
  "Prints args as a string, delimited by sep"
  (print (table.concat [...] sep)))
,doc print-sep ; -> outputs:
;; (print-sep sep ...)
;;   Prints args as a string, delimited by sep
```

다른 Lisp 언어와 마찬가지로 Fennel은 주석에 세미콜론을 사용합니다.

`fn`으로 정의된 함수는 빠르며 Lua에 비해 런타임 오버헤드가 없습니다. 하지만 인자 수 검사도 없습니다. (즉, 잘못된 개수의 인자로 함수를 호출해도 오류가 발생하지 않습니다.) 더 안전한 코드를 위해 `lambda`를 사용하면 정의된 개수만큼의 인자를 받을 수 있습니다. `lambda`를 사용할 때 인자의 이름 앞에 `?`를 붙여 생략 가능한 인자를 지정할 수 있습니다.

```fennel
(lambda print-calculation [x ?y z]
  (print (- x (* (or ?y 1) z))))

(print-calculation 5) ; -> error: Missing argument z
```
두 번째 인수 `?y`는 `nil`이 될 수 있지만 `z`는 그렇지 않습니다.

```fennel
(print-calculation 5 nil 3) ; -> 2
```

`fn`과 마찬가지로 람다는 인수 목록 뒤에 선택적 docstring을 허용합니다.

### 로컬(locals)과 변수(variables)

로컬은 `let`을 사용하여 정의되며, 이름과 값은 단일 대괄호로 묶입니다.

```fennel
(let [x (+ 89 5.2)
      f (fn [abc] (print (* 2 abc)))]
  (f x))
```

여기서 `x`는 89와 5.2를 더한 결과에 바인딩되고, `f`는 인수의 두 배를 출력하는 함수에 바인딩됩니다. 이러한 바인딩은 `let` 호출 본문 내에서만 유효합니다.

`local`을 사용하여 로컬을 사용할 수도 있습니다. 이는 전체 파일에서 사용될 때 유용하지만, 일반적으로 함수 내부에서는 `let`을 사용하는 것이 더 좋습니다. 값을 사용할 위치를 한눈에 알 수 있기 때문입니다.

```fennel
(local tau-approx 6.28318)
```

이렇게 설정된 로컬에는 새로운 값을 지정할 수 없습니다. 하지만 외부 이름을 가리는 새로운 로컬을 정의할 수 있습니다.

```fennel
(let [x 19]
  ;; (set x 88) <- not allowed!
  (let [x 88]
    (print (+ x 2))) ; -> 90
  (print x)) ; -> 19
```

변수의 값을 변경해야 하는 경우 `var`를 사용할 수 있습니다. `var`는 `local`과 유사하지만 `set`을 사용할 수 있습니다. `var`는 `let`과 같이 중첩되지 않습니다.

```fennel
(var x 19)
(set x (+ x 8))
(print x) ; -> 27
```

### 숫자와 문자열

당연하게도, `+`, `-`, `*`, `/`와 같은 모든 표준 산술 연산자는 접두사 형태로 작동합니다. 정수가 도입된 Lua 5.3 이전, Lua 5.2 이하의 모든 Lua 버전에서는 숫자가 배정밀도 부동 소수점 수(Double)라는 점에 유의하세요. Fennel이 Lua 5.3 이상에서 작동할 때에는 정수 나눗셈에 `//`를 사용하고 비트 연산에는 `lshift`, `rshift`, `bor`, `band`, `bnot`, `xor`를 사용합니다. 호스트 Lua 환경이 5.3 이전 버전인 경우 비트 연산자와 정수 나눗셈은 작동하지 않습니다.

긴 숫자의 섹션을 구분하기 위해 밑줄을 사용할 수도 있습니다. 밑줄은 값에 영향을 미치지 않습니다.

```fennel
(let [x (+ 1 99)
      y (- x 12)
      z 100_000]
  (+ z (/ y 10)))
```

문자열은 본질적으로 변경 불가능한 바이트 배열입니다. UTF-8 지원은 [Lua 5.3 이상][15]에서는 `utf8` 테이블에서 제공되며, 이전 버전에서는 [3rd Party 라이브러리][4]를 통해 제공됩니다. 문자열은 `..`를 사용하여 연결됩니다.

```fennel
(.. "hello" " world")
```

### 테이블

Lua(그리고 Fennel)에서는 테이블이 유일한 데이터 구조입니다. 테이블의 주요 구문은 중괄호 안에 키/값 쌍을 사용하는 것입니다.

```fennel
{"key" value
 "number" 531
 "f" (fn [x] (+ x 2))}
```

`.`를 사용하면 테이블에서 값을 가져올 수 있습니다.

```fennel
(let [tbl (function-which-returns-a-table)
      key "a certain key"]
  (. tbl key))
```

그리고 `tset`을 사용하여 값을 삽입할 수 있습니다.

```fennel
(let [tbl {}
      key1 "a long string"
      key2 12]
  (tset tbl key1 "the first value")
  (tset tbl key2 "the second one")
  tbl) ; -> {"a long string" "the first value" 12 "the second one"}
```

### 순차 테이블

어떤 경우 테이블은 순차적으로 사용되는 데이터를 저장하는 데 사용됩니다. 이 경우 키는 1부터 시작하여 증가하는 숫자입니다. Fennel은 이러한 테이블에 대한 대체 구문을 대괄호를 사용하여 제공합니다.

```fennel
["abc" "def" "xyz"] ; equivalent to {1 "abc" 2 "def" 3 "xyz"}
```

Lua의 내장 함수 `table.insert`는 순차 테이블에서 사용하도록 만들어졌습니다. 삽입된 값 이후의 모든 값은 인덱스가 하나씩 증가합니다. `table.insert`에 인덱스를 지정하지 않으면 테이블의 끝에 추가됩니다.

`table.remove` 함수도 비슷하게 작동합니다. 테이블과 인덱스(기본값은 테이블의 끝으로 지정됨)를 받아서 해당 인덱스에 있는 값을 제거한 후 반환합니다.

```fennel
(local ltrs ["a" "b" "c" "d"])

(table.remove ltrs)       ; Removes "d"
(table.remove ltrs 1)     ; Removes "a"
(table.insert ltrs "d")   ; Appends "d"
(table.insert ltrs 1 "a") ; Prepends "a"

(. ltrs 2)                ; -> "b"
;; ltrs is back to its original value ["a" "b" "c" "d"]
```

`length`는 순차적인 테이블과 문자열의 길이를 반환합니다.

```fennel
(let [tbl ["abc" "def" "xyz"]]
  (+ (length tbl)
     (length (. tbl 1)))) ; -> 6
```

간격(gap)이 있는 테이블의 길이는 정의되지 않은 동작입니다. nil 값과 nil이 아닌 값 사이의 테이블의 "경계" 위치에 해당하는 숫자를 반환할 수 있습니다.

Lua의 표준 라이브러리는 매우 작기 때문에 당연히 있을것이라 예상할 수 있는 `map`, `reduce`, `filter`와 같은 함수가 없습니다. Fennel에서는 대신 매크로를 사용합니다. `icollect`, `collect`, `accumulate`를 참조하세요.

### 반복 (Iteration)

테이블 요소에 대한 반복은 `each` 함수와 `pairs`(일반 테이블에 사용됨)나 `ipairs`(순차 테이블에 사용됨)와 같은 반복자를 사용하여 수행됩니다.

```fennel
(each [key value (pairs {"key1" 52 "key2" 99})]
  (print key value))

(each [index value (ipairs ["abc" "def" "xyz"])]
  (print index value))
```

테이블이 순차적인지 여부는 테이블의 고유한 속성이 아니라 어떤 반복자가 사용되었는지에 따라 달라집니다. 어떤 테이블에 대해서든 `ipairs`를 호출할 수 있으며, 1부터 시작하여 `nil`을 만날 때까지 숫자 키만 반복합니다.

`each`와 함께 어떤 [Lua 반복자][6]도 사용할 수 있습니다. [문자열의 일치][7]를 살펴보는 예제는 다음과 같습니다.

```fennel
(var sum 0)
(each [digits (string.gmatch "244 127 163" "%d+")]
  (set sum (+ sum (tonumber digits))))
```

결과를 다시 테이블로 반환받으려면 `icollect`(순차 테이블) 또는 `collect`(키/값 테이블)을 사용합니다. 본문이 nil을 반환하면 결과 테이블에서 항목이 제외됩니다.

```fennel
(icollect [_ s (ipairs [:greetings :my :darling])]
  (if (not= :my s)
      (s:upper)))
;; -> ["GREETINGS" "DARLING"]

(collect [_ s (ipairs [:greetings :my :darling])]
  s (length s))
;; -> {:darling 7 :greetings 9 :my 2}
```

저수준의 반복은 `for`로, 제공된 시작 값에서 종료 값(포함)까지 숫자로 반복합니다.

```fennel
(for [i 1 10]
  (print i))
```

선택적으로 값의 단계를 지정할 수 있습니다. 이 루프는 10 미만의 홀수만 출력합니다.

```fennel
(for [i 1 10 2]
  (print i))
```

### 반복 (Looping)

반복해야 하지만 반복 횟수를 모르는 경우 `while`을 사용할 수 있습니다.

```fennel
(while (keep-looping?)
  (do-something))
```

### 조건문

마지막으로 조건문이 있습니다. Fennel의 `if` 형식은 다른 Lisp 언어와 같은 방식으로 사용할 수 있지만, `elseif` 분기로 컴파일되는 여러 조건에 대한 `cond`로도 사용할 수 있습니다.

```fennel
(let [x (math.random 64)]
  (if (= 0 (% x 2))
      "even"
      (= 0 (% x 9))
      "multiple of nine"
      "I dunno, something else"))
```

인자의 개수가 홀수이면 마지막 절은 "else"로 해석됩니다.

Fennel은 Lisp이므로 명령문(Statement)이 없습니다. `if`는 표현식(Expression)으로, 값을 반환합니다. Lua 프로그래머라면 단순히 값을 얻기 위해 `and`/`or`를 불안정하게 연결할 필요가 없다는 사실에 기뻐할 것입니다!

다른 조건문은 `when`으로, 임의의 숫자의 Side-Effect에 사용되며 else 절이 없습니다.

```fennel
(when (currently-raining?)
  (wear "boots")
  (deploy-umbrella))
```

## 다시 테이블로

공백이나 예약된 문자가 없는 문자열은 `:shorthand` 구문을 사용할 수 있습니다. 이 구문은 종종 테이블 키에 사용됩니다.

```fennel
{:key value :number 531}
```

테이블에 이와 같은 문자열 키가 있는 경우, 키를 미리 알고 있다면 점(.)을 사용하여 쉽게 값을 추출할 수 있습니다.

```fennel
(let [tbl {:x 52 :y 91}]
  (+ tbl.x tbl.y)) ; -> 143
```

`set`과 함께 이 구문을 사용할 수도 있습니다.

```fennel
(let [tbl {}]
  (set tbl.one 1)
  (set tbl.two 2)
  tbl) ; -> {:one 1 :two 2}
```

테이블 키의 이름이 설정하려는 변수의 이름과 같으면 키 이름을 생략하고 대신 `:`를 사용할 수 있습니다.

```fennel
(let [one 1 two 2
      tbl {: one : two}]
  tbl) ; -> {:one 1 :two 2}
```

마지막으로 `let`은 테이블을 여러 개의 로컬로 구조 분해할 수 있습니다.

위치적 구조 분해:

```fennel
(let [data [1 2 3]
      [fst snd thrd] data]
  (print fst snd thrd)) ; -> 1       2       3
```

테이블 키를 사용한 구조 분해:

```fennel
(let [pos {:x 23 :y 42}
      {:x x-pos :y y-pos} pos]
  (print x-pos y-pos)) ; -> 23      42
```

위와 같이 테이블 키의 이름이 구조 분해하려는 변수의 이름과 같으면 키 이름을 생략하고 대신 `:`를 사용할 수 있습니다.

```fennel
(let [pos {:x 23 :y 42}
      {: x : y} pos]
  (print x y)) ; -> 23      42
```

이것을 중첩하고 섞어 일치시킬 수 있습니다.

```fennel
(let [f (fn [] ["abc" "def" {:x "xyz" :y "abc"}])
      [a d {:x x : y}] (f)]
  (print a d)
  (print x y))
```

테이블 크기가 바인딩할 로컬의 개수와 일치하지 않으면 누락된 값은 `nil`로 채워지고 넘치는 값은 삭제됩니다. 많은 언어와 달리 Lua에서 `nil`은 실제로 값이 없음을 나타내므로 테이블에 `nil`이 포함될 수 없습니다. `nil`을 키로 사용하는 것은 오류이며, `nil`을 값으로 사용하면 해당 키에 있던 항목이 모두 제거됩니다.

## 오류 처리

Lua에서 오류는 두 가지 형태로 나타날 수 있습니다. Lua 함수는 여러 값을 반환할 수 있으며, 실패할 수 있는 대부분의 함수는 `nil`과 실패 메시지 문자열을 반환하는 두 가지 반환 값을 사용하여 실패를 나타냅니다. Fennel에서는 대괄호 대신 괄호를 사용하여 이러한 유형의 함수를 분해할 수 있습니다.

```fennel
(case (io.open "file")
  ;; io.open이 성공하면 파일을 반환하지만
  ;; 실패하면 nil과 그 이유를 설명하는 err-msg 문자열을 반환합니다.
  f (do (use-file-contents (f:read :*all))
        (f:close))
  (nil err-msg) (print "Could not open file:" err-msg))
```

`values`를 사용하여 여러 값을 반환하는 함수를 직접 작성할 수 있습니다.

```fennel
(fn use-file [filename]
  (if (valid-file-name? filename)
      (open-file filename)
      (values nil (.. "Invalid filename: " filename))))
```

**참고**: 오류는 함수에서 여러 값을 반환하는 가장 흔한 이유이지만, 다른 경우에도 사용될 수 있습니다. 이는 Lua에서 가장 복잡한 부분이며, 이 튜토리얼에서는 자세히 다루지 않지만 [다른 곳][18]에서 자세히 다루고 있습니다.

이러한 유형의 오류는 구성이 잘 되지 않는다는 문제점이 있습니다. 오류 상태는 내부에서 외부로 호출 체인을 따라 전파되어야 합니다. 이 문제를 해결하려면 `error`를 사용할 수 있습니다. 이 함수는 보호된 호출 내부가 아닌 경우 전체 프로세스를 종료합니다. 이는 다른 언어에서 try/catch 내부가 아닌 경우 예외를 throw하면 프로그램이 중단되는 방식과 유사합니다. `pcall`을 사용하여 보호된 호출을 만들 수 있습니다.

```fennel
(let [(ok? val-or-msg) (pcall potentially-disastrous-call filename)]
  (if ok?
      (print "Got value" val-or-msg)
      (print "Could not get value:" val-or-msg)))
```

`pcall` 호출은 `(potentially-disastrous-call filename)`을 보호 모드에서 실행한다는 것을 의미합니다. `pcall`은 임의의 개수의 인자를 받으며, 이 인자들은 함수에 전달됩니다. `pcall`은 호출 성공 여부를 나타내는 부울 값(여기서는 `ok?`)과, 성공했을 때는 결과, 실패했을 때는 오류 메시지가 설정되는 두 번째 값(`val-or-msg`)을 반환합니다.

`assert` 함수는 값과 오류 메시지를 받습니다. 값이 `nil`이면 `error`를 호출하고, 그렇지 않으면 값을 반환합니다. 이 함수는 여러 값의 실패를 `error`로 변환하는 데 사용할 수 있습니다(`error`를 여러 값의 실패로 변환하는 `pcall`의 반대 개념).

```fennel
(let [f (assert (io.open filename))
      contents (f.read f "*all")]
  (f.close f)
  contents)
```

이 예시에서 `io.open`은 실패 시 `nil`과 오류 메시지를 반환하므로, 실패하면 `error`가 발생하고 실행이 중단됩니다.

## 가변 함수

Fennel은 많은 언어처럼 가변 함수(즉, 여러 개의 인수를 받는 함수)를 지원합니다. 함수에 가변 개수의 인수를 받는 구문은 `...` 기호이며, 함수의 마지막 매개변수여야 합니다. 이 구문은 Lisp가 아닌 Lua에서 파생되었습니다.

`...` 형태는 목록(list)이나 일급 값(first class value)이 아니며, 인라인으로 여러 값으로 확장됩니다. 가변 인자의 개별 요소에 접근하려면 괄호를 사용하여 구조를 q분해하거나, 먼저 테이블 리터럴(`[...]`)로 감싸 일반 테이블처럼 색인하거나, Lua 코어 라이브러리의 `select` 함수를 사용할 수 있습니다. 가변 인자는 바인딩할 필요 없이 `print`와 같은 다른 함수에 직접 전달될 수 있는 경우가 많습니다.

```fennel
(fn print-each [...]
  (each [i v (ipairs [...])]
    (print (.. "Argument " i " is " v))))

(print-each :a :b :c)
```

```fennel
(fn myprint [prefix ...]
  (io.write prefix)
  (io.write (.. (select "#" ...) " arguments given: "))
  (print ...))

(myprint ":D " :d :e :f)
```

가변 인수는 다른 변수와 범위가 다릅니다. 즉, 생성된 함수에서만 접근할 수 있습니다. 일반 값과 ​​달리 함수는 가변 인수를 내포할 수 없습니다. 즉, 내부 함수에서는 가변 인수가 범위를 벗어나므로 다음 코드는 작동하지 않습니다.

```fennel
(fn badcode [...]
  (fn []
    (print ...)))
```

## 엄격한 글로벌 검사

`unknown global in strict mode`라는 오류가 발생하는 경우, Fennel 컴파일러가 인식하지 못하는 전역 변수를 사용하는 코드를 컴파일하려고 한다는 의미입니다. 대부분의 경우 코딩 실수가 원인입니다. 하지만 경우에 따라 올바른 전역 변수 참조를 사용하는 경우에도 이 오류가 발생할 수 있습니다. 이 경우 Fennel의 내재된 한계 때문일 수 있습니다. `_G.myglobal`과 같은 형태를 사용하면 이러한 검사를 우회하여 이 변수가 실제로 전역 변수라는 사실을 알릴 수 있습니다.

이 오류의 또 다른 가능한 원인은 [함수 환경][16]이 수정된 것입니다.
해결 방법은 Fennel 사용 방식에 따라 다릅니다.

* 임베딩 된 Fennel은 `allowedGlobals` 매개변수를 통해 특정(또는 모든) 전역 변수를 무시하도록 검색기를 수정할 수 있습니다. 자세한 내용은 [Lua API][17]를 참조하세요.
* Fennel의 CLI에는 `--globals` 매개변수가 있는데, 이 매개변수는 무시할 전역 변수 목록을 쉼표로 구분하여 지정합니다. 예를 들어, 전역 변수 x, y, z에 대해 엄격 모드를 비활성화하려면 다음과 같이 합니다.
  ```shell
  fennel --globals x,y,z yourfennelscript.fnl
  ```

## 함정들

숙련된 리스퍼(lisper)들을 놀라게 할 만한 몇 가지 놀라운 점이 있습니다. 이 중 대부분은 Fennel이 Lua에 런타임 오버헤드를 전혀 주지 않겠다고 고집한 데서 비롯됩니다.

* 산술, 비교, 부울 연산자는 일급 함수가 아닙니다. 여러 개의 반환 값을 갖는 함수에서는 예상치 못한 방식으로 동작할 수 있는데, 이는 컴파일 타임에 인수의 개수를 알아야 하기 때문입니다.

* `apply` 함수가 없습니다. 대신 Lua 버전에 따라 `table.unpack` 또는 `unpack`을 사용하세요: `(f 1 3 (table.unpack [4 9]))`.

* [Baker][8]에 따르면 테이블은 내용의 값이 아닌 동일성을 기준으로 동등성을 비교합니다.

* repl에서는 반환 값이 보기 좋게 출력되지만, `(print tbl)`을 호출하면 `table: 0x55a3a8749ef0`과 같은 출력이 생성됩니다. 디버깅을 위해 `fennel.view`를 호출한 후 출력하는 프린터 함수를 정의하는 것이 좋습니다. `(local fennel (require :fennel)) (fn _G.pp [x] (print (fennel.view x)))`처럼요. 이 정의를 `~/.fennelrc` 파일에 추가하면 표준 repl에서 사용할 수 있습니다.

* Lua 프로그래머는 Fennel 함수가 조기 반환을 수행할 수 없다는 점에 유의해야 합니다.

## 작동하는 다른것들

`math.random`과 같은 [Lua 표준 라이브러리][10]의 내장 함수는 번거로움 없이, 오버헤드 없이 호출할 수 있습니다.

여기에는 다른 언어에서는 특수 구문을 사용하여 구현되는 코루틴과 같은 기능이 포함됩니다. 코루틴을 사용하면 [콜백 없이 Non-Blocking 작업을 표현할 수 있습니다][11].

Lua의 테이블은 다소 제한적인 것처럼 보일 수 있지만, [메타테이블][12]은 훨씬 더 많은 유연성을 제공합니다. 메타테이블의 모든 기능은 Lua에서와 마찬가지로 Fennel 코드에서도 접근할 수 있습니다.

## 모듈과 다수의 파일

`require` 함수를 사용하면 다른 파일에서 코드를 로드할 수 있습니다.

```fennel
(let [lume (require :lume)
      tbl [52 99 412 654]
      plus (fn [x y] (+ x y))]
  (lume.map tbl (partial plus 2))) ; -> [54 101 414 656]
```

Fennel과 Lua의 모듈은 함수와 다른 값을 포함하는 테이블일 뿐입니다. Fennel 파일의 마지막 값은 전체 모듈의 값으로 사용됩니다. 기술적으로 이는 테이블뿐만 아니라 어떤 값이든 될 수 있지만, 테이블을 사용하는 것이 가장 일반적인 데에는 그럴 만한 이유가 있습니다.

하위 디렉터리에 있는 모듈을 require하려면 파일 이름을 가져와서 슬래시를 마침표로 바꾸고 확장자를 제거한 후 `require` 함수에 전달합니다. 예를 들어, `lib.ui.menu` 모듈을 로드할 때 `lib/ui/menu.lua`라는 파일을 읽습니다.

`fennel` 명령어로 프로그램을 실행할 때 `require`를 호출하여 Fennel 또는 Lua 모듈을 불러올 수 있습니다. 하지만 다른 상황(예: Lua로 컴파일한 후 `lua` 명령어를 사용하거나 Lua를 내장한 프로그램)에서는 Fennel 모듈을 인식하지 못합니다. `.fnl` 파일을 찾는 방법을 알려주는 검색기를 설치해야 합니다.

```lua
require("fennel").install()
local mylib = require("mylib") -- will compile and load code in mylib.fnl
```

이 옵션을 추가하면 `require`는 Lua에서처럼 Fennel 파일에서도 작동합니다. 예를 들어 `(require :mylib.parser)`는 Fennel 검색 경로(`fennel.path`에 저장되어 있으며, Lua 모듈을 찾는 데 사용되는 `package.path`와는 별개입니다)에서 "mylib/parser.fnl"을 검색합니다. 이 경로에는 기본적으로 현재 디렉터리를 기준으로 항목을 로드할 수 있는 항목이 포함되어 있습니다.

## 상대적 require

모듈을 사용하는 라이브러리를 작성하는 방법은 여러 가지가 있습니다. 그중 하나는 LuaRocks와 같은 도구를 사용하여 라이브러리 설치 및 라이브러리와 모듈의 가용성을 관리하는 것입니다. 또 다른 방법은 상대적인 require 스타일을 사용하여 중첩된 모듈을 로드하는 것입니다. 상대적인 require 스타일을 사용하면 라이브러리는 내부 모듈 경로를 확인할 때 루트 디렉터리 이름이나 위치에 의존하지 않습니다.

예를 들어, `init.fnl` 파일과 모듈이 루트 디렉토리에 포함된 작은 `example` 라이브러리는 다음과 같습니다.

```fennel
;; file example/init.fnl:
(local a (require :example.module-a))

{:hello-a a.hello}
```

여기서, 메인 모듈에는 구현을 담고 있는 추가적인 `example.module-a` 모듈이 필요합니다.

```fennel
;; file example/module-a.fnl
(fn hello [] (print "hello from a"))
{:hello hello}
```

여기서 가장 큰 문제는 라이브러리 경로가 정확히 `example`이어야 한다는 것입니다. 예를 들어, 라이브러리가 작동하려면 `(require :example)`로 require해야 하는데, 라이브러리 사용자에게 이를 강제할 수 없습니다. 예를 들어, 복잡함을 피하기 위해 라이브러리를 프로젝트의 `libs` 디렉터리로 이동하고 `(require :libs.example)`로 require하면 런타임 오류가 발생합니다. 이는 라이브러리 자체가 올바른 모듈 경로인 `:libs.example.module-a`가 아닌 `:example.modulea`를 require하려고 하기 때문입니다.

    runtime error: module 'example.module-a' not found:
            no field package.preload['example.module-a']
            ...
            no file './example/module-a.lua'
            ...
    stack traceback:
      [C]: in function 'require'
      ./libs/example/init.fnl:2: in main chunk

LuaRocks는 디렉터리 이름과 설치 경로를 모두 강제로 적용하고, 라이브러리를 사용할 수 있도록 `LUA_PATH` 환경 변수를 설정하여 이 문제를 해결합니다. 물론, 빌드 파이프라인에서 프로젝트별로 `LUA_PATH`를 설정하고 해당 디렉터리를 지정하여 수동으로 설정할 수도 있습니다. 하지만 이 방법은 투명하지 않으며, 프로젝트 로컬 라이브러리가 필요한 경우 `LUA_PATH`가 수정된 위치를 찾는 것보다 프로젝트의 파일 구조에 직접 매핑되는 전체 경로를 확인하는 것이 더 좋습니다.

Fennel 생태계에서는 프로젝트 종속성을 관리하는 더 간단한 방법을 권장합니다. 일반적으로 라이브러리를 프로젝트 트리에 추가하거나 git 서브모듈을 사용하는 것만으로도 충분하며, require 경로는 라이브러리 자체에서 처리해야 합니다.

`libs/example/init.fnl`에 상대적인 require 경로를 지정하여 이름/경로에 구애받지 않는 방법은 다음과 같습니다. 단, `example` 라이브러리를 해당 위치로 옮겼다고 가정합니다.

```fennel
;; file libs/example/init.fnl:
(local a (require (.. ... :.module-a)))

{:hello-a a.hello}
```

이제 라이브러리 이름이나 위치는 중요하지 않으며, 어디에서든 require할 수 있습니다. `(require :lib.example)`로 라이브러리를 require할 때 `...`의 첫 번째 값은 `"lib.example"` 문자열을 포함하기 때문에 작동합니다. 이 문자열은 `".module-a"`와 연결되고, `require`는 런타임에 `"lib.example.module-a"` 경로에서 중첩된 모듈을 제대로 찾아 로드합니다. 이 기능은 Fennel에만 국한된 것이 아니라 Lua의 기능이며, 라이브러리가 Lua로 AOT 컴파일된 경우에도 동일하게 작동합니다.

### 컴파일 타임 상대적 include

Fennel v0.10.0부터 `include` 특수 플래그나 `--require-as-include` 플래그를 사용할 경우 컴파일 타임에도 이 기능이 작동합니다. 단, 표현식이 컴파일 타임에 계산될 수 있어야 한다는 제약 조건이 있습니다. 즉, 표현식은 자체적으로 포함되어야 합니다. 즉, 로컬이나 전역 변수를 참조하지 않고 모든 값을 직접 포함해야 합니다. 다시 말해, 다음 코드는 런타임에만 작동하지만 `include` 또는 `--require-as-include` 플래그를 사용할 경우 `current-module`이 컴파일 타임에 알려지지 않기 때문에 작동하지 않습니다.

```fennel
(local current-module ...)
(require (.. current-module :.other-module))
```

반면에 이것은 런타임과 컴파일 타임 모두에서 작동합니다.

```fennel
(require (.. ... :.other-module))
```

`...` 모듈 인수는 컴파일 중에 전파되므로 이 라이브러리를 사용하는 애플리케이션이 컴파일되면 모든 라이브러리 코드가 Lua 파일에 올바르게 포함됩니다.

`--require-as-include`를 사용하여 이 `example` 라이브러리를 사용하는 프로젝트를 컴파일하면 결과 Lua 코드에 다음 섹션이 포함됩니다.

```lua
package.preload["libs.example.module-a"] = package.preload["libs.example.module-a"] or function(...)
  local function hello()
    return print("hello from a")
  end
  return {hello = hello}
end
```

`package.preload` 항목에는 컴파일 시간에 확인된 완전한 경로 `"libs.example.module-a"`가 포함되어 있습니다.

### `init.fnl` 이외의 모듈에서 모듈 require

`init` 모듈이 아닌 다른 모듈에서 모듈을 require 하려면 현재 모듈까지의 경로는 유지하되 모듈 이름은 제거해야 합니다. 예를 들어, `libs/example/utils/greet.fnl`에 `greet` 모듈을 추가하고 `libs/example/module-a.fnl`에서 해당 모듈을 require 해 보겠습니다.

```fennel
;; file libs/example/utils/greet.fnl:
(fn greet [who] (print (.. "hello " who)))
```

이 모듈은 다음과 같이 require 될 수 있습니다.

```fennel
;; file libs/example/module-a.fnl
(local greet (require (.. (: ... :match "(.+)%.[^.]+") :.utils.greet)))

(fn hello [] (print "hello from a"))

{:hello hello :greet greet}
```

부모 모듈 이름은 현재 모듈 이름 문자열(`...`)에 `match` 메서드를 호출하여 결정됩니다.

[1]: https://stopa.io/post/265
[2]: http://danmidwood.com/content/2014/11/21/animated-paredit.html
[3]: https://www.lua.org/manual/5.4/
[4]: https://github.com/Stepets/utf8.lua
[6]: https://www.lua.org/pil/7.1.html
[7]: https://www.lua.org/manual/5.4/manual.html#pdf-string.gmatch
[8]: https://p.hagelb.org/equal-rights-for-functional-objects.html
[10]: https://www.lua.org/manual/5.4/manual.html#6
[11]: https://leafo.net/posts/itchio-and-coroutines.html
[12]: https://www.lua.org/pil/13.html
[13]: https://love2d.org
[14]: https://fennel-lang.org/lua-primer
[15]: https://www.lua.org/manual/5.3/manual.html#6.5
[16]: https://www.lua.org/pil/14.3.html
[17]: https://fennel-lang.org/api
[18]: https://benaiah.me/posts/everything-you-didnt-want-to-know-about-lua-multivals/
[19]: https://fennel-lang.org/see
