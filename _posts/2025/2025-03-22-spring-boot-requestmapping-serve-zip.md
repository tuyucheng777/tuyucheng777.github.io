---
layout: post
title:  如何使用Spring Boot @RequestMapping提供Zip文件
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

有时我们可能需要允许我们的REST API下载ZIP档案，这对于减少网络负载很有用。但是，我们可能会在端点上使用默认配置下载文件时遇到困难。

在本文中，我们将了解如何使用[@RequestMapping](https://www.baeldung.com/spring-requestmapping)注解从我们的端点生成ZIP文件，并且我们将探索一些从它们提供ZIP档案的方法。

## 2. 将Zip存档压缩为字节数组

**提供ZIP文件的第一种方法是将其创建为字节数组并在HTTP响应中返回**，让我们使用返回存档字节的端点创建REST控制器：

```java
@RestController
public class ZipArchiveController {

    @GetMapping(value = "/zip-archive", produces = "application/zip")
    public ResponseEntity<byte[]> getZipBytes() throws IOException {

        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(byteArrayOutputStream);
        ZipOutputStream zipOutputStream = new ZipOutputStream(bufferedOutputStream);

        addFilesToArchive(zipOutputStream);

        IOUtils.closeQuietly(bufferedOutputStream);
        IOUtils.closeQuietly(byteArrayOutputStream);

        return ResponseEntity
                .ok()
                .header("Content-Disposition", "attachment; filename=\"files.zip\"")
                .body(byteArrayOutputStream.toByteArray());
    }
}
```

我们使用@GetMapping作为[@RequestMapping注解的快捷方式](https://www.baeldung.com/spring-new-requestmapping-shortcuts)。在produce属性中，我们选择application/zip，这是ZIP档案的MIME类型。然后我们用[ZipOutputStream](https://www.baeldung.com/java-compress-and-uncompress)包装ByteArrayOutputStream并在其中添加所有需要的文件。最后，我们用attachment值设置[Content-Disposition](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Disposition)标头，这样我们就可以在调用后下载档案。

现在，让我们实现addFilesToArchive()方法：

```java
void addFilesToArchive(ZipOutputStream zipOutputStream) throws IOException {
    List<String> filesNames = new ArrayList<>();
    filesNames.add("first-file.txt");
    filesNames.add("second-file.txt");

    for (String fileName : filesNames) {
        File file = new File(ZipArchiveController.class.getClassLoader()
                .getResource(fileName).getFile());
        zipOutputStream.putNextEntry(new ZipEntry(file.getName()));
        FileInputStream fileInputStream = new FileInputStream(file);

        IOUtils.copy(fileInputStream, zipOutputStream);

        fileInputStream.close();
        zipOutputStream.closeEntry();
    }

    zipOutputStream.finish();
    zipOutputStream.flush();
    IOUtils.closeQuietly(zipOutputStream);
}
```

在这里，我们只需用资源文件夹中的几个文件填充档案。

最后，让我们调用我们的端点并检查是否返回了所有文件：

```java
@WebMvcTest(ZipArchiveController.class)
public class ZipArchiveControllerUnitTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    void givenZipArchiveController_whenGetZipArchiveBytes_thenExpectedArchiveShouldContainExpectedFiles() throws Exception {
        MvcResult result = mockMvc.perform(get("/zip-archive"))
                .andReturn();

        MockHttpServletResponse response = result.getResponse();

        byte[] content = response.getContentAsByteArray();

        List<String> fileNames = fetchFileNamesFromArchive(content);
        assertThat(fileNames)
                .containsExactly("first-file.txt", "second-file.txt");
    }

    List<String> fetchFileNamesFromArchive(byte[] content) throws IOException {
        InputStream byteStream = new ByteArrayInputStream(content);
        ZipInputStream zipStream = new ZipInputStream(byteStream);

        List<String> fileNames = new ArrayList<>();
        ZipEntry entry;
        while ((entry = zipStream.getNextEntry()) != null) {
            fileNames.add(entry.getName());
            zipStream.closeEntry();
        }

        return fileNames;
    }
}
```

正如响应中所预期的那样，我们从端点获得了ZIP存档。我们从那里解压了所有文件，并仔细检查了所有预期文件是否都已到位。

**对于较小的文件，我们可以使用此方法，但较大的文件可能会导致堆消耗问题。这是因为ByteArrayInputStream将整个ZIP文件保存在内存中**。

## 3. 将Zip存档作为流

对于较大的档案，我们应避免将所有内容加载到内存中。**相反，我们可以在创建ZIP文件时将其直接传输到客户端。这可以减少内存消耗，并使我们能够高效地处理大型文件**。

让我们在控制器上创建另一个端点：

```java
@GetMapping(value = "/zip-archive-stream", produces = "application/zip")
public ResponseEntity<StreamingResponseBody> getZipStream() {
    return ResponseEntity
            .ok()
            .header("Content-Disposition", "attachment; filename=\"files.zip\"")
            .body(out -> {
                ZipOutputStream zipOutputStream = new ZipOutputStream(out);
                addFilesToArchive(zipOutputStream);
            });
}
```

**我们在这里使用了[Servlet输出流](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/server/ServletServerHttpResponse.html)而不是ByteArrayInputStream，因此我们所有的文件都将流式传输到客户端，而无需完全存储在内存中**。

让我们调用这个端点并检查它是否返回我们的文件：

```java
@Test
void givenZipArchiveController_whenGetZipArchiveStream_thenExpectedArchiveShouldContainExpectedFiles() throws Exception {
    MvcResult result = mockMvc.perform(get("/zip-archive-stream"))
            .andReturn();

    MockHttpServletResponse response = result.getResponse();

    byte[] content = response.getContentAsByteArray();

    List<String> fileNames = fetchFileNamesFromArchive(content);
    assertThat(fileNames)
            .containsExactly("first-file.txt", "second-file.txt");
}
```

我们成功检索了档案并且所有文件都已找到。

## 4. 控制档案压缩

当我们使用ZipOutputStream时，它已经提供了压缩功能，**我们可以使用zipOutputStream.setLevel()方法调整压缩级别**。

让我们修改其中一个端点代码来设置压缩级别：

```java
@GetMapping(value = "/zip-archive-stream", produces = "application/zip")
public ResponseEntity<StreamingResponseBody> getZipStream() {
    return ResponseEntity
            .ok()
            .header("Content-Disposition", "attachment; filename=\"files.zip\"")
            .body(out -> {
                ZipOutputStream zipOutputStream = new ZipOutputStream(out);
                zipOutputStream.setLevel(9);
                addFilesToArchive(zipOutputStream);
            });
}
```

我们将压缩级别设置为9，这是最大压缩级别。我们可以在0到9之间选择一个值，**较低的压缩级别可加快处理速度，而较高的压缩级别会产生较小的输出，但会减慢存档速度**。

## 5. 添加存档密码保护

**我们还可以为ZIP档案设置密码**。为此，让我们添加[zip4j依赖](https://mvnrepository.com/artifact/net.lingala.zip4j/zip4j)：

```xml
<dependency>
    <groupId>net.lingala.zip4j</groupId>
    <artifactId>zip4j</artifactId>
    <version>${zip4j.version}</version>
</dependency>
```

现在我们将向控制器添加一个新的端点，在那里返回密码加密的存档流：

```java
import net.lingala.zip4j.io.outputstream.ZipOutputStream;

@GetMapping(value = "/zip-archive-stream-secured", produces = "application/zip")
public ResponseEntity<StreamingResponseBody> getZipSecuredStream() {
    return ResponseEntity
            .ok()
            .header("Content-Disposition", "attachment; filename=\"files.zip\"")
            .body(out -> {
                ZipOutputStream zipOutputStream = new ZipOutputStream(out, "password".toCharArray());
                addFilesToArchive(zipOutputStream);
            });
}
```

这里我们使用了zip4j库中的[ZipOutputStream](https://javadoc.io/doc/net.lingala.zip4j/zip4j/2.7.0/index.html)，它可以处理密码。

现在让我们实现addFilesToArchive()方法：

```java
import net.lingala.zip4j.model.ZipParameters;

void addFilesToArchive(ZipOutputStream zipOutputStream) throws IOException {
    List<String> filesNames = new ArrayList<>();
    filesNames.add("first-file.txt");
    filesNames.add("second-file.txt");

    ZipParameters zipParameters = new ZipParameters();
    zipParameters.setCompressionMethod(CompressionMethod.DEFLATE);
    zipParameters.setEncryptionMethod(EncryptionMethod.ZIP_STANDARD);
    zipParameters.setEncryptFiles(true);

    for (String fileName : filesNames) {
        File file = new File(ZipArchiveController.class.getClassLoader()
                .getResource(fileName).getFile());

        zipParameters.setFileNameInZip(file.getName());
        zipOutputStream.putNextEntry(zipParameters);

        FileInputStream fileInputStream = new FileInputStream(file);
        IOUtils.copy(fileInputStream, zipOutputStream);

        fileInputStream.close();
        zipOutputStream.closeEntry();
    }

    zipOutputStream.flush();
    IOUtils.closeQuietly(zipOutputStream);
}
```

**我们使用ZIP条目的EncryptionMethod和EncryptFiles参数来加密文件**。

最后，让我们调用新的端点并检查响应：

```java
@Test
void givenZipArchiveController_whenGetZipArchiveSecuredStream_thenExpectedArchiveShouldContainExpectedFilesSecuredByPassword() throws Exception {
    MvcResult result = mockMvc.perform(get("/zip-archive-stream-secured"))
            .andReturn();

    MockHttpServletResponse response = result.getResponse();
    byte[] content = response.getContentAsByteArray();

    List<String> fileNames = fetchFileNamesFromArchive(content);
    assertThat(fileNames)
            .containsExactly("first-file.txt", "second-file.txt");
}
```

在fetchFileNamesFromArchive()中，我们将实现从ZIP存档中检索数据的逻辑：

```java
import net.lingala.zip4j.io.inputstream.ZipInputStream;

List<String> fetchFileNamesFromArchive(byte[] content) throws IOException {
    InputStream byteStream = new ByteArrayInputStream(content);
    ZipInputStream zipStream = new ZipInputStream(byteStream, "password".toCharArray());

    List<String> fileNames = new ArrayList<>();
    LocalFileHeader entry = zipStream.getNextEntry();
    while (entry != null) {
        fileNames.add(entry.getFileName());
        entry = zipStream.getNextEntry();
    }

    zipStream.close();

    return fileNames;
}
```

这里我们再次使用zip4j库中的[ZipInputStream](https://javadoc.io/doc/net.lingala.zip4j/zip4j/2.7.0/net/lingala/zip4j/io/inputstream/ZipInputStream.html)并设置我们在加密时使用的密码。否则，我们将遇到ZipException。

## 6. 总结

在本教程中，我们探讨了在Spring Boot应用程序中提供ZIP文件的两种方法。对于中小型档案，我们可以使用字节数组。对于较大的文件，我们应该考虑在HTTP响应中直接流式传输ZIP档案，以保持较低的内存使用率。通过调整压缩级别，我们可以控制网络负载和端点的延迟。