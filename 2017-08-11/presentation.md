<!--markpress-opt

{
	"layout": "horizontal",
	"autoSplit": true,
	"sanitize": false,
	"theme": "light",
	"noEmbed": false
}

markpress-opt-->
# 自动化部署（JAVA篇）

## 1.wagon-maven实现自动化部署
> [wagon-maven](http://maven.apache.org/wagon/)是一个maven构建插件，集成maven命令，可以将maven项目进行一键编译、打包、上传、服务器运行，使得JaveEE项目的部署变得非常简单。

## 2.ansible自动化部署(下期分享)

# 1.wagon-maven实现自动化部署

- what：[wagon-maven](http://maven.apache.org/wagon/)是一个maven构建插件，其运行必须依赖于maven，非maven项目是不能使用wagon的。
- why：一个正常的JaveEE项目想要在远程服务器上跑起来，至少需要经历以下步骤：
  + 1.编译、打成jar包或者war包，得到可执行文件。
  + 2.通过ftp、scp命令或者其他途径将打包后的文件上传至服务器。
  + 3.通过SSH或者其他途径登录服务器。
  + 4.重启tomcat或执行相关shell脚本。
  + 而 wagon-maven 则是将以上的命令或者更多的操作集成在一条命令上，只要一运行该命令，则完成了所有的部署操作。
- how：wagon-maven的使用需要在pom.xml中进行相关配置，详细配置见下文。

# 1-1.wagon-maven的使用

- 配置pom.xml
  + 1.在pom.xml中的build模块扩展中加入wagon-ssh的坐标，示例如下：
    ```java
    <extensions>
          <extension>
              <groupId>org.apache.maven.wagon</groupId>
              <artifactId>wagon-ssh</artifactId>
              <version>${wagon-ssh-version}</version>
          </extension>
      </extensions>
    ```

# 1-2.wagon-maven的使用

- 配置pom.xml
  + 1.在pom.xml中的build模块插件中加入wagon-maven插件，指定远程服务器ip和密码，指定需要上传的可执行文件，指定需要执行的命令，指定示例如下：
    ```
    <plugin>
          <groupId>org.codehaus.mojo</groupId>
          <artifactId>wagon-maven-plugin</artifactId>
          <version>${wagon-maven-plugin-version}</version>
          <configuration>
              <fromFile>target/${package-name}</fromFile>
              <url><![CDATA[scp://${server-username}@${server-ip}${service-path}]]></url>
              <commands>
                  <command>pkill -f ${package-name}</command>
                  <command>rm -f ${service-path}/lpdj.log</command>
                  <command>
                      <![CDATA[nohup java -jar ${service-path}/${package-name} --spring.profiles.active=pro > ${service-path}lpdj.log 2>&1 & ]]></command>
                  <command><![CDATA[netstat -nptl]]></command>
                  <command><![CDATA[ps -ef | grep java]]></command>
              </commands>
              <!-- 运行命令 mvn clean package wagon:upload-single wagon:sshexec-->
              <displayCommandOutputs>true</displayCommandOutputs>
          </configuration>
      </plugin>
    ```

# 1-3.wagon-maven的使用

- 一键部署项目
  + 1.项目打包命令：
    ```
    mvn clean package
    ```
  + 2.项目上传命令：
    ```
    mvn wagon:upload-single
    ```
  + 3.项目上传并运行指定linux命令：
    ```
    mvn wagon:upload-single wagon:sshexec
    ```
  + 4.打包、上传、运行项目命令：（在命令行运行以下命令可快速部署springboot项目）
    ```
    mvn clean package wagon:upload-single wagon:sshexec
    ```

# 1-4.wagon-maven的使用

- 简单案例演示 ：``` mvn wagon:upload-single wagon:sshexec ```

# 下期分享
## 1.ansible自动化部署&自动化运维
