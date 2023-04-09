# Chapter 6. 단위 테스트 스타일

# Chapter 6. 단위 테스트 스타일

- 단위 테스트 스타일에 대한 비교
- 함수형 아키텍처와 육각형 아키텍처의 관계
- 출력 기반 테스트로 전환
- 단위 테스트 스타일에는 출력 기반, 상태 기반, 통신 기반이라는 세 가지 스타일이 있다
- 출력 기반 테스트 스타일이 가장 품질이 좋지만 아무데서나 사용할 수 없지만 함수형 프로그래밍 원칙을 사용해 기반 코드가 함수형 아키텍처를 지향하게끔 재구성하여 변환하는 기법 소개

## 6.1 단위 테스트의 세 가지 스타일

- 출력 기반 테스트
- 상태 기반 테스트
- 통신 기반 테스트

### 6.1.1 출력 기반 테스트 정의

- SUT에 입력을 넣고 생성되는 출력을 점검하는 방식
- 출력 기반 테스트는 내부 컬렉션에 상품을 추가하거나 데이터베이스에 저장하지 않음
- 다시 말해 사이드 이펙트가 없는 코드 선호를 강조하는 함수형 프로그래밍에 뿌리를 둠

### 6.1.2 상태 기반 스타일 정의

- 상태 기반 스타일은 작업이 완료된 후 시스템 상태를 확인하는 것
- 상태는 SUT나 협력자 중 하나, 또는 데이터베이스나 파일 시스템 등과 같은 프로세스 외부 의존성 등을 의미

### 6.1.3 통신 기반 스타일 정의

- 목을 사용해 SUT와 협력자 간의 통신을 검증

## 6.2 단위 테스트 스타일 비교

- 좋은 단위 테스트의 4대 요소(회귀 방지, 리팩터링 내성, 빠른 피드백, 유지 보수성)로 스타일을 비교

### 6.2.1 회귀 방지와 피드백 속도 지표로 스타일 비교하기

- 회귀 방지 지표와 피드백 속도는 특정 스타일에 따라 크게 달라지지 않는다.

### 6.2.2 리팩터링 내성 지표로 스타일 비교하기

- 리팩터링 내성은 리팩터링 중에 발생하는 거짓 양성 수에 대한 척도다. 결국 거짓 양성은 식별할 수 있는 동작이 아니라 코드의 구현 세부사항에 결합된 테스트의 결과다.
- **출력 기반 테스트**는 테스트 대상 메서드에만 결합되므로 거짓 양성 방지가 가장 우수
- **상태 기반 테스트**는 SUT 외에도 클래스 상태와 함께 작동하므로 결합도가 높다는 의미가 되며 이는 구현 세부 사항에 테스트가 얽매일 가능성이 높음을 시사함
- **통신 기반 테스트**가 거짓 양성에 가장 취약하다. 테스트 대역으로 상호 작용을 확인하는 테스트는 대부분 깨지기 쉬움(5장 내용)

### 6.2.3 유지 보수성 지표로 스타일 비교하기

- 유지 보수성 지표는 단위 테스트 스타일과 밀접한 관련이 있다
- 유지 보수성 지표
    - 테스트를 이해하기 얼마나 어려운가?
    - 테스트를 실행하기 얼마나 어려운가?
- 출력 기반 테스트의 유지 보수성
    - 출력 기반 테스트는 거의 항상 짧고 간결하므로 유지 보수가 쉽다
- 상태 기반 테스트의 유지 보수성
    - 상태 검증은 종종 출력 검증보다 더 많은 공간을 차지하기에 출력 기반 테스트보다 유지 보수가 어렵다
- 통신 기반 테스트의 유지 보수성
    - 통신 기반 테스트에는 테스트 대역과 상호 작용 검증을 설정해야 하기에 공간을 더 많이 차지한다 → 유지 보수가 어려워진다

### 6.2.4 스타일 비교하기: 결론

- 출력 기반 테스트 > 상태 기반 테스트 > 통신 기반 테스트 순으로 좋은 품질의 단위 테스트를 구성할 수 있다
- 하지만 출력 기반 스타일은 함수형으로 작성된 코드에만 적용되기에 대부분의 객체지향 프로그래밍 언어에는 해당하지 않음
- 이후 내용에서 상태 기반 테스트와 통신 기반 테스트를 출력 기반 테스트로 바꾸는 기법을 안내

## 6.3 함수형 아키텍처 이해

- 함수형 프로그래밍과 함수형 아키텍처가 무엇인지 알아보고, 함수형 아키텍처와 육각형 아키텍처가 어떤 관련이 있는지 살펴봄

