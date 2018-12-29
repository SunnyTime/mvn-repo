# mvn-repo
mvn
## 1. 配置属性

在module 中添加属性配置文件 ，gradle.properties

```
POM_NAME=custom-view-clearedittext
POM_ARTIFACT_ID=custom-view-clearedittext
POM_PACKAGING=aar
VERSION_NAME=1.0.0-SNAPSHOT
VERSION_CODE=1
GROUP=com.yangyang.maven

POM_DESCRIPTION=
POM_URL=
POM_SCM_URL=
POM_SCM_CONNECTION=
POM_SCM_DEV_CONNECTION=
POM_LICENCE_NAME=The Apache Software License, Version 2.0
POM_LICENCE_URL=http://www.apache.org/licenses/LICENSE-2.0.txt
POM_LICENCE_DIST=repo
POM_DEVELOPER_ID=
POM_DEVELOPER_NAME=
RELEASE_REPOSITORY_URL=file:./MavenArr/release
SNAPSHOT_REPOSITORY_URL=file:./MavenArr/snapshots

设置名称与id
POM_NAME=custom-view-seekbar
POM_ARTIFACT_ID=custom-view-seekbar
POM_PACKAGING=aar

设置版本号
VERSION_NAME=1.0.0-SNAPSHOT
VERSION_CODE=1

设置组名称
GROUP=com.dushiguang.maven

设置aar输入地址
RELEASE_REPOSITORY_URL=file:./MavenArr/release
SNAPSHOT_REPOSITORY_URL=file:./MavenArr/snapshots
```

## 2.引用插件

```
apply plugin: 'maven'
apply plugin: 'signing'

def isReleaseBuild() {
    return VERSION_NAME.contains("SNAPSHOT") == false
}

def getReleaseRepositoryUrl() {
    return hasProperty('RELEASE_REPOSITORY_URL') ? RELEASE_REPOSITORY_URL
            : "https://oss.sonatype.org/service/local/staging/deploy/maven2/"
}

def getSnapshotRepositoryUrl() {
    return hasProperty('SNAPSHOT_REPOSITORY_URL') ? SNAPSHOT_REPOSITORY_URL
            : "https://oss.sonatype.org/content/repositories/snapshots/"
}

def getRepositoryUsername() {
    return hasProperty('NEXUS_USERNAME') ? NEXUS_USERNAME : ""
}

def getRepositoryPassword() {
    return hasProperty('NEXUS_PASSWORD') ? NEXUS_PASSWORD : ""
}

afterEvaluate { project ->
    uploadArchives {
        repositories {
            mavenDeployer {
                beforeDeployment { MavenDeployment deployment -> signing.signPom(deployment) }

                pom.groupId = GROUP
                pom.artifactId = POM_ARTIFACT_ID
                pom.version = VERSION_NAME

                repository(url: getReleaseRepositoryUrl()) {
                    authentication(userName: getRepositoryUsername(), password: getRepositoryPassword())
                }
                snapshotRepository(url: getSnapshotRepositoryUrl()) {
                    authentication(userName: getRepositoryUsername(), password: getRepositoryPassword())
                }

                pom.project {
                    name POM_NAME
                    packaging POM_PACKAGING
                    description POM_DESCRIPTION
                    url POM_URL

                    scm {
                        url POM_SCM_URL
                        connection POM_SCM_CONNECTION
                        developerConnection POM_SCM_DEV_CONNECTION
                    }

                    licenses {
                        license {
                            name POM_LICENCE_NAME
                            url POM_LICENCE_URL
                            distribution POM_LICENCE_DIST
                        }
                    }

                    developers {
                        developer {
                            id POM_DEVELOPER_ID
                            name POM_DEVELOPER_NAME
                        }
                    }
                }
            }
        }
    }

    signing {
        required { isReleaseBuild() && gradle.taskGraph.hasTask("uploadArchives") }
        sign configurations.archives
    }

    task androidJavadocs(type: Javadoc) {
        source = android.sourceSets.main.java.srcDirs
        classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
    }

    task androidJavadocsJar(type: Jar, dependsOn: androidJavadocs) {
        classifier = 'javadoc'
        from androidJavadocs.destinationDir
    }

    task androidSourcesJar(type: Jar) {
        classifier = 'sources'
        from android.sourceSets.main.java.sourceFiles
    }

    artifacts {
        archives androidSourcesJar
        archives androidJavadocsJar
    }
}

```
gradle-mvn-push.gradle在根目录下面
在module 中引用插件脚本
apply from: file('../gradle-mvn-push.gradle')
## 3.打包
在AndroidStudio控制台输入：./gradlew uploadArchives -p *** 命令打包 *** 自己的module名称
项目中生成的文件

## 4.上传到gitHab
## 5.设置引用地址
```
allprojects {
    repositories {
        google()
        jcenter()

        //添加地址
        maven {
            url "https://github.com/SunnyTime/mvn-repo/raw/master/snapshots/"
        }
    }
}
```
进入GitHub中方lib的项目中的第一级目录在浏览器中的复制url将 https://github.com/SunnyTime/mvn-repo/tree/master/snapshots 将tree修改成raw。

## 6.引用到项目工程
引用远程版本
```
compile 'com.dushiguang.maven:custom-view-seekbar:1.0.0-SNAPSHOT'
```
上面的信息可以在打包生成的文件中的maven-metadata.xml中找到。

## 7.注意事项
输入命令会检测一些规范，导致编译失败，可以项目中build.gradle文件添加
```
tasks.getByPath(":clearedittext:androidJavadocs").enabled = false
```
有时版本更新不下来可能是Studio缓存问题，可以更改缓存配置：
```
configurations.all {
    // 检查远程依赖是否存在更新
    resolutionStrategy.cacheChangingModulesFor 0, 'seconds'
    // 采用动态版本声明的依赖缓存
    resolutionStrategy.cacheDynamicVersionsFor 0, 'seconds'
}
```
