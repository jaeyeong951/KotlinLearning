# Kotlin 4. 클래스, 객체, 인터페이스

코틀린의 클래스와 인터페이스는 자바의 그것과는 약간 다르다. 코틀린은 인터페이스에 프로퍼티 선언이 들어갈 수 있다. 그리고 자바와 달리 코틀린은 기본 선언이 final이며 public이다. 게다가 중첩 클래스는 기본적으로 내부 클래스가 아니기 떄문에 중첨 클래스에는 외부 클래스에 대한 참조가 없다. (inner로 명시해야함)

코틀린 컴파일러는 번잡스러움을 피하기 위해 유용한 메소드를 자동으로 만들어준다. 클래스 선언 앞에 data 키워드를 붙이면 컴파일러가 일부 표준 메소드를 생성해준다. 그리고 코틀린이 제공하는 위임(delegation)을 사용하면 위임을 처리하기 위한 준비 메소드를 직접 작성할 필요가 없다.

그리고 코틀린에는 object 키워드가 존재하는데 싱글턴 클래스, 동반 객체(companion object), 객체 식(object expression)(자바의 anonymous class)을 표현할 때 object 키워드를 쓴다. 

이번 장에서 모두 자세히 알아보자.

## 클래스 계층 정의

### 코틀린 인터페이스

코틀린의 인터페이스는 자바 8버전의 인터페이스와 비슷하다. 코틀린의 인터페이스 안에는 추상 메소드뿐만 아니라 구현이 있는 메소드도 정의할 수 있다(자바 8의 디폴트 메소드). 다만 인터페이스에는 아무런 상태(필드)도 들어갈 수 없다.

```kotlin
interface Clickable {
	fun click()
}
```

위 코드는 click()이라는 추상 메소드를 가진 인터페이스를 정의한 것이다. 이 인터페이스를 구현하는 모든 구체적 클래스는 click()에 해단 구현을 제공해야 한다.

```kotlin
class Button : Clickable {
	override fun click() = println("I was clicked")
}
```

자바에서는 extends로 클래스의 확장을, implements로 인터페이스의 구현을 처리하지만 코틀린은 뒤에 콜론(:)으로 모두 처리한다. 자바와 마찬가지로 인터페이스는 개수 제한 없이 구현할 수 있지만, 클래스의 확장은 오직 하나만 가능하다.

디폴트 메소드를 제공하기 위해 자바의 경우 default 키워드를 메소드 앞에 붙여야 했지만 코틀린은 그럴 필요가 없다. 클래스 메소드 구현하듯이 본문을 추가하면 된다.

```kotlin
interface Clickable {
	fun click()

	fun showOff() = println("I'm clickable!")
}
```

위 인터페이스를 구현하는 클래스는 click()에 대한 구현을 제공해야 하지만 showOff()의 경우 새로운 동작을 정의할 수도 있고 그냥 정의를 생략해서 디폴트 구현을 사용할 수도 있다.

```kotlin
interface Clickable {
	fun click()

	fun showOff() = println("I'm clickable!")
}

interface Focusable {
	fun showOff() = println("I'm focusable!")
}
```

만약 위같이 같은 이름의 디폴트 메소드가 구현되어있는 두개의 인터페이스를 하나의 클래스가 구현하면 어떻게 될까? 답은 어느 쪽도 선택되지 않고 다음과 같은 컴팡일러 오류가 발생한다.

`The class 'Button' must override public open fun showOff() because it inherits many implementations of it.`

코틀린 컴파일러는 두 메소드를 아우르는 구현을 하위 클래스에 직접 구현하게 강제한다.

```kotlin
class Button : Clickable, Focusable {
	override fun click() = println("I was clicked")
	override fun showOff() {
		/*
		이름과 시그니처가 같은 멤버 메소드에 대해 둘 이상의 디폴트 구현이 있는 경우
	 인터페이스를 구현하는 하위 클래스에서 명시적으로 새로운 구현을 제공해야 한다.
		*/
		super<Clickable>.showOff()
		super<Focusable>.showOff()
	}
}
```

### open, final, abstract : 기본적으로 final

