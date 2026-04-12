# equals & hashCode 정리

## 개념

- equals: 두 객체가 "논리적으로 같은지" 비교
- hashCode: 객체를 해시값으로 변환하여 "저장 위치"를 결정

👉 HashMap / HashSet에서는  
- hashCode로 위치를 찾고  
- equals로 실제 같은 객체인지 비교한다

---

## 핵심 정리

> 논리적으로 같은 객체로 판단하려면 equals를 재정의해야 하고,  
> Hash 기반 자료구조에서 정상 동작하려면 hashCode도 함께 재정의해야 한다.

👉 하나만 하면 안됨  
👉 하나라도 바꾸면 둘 다 맞춰야 함

---

## 기본 문제 상황

```java
Menu m1 = new Menu("짜장", 5000);
Menu m2 = new Menu("짜장", 5000);
```

```text
m1.equals(m2) → false
```

👉 이유: 기본 equals는 "주소 비교"

---

## equals 재정의

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof Menu)) return false;
    Menu m = (Menu) o;
    return price == m.price && name.equals(m.name);
}
```

👉 이제 논리적으로 같으면 true

---

## 하지만 hashCode 없으면 문제 발생

```java
Map<Menu, String> map = new HashMap<>();

map.put(new Menu("짜장", 5000), "메뉴");

map.get(new Menu("짜장", 5000));
```

```text
→ null
```

👉 이유:
- equals는 같음
- hashCode가 달라서 다른 위치에 저장됨

---

## 올바른 구현

```java
@Override
public int hashCode() {
    return Objects.hash(name, price);
}
```

---

## 실무에서 쓰는 이유

### 1. Entity 비교

- JPA Entity를 비교할 때 사용
- 같은 DB row를 같은 객체로 판단해야 할 때 필요

👉 잘못하면:
- 같은 데이터인데 다른 객체로 인식됨

---

### 2. DTO 중복 제거

```java
Set<Menu> set = new HashSet<>();
```

👉 중복 제거하려면 equals/hashCode 필수

👉 없으면:
- 같은 값인데 여러 개 들어감

---

### 3. Set / Map 사용

- HashSet → 중복 제거
- HashMap → key 기반 조회

👉 equals/hashCode 없으면:
- 조회 안됨
- 중복 발생
- 데이터 꼬임

---

## 실무에서 터지는 문제들

- map.get() 했는데 null 나옴
- Set에 중복 데이터 들어감
- 캐시 key 비교 실패
- Entity 비교 오류

👉 대부분 equals/hashCode 문제

---

## 한줄 정리

> equals는 "논리적 동일성",  
> hashCode는 "같은 위치로 보내기 위한 값"

👉 둘은 항상 같이 가야 한다

---

## 최종!

```text
논리적으로 같은 객체로 판단하려면 equals를 재정의해야 하며, 기본적으로는 객체의 주소로 비교되기 때문에 서로 다른 객체로 인식됩니다.
또한 Hash 기반 자료구조에서 같은 위치에 저장되도록 하기 위해 hashCode를 함께 재정의해야 하며, hashCode는 객체를 정수값으로 변환하여 저장 위치 계산에 사용됩니다.
```