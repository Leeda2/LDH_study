## Java 16 "record"
자바의 record는 데이터 전달을 목적으로 하는 불변(Immutable) 데이터 객체를 아주 간결하게 정의하기 위해 자바 14에 도입되어 자바 16부터 정식 추가된 클래스 타입입니다.

기존 DTO(Data Transfer Object)나 VO(Value Object)를 만들 때 쓰이던 불필요한 코드를 줄여줍니다. = 생성자와 getter를 포함하던 클래스를 줄여줌.

### 1. 기존 클래스 vs Record 비교
사용자의 이름과 나이를 담는 데이터 객체를 만들 때의 차이입니다.

기존 방식 (Class)

```
private static class Destination {
    private final String country;
    private final String city;
    private final String airportCode;

    public Destination(String country, String city, String airportCode) {
        this.country = country;
        this.city = city;
        this.airportCode = airportCode;
    }
}
```
Record 방식
```
private record Destination(
    String country,
    String city,
    String airportCode
) { }
```
=> 컴파일러가 위 기존 방식의 모든 코드를 자동으로 생성

모든 필드를 초기화하는 생성자

필드 이름을 딴 Getter (예: getName()이 아니라 name() 형태)

equals(), hashCode(), toString() 메서드

강제 불변성(Immutable):

모든 필드가 자동으로 private final로 선언됩니다.

생성된 후에는 값을 변경할 수 없으며, Setter 메서드는 제공되지 않습니다.

record는 내부적으로 이미 java.lang.Record 클래스를 상속받고 있으므로, 다른 클래스를 extends 할 수 없습니다. -> 상속불가
(단, interface 구현은 가능합니다.)

### 2. 활용법: 컴팩트 생성자 (Compact Constructor)
데이터 검증(Validation) 로직이 필요할 때는 매개변수를 생략한 컴팩트 생성자를 사용할 수 있습니다.

Java

```
public record Person(String name, int age) {
    // 컴팩트 생성자
    public Person {
        if (age < 0) {
            throw new IllegalArgumentException("나이는 음수일 수 없습니다.");
        }
    }
}
```
