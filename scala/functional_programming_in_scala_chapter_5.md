## 5.1 More Functions on Lists

이번 챕터에서는 스칼라 List의 다른 메서드 들을 알아본다.
xs는 list의 object를 뜻한다.

#### Sublists and element access
* xs.length			xs의 길이
* xs.last 			xs의 마지막 item return, xs가 비어있으면 exception 발생
* xs.init 			마지막 item을 제외한 list reutnr, xs가 비어있으면 exception 발생
* xs take n 		처음부터 n개의 element의 list 리턴, n이 xs의 length보다 크면 n개만 리턴
* xs drop n 		n개를 제외한 나머지 리스트 리턴
* xs(n)				n번째 item 리턴

#### Creating new lists
* xs ++ ys 			두 list 더하기, :::와 같은 기능을 함
* xs.reverse 		역순의 리스트 생성
* xs updated (n, x) n번째 item만 x로 바뀐 list 생성

#### Finding elements
* xs indexOf x 		x와 같은 첫번째 element의 index 값 리턴, 없으면 -1
* xs contains x 	indexOf x >= 0 과 같음

last가 과연 필요한지 모르겠지만(tail을 recursive하게 반복하면 찾을 수 있음), 유용하게 쓰일 수 있다면 last의 복잡도는 어떻게 될까?
```
def last[T](xs: List[T]): T = xs match {
  case List() => throw new Error("last of empty list")
  case List(x) => x
  case y :: ys => lsat(ys) 	
}
```
위와 같이 list의 길이와 같으므로, 복잡도는 O(n)이 되겠다.
init 메서드는 어떨까?
```
def init[T](xs: List[T]): List[T] = xs match {
  case List() => throw new Error("init of empty list")
  case List(x) => List()
  case y :: ys => y :: init(ys) 	
}
```
마찬가지로 O(n)
그다음은 concat(Same as :::)
```
def concat[T](xs: List[T], ys: List[T]) = xs match {
  case List() => ys
  case z :: zs => z :: concat(zs, ys)	
}
```
복잡도를 |xs|라고 표기하던데 이건 뭘까??
다음은 reverse
```
def reverse[T](xs: List[T]): List[T] = xs match {
  case List() => xs
  case y :: ys => reverse(ys) ++ List(y)
}
```
reverse(ys) :: y 가 아니라 reverse(ys) ++ List(y)인 이유는 ::의 마지막엔 Nil이 와야하니깐 y가 Nil이 아니기 때문이 아닐까 생각한다.
복잡도는 각 요소마다 concatenating을 해주고 list의 length만큼 reverse를 해야하므로 O(n2)이 되겠다. reverse는 다소 실망스러운 성능을 보여주는데, 앞으로 더 개선해보도록 하겠다.

마지막으로 removeAt
```
def removeAt[T](n: Int, xs: List[T]) = (xs take n) ::: (xs drop n+1)
```

## 5.2 Paires and Tuples

앞서 살펴보앗던 insertion sort보다 더 개선된 merge sort 알고리즘에 대해서 살펴보자. 기본적인 개념은 zero or one element 리스트는 이미 sorted 하다는 것.

```
def msort(xs: List[Int]): List[Int] = {
  val n = xs.length/2
  if (n == 0) xs
  else {
    // merge 메서드는 앞으로 더 개선해 나갈 예정임
    def merge(xs: List[Int], ys: List[Int]) = 
      xs mathch {
        case Nil => ys
        case x :: xs1 =>
          ys match {
            case Nil => xs
            case y :: ys1 =>
              if (x < y) x :: merge(xs1, ys)
              else y :: merge(xs, ys1)
          }
      }

    val (fst, snd) = xs splitAt n
    merge(msort(fst), msort(snd))
  }
}
```
밑에서 나오는 splitAt 함수는 index n을 기준으로 리스트를 두개로 쪼개서 리턴한다. 여기서 리턴된 val의 모양을 보자. fst와 snd 두개의 타입으로 묶여져 있다. 이를 Pair라고 한다. 예를 들면
```
val pair = ("answer", 42) > pair: (String, Int) = (answer,42)  

val (label, value) = pare > label: String = answer | value : Int = 42
```
위와 같이 타입으로도 쓰일 수 있고, 패턴으로도 사용될 수 있다. 이때 2개 이상의 요소를 가지면 Tuples라 한다. Tuples는 다양하게 사용될 수 있는데, parameterized type으로 사용될 경우, function applictaion으로 사용될 경우, constructor 패턴으로 사용될 경우 각각 
```
scala.Tuplen[T1, ..., Tn]
scala.Tuplen(e1, ..., en)
scala.Tuplen(p1, ..., pn)
```
과 같이 사용할 수 있다. (여기서 Tuplen의 n은 파라미터 개수 ex. Tuple2)
튜플의 각 element는 _1, _2와 같이 접근할 수 있다.
이제 merge 메소드를 개선해보자.
```
def merge(xs: List[Int], ys: List[Int]): List[Int] = (xs, ys) match {
  case (Nil, ys) => ys
  case (xs, Nil) => xs
  case (x :: xs1, y :: ys1) => 
    if (x < y) x :: merge(xs1, ys)
    else y :: merge(xs, ys1)
}
```
훨씬 깔끔해졌다.