자바에서는 final 키워드를 사용해 명시적으로 상속을 금지하지 않는 이상 모든 클래스를 다른 클래스가 상속할 수 있다. 이는 편리해 보이지만 문제가 생기는 경우도 많다.

취약한 기반 클래스(fragile base class)라는 문제에 대해 들어본 적이 있는가? 하위 클래스가 기반 클래스에 대해 가졌던 가정이 기반 클래스의 변경으로 인해 깨져버린 경우에 발생하는 문제를 일컫는다. 

어떤 클래스가 자신을 상속하는 방법에 대해 정확한 규칙(어떤 메소드를 어떻게 오버라이드해야 하는지 등)을 명세하지 않는다면 그 클래스의 클라이언트는 기반 클래스를 작성한 사람의 의도와 다른 방식으로 메소드를 오버라이드할 위험이 있다. 상속하는 모든 하위 클래스를 분석하는 것은 불가능하므로 기반 클래스를 변경하는 경우 하위 클래스의 동작이 예기치 않게 바뀔 수도 있다는 면에서 기반 클래스는 '취약'하다.

이 문제를 해결하기 위해 자바 프로그래밍 기법에 대한 책 중 가장 유명한 책인 조슈아 블로크의 Effective Java 에서는 "상속을 위한 설계와 문서를 갖추거나, 그럴 수 없다면 상속을 금지하라"는 조언을 한다. 이는 곧 하위 클래스에서 오버라이드하도록 의도된 클래스나 메소드가 아니라면 모두 final로 만들라는 뜻이다.

코틀린은 이 철학을 그대로 따른다. 그래서 코틀린의 모든 클래스와 메소드는 기본적으로 final이다. 그래서 어떤 클래스의 상속을 허용하려면 클래스 앞에 open 변경자를 붙여야 한다. 더불어 오버라이드를 허용하고 싶은 메소드나 프로퍼티 앞에서 open 변경자를 붙여야만 한다.

```kotlin
open class RichButton : Clickable {
	fun disable() { /* ... */ } // final 함수이므로 하위 클래스가 오버라이드할 수 없다.
	open fun animate() { /* ... */ } // open 함수이므로 하위 클래스가 오버라이드할 수 있다.
	override fun click() { /* ... */ } // 상위 클래스에서 선언된 open 함수를 오버라이드 한 것. 
	// 오버라이드한 메소드는 기본적으로 open이다. 오버라이드한 메소드의 구현을 하위 클래스에서 못하게 하려면 
	// final앞에 final을 명시해야한다.
}
```

```kotlin
open class RichButton : Clickable {
	final override fun click() { /* ... */ } 
	// 위 final은 쓸데 없는 중복이 아니다.
	// override한 메소드는 기본적으로 open이다.
}
```

### 가시성 변경자 : 기본적으로 public

코틀린의 가시성 변경자는 자바와 비슷하게 public, protected, private 가 존재한다. 하지만 코틀린의 기본 가시성은(아무 변경자도 선언하지 않은 경우) 자바와 달리 public이다.
자바의 기본 가시성인 package-private은 코틀린에 없다. 코틀린은 패키지를 namespace를 관리하기 위한 용도로만 사용한다.

package-private의 대안으로 코틀린에서는 internal이라는 새로운 변경자를 도입했다. internal 변경자는 같은 모듈 내부에서만 볼 수 있다는 뜻으로 모듈은 한 번에 컴파일되는 코틀린 파일들을 의미한다. 
또한 코틀린은 최상위 선언에 대해 private 가시성을 허용한다. 그런 최상위 선언에는 클래스, 함수, 프로퍼티 등이 포함되며 이들은 그 선언이 들어있는 파일 내부에서만 사용할 수 있다. 이 또한 하위 시스템의 자세한 구현 사항을 외부에 감추고 싶을 때 유용한 방법이다.

protected 멤버는 오직 어떤 클래스나 그 클래스를 상속한 클래스 안에서만 보인다. 

클래스를 확장한 확장 함수는 그 클래스의 private이나 protected 멤버에 접근할 수 없다.

코틀린과 자바의 가시성 규칙의 또 다른 차이는 코틀린에서는 외부 클래스가 내부 클래스나 중첩된 클래스의 private 멤버에 접근할 수 없다는 점이다.