### 6.3.1 함수형 프로그래밍이란?

- 함수형 프로그래밍은 수학적 함수를 사용한 프로그래밍
- 즉 숨은 입출력이 없고 호출 횟수에 상관없이 주어진 입력에 대해 동일한 출력을 생성해야함
- 테스트를 어렵게 만드는 숨은 입출력이란?
    - 사이드 이펙트: 사이드 이펙트는 메서드 시그니처에 표시되지 않는 출력
    - 예외: 메서드가 예외를 던지면 메서드 시그니처에 설정된 계약을 우회하는 경로를 가짐
    - 내외부 상태에 대한 참조: Datetime.now, 데이터베이스 조회 등
- 수학적 함수 예제

```java
public int Increment(int x){
  return x+1;
}
```

- 수학적 함수가 아닌 예제

```java
int x = 0;
public int Increment(){
  return ++x;
}
```

```java
public Comment addComment(String text){
   var comment = new Comment(text);
   _comments.add(comment);
   return comment;
}
```

### 6.3.2 함수형 아키텍처란?

- 사이드 이펙트를 일으키지 않는 애플리케이션은 비현실적
- 함수형 프로그래밍의 목표는 사이드 이펙트를 완전히 제거하는 것이 아닌 비즈니스 로직을 처리하는 코드와 사이드 이펙트를 일으키는 코드를 분리하는 것
- 하지만 이 두 가지를 모두 고려하면 복잡도가 배가되고 장기적으로 코드 유지 보수성을 방해한다
- 함수형 아키텍처는 사이드 이펙트를 비즈니스 연산 끝으로 몰아서 비즈니스 로직을 사이드 이펙트와 분리하는 것
- 비즈니스 로직과 사이드 이펙트를 분리하기 위한 두 가지 코드유형
    - 결정을 내리는 코드: 사이드 이펙트 없음, 수학적 함수
    - 해당 결정에 따라 작용하는 코드: 수학적 함수에 의해 이뤄진 모든 결정을 데이터베이스 변경과 같은 가지적인 부분으로 변환
- 결정을 내리는 코드를 functional core 또는 immutable core라 칭하고 해당 결정에 따라 작용하는 코드를 mutable shell이라 칭함
- 함수형 코어와 가변 셸의 협력 방식
    - 가변 셸은 모든 입력을 수집
    - 함수형 코어는 결정을 생성
    - 가변 셸은 결정을 사이드 이펙트로 변환

### 6.3.3 함수형 아키텍처와 육각형 아키텍처 비교

- 두 가지 모두 관심사 분리하는 아이디어를 기반으로 함
- 유사점 1.
    - 육각형 아키텍처는 도메인 계층과 애플리케이션 서비스 계층으로 분리
    - 도메인 계층: 비즈니스 로직
    - 애플리케이션 서비스 계층: 데이터베이스나 외부 애플리케이션과의 통신 책임
    - 이는 결정과 실행을 분리하는 함수형 아키텍처와 매우 유사
- 유사점 2.
    - 의존성 간의 단방향 흐름
    - 도메인 계층 내 클래스는 서로에게만 의존하듯 함수형 아키텍처의 불변 코어는 가변 셸에 의존하지 않는다
- 두 아키텍처의 차이점은 사이드 이펙트에 대한 처리이다
- 함수형 아키텍처는 모든 사이드 이펙트를 불변 코어에서 비즈니스 연산 가장자리로 밀어내고 가장자리는 가변 셸이 처리함
- 반면 육각형 아키텍처의 모든 수정 사항은 도메인 계층 내에 있어야 하며 계층의 경계를 넘어서는 안됨

## 6.4 함수형 아키텍처와 출력 기반 테스트로의 전환

- 함수형 아키텍처 리팩터링 단계
    - 프로세스 외부 의존성에서 목으로 변경
    - 목에서 함수형 아키텍처로 변경

### 6.4.1 감사 시스템 소개

