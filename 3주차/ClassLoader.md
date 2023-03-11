# Class Loader

클래스로더의 역할

- Loading : 클래스 파일을 JVM 메모리에 탑재
    
    컴파일러로 컴파일된 .class 파일을 각 디렉토리로 부터 찾아 JVM 메모리에 탑재해줍니다.
    
    클래스 로더는 Bootstrap → Extension → Application 순으로 우선순위를 가지며 다음과 같은 역할을 합니다.
    
    1. Bootstrap ClassLoader : 모든 클래스 로더의 부모 클래스 로더로 최우선 순위를 가집니다. JVM 구동의 필수적인 라이브러리 클래스들을 JVM에 탑재 합니다. 각 OS에 맞는 네이티크 코드로 작성되어 있습니다.
    2. Extension ClassLoader : 표준 핵심 자바 클래스 라이브러리들을 JVM에 탑재 합니다.
    3. Application ClassLoader : ClassPath에 있는 클래스들 즉, 개발자가 작성한 자바 코드들을 JVM에 탑재 합니다. 흔히 src폴더 밑에 있는 파일들을 탑재합니다.
    
    이 세 단계의 클래스 로더를 거쳐도 클래스 파일을 찾을 수 없다면 ClassNotFoundException를 던집니다.
    
- Linking : 탑재된 클래스 파일을 검증
    
    로드된 클래스 파일들을 검증합니다. Linking은 세 가지 과정을 거칩니다.
    
    1. Verification : 클래스 파일이 유효한지 확인 합니다.
    2. Preparation : 필요한 static field 메모를 할당하고, 기본값으로 초기화 합니다. 이후 Initialization과정을 통해 코드에 작성한 초기값으로 변경합니다.
    3. Resolution : Symbolic Reference(참조하는 클래스의 단순 이름, 메모리 주소가 아님)값을 Direct Reference(참조하는 클래스의 메모리 주소) 주소 값으로 바꾸어 줍니다.
- Initialization : static field로 정의한 값들을 초기화
    
    클래스 파일의 코드를 읽어 클래스와 인터페이스를 지정한 값으로 초기화 해 줍니다.