### inner 클래스와 nested 클래스 : 기본적으로 nested

자바처럼 코틀린에서도 클래스안에 다른 클래스를 선언할 수 있다. 차이는 코틀린의 중첩 클래스(nested class)는 명시적으로 요청하지 않는 한 바깥쪽 클래스에 대한 접근 권한이 없다는 점이다.

View 클래스가 있고 View의 상태를 직렬화해야 한다고 하자. View 자체를 직렬화하는 일은 어렵기 때문에 필요한 모든 데이터를 직렬화 가능한 다른 도우미 클래스로 복사할 수는 있다. 이를 위해 State 인터페이스를 선언하고 Serializable을 구현한다.

```kotlin
interface State : Serializable

interface View {
	fun getCurrentState() : State
	fun restoreState(state: State)
}
```

Button 클래스의 상태를 저장하는 클래스는 Button 클래스의 내부에 선언하면 편하다. 자바로 먼저 살펴본다.

```kotlin
/* 자바 */
public class Button implements View {
	@Override
	public State getCurrentState() {
		return new ButtonState();
	}
	
	@Override
	public void restoreState(State state) { /* ... */ }

	public class ButtonState implements State {
		/* 버튼에 대한 정보 저장... */
	}
}
```

State 인터페이스를 구현한 ButtonState 클래스를 내부에 정의해서 Button에 대한 구체적인 정보를 저장한다.

하지만 위의 코드는 잘못되었다. 위와 같이 선언한 버튼의 상태를 직렬화 하면 java.io.NotSerializableException: Button 이라는 오류가 발생한다. 직렬화하려는 변수는 ButtonState 타입의 state였는데 왜 Button을 직렬화 할 수 없다는 예외가 발생할까?

자바에서는 클래스안에 정의한 클래스는 자동으로 내부 클래스(inner class)가 된다. 그래서 ButtonState 클래스는 바깥쪽 Button 클래스에 대한 참조를 포함한다. 그 참조로 인해 ButtonState를 직렬화할 수 없는 것이다. 직렬화할 수 없는 Button 객체에 대한 참조가 ButtonState에 대한 직렬화를 방해한 것이다.

이 문제를 해결하려면 ButtonState를 static 클래스로 선언해야한다. 중첩 클래스를 static으로 선언하면 외부 클래스에 대한 참조가 사라지기 때문이다. 

코틀린에서 중첩 클래스가 작동하는 방식은 위의 자바와 정반대다.

```kotlin
class Button : View {
	override fun getCurrentState(): State = ButtonState()
	
	override fun restoreState(state: State) { /* ... */}

	class ButtonState : State {
		/* 버튼에 대한 정보 저장... */
	}
}
```

코틀린에서 중첩 클래스에 아무런 변경자가 붙지 않으면 자바의 static 중첩 클래스와 동일하다. 이를 내부 클래스로 변경해서 바깥쪽 클래스에 대한 참조를 포함하게 만들고 싶다면 inner 변경자를 붙여야한다. 

### Sealed class : 클래스 계층 정의 시 계층 확장 제한

코틀린의 sealed class는 부모 클래스를 상속하는 자식 클래스의 종류를 제한한다.

그래서 하위 클래스 분기 처리를 위해 when을 사용시 else가 필요없다.

```kotlin
interface Expr
class Num(val value: Int) : Expr
class Sum(cal left: Expr, val right: Expr) : Expr

fun eval(e: Expr) : Int = 
	when(e){
		is Num -> e.value
		is Sum -> eval(e.right) + eval(e.left)
		else -> throw IllegalArgumentException("Unknown expression")
		// else 분기 강제
	}
```

하지만 위처럼 interface를 when 분기에 적용하면 컴파일러는 해당 interface를 구현하는 구현체가 얼마나 될지 가늠할 수 없기때문에 디폴트 분기인 else 분기를 덧붙이도록 강제한다.

그리고 디폴트 else 분기가 존재하면 새로운 하위 클래스를 중간에 추가하고 when구문에 추가하는 걸 잊는 경우 디폴트 분기가 선택되기 때문에 심각한 버그를 야기할 수 있다.

코틀린은 이러한 문제를 해결하기 위해 sealed 클래스를 제공했다.

