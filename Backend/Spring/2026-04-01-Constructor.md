## @RequiredArgsConstructor

Lombok에서 제공하는 어노테이션으로,
final 또는 @NonNull 필드를 대상으로 생성자를 자동 생성해준다.

주로 스프링에서 생성자 기반 DI를 위해
private final 필드와 함께 사용된다.


## private final 쓰는 이유

final은 필드가 한 번만 할당되고 이후 변경되지 않도록 보장한다.  
private는 외부에서 직접 접근하거나 변경하지 못하도록 제한한다.

이 둘을 함께 사용하면 의존성이 한 번만 주입되고 변경되지 않아
객체의 안정성을 높일 수 있다.

또한 final을 사용하면 생성자에서 반드시 초기화해야 하므로
생성자 기반 DI를 자연스럽게 유도한다.