```java
public class AuditManager {
    private int maxEntriesPerFile;
    private String directoryName;

    public AuditManager(int maxEntriesPerFile, String directoryName) {
        this.maxEntriesPerFile = maxEntriesPerFile;
        this.directoryName = directoryName;
    }

    public void addRecord(String visitorName, LocalDateTime timeOfVithrows IOException {

        String[] filePaths = Files.list(Paths.get(directoryName))
                .map(Path::toString)
                .toArray(String[]::new);
        List<Tuple<Integer, String>> sorted = sortByIndex(filePaths);

        String newRecord = visitorName + ";" + timeOfVisit;

        if (sorted.isEmpty()) {
            String newFile = Paths.get(directoryName, "audit_1.txt").toString();
            Files.write(Paths.get(newFile), newRecord.getBytes());
            return;
        }

        Tuple<Integer, String> lastFile = sorted.get(sorted.size() - 1);
        List<String> lines = Files.readAllLines(Paths.get(lastFile.getPath()));

        if (lines.size() < maxEntriesPerFile) {
            lines.add(newRecord);
            String newContent = String.join(System.lineSeparator(), lines);
            Files.write(Paths.get(lastFile.getPath()), newContent.getBytes());
        } else {
            int newIndex = lastFile.getIndex() + 1;
            String newName = String.format("audit_%d.txt", newIndex);
            String newFile = Paths.get(directoryName, newName).toString();
            Files.write(Paths.get(newFile), newRecord.getBytes());
        }
    }

    private List<Tuple<Integer, String>> sortByIndex(String[] filePaths) {
        List<Tuple<Integer, String>> tuples = new ArrayList<>();
        for (String filePath : filePaths) {
            File file = new File(filePath);
            String fileName = file.getName();
            if (fileName.matches("audit_\\d+\\.txt")) {
                String indexString = fileName.substring(6, fileName.length() - 4);
                int index = Integer.parseInt(indexString);
                tuples.add(new Tuple<>(index, filePath));
            }
        }
        tuples.sort(Comparator.comparing(Tuple::getIndex));
        return tuples;
    }

    private static class Tuple<K, V> {
        private final K index;
        private final V path;

        public Tuple(K index, V path) {
            this.index = index;
            this.path = path;
        }

        public K getIndex() {
            return index;
        }

        public V getPath() {
            return path;
        }
    }

}
```

- 메소드 설명
    - 작업 디렉터리에서 전체 파일 목록을 검색한다.
    - 인덱스별로 정렬한다(모든 파일 이름은 audit_{index}.txt 와 같은 패턴을 따른다.)
    - 아직 감사 파일이 없으면 단일 레코드로 첫 번째 파일을 생성한다.
    - 감사 파일이 있으면 최신 파일을 가져와서 파일의 항목 수가 한계에 도달했는지에 따라 새 레코드를 추가하거나 새 파일을 생성한다.
- 위의 클래스가 단위 테스트로 부적절한 이유
    - 파일을 올바른 위치에 배치하고, 테스트가 끝나면 해당 파일을 일고 내용을 확인한 후 삭제해야함 → 파일 시스템과 밀접히 연결
    - 단위테스트의 특성인 빠르게 수행하고와 다른 테스트와 별도로 처리한다에 부합하지 않으므로 통합 테스트에 해당

## 6.4.2 테스트를 파일 시스템에서 분리하기 위한 목 사용

- 파일의 모든 연산을 별도의 클래스(IFileSystem)로 도출하고 AuditManager에 생성자로 해당 클래스 주입하여 사용

```java
public class AuditManager {
    private int maxEntriesPerFile;
    private String directoryName;
    private IFileSystem fileSystem;

    public AuditManager(int maxEntriesPerFile, String directoryName, IFileSystem fileSystem) {
        this.maxEntriesPerFile = maxEntriesPerFile;
        this.directoryName = directoryName;
        this.fileSystem = fileSystem;
    }

    public void addRecord(String visitorName, LocalDateTime timeOfVisit) throws IOException {

        String[] filePaths = fileSystem.list(Paths.get(directoryName))
                .map(Path::toString)
                .toArray(String[]::new);
        List<Tuple<Integer, String>> sorted = sortByIndex(filePaths);

        String newRecord = visitorName + ";" + timeOfVisit;

        if (sorted.isEmpty()) {
            String newFile = Paths.get(directoryName, "audit_1.txt").toString();
            fileSystem.write(Paths.get(newFile), newRecord.getBytes());
            return;
        }

        Tuple<Integer, String> lastFile = sorted.get(sorted.size() - 1);
        List<String> lines = fileSystem.readAllLines(Paths.get(lastFile.getPath()));

        if (lines.size() < maxEntriesPerFile) {
            lines.add(newRecord);
            String newContent = String.join(System.lineSeparator(), lines);
            fileSystem.write(Paths.get(lastFile.getPath()), newContent.getBytes());
        } else {
            int newIndex = lastFile.getIndex() + 1;
            String newName = String.format("audit_%d.txt", newIndex);
            String newFile = Paths.get(directoryName, newName).toString();
            fileSystem.write(Paths.get(newFile), newRecord.getBytes());
        }
    }

    private List<Tuple<Integer, String>> sortByIndex(String[] filePaths) {
        List<Tuple<Integer, String>> tuples = new ArrayList<>();
        for (String filePath : filePaths) {
            File file = new File(filePath);
            String fileName = file.getName();
            if (fileName.matches("audit_\\d+\\.txt")) {
                String indexString = fileName.substring(6, fileName.length() - 4);
                int index = Integer.parseInt(indexString);
                tuples.add(new Tuple<>(index, filePath));
            }
        }
        tuples.sort(Comparator.comparing(Tuple::getIndex));
        return tuples;
    }

    private static class Tuple<K, V> {
        private final K index;
        private final V path;

        public Tuple(K index, V path) {
            this.index = index;
            this.path = path;
        }

        public K getIndex() {
            return index;
        }

        public V getPath() {
            return path;
        }
    }

    public interface IFileSystem{
        Stream<Path> list(Path directoryName);
        void write(Path filePath, byte[] bytes);
        List<String> readAllLines(Path filePath);
    }

}
```