```kotlin
sealed class Expr {
	class Num(val value: Int) : Expr()
	class Sum(cal left: Expr, val right: Expr) : Expr()
}

fun eval(e: Expr) : Int = 
	when(e){
		is Num -> e.value
		is Sum -> eval(e.right) + eval(e.left)
	}
```

위처럼 when식에서 sealed 클래스인 Expr의 모든 하위 클래스를 처리한다면 디폴트 else 분기가 필요없다. 

Expr에 새로운 하위 클래스를 추가하고 when 분기에 처리하는 것을 깜박하더라도 컴파일이 되지 않기때문에 실수할 여지도 없다.

sealed 클래스는 자동으로 open이며 private 생성자만을 갖는다.

## 주 생성자와 부 생성자

코틀린도 자바처럼 생성자를 하나 이상 선언할 수 있지만 자바와 다르게 주(primary) 생성자와 부(secondary) 생성자를 구분한다. 또한 코틀린은 초기화 블록(initializer block)을 통해 초기화 로직을 추가할 수 있다.

### 주 생성자와 초기화 블록

```kotlin
class User(val nickname: String)

class User constructor (_nickname: String) {
	val nickname: String
	init {
		nickname = _nickname
	}
}

class User(_nickname: String) {
	val nickname = _nickname
}
```

위의 세가지 클래스 선언은 완전히 동일한 의미이다. 맨 위의 간결한 표현을 풀어서 쓰면 아래 두 표현과 같다.

클래스 이름 바로 뒤의 괄호로 둘러싸인 코드를 주 생성자라고 부른다. 

주 생성자는 생성자 파라미터를 생성하고 그 생성자 파라미터에 의해 초기화되는 프로퍼티를 정의하는 두 가지 역할을 한다.

init 키워드로 시작하는 초기화 블록에는 클래스의 객체가 만들어질 때(인스턴스화될 때) 실행될 초기화 코드가 들어간다. 코틀린은 주 생성자에 어떠한 코드도 들어갈 수 없기에 만들어진 블록이다. 필요에 따라 여러 초기화 블록을 선언하는 것도 가능하다. 

클래스의 본문에서 val 키워드로 프로퍼티를 정의하고 주 생성자의 파라미터로 초기화하는 것도 가능하지만, 생성자 파라미터의 앞에 val을 추가하는 방식으로 프로퍼티의 정의와 초기화를 한번에 수행할 수 있다.

```kotlin
class User(val nickname: String) -> "val"은 이 파라미터에 상응하는 프로퍼티를 생성
```

함수 파라미터와 마찬가지로 클래스의 생성자 파라미터에도 디폴트 값을 정의할 수 있다.

```kotlin
class User(val nickname: String, val isSubscribed: Boolean = true)
```

클래스의 인스턴스를 만들기 위해 new 키워드를 쓸 필요없다. 그저 생성자를 호출하기만 하면 된다.

어떤 클래스를 상속하는 자식 클래스를 정의하려면 주 생성자에서 기반 클래스의 생성자를 호출해야한다. 

```kotlin
open class User(val nickname: String) { ... }
class TwitterUser(nickname: String) : User(nickname) { ... } 
-> 부모 클래스인 User의 생성자 호출
```

이 규칙으로 인해 기반 클래스의 이름 뒤에는 꼭 빈 괄호가 들어가거나 인자가 있는 괄호가 들어간다.

반면 인터페이스는 생성자가 없기 때문에 어떤 클래스가 인터페이스를 구현하는 경우 아무 괄호도 붙지 않는다.

그래서 클래스의 정의를 볼 때 기반 클래스 이름 뒤에 괄호가 있는 없는지를 보면 쉽게 기반 클래스와 인터페이스를 구분할 수 있다. → 자바처럼 implements, extends 로 구분하지 않고 ":" 로 통일한 이유

### Private Constructor

어떤 클래스를 클래스 외부에서 인스턴스화하지 못하게 막고 싶다면 모든 생성자를 private으로 만들면 된다. 다음과 같이 주 생성자에 private 변경자를 붙일 수 있다.

```kotlin
class Secretive private constructor() {}
// 클래스의 유일한 주 생성자가 private
```

