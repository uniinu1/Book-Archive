# 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

많은 클래스가 하나 이상의 자원에 의존한다.

사전은 정적 유틸리티 클래스(아이템4)로 보통 구현함

**코드5-1. 정적 유틸리티를 잘못 하용한 예 - 유연하지 않고 테스트하기 어려움**
```java
public class SpellChecker{
    private static final Lexicon dictionary = ...;
    
    private spellChecker(){}  // 객체 생성 방지
    
    public static boolean isValid(String word){ ... }
    public static List<String> suggestions(String typo){ ... }
}
```

**코드 5-2. 싱글턴을 잘못 사용한 예 - 유연하지 않고 테스트하기 어려움**
```java
public class SpellChecker{
    private final Lexicon dictionary = ...;
    
    private SpellChecker(...){}
    public static SpellChecker INSTANCE = new SpellChecker(...);
    
    public boolean isValid(String word){...}
    public List<String> suggestions(String typo){...}
}
```