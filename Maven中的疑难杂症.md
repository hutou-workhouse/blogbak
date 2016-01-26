title: Maven中的疑难杂症
date: 2016-01-26 18:53:56
categories: 
-	java
-	Maven

tags: 
-	Maven
-	java
-	技术文章
-	构建工具

---

Maven的疑难杂症

<!--more-->


近期在进行工程整合的时候，又有些Maven使用中发现的问题进行整理

1. 发布和编译
在使用Maven build package 的时候，必须先clean一下。否则本次编译中删除的java代码如果在上次打包中存在，则class依然会出现在打好的包中。
2. 配置文件
我习惯自己书写一个setting.xml，这个文件中指定了私服的地址。在eclipse中进行配置。在eclipse中都没有问题，但是在命令行程序中执行会发现有些私服管理的包就找不到了。将这个文件复制到 **%MAVEN_HOME%\conf**下则可以解决这个问题。Maven在执行中首先检查这个目录，然后检查个人目录**C:\Users\Administrator\\.m2**，为了安全，这个目录下我也放置了一份
3. 检查包依赖
在进行包管理中，会有一些意外的包被引入，导致出现问题。使用如下的命令可以查看包的依赖关系
```
mvn dependency:tree -Dverbose
```
使用下面的命令可以进行内容的过滤
```
# 这里是过滤包组织为org.hibernate的依赖关系
mvn dependency:tree -Dverbose -Dincludes=org.hibernate
```
4. 去除包的依赖
从上面的命令中，我们已经可以判断出产生冲突的包是由什么地方被引入的。如果想剔除这个依赖，需要在引入的地方进行如下处理（**拷贝自网络，去除asm包的引用**）
```
        <dependency>
            <groupId>org.unitils</groupId>
            <artifactId>unitils-dbmaintainer</artifactId>
            <version>${unitils.version}</version>
            <exclusions>
                <exclusion>
                    <artifactId>dbunit</artifactId>
                    <groupId>org.dbunit</groupId>
                </exclusion>
                <!-- 这个就是我们要加的片断 -->
                <exclusion>
                    <artifactId>asm</artifactId>
                    <groupId>asm</groupId>
                </exclusion>
            </exclusions>
        </dependency>
```
5. 运行期查看Jar情况
有些问题在运行期出现，多是由于不同版本的包的兼容问题。这个时候可以使用如下的方法（**取自网络[](http://stamen.iteye.com/blog/2030552)**）
```
package com.ridge.util;
import java.io.File;
import java.net.MalformedURLException;
import java.net.URL;
import java.security.CodeSource;
import java.security.ProtectionDomain;
/**
 * @author : chenxh
 * @date: 13-10-31
 */
public class ClassLocationUtils {
    /**
     * 获取类所有的路径
     * @param cls
     * @return
     */
    public static String where(final Class cls) {
        if (cls == null)throw new IllegalArgumentException("null input: cls");
        URL result = null;
        final String clsAsResource = cls.getName().replace('.', '/').concat(".class");
        final ProtectionDomain pd = cls.getProtectionDomain();
        if (pd != null) {
            final CodeSource cs = pd.getCodeSource();
            if (cs != null) result = cs.getLocation();
            if (result != null) {
                if ("file".equals(result.getProtocol())) {
                    try {
                        if (result.toExternalForm().endsWith(".jar") ||
                                result.toExternalForm().endsWith(".zip"))
                            result = new URL("jar:".concat(result.toExternalForm())
                                    .concat("!/").concat(clsAsResource));
                        else if (new File(result.getFile()).isDirectory())
                            result = new URL(result, clsAsResource);
                    }
                    catch (MalformedURLException ignore) {}
                }
            }
        }
        if (result == null) {
            final ClassLoader clsLoader = cls.getClassLoader();
            result = clsLoader != null ?
                    clsLoader.getResource(clsAsResource) :
                    ClassLoader.getSystemResource(clsAsResource);
        }
        return result.toString();
    }
}
```
在出现问题的地方设置好断点，在断点处动态执行下面的代码
```
# 你运行出问题的类设置为参数
ClassLocationUtils.where(org.objectweb.asm.ClassVisitor.class) 
```
可以马上查到这个类是哪个Jar包中的，可以判断是否符合自己的预期

6. 配置文件
下面是一个比较详细的说明，来自网络[Maven配置setting.xml详细说明](http://www.micmiu.com/software/build/maven-setting-xml/)
```
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <!--指定本地仓库存储路径。默认值为~/.m2/repository 即 ${user.home}/.m2/repository。 -->
  <localRepository>d:/.m2/repository</localRepository>

	<!--指定Maven是否需要和用户输入进行交互。true:需要与用户交互;false:使用一个合理的默认值。默认值为true。 -->
  <interactiveMode>true</interactiveMode>

  <!--指定是否使用plugin-registry.xml文件来管理插件版本。设为true表示使用。默认值为false。-->
  <usePluginRegistry>false</usePluginRegistry>

  <!--指定是否在离线模式下运行。设为true表示项目构建要在离线模式下运行，默认值为false。 -->
  <offline>false</offline>

	<!-- 指定插件groupId列表，用于搜索时插件的groupId没有明确规定。 -->
	<pluginGroups>
	 	<!-- 指定使用插件查找进一步的组标识符 -->
		<pluginGroup>com.micmiu.plugins</pluginGroup>
	</pluginGroups>

  <!-- 指定这台机器连接到网络的代理服务器的列表。除非另有规定（系统属性或命令行开关），
       列表中配置的第一代理将被激活使用。-->
  <proxies>
    <!-- 配置代理服务器的相关参数 -->
    <proxy>
    	<!-- 代理标识ID，默认值：default -->
      <id>micmiuProxy</id>
      <!-- 指定是否激活，默认值：true -->
      <active>true</active>
      <!-- 指定代理协议，默认值：http -->
      <protocol>http</protocol>
      <!-- 指定代理认证的用户名 -->
      <username>micmiu</username>
      <!-- 指定代理认证用户的密码 -->
      <password>mypwd</password>
      <!-- 指定代理服务器的主机名 -->
      <host>micmiu.com</host>
      <!-- 指定代理服务的端口 默认值：8080 -->
      <port>80</port>
      <!-- 指定不被代理的主机名列表。多个用|分隔。-->
      <nonProxyHosts>ctosun.com|ctosun.micmiu.com</nonProxyHosts>
    </proxy>
  </proxies>

  <!-- 这是一个认证配置的列表,系统内部根据配置的serverID使用。认证配置用于maven链接到远程服务-->
  <servers>
    <!-- 指定的身份认证信息用于连接到一个特定的服务器时，确定系统内的唯一的名称（简称下面的'id'属性）。-->
    <server>
    	<!-- 这是server的id（注意不是用户登陆的id）。该id与distributionManagement中repository元素的id必须要匹配。-->
      <id>micmiu-releases</id>
      <!-- 服务器认证的用户名 -->
      <username>michael</username>
      <!-- 服务器认证的用户对应的密码 -->
      <password>mypwd</password>
    </server>

    <!-- 另一个示例 私钥/密码 -->
    <server>
      <id>micmiu-snapshots</id>
      <!-- 认证时使用的私钥文件。 -->
      <privateKey>/home/micmiu/.ssh/id_dsa</privateKey>
      <!-- 认证时使用的私钥密码，没有密码就设为空 -->
      <passphrase>mypwd</passphrase>
      <!-- 目录被创建时的权限设置。其值对应了unix文件系统的权限，如664，或者775 -->
      <directoryPermissions>775</directoryPermissions>
      <!-- 仓库文件创建时的权限设置。其值对应了unix文件系统的权限，如664，或者775。 -->
      <filePermissions>664</filePermissions>
    </server>
  </servers>

  <!-- 指定镜像列表，用于从远程仓库下载资源 -->
  <mirrors>
    <!-- 指定仓库的镜像站点，代替一个给定的库。该镜像藏库有一个ID相匹配的mirrorOf元素。
         ID是用于继承和直接查找目的，必须是唯一的。-->
    <mirror>
    	<!--该镜像的唯一标识符。id用来区分不同的mirror元素。 -->
      <id>mirrorId</id>
      <!--被镜像的服务器的id，比如：central，不能和id匹配。-->
   		<mirrorOf>central</mirrorOf>
      <name>micmiu for dev.</name>
      <url>http://dev.micmiu.com/repo/maven2</url>
    </mirror>

  </mirrors>

  <!-- 这是一个可以在各种不同的方式激活的配置文件列表，并可以修改构建过程。在settings.xml中提供的信息，
       旨在提供本地机器允许建立在本地环境中工作的具体路径和库位置。有多种方式可以激活配置属性：一种在settings.xml中<activeProfiles>指定；
       另一种实质上依赖于系统属性，无论是匹配特定的属性值或只是测试到它的存在.配置文件也可以根据JDK版本的前缀进行激活，1.4 可以激活1.4.2_07
    注：对于在settings.xml中定义的配置，你仅限于指定资源仓库、插件仓库和用于插件在POM中变量的自由形式属性的定义 -->
  <profiles>
    <!-- 指定生成过程的介绍，使用一个或多个上述机制被激活。对于继承而言，激活通过<activatedProfiles/>或命令行配置文件，
         配置文件必须有一个唯一的ID。此配置文件的例子使用的JDK版本触发激活。-->
    <profile>
    	<!--该配置的唯一标识符。 -->
      <id>jdk-1.4</id>

			<!--自动触发配置文件的逻辑定义。Activation的逻辑配置决定了是否开启该profile。activation元素并不是激活profile的唯一方式。
			    settings.xml文件中的activeProfile元素可指定需要激活的profile的id。
			    profile也可以通过在命令行，使用-P标记和逗号分隔的列表来显式的激活 -->
      <activation>
      	<!--指定是否激活的标识 默认值为false-->
    		<activeByDefault>false</activeByDefault>

      	<!--当匹配的jdk被检测到，profile被激活。例如，1.4激活JDK1.4，1.4.0_2，而!1.4激活所有不是以1.4开头的JDK版本。-->
        <jdk>1.4</jdk>

        <!-- 当检测到匹配的操作系统属性时，指定该配置文件将被激活， -->
       	<os>
		     	<!--激活profile的操作系统的名字 -->
		     	<name>windows 7</name>
		     	<!--激活profile的操作系统所属家族(如 'windows')  -->
		     	<family>windows</family>
		    	<!--激活profile的操作系统体系结构  -->
		     	<arch>x86</arch>
		     	<!--激活profile的操作系统版本-->
		     	<version>6.1</version>
	    	</os>

	    	<!-- 检测系统对应的属性和值(该值可在POM中通过${属性名称}引用)，配置就会被激活。
	    	    如果值字段是空的，那么存在属性名称字段就会激活 -->
		    <property>
		     	<!-- 属性的名称 -->
		     	<name>mavenVersion</name>
		     	<!-- 属性的值 -->
		    	<value>3.0.4</value>
		    </property>
		    <!-- 通过检测该文件的是否存在来激活配置。missing检查文件是否存在，如果不存在则激活profile；exists则会检查文件是否存在，如果存在则激活。-->
		    <file>
		    	<!--如果指定的文件存在，则激活profile。 -->
		    	<exists>/usr/local/micmiu/workspace/myfile</exists>
		    	<!--如果指定的文件不存在，则激活profile。-->
		    	<missing>/usr/local/micmiu/workspace/myfile</missing>
		    </file>
      </activation>

			<!-- 对应profile的扩展属性列表。Maven属性和Ant中的属性一样，可以用来存放一些值。这些值可以在POM中的任何地方使用标记${X}来使用，
			     这里X是指属性的名称。属性有五种不同的形式，并且都能在settings.xml文件中访问。
   					1. env.X: 表示系统环境变量。例如,"env.PATH" 等同于 $path环境变量（在Windows上是%PATH%）。
					  2. project.x：表示 POM中对应的属性值。
					  3. settings.x: 表示 settings.xml中对应属性值。
					  4. Java系统属性: 所有可通过java.lang.System.getProperties()访问的属性都能在POM中使用该形式访问。
					  5. x: 在<properties/>元素中，或者外部文件中设置，以${someVar}的形式使用。 -->
		 	<properties>
		  	<user.blog>www.micmiu.com</user.blog>
		  </properties>

    </profile>

    <!-- 这是另一个配置文件，根据系统属性来激活 -->
    <profile>
    	<!--该配置的唯一标识符。 -->
      <id>env-dev</id>
      <activation>
        <property>
        	<!-- 被用来激活配置文件的属性的名称 -->
          <name>target-env</name>
          <!-- 被用来激活配置文件的属性的值 -->
          <value>dev</value>
        </property>
      </activation>
      <!-- 指定配置文件的扩展配置 内容采取property.value的形式 -->
      <properties>
        <tomcatPath>/path/to/tomcat/instance</tomcatPath>
      </properties>
    </profile>

    <profile>
			<id>repo-dev</id>
			<!-- 配置远程仓库列表 -->
			<repositories>
				<!-- 远程仓库的配置信息 -->
				<repository>
					<!-- 远程仓库唯一标识-->
					<id>nexus</id>
					<!-- 远程仓库名称 -->
					<name>nexus for develop</name>
					<!-- 远程仓库URL -->
					<url>http://192.168.1.8:8080/nexus/content/groups/public/</url>
					<layout>default</layout>
					<releases>
						<!--是否使用这个资源库下载这种类型的构件 默认值：true-->
						<enabled>true</enabled>
						<!--指定下载更新的频率。这里的选项是：always（一直），daily（每日，默认值），interval：X（这里X是指分钟），或者never（从不）。 -->
			      <updatePolicy>daily</updatePolicy>
			      <!--当Maven验证构件校验文件失败时该怎么做fail（失败）或者warn（告警）。-->
			      <checksumPolicy>warn</checksumPolicy>
					</releases>
					<snapshots>
						<!--是否使用这个资源库下载这种类型的构件 默认值：true-->
						<enabled>true</enabled>
						<!--指定下载更新的频率。这里的选项是：always（一直），daily（每日，默认值），interval：X（这里X是指分钟），或者never（从不）。 -->
			      <updatePolicy>daily</updatePolicy>
			      <!--当Maven验证构件校验文件失败时该怎么做fail（失败）或者warn（告警）。-->
			      <checksumPolicy>warn</checksumPolicy>
					</snapshots>
				</repository>
			</repositories>
			<pluginRepositories>
				<pluginRepository>
					<id>nexus</id>
					<name>local nexus</name>
					<url>http://192.168.1.8:8080/nexus/content/groups/public</url>
					<layout>default</layout>
					<releases>
						<enabled>true</enabled>
					</releases>
					<snapshots>
						<enabled>true</enabled>
					</snapshots>
				</pluginRepository>
			</pluginRepositories>
		</profile>

  </profiles>

  <!-- 指定被激活的配置文件-->
  <activeProfiles>
    <activeProfile>repo-dev</activeProfile>
  </activeProfiles>
</settings>
```
下面是我个人使用的Setting.xml文件
```
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" 
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <pluginGroups>
    <!-- pluginGroup
     | Specifies a further group identifier to use for plugin lookup.
    <pluginGroup>com.your.plugins</pluginGroup>
    -->
  </pluginGroups>

  <proxies>
    <!-- proxy
     | Specification for one proxy, to be used in connecting to the network.
     |
    <proxy>
      <id>optional</id>
      <active>true</active>
      <protocol>http</protocol>
      <username>proxyuser</username>
      <password>proxypass</password>
      <host>proxy.host.net</host>
      <port>80</port>
      <nonProxyHosts>local.net|some.host.com</nonProxyHosts>
    </proxy>
    -->
  </proxies>

  <servers>
    <!-- server
     | Specifies the authentication information to use when connecting to a particular server, identified by
     | a unique name within the system (referred to by the 'id' attribute below).
     | 
     | NOTE: You should either specify username/password OR privateKey/passphrase, since these pairings are 
     |       used together.
     |
    <server>
      <id>deploymentRepo</id>
      <username>repouser</username>
      <password>repopwd</password>
    </server>
    -->
    
    <!-- Another sample, using keys to authenticate.
    <server>
      <id>siteServer</id>
      <privateKey>/path/to/private/key</privateKey>
      <passphrase>optional; leave empty if not used.</passphrase>
    </server>
    -->
	<server>
		  <id>releases</id>
		  <username>admin</username>
		  <password>admin123</password>
    </server>
	<server>
		  <id>snapshots</id>
		  <username>admin</username>
		  <password>admin123</password>
    </server>
  </servers>

  <mirrors>
    <!-- mirror
     | Specifies a repository mirror site to use instead of a given repository. The repository that
     | this mirror serves has an ID that matches the mirrorOf element of this mirror. IDs are used
     | for inheritance and direct lookup purposes, and must be unique across the set of mirrors.
     |
    <mirror>
      <id>mirrorId</id>
      <mirrorOf>repositoryId</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://my.repository.com/repo/path</url>
    </mirror>
     -->
	 <mirror>
      <!--This sends everything else to /public -->
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
      <url>http://192.168.24.15:8081/nexus/content/groups/public/</url>
    </mirror>

	<mirror>
      <id>internet</id>
      <mirrorOf>*</mirrorOf>
      <url>http://repo1.maven.org/maven2/</url>
    </mirror>
  </mirrors>
  
  <profiles>

	<profile>
      <id>nexus</id>
      <!--Enable snapshots for the built in central repo to direct -->
      <!--all requests to nexus via the mirror -->
      <repositories>
        <repository>
            <id>nexus</id>
            <name>local private nexus</name>
            <url>http://192.168.24.15:8081/nexus/content/groups/public/</url>
            <releases><enabled>true</enabled></releases>
            <snapshots><enabled>false</enabled></snapshots>
        </repository>
        
      </repositories>
      <pluginRepositories>
        <pluginRepository>
            <id>nexus</id>
            <name>local private nexus</name>
            <url>http://192.168.24.15:8081/nexus/content/groups/public/</url>
            <releases><enabled>true</enabled></releases>
            <snapshots><enabled>false</enabled></snapshots>
        </pluginRepository>
        
       </pluginRepositories>
    </profile>
  </profiles>

  <activeProfiles>
    <!--make the profile active all the time -->
    <activeProfile>nexus</activeProfile>
  </activeProfiles>
</settings>
```
7. 导出所有依赖包
在开发中有这样一个需求，需要在非Maven环境下进行开发。这个时候需要将依赖的jar包导出。可以使用如下的命令完成：
```
mvn dependency:copy-dependencies -DoutputDirectory=lib -DincludeScope=compile -sD:\eclipseallinone\maven\settings.xml
```
这个命令指定了导出Jar包到那个目录，包括那个Scope以及指定的配置文件位置。

8. 依赖本地Jar
在开发过程中，有些包可能在服务器中找不到，需要对私服进行管理。有些只需要在本地加载即可
```
<dependency>
<groupId>javax.servlet</groupId>
<artifactId>servlet-api</artifactId>
        <version>1.1.1< ersion>
        <scope>system</scope>
        <!--本地jar的路径,相对或者绝对都可以-->
        <systemPath>path/to/yourLocalJar.jar</systemPath>
</dependency>
```