Secretive 클래스는 주 생성자밖에 없고 주 생성자는 private이다. 그래서 외부에서 Secretive 클래스를 인스턴스화할 수 없다.  companion object 안에서 이런 private 생성자를 호출하면 좋다.

자바의 경우 유틸리티 함수들을 담아두기 위해 인스턴스화 할 필요가 없는 정적 유틸리티 클래스를 정의해야했고, 싱글턴 클래스는 미리 정한 팩토리 메소드 등의 생성 방법을 통해서만 객체를 생성해야 했다. 이 경우 자바는 private 생성자를 정의해서 클래스를 다른 곳에서는 인스턴스화하지 못하게 막았다. 코틀린은 이런 경우를 언어에서 기본적으로 지원한다. 
정적 유틸리티 함수를 담는 클래스는 필요없다. 최상위 함수를 사용하면 된다.
싱글턴 클래스를 만들고 싶으면 object를 선언하면 된다.

### 부 생성자

코틀린은 생성자를 여러개 생성하는 경우가 자바보다 훨씬 적다. 디폴트 파라미터를 이용하면 굳이 생성자를 여러개 만들 필요가 없기 때문이다. 

주 생성자는 constructor 키워드를 생략할 수 있지만 부 생성자는 그럴수 없습니다.

부 생성자의 중요한 특징으로는 주 생성자가 존재할 경우 부 생성자는 반드시 주 생성자에게 생성자를 위임해야합니다.

```kotlin
class Person(val name: String) {

    var age: Int = 20
    var height: Int = 500

//    컴파일 에러
//    constructor(name: String, age: Int) {
//        this.age = age
//    }

    constructor(name: String, age: Int) : this(name) {
        this.age = age
    }

    constructor(name: String, age: Int, height: Int) : this(name, age) {
        this.height = height
    }
}
```

### 인터페이스에 프로퍼티 선언

코틀린은 인터페이스에 추상 프로퍼티를 선언할 수 있다.

```kotlin
interface User {
	val nickname: String
}
```

위는 User 인터페이스를 구현하는 클래스는 nickname의 값을 제공해야한다는 뜻이다. 
User 인터페이스를 구현하는 여러가지 방법을 보자.

```kotlin
class PrivateUser(override val nickname: String) : User // 주 생성자에 프로퍼티 선언

class SubscribingUser(val email: String) : User {
	override val nickname: String
		get() = email.substringBefore('@') // 커스텀 게터
}

class FacebookUser(val accountId: Int) : User {
	override val nickname = getFacebookName(accountId) // 프로퍼티 초기화 식
}
```

PrivateUser는 주 생성자 안에 프로퍼티를 직접 선언하는 간결한 구문을 사용했다.

SubscribingUser는 커스텀 게터를 사용해 nickname 프로퍼티를 정의한다. 이 프로퍼티는 backing field에 갑을 저장해놓지 않고 매번 email 스트링에서 nickname을 계산해 값을 반환한다.

FacebookUser는 getFacebookUser 메소드를 호출해서 nickname을 초기화한다. getFaceBookUser 메소드는 비용이 많이 들 수 있기 때문에 객체를 초기화하는 단계에 한 번만 호출하게 설계했다.

SubscribingUser의 nickname은 호출될 때 마다 매번 계산하는 커스텀 게터를 사용했고, FacebookUser의 nickname은 객체 초기화 시 계산한 값을 backing field에 저장했다가 불러오는 방식을 사용했음에 유의하자.

그리고 인터페이스는 아래 코드처럼 추상 프로퍼티뿐만 아니라 게터와 세터가 있는 프로퍼티를 선언할 수 있다. 하지만 당연히 인터페이스의 게터와 세터는 backing field를 참조할 수 없다. 인터페이스는 상태를 저장할 수 없기 때문이다. 대신 매번 결과를 계산해 반환하는 방식을 사용하자.

```kotlin
interface User {
	val email: String
	val nickname: String
		get() = email.substringBefore('@')
}
```

인터페이스와 달리 클래스의 프로퍼티는 backing field에 마음대로 접근하고 사용할 수 있다.

### 게터와 세터의 backing field 사용

