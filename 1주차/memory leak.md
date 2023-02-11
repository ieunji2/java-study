- memory leak란?
	- 자바에서 더 이상 사용되지 않는 객체가 GC에 의해 회수되지 않고 누적되는 현상
	- Old 영역에서 객체가 지워지지 않고 계속 쌓이므로 Heap Memory가 변하지 않음
	- 결국 Out of Memory가 발생함

1. Static 변수의 무분별한 사용
	- 클래스 변수(Static Variable)의 라이프 사이클은 static 객체 생성 후 프로그램 종료까지 입니다.
	- public static List<double> a = new ArrayList();이라는 a 리스트에 객체를 추가하면 a 리스트는 heap 메모리에 위치하지만 static 변수이기 때문에 gc 대상이 아닙니다. 때문에 해당 리스트는 크기가 커져있고 메모리를 계속 차지합니다.

2. Through Unclosed Resources
	- 자바에서 DBConnection을 사용하거나 Input Stream을 사용했을 때 새로운 커넥션을 만들고 리소스를 close()하지 않으면 memory leak가 발생합니다.
	- try ~ finally로 항상 리소스를 반환해야 합니다.
	- finally 블록에서 예외가 발생할 수 있기 때문에 finally 블록에서도 try catch를 사용해야합니다.
	- 때문에 try-with-resources block로 자동 반환하도록 하는걸 추천합니다.

3. equals, hashCode 재정의를 안했을 때
	- 새로운 클래스를 정의했을 때 equals, hashCode를 재정의 하지 않으면 메모리 누수가 발생할 수 있습니다.
	- hashCode를 재정의 하지 않으면 Map 객체에 키값은 같은 value를 가지더라도 다른 키로 인식되어 메모리가 낭비됩니다.

4. Inner Class 참조
	- 내부 클래스를 만들 때 static으로 선언하지 않으면 내부 클래스는 외부 클래스를 참조하게 됩니다.
	- 외부 클래스가 생성될 때 마다 내부 클래스도 함께 생성? 잘 모르겠다.
