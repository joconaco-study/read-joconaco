# Chap8. 코드를 모듈화 하라.

<br>

## Description

---

- 모듈화된 코드의 이점
- 이상적인 코드 모듈화가 되지 않는 일반적인 방식
- 코드를 좀 더 모듈화하기 위한 방법

모듈화의 주된 목적 중 하나는 코드가 향후에 어떻게 변경되거나 재구성 될지 정확히
알지 못한 상태에서 변경과 재구성이 용이한 코드를 작성하는 것이다.

<br>
<br>
<br>

## ❐ 8.1 의존성 주입을 고려하라.

### 8.1.1 하드 코드화된 의존성은 문제가 될 수 있다. (구현체 의존)
```java
public class CarService {
    private final MoveStrategy moveForwardStrategy; // 구현체 의존!
    
    //...
}
```
이렇게 DI를 할 경우 뒤로 이동, 옆으로 이동과 같은 추가 요구사항을 대응하는데 있어 큰 걸림돌이 된다.

### 8.1.2 해결책 : 의존성 주입을 사용하라
```java
public class CarService {
    private final MoveStrategy moveStrategy;
  
    //...
}
```
이렇게 DI를 하면 CarService 클래스의 생성자가 좀 더 복잡해진다는 단점이 있다.

이런 경우 MoveStrategyFactoryFactory 를 만들어줘야 한다. 최종적으로 의존성 주입 프레임워크를<br> 
사용하면 의존성 주입과 관련되 많은 작업을 수동으로 하지 않아도 되기 때문에 개발 작업이 쉬워진다.

<br>

### 8.1.3 의존성 주입을 염두에 두고 코드를 설계하라.
코드를 작성할 때 의존성 주입을 사용할 수 있다는 점을 의식적으로 고려하는 것이 유용할 때가 있다.
이번 장에 `정적 매달림`이라는 용어가 나왔는데 p250 참고하시면 될 것 같습니다.

<br>
<br>
<br>

## ❐ 8.2 인터페이스에 의존하라
### 8.2.1 구체적인 구현에 의존하면 적응성이 제한된다.
> 8.1.1과 동일한 내용


### 8.2.2 해결책 : 가능한 경우 인터페이스에 의존하라
> 8.1.2와 동일한 내용

<br>
<br>
<br>

## ❐ 8.3 클래스 상속을 주의하라

### 8.3.1 클래스 상속은 문제가 될 수 있다.
```java
public interface FileValueReader {
    Optional<String> getNextValue();
    void close();
}

---

public interface FileValueWriter {

    void writeValue(String value);
    void close();
}

---

/**
 * 쉼표로 구분된 값을 가지고 있는 파일을 읽거나 쓰기 위한 클래스
 */
public class CsvFileHandler implements FileValueReader, FileValueWriter {

    public CsvFileHandler(File file) { ... }

    @Override
    public Optional<String> getNextValue() { ... }

    @Override
    public void writeValue(String value) { ... }

    @Override
    public void close() { ... }
}
```

```java
// 서브클래스인 IntegerFileReader는 슈퍼클래스인 CsvFileHandler를 '확장'한다.
public class IntegerFileReader extends CsvFileHandler {

    // IntegerFileReader 생성자는 슈퍼 클래스인 CsvFileHandler의 생성자를 호출한다.
    public IntegerFileReader(File file) {
        super(file);
    }

    public Optional<Integer> getNextInteger() {
        // 슈퍼클래스인 CsvFileHandler로 부터 getNextValue() 메소드를 호출한다.
        Optional<String> nextValue = getNextValue();
        if (nextValue.isEmpty()) {
            return Optional.empty();
        }
        return Integer.parseInt(nextValue, Radix.BASE_10);
    }
}

```
<br>

#### **상속은 추상화 계층에 방해가 될 수 있다.**

한 클래스가 다른 클래스를 확장(상속)하면 슈퍼 클래스의 모든 기능을 사용할 수 있다.<br>
위에서 본`close()`메소드를 사용하는 경우처럼 유용할 때도 있지만, 의도한 것보다 더 많은 기능이 노출될 수 있다.<br> 
CsvFileHandler를 구현한 개발자는 public인 서브 클래스의 메소드인 `getNextInteger()`메소드와 외부에서 사용하는<br> 
슈퍼 클래스의`close()`메소드만 외부에 노출되기를 기대한다. 하지만, 외부에 노출되는 정보는 다음과 같다.

1. getNextInteger() : 외부에서 사용하는 서브 클래스의 메소드
2. close() : 외부에서 사용하는 슈퍼 클래스 메소드
3. getNextValue() : 외부에서 사용하지 않는 슈퍼 클래스의 메소드 1
4. writeValue() : 외부에서 사용하지 않는 슈퍼 클래스의 메소드 2

추상화 계층에서 IntegerFileReader를 사용하는 입장에서는 상위 계층에 대한 정보를 몰라야 한다.<br>
하지만 이러한 정보들이 상속으로 인해 하위 계층으로 노출된다.