위에서 프로퍼티의 두 가지 유형에 대해 살펴봤다. 이 두 유형을 조합해 프로퍼티의 값을 변경하거나 읽을 때 마다 어떠한 로직을 실행할 수 있도록 만들어본다.

프로퍼티의 값이 변경될 때 마다 그 이력을 로그에 남길수 있도록 만들고 싶다. 이 경우 변경 가능한 프로퍼티를 정의하고 프로퍼티의 값이 바뀔 때 마다 약간의 코드가 추가로 실행되어야 한다.

```kotlin
class User(val name: String) {
	var address: String = "unspecified"
		set(value: String) {
			println("""
				Address was changed for ${name}":
				"${field}" -> "${value}.""".trimIndent()) // 기존 backing field를 읽고 원하는 작업을 수행
			field = value // backing field의 값 변경(set 로직)
		}
}

>>> val user = User("Alice")
>>> user.address = "Emiles 47, 29830 Muil"
Address was changed for Alice:
"unspecified" -> "Emiles 47, 29830 Muil".
```

코틀린에서 프로퍼티의 값을 바꿀 때는 user.address = "new value" 처럼 사용한다. 이 코드는 내부적으로는 address의 세터를 호출한다.

위에서 field라는 특별한 식별자를 통해 backing field에 접근했다. 게터에서는 field 값을 읽기만, 세터에서는 field 값을 읽거나 쓸 수 있다.

### 접근자의 가시성 변경

접근자의 가시성은 따로 작업을 하지 않는다면 프로퍼티의 가시성과 같다. 하지만 get이나 set앞에 가시성 변경자를 추가하여 접근자의 가시성을 변경할 수 있다.

```kotlin
class LengthCounter {
	var counter: Int = 0
		private set

	fun addWord(word: String) {
		counter += word.length
	}
}
```

위처럼 세터를 private로 선언하면 해당 프로퍼티는 클래스 내부에서만 변경이 가능하고 클래스 외부에서는 읽기만 가능하다.

## data class와 클래스 위임(delegation)

자바에서는 클래스가 equals, hashCode, toString 등의 메소드를 직접 구현해야한다. 코틀린 컴파일러는 이런 메소드를 자동으로 보이지 않는 곳에서 해준다. 

### 모든 클래스가 정의해야 하는 메소드

자바와 마찬가지로 코틀린의 클래스로 toString, equals, hashCode 등을 오버라이드할 수 있다.

```kotlin
class Client(val name: String, val postalCode: Int)
```

### toString() : 문자열 표현

코틀린에서 기본적으로 제공되는 toString 메소드는 Client@5e3493d 와 같은 형식인데 이는 그다지 유용하지 않다. toString 메소드를 오버라이드하자.

```kotlin
class Client(val name: String, val postalCode: Int) {
	override fun toString() = "Client(name = ${name}, postalCode = ${postalCode})"
}
```

### equals() : 객체의 동등성

코틀린에서 == 연산자는 자바와 달리 참조 동일성을 검사하지 않고 객체의 동등성을 검사한다. 따라서 == 연산은 내부적으로 equals 메소드를 호출하는 식으로 컴파일된다. 참조 비교를 위해서는 === 연산자를 사용한다.

```kotlin
>>> val client1 = Client("David", 4122)
>>> val client2 = Client("David", 4122)
>>> println(client1 == client2)
false
```

위 예제에서는 두 객체가 동일하지 않다. 우리가 원하는 결과가 나오려면 equals 메소드를 오버라이드 하여 수정해야한다.

```kotlin
class Client(val name: String, val postalCode: Int) {
	override fun equals(others: Any?): Boolean {
		if(other == null || other !is Client)
			return false
		return name == other.name && postalCode == other.postalCode
	}

	override fun toString() = "Client(name = ${name}, postalCode = ${postalCode})"
}
```

Any는 java.lang.Object에 대응하는 클래스로, 코틀린의 모든 클래스의 최상위 클래스다. 코틀린의 is 메소드는 자바의 instanceOf와 같다. 위에서는 is 메소드로 Client 인지 검사한다. 

위 처럼 작성하면 프로퍼티 값이 모두 같은 두 Client 객체는 동등하다. 하지만 더 복잡한 작업을 수행해보면 제대로 작동하지 않는 경우가 있다. hashCode의 정의를 빠뜨려서 그렇다.

