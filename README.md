# call by value vs. call by reference??

어느 날 후배 동료분이 문득 자바와 관련된 질문을 툭 던졌다. 사실 회사에서는 nodeJs로 개발을 하는지라 상당히 뜬금없는 질문이자 고전적인 질문을 던진것이다.

아무래도 json객체를 다루면서 어떤 함수를 거쳤을때 json객체의 값들이 변경되는 것을 보고 물어 본 듯 싶다.

"Call By Value 또는 Pass By Value와 Call By Reference, Pass By Reference의 정확한 의미를 모르겠어요."

'어라? 엄청 뜬금없는 질문이네?'

하고 잠시 생각한 이후에 하나씩 설명을 해주기 시작했다.

# 자바는 Call By Value이다.
자바는 Go (포인터가 존재한다)나 c#, c/c++같은 언어처럼 포인터를 지정할 수 있는 방법이 없다. 이게 무슨 말이냐면

Go를 예로 들어보자.

```
package main

import "fmt"

func testFunc(a int) {
	a = 1
}

func main() {
	fmt.Println("hi! go")

	var a int = 0

	fmt.Println(a)
	testFunc(a)
	fmt.Println(a)

}
실행
basquiatyoonui-MacBookPro:go basquiat$ go run hello.go

```
위와 같이 실행을 하게 되면 기대되는 결과같은 다음과 같다.

```
hi! go
0
0
```

음 당연한거 아닌가?

그런데 다음과 같이 작성을 해보자

```
import "fmt"

func testFunc(a *int) { // *가 이 변수는 레퍼런스라는 것을 표시한다.
	*a = 1
}

func main() {
	fmt.Println("hi! go")

	var a int = 0

	fmt.Println(a)

	testFunc(&a) // &는 a의 레퍼런스를 넘기겠다는 의미이다.
	fmt.Println(a)

}

실행
basquiatyoonui-MacBookPro:go basquiat$ go run hello.go

```   

결과는요??

```
hi! go
0
1

```

어? 바꼈넹?

시작한 언어가 자바였던지라 신입때 선배로부터 이와 관련 이야기를 들었을 때는 엄청 신기했다.

이유는 레퍼런스의 개념이기 때문이다. 이것을 여기서 주절이 푸는 것은 약간 한계가 있다. 일단 내가 굉장히 해박하게 아는게 아니고 그냥 개념 정도로만 이해하고 넘어가기 때문에 디테일한 부분까지 넘어가면 일단 한계가...

하지만 자바나 자바스크립트 (아직 파이썬이나 Rusty는 현재 조금씩 보고 있는지라)의 경우에는 저렇게 할 수 있는 방법이 없다.

위에서 포인터를 쓴 go의 예제는 testFunc를 호출 시 전달되는 해당 파라미터의 주소값을 념겼기 때문에 변경시 주소값도 변경되서 위와 같은 결과가 나온다.

결국 Call By Value 또는 Pass By Value는 메소드나 함수가 호출시에 넘어가는 파라미터는 주소값이 아닌 그 값을 복사해서 넘겨준다고 보면 이해하기 쉽다.

결국 전혀 다른 녀석이라는 의미이다.

하지만 오해의 소지가 있는 경우도 있다. 아마도 이런 부분 때문에 이런 생각을 하게 된다.

"봐요. 이 경우에는 변경되잖아요. 그러니깐 자바도 Call By Reference가 적용되는거 아니에요?"

위에서 말한 상황이 어떤 걸까? 코드로 보자


간단한 Test 객체 하나 만들고

```
package io.basquiat.model;

public class Test {

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Test{" +
                "name='" + name + '\'' +
                '}';
    }
}

```

그리고 다음과 같이


```
package io.basquiat;

import io.basquiat.model.Test;

public class jpaMain {

    public static void change(Test t) {
        t.setName("changed");
    }

    public static void main(String[] args) {
        Test test = new Test();
        test.setName("before");
        System.out.println(test.toString());
        change(test);
        System.out.println(test.toString());
    }
}

```

그럼 실행 결과는요??

```

Test{name='before'}
Test{name='changed'}

``` 

"봐요? 변경되었잖아요?"

음... 진짜 한참을 고민했었다...

'어? 내가 잘못 알고 있었던 것인 것인가???'

그래서 다음과 같이 코드를 짠다.

```
package io.basquiat;

import io.basquiat.model.Test;

public class jpaMain {

     public static void change(Test t) {
        t.setName("changed");
        t = new Test();
        t.setName("new Test");
    }

    public static void main(String[] args) {

        Test test = new Test();
        test.setName("before");
        System.out.println(test.toString());
        change(test);
        System.out.println(test.toString());
    }
}


```

"그럼 후배님 말씀대로라면요 이것이 넘겨 받은 t를 새로운 객체를 만들어서 저렇게 세팅하면 뒤에 나오는 객체는 name이 'new Test'가 되어야겠군요?"

맞다면 밑에 처럼 결과가 나와야 한다.

```
Test{name='before'}
Test{name='new Test'}
```


하지만 생각대로 되지 않는다는 것을 알아야 하는 것인 것이다.

실제 결과는 다음과 같이...

```
Test{name='before'}
Test{name='changed'}
```

"어랍쇼? 그대로네요?"

이런 부분에서 오해를 할 수 있는 경우인데 이 경우에는 change 메소드가 호출되는 시점에 객체인 test는 복사되어 넘겨진게 맞다. 
객체인 test는 '어떻게 보면' reference 타입처럼 넘어간 것처럼 보이지만 실제 내부 메커니즘은 reference의 사본이 넘어간다고 하니 엄밀히 말하면 이것은 Call By Reference가 아니라는 말이다.

솔직히 내가 써놓고도 헛갈린다.

# At A Glance
사실 이건 신입 시절 이후 한번도 고민해 본적이 없었던 것이다. 
그러다가 go에 관심을 가지면서 고민을 해보긴 했지만 현재 업무는 그렇지 않아서 잊고 있었는데 다시 한번 상기한다는 마인드로 흔적을 남기고자 한다.


# 그럼 자바스크립트도 그런 거에요?

넹 그런겁니다.

[repl](https://repl.it)

요기에서 테스트 해볼 수 있다.

```
function test(value) {
  value.key = "change";
  value = {};
}


let a = {'key': 'value'};
console.log(a);
test(a);
console.log(a);

------- 결과는요? -------
{ key: 'value' }
{ key: 'change' }

```