<br>

#### 상속은 적응성 높은 코드의 작성을 어렵게 만들 수 있다.

추가 요구사항 ;세미콜론으로 구분된 값도 읽을 수 있어야 한다.
하지만 이미 세미콜론으로 구분할 수 있는 클래스가 존재한다.
```java
/**
 * 세미콜론으로 구분된 값을 가지고 있는 파일을 읽거나 쓰기 위한 클래스
 */
public class SemicolonFileHandler implements FileValueReader, FileValueWriter {

    public SemicolonFileHandler(File file) { ... }

    @Override
    public Optional<String> getNextValue() { ... }

    @Override
    public void writeValue(String value) { ... }

    @Override
    public void close() { ... }
}
```

하지만 중복되는 코드가 많이 보인다. 이는 자바가 다중 상속을 지원하지 않기 때문이다.

<br>

### 8.3.2 해결책 : 구성(Composition)을 사용하라

```java
public class IntFileReader {

    // IntFileReader는 FileValueReader 인터페이스를 구현하는 클래스의 인스턴스를 갖는다.
    private final FileValueReader valueReader;
  
    public IntFileReader(FileValueReader valueReader) {
        this.valueReader = valueReader;
    }
  
    //...	
}
```
상속 대신 구성을 사용하면,
- 코드 재사용의 이점을 얻을 수 있고, 상속과 관련된 문제도 피할 수 있다.
- 전달이나 위임을 사용하지 않는한 CsvFileHandler 클래스의 기능이 외부로 기능이 노출되지 않는다.
- 앞선 요구사항에 큰 무리 없이 대응할 수 있다.

<br>

### 8.3.3 진정한 is-a 관계는 어떤가?

- Car와 Mustang은 완벽한 is-a 관계가 맞음
- IntFileReader와 CsvFileHandler는 is-a 관계가 아님

Car와 Mustang과 같이 완벽한 is-a 관계가 맞더라도 상속은 여전히 문제가 됨.

1. 취약한 베이스 클래스 문제
   - 슈퍼 클래스가 변경되면 서브 클래스도 변경해야 한다.
2. 다이아몬드 문제
   - 다중 상속이 가능하면, 컴파일러가 상위 클래스 중 어떤 클래스의 메서드를 사용해야 할지 모르게 된다.
3. 문제가 있는 계층 구조
   - Car와 Aircraft 가 있는 상태에서 하늘을 나는 자동차를 만들려면?
   - 다중 상속을 할 수 없기 때문에 상속으로는 해결 할 수 없다.

위 3번 같은 경우는 인터페이스를 사용하면 해결 할 수 있다.
자바는 인터페이스의 다중 상속은 혀용한다. (기능 구현은 구현 클래스에서 하기 때문)
```java
public class 하늘을나는자동차 implements Car, AirCraft {
    @Override
    public void drive() {...}
  
    @Override
    public void fly() {...}
}
```




















<br>
<br>
<br>

## ❐ 8.4 클래스는 자신의 기능에만 집중해야 한다.

---

모듈화의 핵심 목표 중 하나는 요구사항이 변경되면 그 변경과 직접 관련된 코드만 수정한다는 것.

### 8.4.1 다른 클래스와 지나치게 연관되어 있으면 문제가 될 수 있다.
```java
public class Book {
    private final List<Chapter> chapters;
  
    public int wordCount() {
        return chapters.map(getChaterWordCount)
                    .sum();
    }
    
    // 이 함수는 Chapter 클래스에 대한 것만 다룬다.
    private static int getChapterWordCount(Chapter chapter) {
        //...
    }
}

```

- `getChapterWordCount()`를 Book 클래스에 두면 모듈화가 되지 않음.
- 요구 사항이 변해 Chapter 관련된 기능을 수정해야 하면, Book도 수정해줘야 함.

결론적으로 변경이 Book 까지 전파 된다.

<br>
<br>

### 8.4.2 해결책 : 자신의 기능에만 충실한 클래스를 만들라.
```java
public class Book {
    private final List<Chapter> chapters;
  
    // 단어를 counting하는 역할을 Chapter로 위임
    public int wordCount() {
        return chapters.map(chapter -> chapter.wordCount())
                    .sum();
    }
}
```
단어를 counting하는 역할을 Chapter로 위임했기 때문에, Chapter 관련 변경 사항을 수정할 때
Book에는 변경이 전파되지 않는다.

<br>
<br>
<br>

## ❐ 8.5 관련있는 데이터는 함께 캡슐화하라

---

하나의 클래스에 너무 많은 것들을 두지 않도록 주의해야 하지만,<br> 한 클래스 안에
함께 두는 것이 합리적일 때는 그렇게 하는 것의 이점을 놓쳐서도 안된다.

### 8.5.1 캡슐화되지 않은 데이터는 취급하기 어려울 수 있다.

