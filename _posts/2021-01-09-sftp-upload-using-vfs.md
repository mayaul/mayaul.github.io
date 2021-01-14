---
layout: post
title: "SFTP 파일 업로드"
excerpt: "apache vfs 를 이용한 sftp 파일 업로드"
date: 2021-01-09 00:00:00 +0900
comments: true
published : true
tag:
- sftp
- java
- apache common vfs
---
### 들어가면서
* SFTP 를 이용해서 파일 업로드 하는 방법을 알아보겠습니다.
* 여러가지 방법이 있지만, 저는 apache 에 있는 VFS 를 이용해서 진행을 하겠습니다.

### 의존성
* 기본적으로 [vfs2](https://mvnrepository.com/artifact/org.apache.commons/commons-vfs2) 의존성을 넣어주면 됩니다.
* 하지만, [jsch](https://mvnrepository.com/artifact/com.jcraft/jsch) 를 넣어주지 않으면 sftp 로 전송을 할 수 없습니다.
``` groovy
compile('org.apache.commons:commons-vfs2:2.7.0')
compile('com.jcraft:jsch:0.1.55')
```

### 업로드 코드
``` java
public void upload(File file, String targetDir, String targetFilename, ConnectionConfig connectionConfig) {

    try (StandardFileSystemManager manager = new StandardFileSystemManager()) {
        manager.init();
        
        FileObject localFile = manager.toFileObject(file);
    
        String connectionString = createConnectionString(connectionConfig.getHost(), connectionConfig.getUsername(), connectionConfig.getPassword(), targetDir + "/" + targetFilename);
        FileObject remoteFile = manager.resolveFile(connectionString, createDefaultOptions());
        
        remoteFile.copyFrom(localFile, Selectors.SELECT_SELF);
    
        System.out.println("File upload success");
    } catch (Exception e) {
        log.error("파일업로드에 실패했습니다.", e);
    }
}

public static String createConnectionString(String hostName, String username, String password, String remoteFilePath) {
    return "sftp://" + username + ":" + password + "@" + hostName + remoteFilePath;
}

public static FileSystemOptions createDefaultOptions() throws FileSystemException {
    FileSystemOptions opts = new FileSystemOptions();

    // SSH Key checking
    SftpFileSystemConfigBuilder.getInstance().setStrictHostKeyChecking(opts, "no");

    // Root directory set to user home
    SftpFileSystemConfigBuilder.getInstance().setUserDirIsRoot(opts, false);

    // Timeout is count by Milliseconds
    SftpFileSystemConfigBuilder.getInstance().setSessionTimeoutMillis(opts, 10000);

    return opts;
}
```

``` java
public class ConnectionConfig {
    private String hostname;
    private int port;
    private String username;
    private String password;

    @Builder
    public ConnectionConfig(String hostname, int port, String username, String password) {
        this.hostname = hostname;
        this.port = port;
        this.username = username;
        this.password = password;
    }

    public String getHost() {
        return String.format("%s:%s", hostname, port);
    }
}
```

### 문제 해결
#### jcsh 의존성이 없을 때
* vfs2 패키지에 들어가서 보면 `providers.xml` 파일이 있습니다.
    ![providers.xml](/assets/img/posts/sftp/sftp-1.png){: width="30%" height="30%"}
* 여기서 sftp schema 설정이 되어있는데, `com.jcraft.jsch.JSch` 를 필요로 합니다.

    ``` xml
    <providers>
        <!-- 중략..  -->
    
        <provider class-name="org.apache.commons.vfs2.provider.sftp.SftpFileProvider">
            <scheme name="sftp"/>
            <if-available class-name="javax.crypto.Cipher"/>
            <if-available class-name="com.jcraft.jsch.JSch"/>
        </provider>
        <!-- 중략..  -->
    </providers>
    ```
* 이 설정이 없다면 `DefaultFileSystemManager#resolveFile` 에서 에러가 발생이 됩니다.
* 오류 StackTrace

    ``` java
    org.apache.commons.vfs2.FileSystemException: Could not find file with URI "sftp://user:***@host/path/filename.csv" because it is a relative path, and no base URI was provided.
        at org.apache.commons.vfs2.FileSystemException.requireNonNull(FileSystemException.java:87)
        at org.apache.commons.vfs2.impl.DefaultFileSystemManager.resolveFile(DefaultFileSystemManager.java:728)
        at org.apache.commons.vfs2.impl.DefaultFileSystemManager.resolveFile(DefaultFileSystemManager.java:648)
    ```

#### 비밀번호에 `@` 가 있으면 안된다. 
* `HostFileNameParser#extractToPath` 에 보면 전달 된 URI 를 파싱하고 있습다.
``` java
// 중략
final String userInfo = extractUserInfo(name);
final String userName;
final String password;
// 중략
```
* `extractUserInfo` 소스코드를 보면 URI 에서 `@` 를 찾아서 거기까지 잘라서 userInfo 로 리턴을 합니다.
    - 앞에서 순차검색을 해서 `@` 까지 잘라서, userInfo 로 자르기 때문에 잘못된 파싱이 되어서 에러가 발생이 됩니다.
  
      ``` java
      final int maxlen = name.length();
      for (int pos = 0; pos < maxlen; pos++) {
          final char ch = name.charAt(pos);
          if (ch == '@') {
              // Found the end of the user info
              final String userInfo = name.substring(0, pos);
              name.delete(0, pos + 1);
              return userInfo;
          }
          if (ch == '/' || ch == '?') {
              // Not allowed in user info
              break;
          }
      }
      
      return null;
      ```
* 오류 StackTrace
``` java
org.apache.commons.vfs2.FileSystemException: Invalid absolute URI "sftp://user:***@host/path/filename.csv".
	at org.apache.commons.vfs2.provider.AbstractOriginatingFileProvider.findFile(AbstractOriginatingFileProvider.java:52)
	at org.apache.commons.vfs2.impl.DefaultFileSystemManager.resolveFile(DefaultFileSystemManager.java:711)
	at org.apache.commons.vfs2.impl.DefaultFileSystemManager.resolveFile(DefaultFileSystemManager.java:648)
```

#### 비밀번호에 `#` 이 있으면 안된다. 
* 위에 `@` 문제를 통과를 하고나면, `UriParser#checkUriEncoding` decoding 을 하게 됩니다. 
* 여기서 `#` 문자열이 있다면 HEX값으로 생각을 하고 decoding 을 하게 됩니다. 
* 그래서 저는 password 를 encoding 해서 설정했습니다. 
    ``` java 
    public static String createConnectionString(String hostName, String username, String password, String remoteFilePath) throws UnsupportedEncodingException {
        return "sftp://" + username + ":" + URLEncoder.encode(password, StandardCharsets.UTF_8.name()) + "@" + hostName + remoteFilePath;
    }
    ```
* 위 예제 코드에 있는 `createConnectionString` 를 encoding 하는것으로 변경하면 더이상 에러는 발생하지 않을겁니다.
* 바로 위에 있는 `@` 문제도 encoding 을 하니까 해결이 되네요
* 오류 StackTrace
```
org.apache.commons.vfs2.FileSystemException: Invalid absolute URI "sftp://user:***@host/path/filename.csv".
	at org.apache.commons.vfs2.provider.AbstractOriginatingFileProvider.findFile(AbstractOriginatingFileProvider.java:52)
	at org.apache.commons.vfs2.impl.DefaultFileSystemManager.resolveFile(DefaultFileSystemManager.java:711)
	at org.apache.commons.vfs2.impl.DefaultFileSystemManager.resolveFile(DefaultFileSystemManager.java:648)
```

### 참고 사이트
> https://commons.apache.org/proper/commons-vfs/  
> https://m.blog.naver.com/PostView.nhn?blogId=racoon_z&logNo=220915890265&proxyReferer=https:%2F%2Fwww.google.com%2F  
> https://www.baeldung.com/java-file-sftp  