## 5.3 Implicit Parameters

이전 장에서 보았던 msort는 List[Int] 타입으로 지정되어 있는데 parameterize를 통해서 Int 말고도 다른 타입이 들어올 수 있도록 임의의 타입 T로 변경해보자

```
object mergesort {
  def msort[T](xs: List[T]): List[T] = {
    val n = xs.length/2
    if (n == 0) xs
    else {
      // merge 메서드는 앞으로 더 개선해 나갈 예정임
      def merge(xs: List[T], ys: List[T]): List[T] = (xs, ys) match {
        case (Nil, ys) => ys
        case (xs, Nil) => xs
        case (x :: xs1, y :: ys1) =>
          if (x < y) x :: merge(xs1, ys)
          else y :: merge(xs, ys1)
      }

      val (fst, snd) = xs splitAt n
      merge(msort(fst), msort(snd))
    }
  }

  val nums = List(2, -4, 5, 7, 1)
  msort(nums)
}
```
x < y 부분에서 에러가 발생한다. 왜냐하면 comparison '<'가 임의의 타입 T에 정의되어 있지 않기 때문이란다....
그래서 우리는 comparison 함수가 필요하다. 이 때 가장 유연한 방법은 msort 함수에 comparison operation을 추가적인 파라미터로 붙이는 것이다. 아래처럼
```
def msort[T](xs: List[T])(lt: (T, T) => Boolean) = {
	...
	merge(msort(fst)(lt), msort(snd)(lt))
}
```
그래서 원래 mergesort에 적용하면 다음과 같다.
```
object mergesort {
  def msort[T](xs: List[T])(lt: (T, T) => Boolean): List[T] = {
    val n = xs.length/2
    if (n == 0) xs
    else {
      // merge 메서드는 앞으로 더 개선해 나갈 예정임
      def merge(xs: List[T], ys: List[T]): List[T] = (xs, ys) match {
        case (Nil, ys) => ys
        case (xs, Nil) => xs
        case (x :: xs1, y :: ys1) =>
          if (lt(x, y)) x :: merge(xs1, ys)
          else y :: merge(xs, ys1)
      }

      val (fst, snd) = xs splitAt n
      merge(msort(fst)(lt), msort(snd)(lt))
    }
  }

  val nums = List(2, -4, 5, 7, 1)
  msort(nums)((x, y) => x < y)

  val fruits = List("apple", "pineapple", "banana", "orange")
  msort(fruits)((x, y) => x.compareTo(y) < 0)
}
```
이제 Int 타입 뿐만 아니라 String과 같은 다른 타입도 정렬이 가능해졌다. 이 때 lt에 들어오는 함수 파라미터에 타입 붙이는 걸 생략해도 되는데, 컴파일러가 앞에 있는 리스트의 타입을 보고 유추할 수 있기 때문이란다. 즉 파라미터 셋의 마지막에 function value가 들어오게 되면, 컴파일러가 타입 체크를 미뤄버린다.

#### scala.math.Ordering[T]
사실 ordering을 위한 스탠다드 라이브러리 클래스가 있다. 
> scala.math.Ordering[T]
그래서 lt 명령어를 parameterizing 하는 대신 Orderging 클래스로 parameterize 할 수 있다.
```
def msort[T](xs: List[T])(ord: Ordering) = 

  def merge(xs: List[T], ys: List[T]) =
    ... if (ord.lt(x, y)) ...

  ... merge(msort(fst)(ord), msort(snd)(ord)) ...
```

#### implicit
대체로 완성된 느낌이 나지만, Ordering 함수가 처음 콜 될때부터 계속 전달되는게 좀 비효율적으로 보인다. 그래서 여기에다가 또하나를 추가해보자.
ord 파라미터에 implicit(절대적인이란 뜻) 키워드를 앞에 붙여보자. 그러면, 함수를 실제로 호출하는 부분에서 실제 파라미터를 넣어줄 필요가 없다.
```
def msort[T](xs: List[T])(implicit ord: Ordering) = 

  def merge(xs: List[T], ys: List[T]) =
    ... if (ord.lt(x, y)) ...

  ... merge(msort(fst), msort(snd)) ...

val nums = List(2, -4, 5, 7, 1)
msort(nums)
```

