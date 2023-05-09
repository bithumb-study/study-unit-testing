# 📚 6장 단위 테스트 스타일

## 📖 6.4 함수형 아키텍처와 출력 기반 테스트로의 전환

___

이 절에서는 샘플 코드를 함수형 아키텍처로 리팩터링 하는 것을 보여준다. 두 가지 리팩터링 단계를 보여준다.

1. 프로세스 외부 의존성에서 Mock으로 변경
2. Mock에서 함수형 아키텍처로 변경

### 🔖 6.4.1 감사 시스템 소개

조직의 모든 방문자를 추적하는 감사 시스템이 샘플이다.

아래 코드는 초기 버전이다.

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

`AuditorManager` class는 파일 시스템과 밀접하게 연결돼 있어 그대로 테스트하기가 어렵다.

### 🔖 6.4.2 테스트를 파일 시스템에서 분리하기 위한 목 사용

파일의 모든 연산을 인터페이스로 도출한 후 AuditManager 클래스에 IFileSystemImpl 구현 클래스를 주입 받아 사용할 수 있다.

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

이제 공유 의존성이 사라지고 테스트를 서로 독립적으로 실행할 수 있다. Mock을 사용한다면 더 이상 파일 시스템에 접근하지 않으므로 더 빨리 실행된다. 초기 버전보다 더욱 개선이 된 것이다.

### 🔖 6.4.3 함수형 아키텍처로 리팩터링하기

`AuditorManager` 는 함수형 코어에 해당된다.

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

위와 같이 함수형 코어와 가변 셸을 붙이면 진입점이 제공된다. 모든 테스트는 작업 디렉터리의 가상 상태를 제공하고 `AuditorManager`가 내린 결정을 검증하는 것으로 단축됐다.

||초기 버전|Mock 사용|출력 기반
|:---:|:---:|:---:|:---:
|회귀 방지|좋음|좋음|좋음
|리팩터링 내성|좋음|좋음|좋음
|빠른 피드백|나쁨|좋음|좋음
|유지 보수성|나쁨|중간|좋음

### 🔖 6.4.4 예상되는 추가 개발

함수형 아키텍처는 추가 요구사항에 대해 쉽게 개발과 대응을 할 수 있다.

## 📖 6.5 함수형 아키텍처의 단점 이해하기

___

함수형 아키텍처라고해도, 코드베이스가 커지고 성능에 영향을 미치면서 유지 보수성의 이점이 상쇄된다.

### 🔖 6.5.1 함수형 아키텍처 적용 가능성

숨은 입력이 생기면 수학적 함수가 될 수 없으며, 출력 기반 테스트를 적용할 수 없다.

이에 대한 해결은 성능 저하 혹은 아키텍처가 망가지는 결과를 가져온다.

### 🔖 6.5.2 성능 단점

함수형 아키텍처와 전통적인 아키텍처 사이의 선택은 성능과 코드 유지 보수성 간의 절충이다. 성능 영향이 그다지 눈에 띄지 않는 일부 시스템에서는 함수형 아키텍처를 사용해 유지 보수성을 향상시키는 편이 낫다.

### 🔖 6.5.3 코드베이스 크기 증가

함수형 아키텍처는 코드 복잡도가 낮아지고 유지 보수성이 증가하지만, 초기에 코딩이 더 필요하다.
또한, 함수형 방식에서 순수성에 많은 비용이 든다면 순수성을 따르지 말라.
