# JAVA_HOME
```sh
sudo vi /etc/profile

export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64

export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$CLASSPATH

export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH

source /etc/profile
```

# 单例
```java
public class Singleton {  
    private static class SingletonHolder {  
    private static final Singleton INSTANCE = new Singleton();  
    }  
    private Singleton (){}  
    public static final Singleton getInstance() {  
    return SingletonHolder.INSTANCE;  
    }  
}  
```