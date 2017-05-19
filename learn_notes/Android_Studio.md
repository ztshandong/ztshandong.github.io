Error:A problem was found with the configuration of task ':watch:packageOfficialDebug'.
> File 'D:\Code\XTC_VersionCompatible\watch\build\intermediates\res\resources-official-debug-stripped.ap_' specified for property 

# 全局加速
```java
在~/.gradle目录下新建init.gradle输入以下可全局加速，但如果用的是jcenter()这种格式就无效了
allprojects{
    repositories {
        def REPOSITORY_URL = 'http://maven.aliyun.com/nexus/content/groups/public/'
        all { ArtifactRepository repo ->
            def url = repo.url.toString()
            if ((repo instanceof MavenArtifactRepository) && (url.startsWith('https://repo1.maven.org/maven2') || url.startsWith('https://jcenter.bintray.com'))) {
                project.logger.lifecycle 'Repository ${repo.url} replaced by $REPOSITORY_URL .'
                remove repo
            }
        }
        maven {
            url REPOSITORY_URL
        }
    }
}
```
# 每个项目的根目录build.gradle加速，下载的包在gradle/wrapper/dists下，也可手动下载放进去
```java
buildscript {
    repositories {
        maven{ url 'http://maven.aliyun.com/nexus/content/groups/public/'}
    }
allprojects {
    repositories {
        maven{ url 'http://maven.aliyun.com/nexus/content/groups/public/'}
    }
}
```
# app/src/build.gradle
```java

 debug {//调试模式
            // 显示Log
            buildConfigField "boolean", "LOG_DEBUG", "true"
            versionNameSuffix "-debug"
            //不做混淆
            minifyEnabled false
            zipAlignEnabled true
            shrinkResources false
            signingConfig signingConfigs.release//设置签名信息
        }

        release {//发布模式
            //不显示Log
            buildConfigField "boolean", "LOG_DEBUG", "false"
            //做混淆
            minifyEnabled false
            //Zipalign优化
            zipAlignEnabled true
            //移除无用的resource文件
            shrinkResources false
        }
```
