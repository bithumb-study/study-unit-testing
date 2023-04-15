# 6. 단위 테스트 스타일 

## 6.4 함수형 아키텍처와 출력 기반 테스트로의 전환
- 프로세스 외부 의존성에서 mock 으로 변경
- mock 에서 함수형 아키텍처로 변경 

이 장에서는 샘플 프로젝트(감사 시스템)을 만들어서 리팩토링 한다. 

### 6.4.1 감사 시스템 소개 (Sample project)

```java
public class AuditManager {
    private final int maxEntriesPerFile;
    private final String directoryName;

    public AuditManager(int maxEntriesPerFile, String directoryName) {
        this.maxEntriesPerFile = maxEntriesPerFile;
        this.directoryName = directoryName;
    }

    public void addRecord(String visitorName, LocalDateTime localDateTime) {
        try {

            // 작업 디렉토리에서 전체 파일 목록을 검색 후 이름순 정렬 
            List<String> files = Arrays.asList(new File(directoryName).listFiles()).stream().sorted(Comparator.comparing(File::getName)).map(f -> f.getPath()).collect(Collectors.toList());

            String newRecord = visitorName + ";" + localDateTime;

            // 아직 감사 파일이 없으면 첫 번째 파일을 생성
            if(files.size() == 0) {
                File file = new File(directoryName + "\\audit_1.txt");
                file.createNewFile();

                FileWriter fileWriter = new FileWriter(file);
                BufferedWriter writer = new BufferedWriter(fileWriter);
                writer.write(newRecord);
                writer.close();
                return;
            }

            // 감사 파일이 있으면 최신 파일을 가져와서
            String lastFile = files.get(files.size()-1);
            FileReader fileReader = new FileReader(lastFile);
            BufferedReader bufferedReader = new BufferedReader(fileReader);
            List<String> lines = bufferedReader.lines().collect(Collectors.toList());

            // 파일의 항목 수가 한계에 도달했는지 확인 후 입력  
            if(lines.size() < maxEntriesPerFile) {
                FileWriter fileWriter = new FileWriter(lastFile);
                BufferedWriter writer = new BufferedWriter(fileWriter);
                lines.add(newRecord);

                StringJoiner stringJoiner = new StringJoiner("\r\n");
                writer.write(lines.stream().map(s -> stringJoiner.add(s)).toString());
                writer.close();
            } else {
                int newIndex = files.size() + 1;
                String newFileName = "audit_" + newIndex + ".txt";
                File file = new File(directoryName + "\\" + newFileName);
                file.createNewFile();

                FileWriter fileWriter = new FileWriter(lastFile);
                BufferedWriter writer = new BufferedWriter(fileWriter);
                writer.write(newRecord);
                writer.close();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
- 회귀 방지 : 좋음
- 리팩토링 내성 : 좋음
- 빠른 피드백 : 나쁨
- 유지 보수성 : 나쁨 

테스트 또한 단위 테스트가 아닌 통합 테스트 범주에 속함

### 6.4.2 테스트를 파일 시스템에서 분리하기 위한 mock 사용 

```java
/**
 * 파일 시스템 작업을 캡슐화한 새로운 사용자 정의 인터페이스
 */
public interface IFileSystem {
    List<String> getFiles(String directoryName);
    void writeAllText(String filePath, String content);
    List<String> readAllLines(String filePath);
}
```

```java
public class AuditManager {
    private final int maxEntriesPerFile;
    private final String directoryName;
    private final IFileSystem iFileSystem;  // used new interface

    public AuditManager(int maxEntriesPerFile, String directoryName, IFileSystem iFileSystem) {
        this.maxEntriesPerFile = maxEntriesPerFile;
        this.directoryName = directoryName;
        this.iFileSystem = iFileSystem;  // used new interface
    }

