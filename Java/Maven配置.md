



## 配置JDK版本


修改settings.xml ,添加如下内容，配置为JDK1.8


```xml
<profile> 
    <id>jdk-1.8</id>  
    <activation>
  <activeByDefault>true</activeByDefault>
      <jdk>1.8</jdk>  
    </activation>  

    <properties>  
      <maven.compiler.source>1.8</maven.compiler.source>  
      <maven.compiler.target>1.8</maven.compiler.target>  
      <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>  
    </properties>   
  </profile> 
```


## 配置Maven阿里镜像源


修改settings.xml ,添加如下内容，配置为阿里云镜像


```xml
<mirror>
  <id>nexus-aliyun</id>
  <mirrorOf>central</mirrorOf>
  <name>Nexus aliyun</name>
  <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```


## 配置Maven本地仓库


修改settings.xml ,添加如下内容


```xml
<localRepository>/opt/apache-maven-3.6.0/repository</localRepository>
```