### hashCode() : 해시 컨테이너

자바에서는 equals 메소드를 오버라이드할 때 만드시 hashCode도 함께 오버라이드하도록 강제한다. 그 이유는 무엇일까?

Client로 이루어진 집합으로 실험해보자.

```kotlin
>>> val processed = hashSetOf(Client("David", 4122))
>>> println(processed.contatins(Client("David", 4122))
false
```

프로퍼티가 완전히 같은 새 인스턴스와 기존 인스턴스는 동등함에도 속했는지 여부를 검사하면 false가 나온다. 이는 Client 클래스가 hashCode 메소드를 정의하지 않았기 때문이다.

JVM 언어에서는 "equals()가 true를 반환하는 두 객체는 반드시 같은 hashCode()를 반환해야 한다"라는 제약이 있는데 Client 클래스는 이를 어기고있다. 위의 예제의 경우 두 Client 인스턴스는 해시코드가 다르기 때문에 저런 결과가 나온 것이다. Client 클래스에 hashCode 메소드를 구현해보자.

```kotlin
class Client(val name: String, val postalCode: Int) {
	override fun hashCode(): Int = name.hashCode() * 31 + postalCode

	override fun equals(others: Any?): Boolean {
		if(other == null || other !is Client)
			return false
		return name == other.name && postalCode == other.postalCode
	}

	override fun toString() = "Client(name = ${name}, postalCode = ${postalCode})"
}
```

비로소 이 클래스는 예상대로 작동한다. 하지만 너무 성가신 작업이다. 다행히도 코틀린 컴파일러는 이 모든 메소드를 자동으로 생성해줄 수 있다. 

### data class : 필수적인 메소드 자동 생성

어떤 클래스가 데이터를 저장하는 역할만을 수행한다면 toString, equals, hashCode 메소드를 반드시 오버라이드해야 한다. 코틀린은 data라는 키워드를 class 앞에 붙이면 이런 메소드를 컴파일러가 자동으로 만들어준다. data 키워드가 붙은 클래스를 data class 라고 부른다.

위의 Client를 데이터 클래스로 선언해보자.

```kotlin
data class Client(val name, val postalCode: Int)
```

위의 10줄짜리 코드와 동일한 기능을 하는 1줄짜리 코드이다.

위 1줄짜리 코드는 

- 인스턴스 간 비교를 위한 equals
- 해시 기반 컨테이너에서 키로 사용할 수 있는 hashCode
- 클래스의 각 필드를 선언 순서대로 표시하는 문자열을 만들어주는 toString

를 모두 포함한다.

equals와 hashCode는 주 생성자에 나열된 모든 프로퍼티를 고려해 만들어진다. 주 생성자 밖에 전의된 프로퍼티는 equals나 hashCode를 계산할 때 고려의 대상이 아니라는 점에 유의하자.

### data class와 불변성 : copy()

데이터 클래스의 프로퍼티가 꼭 val일 필요는 없다. var도 사용 가능 하지만 데이터 클래스의 모든 프로퍼티를 읽기 전용으로 만들어서 데이터 클래스를 불변(immutable)클래스로 만들기를 권장한다.

데이터 클래스 객체를 키값으로 하는 컨테이너에 담은 다음 객체의 프로퍼티를 변경하면 상태가 잘못될 수 있다. 게다가 불변 객체를 사용하면 다중 스레드 프로그램에서 훨씬 안정적이다.

불변 데이터 클래스 객체를 훨씬 유용하게 활용하기 위해 copy() 메소드가 존재한다.

copy() 메소드는 객체를 복사하면서 일부 프로퍼티를 바꿀 수 있게 해준다. 객체를 메모리상에서 직접 바꾸지 않고 복사본을 만드는 편이 낫다. 복사를 하면서 일부 프로퍼티를 바꾸거나 복사본을 제거해도 프로그램에서 원본을 참조하는 다른 부분에 전혀 영향을 끼치지 않는다.

```kotlin
val Client1 = Client("David", 4122)
val Client1_revised = Client1.copy(postalCode = 4000)
```

### by : 클래스 위임(delegation)
