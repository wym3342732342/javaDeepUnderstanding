# 本地setting.xml

```xml
<servers>
        <server>
            <id>rdc-releases</id>
            <username>608bb3b56d822d6b010a051c</username>
            <password>Sie7JeXuqY=e</password>
        </server>
        <server>
            <id>rdc-snapshots</id>
            <username>608bb3b56d822d6b010a051c</username>
            <password>Sie7JeXuqY=e</password>
        </server>
 </servers>
```



# 项目POM.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>test</artifactId>
    <version>${project.release.version}</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.release.version>0.5</project.release.version>

    </properties>
    <distributionManagement>
        <repository>
            <id>rdc-releases</id>
            <name>阿里云正式</name>
            <url>https://packages.aliyun.com/maven/repository/2100373-release-vf3cOI/</url>
        </repository>
        <snapshotRepository>
            <id>rdc-snapshots</id>
            <name>阿里云快照版</name>
            <url>https://packages.aliyun.com/maven/repository/2100373-snapshot-lKMgAl/</url>
        </snapshotRepository>

    </distributionManagement>
    <!-- 单一项目使用该仓库 -->
    <repositories>
        <repository>
            <id>central</id>
            <url>https://maven.aliyun.com/nexus/content/groups/public</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
        <repository>
            <id>snapshots</id>
            <url>https://maven.aliyun.com/nexus/content/groups/public</url>
            <releases>
                <enabled>false</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
        <repository>
            <id>rdc-releases</id>
            <url>https://packages.aliyun.com/maven/repository/2100373-release-vf3cOI/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
        <repository>
            <id>rdc-snapshots</id>
            <url>https://packages.aliyun.com/maven/repository/2100373-snapshot-lKMgAl/</url>
            <releases>
                <enabled>false</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>central</id>
            <url>https://maven.aliyun.com/nexus/content/groups/public</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </pluginRepository>
        <pluginRepository>
            <id>snapshots</id>
            <url>https://maven.aliyun.com/nexus/content/groups/public</url>
            <releases>
                <enabled>false</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </pluginRepository>
        <pluginRepository>
            <id>rdc-releases</id>
            <url>https://packages.aliyun.com/maven/repository/2100373-release-vf3cOI/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </pluginRepository>
        <pluginRepository>
            <id>rdc-snapshots</id>
            <url>https://packages.aliyun.com/maven/repository/2100373-snapshot-lKMgAl/</url>
            <releases>
                <enabled>false</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </pluginRepository>
    </pluginRepositories>
    <build>
        <plugins>
            <!-- 打source包 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
                <version>3.0.1</version>
                <configuration>
                    <attach>true</attach>
                </configuration>
                <executions>
                    <execution>
                        <phase>compile</phase>
                        <goals>
                            <goal>jar</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

