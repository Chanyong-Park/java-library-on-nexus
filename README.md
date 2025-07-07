# 개요
  - java gradle 프로젝트의 공통 라이브러리를 개발하고 private 저장소인 넥서스에 업로드하여 관리
  - [넥서스 설치](https://github.com/Chanyong-Park/nexus-settings/blob/main/README.md)

## Nexus repository에 배포하기
### gradle.properties 파일 생성
  - 프로젝트 root 디렉토리에 생성하거나 C:\Users\<user명>\.gradle\ 디렉토리에 파일을 생성
  - 파일 내용
```
    nexusUrl=http://localhost:8081/repository/maven-public/
    nexusUrl-releases=http://localhost:8081/repository/maven-releases/
    nexusUrl-snapshots=http://localhost:8081/repository/maven-snapshots/
    nexusUsername=admin
    nexusPassword=admin
    systemProp.org.gradle.internal.http.connectionTimeout=180000
    systemProp.org.gradle.internal.http.socketTimeout=180000
```

### gradle.build 설정
```
plugins {
    id 'java-library'    // 또는 id 'java'
    id 'maven-publish'
}

group = 'com.cooldragon'
version = '0.0.3-SNAPSHOT'

...

publishing {
    publications {
        mavenJava(MavenPublication) {
            from components.java
        }
    }
    repositories {
        maven {
            name = "nexus"
            url = version.endsWith('SNAPSHOT') ? uri(findProperty("nexusUrl-snapshots"))
                                               : uri(findProperty("nexusUrl-releases"))
            credentials {
                username = findProperty("nexusUsername")
                password = findProperty("nexusPassword")
            }
			allowInsecureProtocol = true // HTTP 허용
        }
    }
}
```

### 빌드 및 jar 업로드
```
$ ./gradlew clean build
$ ./gradlew publish
```

### nexus의 repository에서 생성 확인
(1) 해당 리포지토리에서 생성여부를 확인

## 다른 프로젝트에서 해당 Library 사용하기
### gradle.properties 을 동일하게 가져와서 사용

### gradle.build 설정
```
...
repositories {
	mavenCentral()
    maven {
		url = uri(findProperty("nexusUrl"))
		allowInsecureProtocol = true // HTTP 허용
		credentials {
			username = findProperty("nexusUsername")
			password = findProperty("nexusPassword")
		}
    }
}
...
dependencies {
...
    implementation 'com.cooldragon:common:0.0.3-SNAPSHOT'   // 배포된 라이브러리 group, artifact, 버전 설정
...
}
...
```
