### 10장 null 대신 Optional

#### 10.1 값이 없는 상황을 어떻게 처리할까?

- 자동차와 자동차 보험을 갖고 있는 사람 객체를 중첩 구조로 구현

```java
public class Person {
    private Car car;
    public Car getCar() { return Car; }
}

public class Car {
    private Insurance insurance;
    public Insurance getInsurace() { return insurance; }
}

public class Insurance {
    private String name;
    public String getName() { return name; }    
}
```



- 발생할 수 있는 문제들

```java
public String getCarInsuranceName(Person person){
    return person.getCar().getInsurance().getName();
}
```

- person이 null이라면?
- 사람은 있는데 차를 소유하지 않은 사람이라면? (->  getInsurance는  null의 보험 정보를 반환하려 하므로 NullPointerException)
- 사람도 있고 차도 있는데 보험이 가입되어 있지 않은 차라면?

##### 10.1.1 보수적인 자세로 NullPointerException 줄이기

```java
public String getCarInsuranceName(Person person){
    if (person != null) {
        Car car = person.getCar();
        if (car != null) {
            Insurance insurance = car.getInsurance();
            if(insurance != null) {
                return insurance.getName();
            }
        }
    }
    return "Unknown";
}
```

- 변수를 접근할 때마다 중첩된 if가 추가되면서 코드 들여쓰기 수준이 증가함
- 깊은 의심 deep doubt
- 코드의 구조도 엉망, 가독성도 떨어짐



```java
public String getCarInsuranceName(Person person){
    if (person == null){
    return "Unknown";
    }
    Car car = person.getCar();
    if (car == null) {
        return "Unknown";
    }
    Insurance insurance = car.getInsurance();
    if (insurance == null) {
        return "Unknown";
    }
    return insurance.getName();
}
```

- 메서드에 너무 많은 출구가 생겨서 유지보수가 어려워짐
- 값이 있거나 없음을 표현할 수 있는 다른 좋은 방법?



##### 10.1.2 null 때문에 발생하는 문제

1. 에러의 근원이다
2. 코드를 어지럽힌다 (null 확인 코드 추가)
3. 아무 의미가 없다 (정적 형식 언어에서 값이 없음을 표현하는 방법으로는 적절하지 않다 <- 무슨 말?)
4. 자바 철학에 위배된다 (개발자로부터 숨겨지지 않은 유일한 포인터)
5. 형식 시스템에 구멍을 만듬 (다른 곳으로 null이 퍼졌을 때 애초에 어떤 의미로 null이 사용되었는지 알 수 없음)

##### 10.1.3 다른 언어는 null 대신 무얼 사용하나?

- groovy

  ```groovy
  def carInsuranceName = person?.car?.insurance?.name
  ```

  호출 체인에 null인 레퍼런스가 있으면 결과로 null이 반환

- Haskell, Scala

  - 하스켈 : 선택형 값(optional value)을 저장할 수 있는 형식 Maybe
  - 스칼라 : T 형식의 값을 갖거나 아무 값도 갖지 않을 수 있는 Option[T]

#### 10.2 Optional 클래스 소개

- java.util.Optional<T>

  - 선택형 값을 캡슐화하는 클래스
  - 값이 있으면 해당 값을 감싸고
  - 값이 없으면 Optional.empty로 옵셔널(싱글턴 인스턴스)을 반환

- Optional<Car> : 값이 없을 수 있음을 명시적으로 보여주는 것임

  ```java
  public class Person {
      private Optional<Car> car;
      public Optional<Car> getCar() { return car; }
  }
  
  public class Car {
      private Optional<Insurance> insurance;
      public Optional<Insurance> getInsurance() { return insurance; }
  }
  
  public class Insurance {
      private String name;
      public String getName() { return name; }
  }
  ```

  - 사람은 차를 소유했을 수도 아닐 수도 있음 (옵셔널)
  - 차는 보험에 가입되어 있을 수도 아닐 수도 있음 (옵셔널)
  - 보험회사는 반드시 이름을 가져야 하므로 옵셔널이 아님

- 이와 같은 방식으로 값이 없는 상황이 *데이터의 문제인지* 또는 *알고리즘의 버그인지* 명확히 구분할 수 있음

- 모든 null 레퍼런스를 옵셔널로 대체하는 것은 좋은 선택이 아니다!



#### 10.3 Optional 적용 패턴

##### 10.3.1. Optional 객체 만들기

- 빈 Optional

  ```java
  Optional<Car> optCar = Optional.empty();
  ```

  

- null이 아닌 값으로 Optional 만들기

  ```java
  Optional<Car> optCar = Optional.of(car);
  ```

  

- null값으로 Optional 만들기

  ```java
  Optional<Car> optCar = Optional.ofNullable(car);
  ```

  - ofNullable은 정적 팩토리 메서드

##### 10.3.2 맵으로 Optional의 값을 추출하고 변환하기

- 보험회사의 이름을 추출하려면 먼저 보험회사 객체가 null인지 확인한다.

```java
String name = null;
if (insurance != null) {
    name = insurance.getName();
}
```

- 이러한 유형의 패턴에 사용 : map 메소드

```java
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);

Optional<String> name = optInsurance.map(Insurance::getName);
```

- 스트림의 map 메소드와 개념적으로 유사함
- Optional이 값을 포함하면 map의 인수로 제공된 함수가 값을 바꾸고, 비어있으면 아무 일도 일어나지 않음.

##### 10.3.3 flatMap으로 Optional 객체 연결

- 여러 메소드를 안전하게 호출하고 싶다

```java
public String getCarInsuranceName(Person person) {
    return person.getCar().getInsurance().getName();
}
```