- 위와 같이 리팩터링 하게 된다면 AuditManager는 파일 시스템에서 분리되므로 공유 의존성이 사라지고 테스트를 독립적으로 실행할 수 있다

```java
public class AuditManagerTest {

    @Test
    void new_file_is_created_when_the_current_file_overflows() throws Exception {
        IFileSystem fileSystemMock = mock(IFileSystem.class);
        when(fileSystemMock.list(Paths.get("audits"))).thenReturn(Stream.of(
                Paths.get("audits/audit_1.txt"),
                Paths.get("audits/audit_2.txt")
        ));
        when(fileSystemMock.readAllLines(Paths.get("audits/audit_2.txt"))).thenReturn(List.of(
                "Peter; 2019-04-06",
                "Jane; 2019-04-06",
                "Jack; 2019-04-06"
        ));

        AuditManager sut = new AuditManager(3, "audits", fileSystemMock);

        sut.addRecord("Alice", LocalDateTime.parse("2019-04-06T00:00:00"));

        verify(fileSystemMock).write(Paths.get("audits/audit_3.txt"), "Alice;2019-04-06T00:00".getBytes());
    }
}
```

- 위의 테스트 코드에는 복잡한 설정을 포함하기에 유지비 측면에서 이상적이지 않다
- 목 라이브러리가 최선을 다해 도움을 주지만, 작성된 테스트 코드는 여전히 평이한 입출력에 의존하는 테스트인 만큼 읽기가 어려움

### 6.4.3 함수형 아키텍처로 리팩터링하기

- 사이드 이펙트를 클래스 외부로 완전히 이동하여 AuditManager에 의존성을 제거

```java
public class AuditManager {
    private int maxEntriesPerFile;

    public AuditManager(int maxEntriesPerFile) {
        this.maxEntriesPerFile = maxEntriesPerFile;
    }

    public FileUpdate addRecord(FileContent[] files, String visitorName, LocalDateTime timeOfVisit) throws IOException {

        List<Tuple<Integer, FileContent>> sorted = sortByIndex(filePaths);

        String newRecord = visitorName + ";" + timeOfVisit;

        if (sorted.isEmpty()) {
            String newFile = Paths.get(directoryName, "audit_1.txt").toString();
            fileSystem.write(Paths.get(newFile), newRecord.getBytes());
            return;
        }

        Tuple<Integer, String> lastFile = sorted.get(sorted.size() - 1);
        List<String> lines = fileSystem.readAllLines(Paths.get(lastFile.getPath()));

        if (lines.size() < maxEntriesPerFile) {
            lines.add(newRecord);
            String newContent = String.join(System.lineSeparator(), lines);
            fileSystem.write(Paths.get(lastFile.getPath()), newContent.getBytes());
        } else {
            int newIndex = lastFile.getIndex() + 1;
            String newName = String.format("audit_%d.txt", newIndex);
            String newFile = Paths.get(directoryName, newName).toString();
            fileSystem.write(Paths.get(newFile), newRecord.getBytes());
        }
    }
}

// FileContent
// 결정을 내리기 위해 파일 시스템에 대해 알아야 할 모든 것을 포함하는 클래스
public class FileContent{
  public String fileName;
  public String[] lines;

  public FileContent(String fileName, String[] lines){
    this.fileName = fileName;
    this.lines = lines;
  }
}

// FileUpdate
// 작업 디렉터리의 파일을 변경하는 대신, AuditManager는 사이드 이펙트에 대한 명령을 반환
public class FileUpdate{
  public String fileName;
  public String newContent;

  public FileUpdate(String fileName, String newContent){
    this.fileName = fileName;
    this.newContent = newContent;
  }
}

// Persister
public class Persister {
    public FileContent[] ReadDirectory(String directoryName) throws IOException {
        return Files.walk(Paths.get(directoryName))
                .filter(Files::isRegularFile)
                .map(path -> new FileContent(path.getFileName().toString(),
                        Files.readAllLines(path)))
                .toArray(FileContent[]::new);
    }

    public void ApplyUpdate(String directoryName, FileUpdate update) throws IOException {
        String filePath = Paths.get(directoryName, update.fileName).toString();
        Files.write(Paths.get(filePath), update.newContent.getBytes());
    }

```