#### Rules for Implicit Parameters
더 간결해졌다. 
타입이 T인 implicit 파라미터가 있을때, 컴파일러는 
> (1) implicit이 쓰인 파라미터에 (2) T와 호환되는 타입을 가지고 (3) function call에서 보이거나 T와 관련된 companion 오브젝트에서 
single implicit definition을 찾는다. 즉, Ordering[Int]가 함수 call의 파라미터로 존재하지 않지만, implicit으로 처리되어 어딘가에 존재하게 된다.


## 5.4 Higher-Order List Functions
위에서 보았던 예제들은 종종 비슷한 구조를 보여준다. 요약해보면
* 리스트의 각 element를 변경하는 것
* 어떤 조건을 만족하는 모든 element의 리스트를 구하는 것
* 연산자를 사용하여 element들을 결합하는 것

함수형 언어는 higer-order functinos 패턴을 이용하는 generic function을 만들 수 있다.

첫번째 예제는 각 요소를 multiply 하는 것이다.
```
def scaleList(xs: List[Double], factor: Double): List[Double] = xs match {
  case Nil => xs
  case y :: ys => y * factor :: scaleList(ys, factor)
}
```

#### Map
위 예제는 list의 map 메서드를 이용하여 만들 수 있다. 
map 메서드의 구조를 살펴보면 아래와 같다.
```
abstract class List[T] { ...
  def map[U](f: T => U): List[U] = this match {
    case Nil => this
    case x :: xs => f(x) :: xs.map(f)
  }	
}
```
파라미터로 들어온 함수f가 각 element에 적용되어서 새로운 리스트를 만들어 내는 함수가 바로 map이다. map 메서드를 이용하면 훨씬 간단하게 작성할 수 있다
```
def scaleList(xs: List[Double], factor: Double) =
  xs.map(x => x * factor)
```
또하나의 예제를 살펴보자
```
def squareList(xs: List[Int]): List[Int] = xs match {
  case Nil => Nil
  case y :: ys => y * y :: squareList(ys)
}

def squareList(xs: List[Int]): List[Int] =
  xs map (y => y * y)
```

#### Filtering
필터링은 어떤 조건에 맞는 element를 모아 새로운 리스트를 만들어 내는 메서드이다. 
0보다 큰수만 필터링 하는 다음의 함수를 보자
```
def posElems(xs: List[Int]): List[Int] = xs match {
  case Nil => xs
  case y :: ys => if (y > 0) y :: posElems(ys) else posElems(ys)	
}
```
필터를 이용하면 간단하게 해결할 수 있다. 우선은 filter 메서드가 어떻게 생겼는지부터 살펴보도록 하자.
```
abstract class List[T] {
  ...
  def filter(p: T => Boolean): List[T] = this match {
    case Nil => this
    case x :: xs => if (p(x)) x :: xs.filter(p) else xs.filter(p)
  }	
}
```
필터는 특정조건함수(p)가 true이면 :: 연산자를 이용하여 리스트에 붙이고 false이면 제외하는 방식으로 새로운 리스트를 만들어간다.
그럼 위에서 보았던 posElems를 filter를 이용해 재구성해보자
```
def posElems(xs: List[Int]): List[Int] = 
  xs filter(x => x > 0)
```

그외에 유용한 메서드 목록은 아래와 같다.
* xs filterNot p 	xs filter (x => !p(x))와 같다.
* xs partition p 	(xs filter p, xs filterNot) 튜플
* xs takeWhile p 	p를 만족하는 요소들의 가장 긴 리스트 
* xs dropWhile p 	p를 만족하는 요소들의 나머지
* xs span p   		(xs takeWhile p, xs dropWhile p) 튜플

예를 들어보자
```
scala> val nums = List(2, -4, 5, 7, 1)
nums: List[Int] = List(2, -4, 5, 7, 1)

scala> nums filter (x => x > 0)
res0: List[Int] = List(2, 5, 7, 1)

scala> nums filterNot (x => x > 0)
res1: List[Int] = List(-4)

scala> nums partition (x => x > 0)
res2: (List[Int], List[Int]) = (List(2, 5, 7, 1),List(-4))

scala> nums takeWhile (x => x > 0)
res3: List[Int] = List(2)

scala> nums dropWhile (x => x > 0)
res4: List[Int] = List(-4, 5, 7, 1)

scala> nums span (x => x > 0)
res5: (List[Int], List[Int]) = (List(2),List(-4, 5, 7, 1))
```

## 5.5 Reductino of Lists