- map을 이용하여 위 코드를 재구현
  - 중첩된 Optional 객체 때문에 컴파일되지 않음

```java
Optioanl<Person> optPerson = Optional.of(person);
Optional<String> name = 
	optPerson.map(Person::getCar)
			 .map(Car::getInsurance)
			 .map(Insurance.getName);
```

- flatMap을 사용하여 재구현

  ```java
  public String getCarInsuranceName(Optional<Person> person) {
      return person.flatMap(Person::getCar)
          	     .flatMap(Car::getInsurance)
          		 .map(Insurance::getName)
          		 .orElse("Unknown");
  }
  ```

  - Optional을 인수로 받거나 Optional을 반환하는 메소드를 정의함으로서
    - 이 메소드를 사용하는 모든 사람에게 이 메소드가 빈 값을 받거나 빈 결과를 반환할 수 있음을 잘 문서화해서 제공하는 것

##### *참고 : 도메인 모델에 Optional을 사용하면 데이터 직렬화 불가능

```java
public class Person {
    private Car car;
    public Optional<Car> getCarAsOptional() {
        return Optional.ofNullable(car);
    }
}
```

##### 10.3.4 디폴트 액션과 Optional 언랩

- get() : 가장 간단하지만 가장 안전하지 않은 메소드
  - 값이 있으면 값 반환, 값이 없으면 NoSuchElementException
- orElse("디폴트값") : 값이 없을 때 디폴트값 제공
- orElseGet(Supplier<? extends T> other) : Optional에 값이 없을 때만 Supplier 실행 (?)
- orElseThrow : Optional이 비어 있을 때 사용자가 선택한 종류의 예외 발생
- ifPresent :  값이 존재할 때 인수로 넘겨 준 동작 실행 (없으면 아무 일도 일어나지 않음)

##### 10.3.5 두 Optional 합치기

```java
public Insurance findCheapestInsurance(Person person, Car car) {
    //다양한 보험회사가 제공하는 서비스 조회
    //모든 결과 데이터 비교
    return cheapestCompany;
}
```



- isPresent 활용하여 개선

```java
public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
    if(person.isPresent() &* car.isPresent()) {
        return Optional.of(findCheapestInsurance(person.get(), car.get()));
    } else {
        return Optional.empty();
    }
}
```

​	null 확인 코드와 크게 다른 점이 없다

- Optional 언랩하지 않고 두 Optional 합치기

  ```java
  public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
      return person.flatMap(p -> car.map(c -> findCheapestInsurance(p,c)));
  }
  ```

##### 10.3.6 필터로 특정값 거르기

- 보험회사 이름이 'CambridgeInsurance'인지 확인하기

  ```java
  Insurance insurance = ...;
  if(insurance != null && "CambridgeInsurance".equals(insurance.getName())) {
      System.out.println("ok");
  }
  ```

- Optional 객체에 filter 메소드 이용해서 재구현

  ```java
  Optional<Insurance> optInsurance = ...;
  optInsurance.filter(insurance ->
                     "CambridgeInsurance".equals(insurance.getName()))
      				. ifPresent(x -> System.out.println("ok"));
  ```

  - Optional 객체가 값을 가지고 && 프레디케이트와 일치하면 값을 반환하고
  - 그렇지 않으면 빈 Optional 객체를 반환

- Optional 필터링 활용 예제

  - 인수 person이 minAge 이상의 나이일 때만 보험회사 이름을 반환

  ```java
  public String getCarInsuranceName(Optional<Person> person, int minAge) {
      return person.filter(p -> p.getAge() >= minAge)
          		 .flatMap(Person::getCar)
          	     .flatMap(Car::getInsurance)
          		 .map(Insurance::getName)
          		 .orElse("Unknown");
  }
  ```

  

#### 10.4 Optional을 사용한 실용 예제

- 코드 구현만 바꾸는 것이 아니라 네이티브 자바 API와 상호작용하는 방식도 바꿔야 한다.
- Optional 기능을 활용할 수 있도록 우리 코드에 작은 유틸리티 메소드를 추가(?)

##### 10.4.1 잠재적으로 null이 될 수 있는 대상을 Optional로 감싸기

```java
//Without Optional
Object value = map.get("key");

//With Optional
Optional<Object> value = Optional.ofNullable(map.get("key"));
```

##### 10.4.2 예외와 Optional

- 값을 제공할 수 없을 때 null 반환 대신 예외를 발생시키는 경우

  - Integer.parseInt(String) : NumberFormatException

  ```java
  public static Optional<Integer> stringToInt(String s) {
      try {
          return Optional.of(Integer.parseInt(s));
      } catch (NumberFormatException e) {
          return Optional.empty();
      }
  }
  ```

- 기본형 Optional과 이를 사용하지 말아야 하는 이유

  - 스트림처럼 기본형으로 특화된 OptionalInt, OptionalLong, ... 제공
  - 하지만 map, flatMap, filter 등 지원하지 않음

##### 10.4.3 응용 

```java
Properties props = new Properties();

props.setProperty("a","5");
props.setProperty("b","true");
props.setProperty("c","-3");

public int readDuration(Properties props, String name) {
    String value = props.getProperty(name);
    if (value != null) {
        try {
            int i = Integer.parseInt(value);
            if (i > 0) { return i; }
        } catch (NumberFormatException nfe) {}
    }
    return 0;
}

assertEquals(5, readDuration(param, "a"));
assertEquals(0, readDuration(param,"b"));
```

```java
public int readDuration(Properties props, String name) {
    return Optional.ofNullable(props.getProperty(name))
        		   .flatMap(OptionalUtility::stringToInt)
        .filter(i -> i > 0)
        .orElse(0);
}
```