```java
public class TextBox {
	
    public void renderText (
        String text,
        Font font,
        Double fontSize,
        Double lineHeight,
        Color textColor) {
        ...
    } 
}
```

```java
public class UserInterface {
    private final TextBox messageBox;
    private final UiSettings uiSettings;
  
    public void displayMessage(String message) {
        messageBox.renderText(...)
    }
}
```

text를 랜더링 해줄 때 필요한 속성들은 모두 파라미터로 받고있다.
- Prameter가 많아져서 가독성 저하
- `displayMessage()`가 텍스트 스타일링의 세부 사항에 대해 자세히 알아야 한다.

<br>
<br>

### 8.5.2 해결책 : 관련된 데이터는 객체 또는 클래스로 그룹화하라.
```java
public class TextOptions {
    private final Font font,
    private final Double fontSize;
    private final Double lineHeight;
    private final Color textColor;
}
```
text를 랜더링 해줄 때 필요한 속성들을 TextOptions 클래스로 그룹화.

<br>
<br>
<br>

## ❐ 8.6 반환 유형에 구현 세부 정보가 유출되지 않도록 주의하라.

---

### 8.6.1 반환 형식에 구현 세부 사항이 유출될 경우 문제가 될 수 있다.
```java
public class ProfilePictureService {
    private final HttpFetcher httpFetcher;
    ProfilePictureResult getProfilePicture(int userId) {...}
}
```

```java
public class ProfilePictureResult {
    // 요청 성공 여부
    HttpResponse.Status getStatus() {...}
  
    // 프로필 사진이 발견된 경우 해당 데이터 
    HttpResponse.Payload? getImageData() {...}
}
```

위 구조에서 ProfilePictureService를 사용하는 개발자는 아래와 같은 문제를 겪는다.
- `HttpResponse` 와 관련된 여러 개념을 처리해야 함.
- ProfilePictureService의 구현을 변경하는 것은 매우 어려움.
    - HttpResponse.Status 까지 다뤄야 한다.
    - HttpResponse.Payload 까지 다뤄야 한다.

<br>
<br>

### 8.6.2 해결책 : 추상화 계층에 적합한 유형을 반환하라.

ProfilePictureService 클래스가 해결해야 하는 문제는?
-> 사용자의 프로필 사진을 가져오는 것.

```java
public class ProfilePictureService {
    private final HttpFetcher httpFetcher;
    
    ProfilePictureResult getProfilePicture(int userId) {...}
}
```

```java
public class ProfilePictureResult {

    enum Status {
    	SUCCESS, ...
    }
    
    // 요청 성공 여부
    Status getStatus() {...}
    
    // 프로필 사진이 발견된 경우 해당 데이터 
    List<Byte>? getImageData() {...}
}
```

**해결 방법 : 반환 유형을 추상화 계층과 일치 시켜라**
- 사용자 지정 `Status`를 사용
- Payload 대신 Byte리스트를 반환

<br>
<br>
<br>

## ❐ 8.7 예외 처리 시 구현 세부 사항이 유출되지 않도록 주의하라.

> 호출 당하는 쪽의 정보가 호출하는 쪽에게 유출이 되면 안된다.

<br>

### 8.7.1 예외 처리 시 구현 세부 사항이 유출되면 문제가 될 수 있다.

> 2주차 과제 기반으로 간단 예제를 만들어봤습니다.

```java
public class CarService {
    private final MoveStrategy moveStrategy;
  
    public void moveCars(List<Car> cars) {
        cars.forEach(car -> {
            int distance = moveStrategy.getCondition();
            car.move(distance);
        })
    }
}
```

```java
public class Forward implements MoveStrategy {
    @Override
    public int getCondition() {
        ...
        throw new CannotGoForwardException();
    }
}
```

```java
public class Back implements MoveStrategy {
    @Override
    public int getCondition() {
        ...
        throw new CannotGoBackException();
    }
}
```

이런 경우 각 구현체마다 서로 다른 예외를 던지기 때문에 이에 대한 처리를 CarService에서 처리해주야 함.<br>
**→** CarService가 구현 세부사항의 모든 예외를 알게 됨.

<br>
<br>

### 8.7.2 해결책: 추상화 계층에 적절한 예외를 만들라
```java
public class BusinessException extends RuntimeException {...}
```
만약 위와 같이 추상화 계층에 적절한 예외를 만들면 해결 됨.

```java
public class Forward implements MoveStrategy {
    @Override
    public int getCondition() {
        ...
        throw new BusinessException("앞으로 못감");
    }
}
```

```java
public class Back implements MoveStrategy {
    @Override
    public int getCondition() {
        ...
        throw new CannotGoBackException("뒤로 못감");
    }
}
```

(길준) 혹은 static factory method를 사용하여 네이밍을 추가해도 될 듯 합니다.
```java
public class Forward implements MoveStrategy {
    @Override
    public int getCondition() {
        ...
        throw BusinessException.cannotGoForward();
    }
}
```

<br>
