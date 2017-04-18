Error:A problem was found with the configuration of task ':watch:packageOfficialDebug'.
> File 'D:\Code\XTC_VersionCompatible\watch\build\intermediates\res\resources-official-debug-stripped.ap_' specified for property 


build.gradle
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