    public void addRecord(String visitorName, LocalDateTime localDateTime) {
        try {

            List<String> files = iFileSystem.getFiles(directoryName);  // used new interface

            String newRecord = visitorName + ";" + localDateTime;

            if(files.size() == 0) {
                File file = new File(directoryName + "\\audit_1.txt");
                file.createNewFile();
                iFileSystem.writeAllText(file.getPath(), newRecord);  // used new interface
                return;
            }

            String lastFile = files.get(files.size()-1);
            List<String> lines = iFileSystem.readAllLines(lastFile); // used new interface

            if(lines.size() < maxEntriesPerFile) {
                lines.add(newRecord);
                StringJoiner stringJoiner = new StringJoiner("\r\n");

                iFileSystem.writeAllText(lastFile, lines.stream().map(s -> stringJoiner.add(s)).toString()); // used new interface
            } else {
                int newIndex = files.size() + 1;
                String newFileName = "audit_" + newIndex + ".txt";
                File file = new File(directoryName + "\\" + newFileName);
                file.createNewFile();

                iFileSystem.writeAllText(file.getPath(), newRecord); // used new interface
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

파일 시스템을 분리하여 공유 의존성을 제거 -> 테스트를 서로 독립적으로 실행   

테스트 코드 
```java
class AuditManagerTest {

    /**
     * 검증 : 현재 파일의 항목 수가 한계(3)에 도달했을 때, 감사 항목이 하나 있는 새 파일을 생성
     */
    @Test
    void a_new_file_is_created_when_the_current_file_overflows() {
        IFileSystem iFileSystem = mock(IFileSystem.class);

        when(iFileSystem.getFiles("audits")).thenReturn(Arrays.asList("audits\\audit_1.txt", "audits\\audit_2.txt"));
        when(iFileSystem.readAllLines("audits\\audit_2.txt")).thenReturn(Arrays.asList("Peter; 2023-04-10Y17:00:00", "Jane; 2023-04-10Y17:00:00", "Jack; 2023-04-10Y17:00:00"));

        AuditManager sut = new AuditManager(3, "audits", iFileSystem);
        LocalDateTime now = LocalDateTime.now();
        sut.addRecord("Alice", now);

        verify(iFileSystem).writeAllText("audits\\audit_3.txt", "Alice;" + now.toString());
    }
}
```

- 회귀 방지 : 좋음
- 리팩토링 내성 : 좋음
- 빠른 피드백 : 나쁨 -> 좋음
- 유지 보수성 : 나쁨 -> 중간 


### 6.4.3 함수형 아키텍처로 리팩터링 하기
- 6.4.2 예제는 캡슐화 된 인터페이스에 부작용을 숨긴 뒤 본체에 주입 시킨 형식

[ 함수형 아키텍처 ]
- 부작용을 클래스 외부로 완전히 이동시킨 뒤 `AuditManager` 는 파일에 수행할 작업을 둘러싼 결정만 책임
```java
public class FileUpdate {

    public final String fileName;
    public final String newContent;

    public FileUpdate(String fileName, String newContent) {
        this.fileName = fileName;
        this.newContent = newContent;
    }
}

public class FileContent {

    public final String fileName;
    public final String[] lines;

    public FileContent(String fileName, String[] lines) {
        this.fileName = fileName;
        this.lines = lines;
    }
}

public class AuditManager2 {

    private final int maxEntriesPerFile;

    public AuditManager2(int maxEntriesPerFile) {
        this.maxEntriesPerFile = maxEntriesPerFile;
    }

    public FileUpdate addRecord(List<FileContent> files, String visitorName, LocalDateTime timeOfVisit) {
        List<FileContent> sorts = files.stream().sorted(Comparator.comparing(fileContent -> fileContent.fileName)).collect(Collectors.toList());

        String newRecord = visitorName + ";" + timeOfVisit;

        if (sorts.size() == 0) {
            return new FileUpdate("audit_1.txt", newRecord);
        }

        FileContent currentFile = sorts.get(sorts.size() - 1);
        List<String> lines = Arrays.asList(currentFile.lines);

        if (lines.size() < maxEntriesPerFile) {
            lines.add(newRecord);
            StringJoiner joiner = new StringJoiner("\r\n");
            String newContent = lines.stream().map(s -> joiner.add(s)).toString();
            return new FileUpdate(currentFile.fileName, newContent);
        } else {
            int newIndex = files.size() + 1;
            String newFileName = "audit_" + newIndex + ".txt";
            return new FileUpdate(newFileName, newRecord);
        }
    }
}
```

```java
public class Persister {

    public List<FileContent> readDirectory(String directoryName) throws Exception {
        return Arrays.stream(new File(directoryName).listFiles()).map(file -> {
            String fileName = file.getName();
            String[] lines = {};
            try {
                FileReader fileReader = new FileReader(fileName);
                BufferedReader reader = new BufferedReader(fileReader);
                lines = reader.lines().toArray(String[]::new);
                reader.close();
            } catch (Exception e) {
                e.printStackTrace();
            }

            return new FileContent(fileName, lines);
        }).collect(Collectors.toList());
    }
    
    public void applyUpdate(String directoryName, FileUpdate update) throws Exception {
        File file = new File(directoryName);

        FileWriter fileWriter = new FileWriter(file);
        BufferedWriter writer = new BufferedWriter(fileWriter);
        writer.write(update.newContent);
        writer.close();
    }
}
```

```java
public class ApplicationService {
    private final String directoryName;
    private final AuditManager2 auditManager;
    private final Persister persister;
    
    public ApplicationService(String directoryName, int maxEntriesPerFile) {
        this.directoryName = directoryName;
        this.auditManager = new AuditManager2(maxEntriesPerFile);
        this.persister = new Persister();
    }
    
    public void addRecord(String visitorName, LocalDateTime timeOfVisit) throws Exception {
        List<FileContent> files = persister.readDirectory(directoryName);
        FileUpdate fileUpdate = auditManager.addRecord(files, visitorName, timeOfVisit);
        persister.applyUpdate(directoryName, fileUpdate);
    }
}
```

테스트 코드 
```java
class AuditManagerTest {
    /**
     * mock 없이 작성된 테스트
     */
    @Test
    void a_new_file_is_created_when_the_current_file_overflows_without_mock() {
        LocalDateTime now = LocalDateTime.now();

        AuditManager2 sut = new AuditManager2(3);
        List<FileContent> files = Arrays.asList(new FileContent("audit_1.txt", new String[0]), new FileContent("audit_2.txt", new String[]{"Peter; 2023-04-10Y17:00:00", "Jane; 2023-04-10Y17:00:00", "Jack; 2023-04-10Y17:00:00"}));
        FileUpdate update = sut.addRecord(files, "Alice", now);

        assertEquals("audit_3.txt", update.fileName);
        assertEquals("Alice;" + now, update.newContent);
    }
}
```
초기버전 -> mock 사용 -> 출력 기반 
- 회귀 방지 : 좋음 -> 좋음 -> 좋음
- 리팩토링 내성 : 좋음 -> 좋음 -> 좋음
- 빠른 피드백 : 나쁨 -> 좋음 -> 좋음
- 유지 보수성 : 나쁨 -> 중간 -> 좋음 


## 6.5 함수형 아키텍처의 단점 이해하기 

### 6.5.1 함수형 아키텍처 적용 가능성

### 6.5.2 성능 단점
- 전통적 아키텍처 : 성능 vs 함수형 아키텍처 : 유지 보수성 
- 함수형 아키텍처는 유지 보수성 향상을 위해 성능을 희생함 
- 상황에 맞게 선택해야 함 

### 6.5.3 코드베이스 크기 증가
- 초기에 코드베이스의 크기가 증가함 
- 시스템의 복잡도와 중요성을 고려해 함수형 아키텍처를 전략적으로 적용해야 함 
- 함수형 방식에서 순수성에 많은 비용이 든다면 순수성을 따르지 말 것