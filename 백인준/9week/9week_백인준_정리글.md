# [Unit Testing] 6장 단위 테스트 스타일

## 6.4 함수형 아키텍처와 출력 기반 테스트로의 전환

- 프로세스 외부 의존성에서 목으로 변경
- 목에서 함수형 아키텍처로 변경

### 감사시스템

```java

@RequiredArgsConstructor
public class AuditManager {
    private final int maxEntriesPerFile;
    private final String directoryName;

    public void addRecord(String visitorName, Date timeOfVisit) throws IOException {
        String[] filePaths = new File(directoryName).list();
        Arrays.sort(Objects.requireNonNull(filePaths), Comparator.comparingInt(this::extractIndex));
        String newRecord = visitorName + ";" + timeOfVisit.toString();
        if (filePaths.length == 0) {
            String newFile = Paths.get(directoryName, "audit_1.txt").toString();
            Files.write(Paths.get(newFile), Collections.singletonList(newRecord));
            return;
        }
        String currentFilePath = filePaths[filePaths.length - 1];
        Path path = Paths.get(currentFilePath);
        List<String> lines = Files.readAllLines(path);
        if (lines.size() < maxEntriesPerFile) {
            lines.add(newRecord);
            Files.write(path, lines);
            return;
        }
        String newFile = Paths.get(directoryName, "audit_" + extractIndex(currentFilePath) + 1 + ".txt").toString();
        Files.write(Paths.get(newFile), Collections.singletonList(newRecord));
    }

    private int extractIndex(String filePath) {
        String fileName = new File(filePath).getName();
        return Integer.parseInt(fileName.substring(6, fileName.lastIndexOf(".")));
    }
}
```

- 감사시스템은 조직의 모든 방문자를 추적하는것으로 텍스트 파일을 기반 저장소로 사용
- 해당 시스템은 파일 시스템과 밀접하게 연결돼 있어 테스트 하기 어려움
- 파일 시스템은 공유의존성

### 테스트를 파일 시스템에서 분리하기 위한 목 사용

```java
public interface FileSystemInterface {
    String[] getFiles(String directoryName);

    void writeAllText(String filePath, String content);

    List<String> readAllLines(String filePath);
}
```

```java

@RequiredArgsConstructor
public class AuditManager {
    private final int maxEntriesPerFile;
    private final String directoryName;
    private final FileSystemImpl fileSystem;

    public void addRecord(String visitorName, Date timeOfVisit) throws IOException {
        String[] filePaths = fileSystem.getFiles(directoryName);
        Arrays.sort(filePaths, Comparator.comparingInt(this::extractIndex));
        String newRecord = visitorName + ";" + timeOfVisit.toString();
        if (filePaths.length == 0) {
            String newFile = Paths.get(directoryName, "audit_1.txt").toString();
            fileSystem.writeAllText(newFile, newRecord);
            return;
        }
        String currentFilePath = filePaths[filePaths.length - 1];
        List<String> lines = fileSystem.readAllLines(currentFilePath);
        if (lines.size() < maxEntriesPerFile) {
            lines.add(newRecord);
            String newContent = String.join("\r\n", lines);
            fileSystem.writeAllText(currentFilePath, newContent);
            return;
        }
        String newName = "audit_" + extractIndex(currentFilePath) + 1 + ".txt";
        String newFile = Paths.get(directoryName, newName).toString();
        fileSystem.writeAllText(newFile, newRecord);
    }

    private int extractIndex(String filePath) {
        String fileName = Paths.get(filePath).getFileName().toString();
        return Integer.parseInt(fileName.substring(6, fileName.lastIndexOf(".")));
    }
}
```

- 파일 시스템을 목으로 처리하면 해당시스템은 파일 시스템에서 분리되므로 공유의존성이 사라지고 테스트를 독립적으로 실행 가능
- 테스트는 더이상 파일 시스템에 접근하지 않아 더빨리 실행 및 파일 시스템을 다루지 않아 유지비도 절감

### 함수형 아키텍처로 리팩터링 하기