- 위의 방식으로 Persister의 코드를 간결하게 개선할 수 있다
- 이런 분리를 유지하려면 FileContent와 FileUpdate의 인터페이스를 프레임워크에 내장된 파일 상호 작용 명령과 최대한 가깝게 두어야 함

```java
// AuditManager와 Persister를 붙이는 애플리케이션 서비스
public class ApplicationService{
    private String directoryName;
    private AuditManager auditManager;
    private Persister persister;
    
    public void addRecord(String visitorName, LocalDateTime timeOfVisit){
        FileContent[] files = persister.readDirectory(directoryName);
        FileUpdate update = auditManager.addRecord(files, visitorName, timeOfVisit);
        persister.ApplyUpdate(directoryName, update);
    }
}
```

- 이러한 설계로 리팩토링하여 애플리케이션 서비스에서 클라이언트 진입점을 생성하고, Persister는 파일 I/O의 책임만을 가지며 AuditManager는 Persister에서 필요한 명령만을 생성함

```java
public class AuditManagerTest {

    @Test
    void new_file_is_created_when_the_current_file_overflows() throws Exception {
        IFileSystem fileSystemMock = mock(IFileSystem.class);
        when(fileSystemMock.list(Paths.get("audits"))).thenReturn(Stream.of(
                Paths.get("audits/audit_1.txt"),
                Paths.get("audits/audit_2.txt")
        ));
        when(fileSystemMock.readAllLines(Paths.get("audits/audit_2.txt"))).thenReturn(List.of(
                "Peter; 2019-04-06",
                "Jane; 2019-04-06",
                "Jack; 2019-04-06"
        ));

        AuditManager sut = new AuditManager(3, "audits", fileSystemMock);

        sut.addRecord("Alice", LocalDateTime.parse("2019-04-06T00:00:00"));

        verify(fileSystemMock).write(Paths.get("audits/audit_3.txt"), "Alice;2019-04-06T00:00".getBytes());
    }
}
```

**리팩토링 기반으로 작성된 테스트**

```java
public class AuditManagerTest {

    @Test
    void new_file_is_created_when_the_current_file_overflows() throws Exception {
        AuditManager sut = new AuditManager(3);
        FileContent[] files = new FileContent[]{
                new FileContent("audit_1.txt", new String[0]),
                new FileContent("audit_2.txt", new String[]{
                        "Peter; 2019-04-06",
                        "Jane; 2019-04-06",
                        "Jack; 2019-04-06"
                })
        };
        
        FileUpdate update = sut.addRecord(files, "Alice", LocalDateTime.parse("2019-04-06"));
        
        Assert.Equal("audit_3.txt", update.getFileName());
        Assert.Equal("Alice;2019-04-06", update.getNewContent());
    }
}
```

## 6.5 함수형 아키텍처의 단점 이해하기

### 6.5.1 함수형 아키텍처 적용 가능성

- 예를 들어 방문자의 접근 레벨에 따라 다른 동작을 수행하고, 방문자의 접근 레벨이 모두 데이터베이스에 저장되어 있다면 addRecord 메서드에 숨은 입력이 생겨 수학적 함수가 될 수 없음
- 이에 대한 동작을 억지로 구현하려하다보면 필요없음에도 데이터베이스를 조회하는 성능저하나 클래스 책임이 모호해질 수 있다

### 6.5.2 성능단점

- 출력 기반 테스트(함수형 아키텍처 사용)는 목을 사용한 테스트만큼 빠르게 동작하지만 시스템은 외부 의존성을 더 많이 호출하기에 성능이 낮아짐
- 함수형 아키텍처와 전통적인 아키텍처 사이의 선택은 성능과 코드 유지 보수성 간의 절충

### 6.5.3 코드베이스 크기 증가

- 위의 예제에서 알 수 있듯 함수형 아키텍쳐를 통해 코드 복잡도가 낮아지고 유지 보수성이 향상되지만, 코딩을 훨씬 많이 요구함
- 이러한 추가 비용을 감수하여도 괜찮을 프로젝트에서만 적용
- 또한 출력 기반 테스트만 의존하는 것이 아닌 통신 기반 스타일을 섞어가며 코드 생성 비용에 대해 고민하고 적용