![함수형](https://user-images.githubusercontent.com/58027908/230746724-bb5b2ddd-b859-45ff-833e-621b6e0e7e74.jpg)

- persister 클래스를 생성 (사이드 이펙트 클래스 해당 시스템에서는 파일 시스템 가변셸 역할)
- 기존 결합돼있던 AuditManager 는 함수형 코어로써만 역할을 한다. (결정만 책임)

```java

@RequiredArgsConstructor
public class AuditManager {
    private final int maxEntriesPerFile;

    public FileUpdate addRecord(FileContent[] files, String visitorName, LocalDateTime timeOfVisit) {
        Arrays.sort(files);
        String newRecord = visitorName + ";" + timeOfVisit;
        if (files.length == 0) {
            return new FileUpdate("audit_1.txt", newRecord);
        }
        int currentFileIndex = files.length - 1;
        FileContent currentFile = files[currentFileIndex];
        List<String> lines = Arrays.asList(currentFile.getLines());
        if (lines.size() < maxEntriesPerFile) {
            lines.add(newRecord);
            return new FileUpdate(currentFile.getFileName(), String.join("\r\n", lines));
        }
        return new FileUpdate("audit_" + currentFileIndex + 1 + ".txt", newRecord);
    }
}
```

- AuditManger 는 파일시스템에 대한 결정을 내리는 모든것을 안다.

```java

@Getter
public record FileContent(String fileName, String[] lines) {
}
```

```java

@Getter
public record FileUpdate(String fileName, String newContent) {
}
```

`Persister`는 가변 셸 역할을 한다.

```java
public class Persister {
    public FileContent[] readDirectory(String directoryName) {
        return Arrays.stream(Objects.requireNonNull(new File(directoryName).listFiles()))
                .map(file -> {
                    try {
                        return new FileContent(file.getName(), Files.readAllLines(file.toPath()).toArray(String[]::new));
                    } catch (IOException e) {
                        throw new RuntimeException(e);
                    }
                })
                .toArray(FileContent[]::new);
    }

    public void applyUpdate(String directoryName, FileUpdate update) {
        String filePath = Paths.get(directoryName, update.getFileName()).toString();
        try {
            Files.write(Paths.get(filePath), update.getNewContent().getBytes());
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

```java
public class ApplicationService {
    private final String directoryName;
    private final AuditManager auditManager;
    private final Persister persister;

    public ApplicationService(String directoryName, int maxEntriesPerFile) {
        this.directoryName = directoryName;
        this.auditManager = new AuditManager(maxEntriesPerFile);
        this.persister = new Persister();
    }

    public void addRecord(String visitorName, LocalDateTime timeOfVisit) {
        FileContent[] files = persister.readDirectory(directoryName);
        FileUpdate update = auditManager.addRecord(files, visitorName, timeOfVisit);
        persister.applyUpdate(directoryName, update);
    }
}
```

- ApplicationService 는 함수형 코어와 가변셸을 붙이고 육각형 아키텍처의 어플리케이션 계층 처럼 외부 클라이언트를 위한 시스템의 진입접 제공
- 목을 사용한 테스트와 같이 빠른 피드백 뿐만 아니라 유지보수성 지표도 향상
- |초기 버전|Mock 사용|출력 기반|\
  |회귀 방지|좋음|좋음|좋음|\
  |리팩터링 내성|좋음|좋음|좋음|\
  |빠른 피드백|나쁨|좋음|좋음|\
  |유지 보수성|나쁨|중간|좋음|

### 예상되는 추가 개발
- 함수형 아키텍처를 사용하게 되면 오류처리가 더욱 간단해지고 명확해진다.

## 6.5 함수형 아키텍처의 단점 이해하기
- 코드베이스가 커지고 성능에 영향을 미치면서 유지 보수성의 이점이 상쇄된다.

### 함수형 아키텍처 적용 가능성
- 함수형 아키텍처에 숨은 입력이 생기게 되면 수학적 함수가 될수 없다.

###성능 단점
- 프로세스 외부 의존성을 더 많이 호출 하여 그결과 성능이 떨어진다.
- 함수형 아키텍처와 전통적인 아키텍처 사이의 선택은 성능과 코드 유지보수성인데 성능영향이 그다지 
크지 않은 일부 시스템에서는 함수형 아키텍처를 적용하여 유지보수성을 향상시키는 편이 낫다.

###코드 베이스 크기 증가
- 함수형 아키텍처는 함수형 코어와 가변셸을 명확히 분리해야 하고 이결과 코드 복잡도가 낮아지고 유지보수성이 
향상 되지만 초기 코딩이 더 필요하다.
- 시스템의 복잡도와 중요성을 고려해 함수형 아키텍처를 전략적으로 적용
- 순수성에 많은 비용이 든다면 순수성을 따르지말라 모든 도메인을 불변으로 할수 없기 때문에