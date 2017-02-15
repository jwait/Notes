# Maven学习笔记

参考资料：《Maven 实战》



## POM

### POM介绍

Maven项目的核心是 pom.xml。POM（Project Object Model，项目对象模型），定义了项目的基本信息，用于描述项目如何构建，声明项目依赖等等。

一个简单的pom.xml文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" 		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

  <modelVersion>4.0.0</modelVersion>
  <packaging>war</packaging>

  <name>WMS</name>
  <groupId>com.ken</groupId>
  <artifactId>WMS</artifactId>
  <version>1.0-SNAPSHOT</version>
  
  <dependencies>
    <!-- https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
        <scope>provided</scope>
    </dependency>
  <dependencies/>
<project/>
  
```

pom.xml 文件的第一行是 XML 头，指定了 xml 文档的版本和编码方式。紧接着是 project 元素，project 元素是所有 pom.xml 的根元素，它声明了一些 POM 相关的命名空间以及xsd元素。

### 定义当前项目信息

根元素下的的前几个子元素：modelVersion、artifactId、groupId、version、name对当前项目的基本信息进行描述，其中包括了项目的坐标。下面对其中涉及的一些元素作简单的解释

* modelVersion：指定当前POM模型的版本，对于Maven2 及 Maven3 来说，它只能是4.0.0
* groupId：定义项目属于哪一个组，这个组往往和项目所在的组织或公司存在关联
* artifactId：定义了当前 Maven 项目在组中的唯一ID
* version：version指定项目当前的版本
* name：声明一个对于用户友好的项目名称




### 声明项目依赖信息

在上面的简单示例中，pom.xml 文件使用 dependencies 子元素来对项目的依赖进行声明，其中包含了多个 dependency 子元素。dependency子元素描述了项目依赖模块的信息，以及依赖的坐标。任何一个Maven 项目都需要定义自己的的坐标，当这一个坐标成为其他 Maven 项目的依赖的时候，这组坐标就体现了其价值




## Maven坐标

Maven 坐标为各种构件引入了秩序，任何一个构建都必须明确定义自己的坐标。那么其他项目就可以通过这个坐标唯一确定构建并引入。

```xml
<dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>4.3.0.RELEASE</version>
</dependency>
```

上面为spring-core的坐标定义。可见，一组Maven坐标是通过一些元素来定义的，它们是groupId、artifactId、version、classifer。

### 坐标元素

下面详细解释以下各个坐标元素：

* **groupId**：当前定义Maven项目隶属的实际项目。需要注意的是：首先，Maven项目和实际项目不一定是一对一关系，例如上面的Springframewo这一实际项目，其对应的Maven项目会有很多，如Spring-core，Spring-context等。其次，groupId不应该对应项目隶属的组织和公司，原因是一个组织或公司可能会有很多实际项目；最后，groupId的表示方式与Java包名表示方式类似，采取域名反向
* **artifactId**：该元素定义实际项目中的一个Maven项目（模块），推荐的做法是使用实际项目名称作为artifactId的前缀
* **version**：该元素定义 Maven 项目的当前所处的版本
* **packaging**：该元素定义 Maven 项目的打包方式。打包的方式与生成的构建文件扩展名对应，并且打包方式会影响构建的生命周期。当不定义是，Maven 使用默认值 jar
* **classifier**：该元素用来帮助定义构建输出的一些附属构建。附属构建与主构建对应，如文档和源代码等。需要注意的是：不能直接定义项目的classifier，因为附属构建不是项目直接默认生成的，而是由附加的插件帮助生成




## 依赖管理

### 依赖的配置

根元素 project 下的dependencies 可以包含一个或多个 dependency 元素，以声明一个或多个项目依赖，可以看到依赖会由基本的 groupId、artifactId 和 version 等元素组成。其实一个依赖声明可以包含如下的一些元素：

#### 元素介绍：

* **groupId**、**artifacId** 和 **version：**依赖的基本坐标，对于任何一个依赖来说，基本坐标是最重要的，Maven 根据坐标才能找到需要的依赖
* **type：**依赖的类型，对应于项目坐标定义的 packaging。大部分情况下，该元素不必声明，其默认值为jar
* **scope：**依赖的范围
* **optional：**标记依赖是否可选
* **exclusions：**用来排除传递性依赖



#### scope 依赖范围

Maven 有 3 种 classpath，分别是：编译classpath、测试classpath、运行classpath。Maven 在编译项目组代码的时候需要使用一套 classpath；在编译和执行测试的时候会使用另一套 classpath；最后在实际运行 Maven 项目的时候，又会使用另一套 classpath

依赖范围scope 就是用来控制依赖与这三种 classpath的关系，Maven 有以下几种依赖范围：

* **compile**：编译依赖范围。如果没有指定，就会默认使用该范围依赖，使用此依赖范围的依赖，对于编译，测试、运行三种 classpath 都有效
* **test**：测试依赖范围。使用此依赖范围的Maven 依赖，只对于测试 classpath 有效，在编译主代码或者运行项目的时候将无法使用该依赖
* **provided**：已提供依赖范围。使用此依赖范围的 Maven 依赖，对于编译和测试 classparh 有效，但在运行时无效
* **runtime**：运行时依赖范围。使用此依赖范围的 Maven 依赖，对于测试和运行 classpath 有效，但在编译主代码时无效
* **system**：系统依赖范围。该依赖与三种 classpath 的关系，和provided依赖范围完全一致。但是使用 system 依赖时必须 通过 systemPath 元素显式的指定依赖文件的路径。由于此类依赖不是从Maven 仓库解析的，往往与本机绑定，可能造成构建的不可移植性
* **import**：导入依赖范围。该依赖范围不会对三种 class path 造成影响



### 传递性依赖机制

#### 什么是传递性依赖

在项目中，有可能存在当引入一个组件依赖的时候，这个组件又依赖依赖其他的一些组件的情况。如果不使用 Maven 进行依赖管理，而是使用手工处理，那么我们需要把这一个组件依赖的其他组件添加进来，这个过程非常的麻烦，有可能出现依赖组件缺少报错有或者是引入了许多不必要的依赖。

Maven 的传递依赖机制就可以很好的解决这一个问题，Maven 会解析各个直接的 POM，将哪些必要的间接依赖，以传递依赖的形式引入到当前的项目中



#### 传递性依赖和依赖范围

依赖范围不仅可以控制依赖与三种 classpath 的关系，还对传递性依赖产生影响。假设 A 依赖于 B，B 依赖于 C，我们说 A 对于 B 是第一直接依赖， B 对于 C 是第二直接依赖，A 对于C 是传递性依赖。第一直接依赖范围和第二直接依赖范围决定了传递性依赖的范围。其影响如下表所示，最左边一列表示第一直接依赖范围，最上面一行表示第二直接依赖范围，中间的交叉单元表示传递依赖的范围

|          | compile  | test | provided | runtime  |
| :------: | :------: | :--: | :------: | :------: |
| compile  | compile  |  -   |    -     | runtime  |
|   test   |   test   |  -   |    -     |   test   |
| provided | provided |  -   | provided | provided |
| runtime  | runtime  |  -   |    -     | runtime  |



#### 传递性依赖的一些问题

Maven 引入的传递性依赖机制，一方面大大简化和方便了依赖声明，另一方面，大部分情况下我们只需要关注项目的直接依赖是什么，而不用考虑这些直接依赖会引入什么传递性依赖。但是有时传递性依赖会引发一些问题，于是就有了下面的**依赖调解**和**可选依赖**。

##### 依赖调解

假如：项目A有这样的一条依赖关系：A->B->C->X(1.0) 、A->D->X(2.0)，X 是 A 的传递性依赖，但是两条依赖路径上有两个版本的X，两个版本都被解析显然是不对的，因为会造成依赖重复，那么哪一个X会被Maven 解析使用？Maven 的**依赖调解（Dependency Mediation）**就是解决这一个问题。

* 依赖调解**第一原则**是：路径最近者优先
* 依赖调解**第二原则**是：第一声明者优先

##### 可选依赖

假如有这样一个依赖关系，项目A依赖于项目B，项目B依赖与项目X和Y，B 对于 X 和 Y 的依赖都是可选依赖（即使用optional标签标记）：A->B、B->X、B->Y，根据传递性依赖的定义，如果所有这三个依赖的范围都是 compile，那么 X、Y 都是 A 的compile 传递性依赖。然而，由于这里 X、Y 是可选依赖，依赖将不会传递，换句话说 X、Y 将不会对 A 有任何影响



## Maven 仓库

### 什么是 Maven 仓库

得益于坐标机制，任何 Maven 项目使用任何一个构件的方式都是相同的，都是通过提供构件的坐标使用，换句话说，相同的依赖坐标指向的是同一个依赖文件。因此，Maven 可以在某个位置同一存储所有 Maven 项目共享的构件，这一个统一的位置就是仓库。实际的 Maven 项目将不再各自存放其依赖文件，它们只需要声明这些依赖的坐标，在需要的时候，Maven 会自动根据坐标找到仓库中的构件并使用它们。

使用 Maven 仓库可以避免 各个 Maven 项目各自存放管理其依赖文件导致的依赖文件大量重复问题



### Maven 仓库分类

对于 Maven 来说，根据位置仓库可以分为两大类：**本地仓库** 和**远程仓库**。而远程仓库又可以分类为：**中央仓库**、**私服**和**其他公共库**

* **本地仓库**：在本机上存放依赖文件的位置，默认情况下，每一个用户在自己用户目录下都会有一个路径名为：`.m2/reposiotry/`的仓库目录。当 Maven 根据坐标寻找构件的时候，它首先会查看本地仓库，如果本地仓库存在此构件，则直接使用。如果本地仓库不存在此构件，或者是需要查看是否有更新的构件版本，Maven 才会去远程仓库查找
* **远程仓库**：其他用于存放依赖构件的服务器。只有当 Maven 查找构件时发现本地没有需要的构件才从远程仓库中获取
* **中央仓库**：特殊的远程仓库。中央仓库时 Maven 核心自带的远程仓库，它包含了绝大部分开源的构件。在默认设置下，当本地仓库没有 Maven 需要的构件的时候，它才会尝试从中央仓库下载
* **私服**：另一种特殊的仓库，私服是在局域网中架设的，供局域网内 Maven 用户使用的，一个私有的仓库服务器，用其代理所有外部的远程仓库，可以节省带宽和时间，加速 Maven 构建



### 配置远程仓库

在很多情况下，默认的中央仓库无法满足项目的需求，可能是因为需要的构件存在于另一个远程仓库中。那么这时候。我们希望 Maven能够从指定的远程仓库中获取构件，这里可以通过配置远程仓库来实现。

例如，若我们需要配置JBoss Maven 远程仓库的时候，可以如下：

```xml
<project>
  ...
	<repositories>
		<repository>
			<id>jboss</id>
			<name>JBoss Repository</name>
			<url>http://repository.jboss.com/maven2/</url>
			<releases>
				<enable>true</enable>
			</releases>
			<snapshots>
				<enable>false</enable>
			</snapshots>
			<layout>default</layout>
		</repository>
	</repositories>
  ...
</project>
```

我们可以在pom.xml文件中添加一个 repositories 元素，在 repositories 元素下可以使用 repository子元素声明一个或多个远程仓库。在 repository 元素中需要使用 id 、name、url 标签对远程仓库的id、名称以及地址进行描述。

另外，配置中的 release 和 snapshots 元素比较重要，它们用来控制 Maven 对于发布版侯构件和快照版构件的下载。对于 release 和 snapshots 元素 除了 enable 元素，它还包含两个子元素：updatePolicy 和 checksumPolicy 分别配置 Maven 从远程仓库检查更新的频率和配置 Maven 检查校验和文件的策略



### 快照版本机制

Maven 中的快照版本机制是在尚不稳定的项目或构件的版本中加上`-SNAPSHOT`，用于解决非稳定版本的项目或构件带来的版本控制问题。

例如：有模块 A 与模块 B 分别由两人同时开发，模块 A 尚未正式发布，而模块 B 的功能依赖于 A，那么在开发过程中，需要经常将模块A构件输出供开发模块 B 的人员使用，问题是，这个工作如何进行？对于这个问题可以由以下方案：

1. 让开发模块 B 的人员自己签出模块A的源码进行构建，这样可以保证开发模块B的人员可以的模块A的最新构件，但是多了很多版本控制和构建的操作，还有可能构建失败，这种方式是低效的
2. 重复部署模块A供开发模块B的人员下载。虽然这样可以保证仓库中的模块A构建是最新的。但是对于Maven 来说，同样的版本和同样的坐标就意味着同样的构建，模块A尚未由稳定的发布版本，不会有太多的版本变化，那么当开发模块B的人员的本地仓库已经有该模块构建，将不会从远程仓库中更新，除非手工清除本地仓库
3. 模块A在开发过程中频繁的更新版本，可以避免方案二中的问题，但是使用这种方式需要模块A和模块B的开发人员都频繁的修改POM以保持模块A最新，并且大量的版本仅仅包含了微小的差异，可能导致版本号的滥用



Maven 的快照版本机制就是为了解决以上的问题。如果使用了 Maven 的快照版本机制，那么开发模块A的开发人员只需要在计划的版本号后加入`-SNAPSHOT`，如：`2.1-SNAPSHOT`，然后发布到远程仓库中。在发布过程中，Maven 会自动为该构建打上时间戳，如：`2.1-20161229221415-3`，有了时间戳，Maven 就可以找到仓库中该构件的最新版本，因此当构建模块B的时候，Maven 会自动从仓库中检查依赖的模块A的`2.1-SNAPSHOT`版本有没有最新构件，根据时间戳发现有更新时便进行下载



需要注意的是：当项目进过完善测试后需要发布的时候，就应该将快照版本更改为发布版本，表示该构件已经稳定，且只对应唯一的构件；另外，快照版本只应该在组织内部或模块间依赖使用，因为快照版本不稳定，对其的依赖可能造成潜在的危险



## 生命周期与插件

在有关 Maven 的日常使用中，命令行的输入往往就对应了生命周期，如 mvn package 就表示执行默认生命周期阶段 package。Maven 的生命周期时抽象的，其实际行为都有插件来完成，如 package 姐u但的任务就可能由 maven-jar-plugin 完成。生命周期和插件协同工作，密不可分

### 什么是生命周期

项目的构建存在生命周期，软件开发人员每天都对项目进行清理、编译、测试和部署。

**Maven 的生命周期**就是对这个过程所有的构建任务进行抽象和统一。Maven 从大量的项目和构建工具中学习和反思，然后总结出一套高度完善的、易扩展的生命周期。这个生命周期包含了项目的清理、初始化、编译、测试、打包、集成测试、验证、部署和站点生成等几乎所有的构建步骤。



### 什么是插件机制

Maven 的生命周期是抽象的，生命周期本身不做任何实际的工作，在 Maven 的设计中，实际的任务都是交由插件来完成，这意味者每个构建步骤都可以绑定一个或多个插件行为。Maven 的插件机制就是为大部分的构建步骤设置默认插件并且允许为 Maven 的构建步骤编写并绑定插件。

这样一方面保证了所有 Maven 项目由一致的构建标准，另一方面由通过默认插件简化和稳定了实际项目的构建。此外插件机制还提供了足够的扩展空间，用户可以通过配置现有的插件或自行编写插件来自定义构建行为



### 生命周期详解

* Maven 拥有三套项目独立的生命周期，它们分别是clean、default和site
* 每个生命周期包含一些阶段（phase），这些阶段是有顺序的，并且后面阶段的依赖于前面的阶段。用户可以直接调用这些阶段，当某个阶段被调用时，其前面的阶段都会被执行，但是后面的阶段则不会被执行
* 三套生命周期是相互独立的，当用户调用某一个生命周期的某一阶段时，其他生命周期产生任何影响



#### clean 生命周期

clean 生命周期的目的是清理项目，它包含三个阶段：

1. **pre-clean**： 执行一些清理前需要完成的工作
2. **clean**：清理上一次构建生成的文件
3. **post-clean**：执行一些清理后需要完成的工作



#### default 生命周期

default 生命周期定义了真正构建时需要执行的所有步骤，它是生命周期中最核心的部分，其包含的的阶段如下

* **validate**
* **initialize**
* **generate-source**
* **process-source**：处理项目主资源文件。一般来说，是对src/main/resources 目录的内容进行变量替换等工作后，复制到项目输出的主 classpath 目录中
* **generate-resources**
* **process-resources**
* **compile**：编译项目的主源码。一般来说，是编译 src/main/java 目录下的Java文件至项目的主 classpath 目录中
* **process-class**
* **generate-test-sources**
* **process-test-sources**：处理项目的测试资源文件。一般来说，是对 src/test/resources 目录的内容进行变量替换等工作后，复制到项目输出的测试 classpath 目录中
* **generate-test-resources**
* **process-test-resources**
* **test-compile**：编译项目的测试代码。一般来说，是编译 src/test/java 目录下的 Java 文件至项目输出的测试 classpath 目录中
* **process-test-classes**
* **test**：使用单元测试框架运行测试，测试代码不会被打包或部署
* **prepare-package**
* **package**：接收编译好的代码，打包成可发布的格式
* **pre-integration-test**
* **integration-test**
* **post-integration-test**
* **verify**
* **install**：将包安装到Maven 本地仓库，提供本地其他 Maven 项目使用
* **deploy**：将最终的包复制到远程仓库，供其他来发人员和Maven 项目使用



#### site 生命周期

site 生命周期的目的是建立和发布项目站点，Maven 能够基于 POM 所包含的信息，自动生成一个友好的站点，方便团队交流和发布项目信息，该生命周期包含如下阶段：

* **pre-site**：执行一些在生成项目站点之前需要完成的工作
* **site**：生成项目站点文档
* **post-site**：执行一些在生成项目站点之后需要完成的工作
* **site-deploy**：将生成的项目站点发布到服务器尚



### 使用命令行调用 Maven 生命周期

从命令行执行 Maven 任务的最主要方式就是调用 Maven 的生命周期。需要注意的是，各个生命周期是相互独立的，而ige生命周期的阶段是有前后依赖关系的，用户可以根据需要调用不同生命周期的不同阶段。

例如：

```
$mvn clean deploy site-deploy
```

该命令结合了三个生命周期，分别执行了clean阶段前、deploy阶段前、site-deploy阶段前的说有任务

由于 Maven 的主要生命周期阶段并不多，而常用的 Maven 命令实际都是基于这些姐u按简单组合而成的，因此只要对 Maven 生命周期有一个基本的理解就可以正确熟练地使用 Maven 命令



### 插件绑定

Maven 的生命周期与插件相互绑定，用以完成实际的构建任务。具体而言，是生命周期的阶段与插件的目标相互绑定

#### 内置绑定

为了能让用户几乎不用任何配置就能构建 Maven 项目，Maven 在核心一些主要的生命周期阶段绑定了很多插件的目标，当用户通过命令行调用生命周期阶段的时候，对应的插件目标就会执行相应的目标

#### 自定义绑定

自定义绑定方式允许用户自己选择某个插件目标绑定到生命周期的某个阶段上，这种插件绑定方式能让Maven 项目在构建过程中执行更富特色的任务

绑定的示例如下：

```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.apache.maven-plugins</groupId>
			<artifactId>maven-source-plugin</artifactId>
			<version>2.1.1</version>
			<execution>
				<id>attach-source</id>
				<phase>verify</phase>
				<globals>
					<global>jar-no-fork</global>
				</globals>
			</execution>
		</plugin>
	</plugins>
</build>
```

在 POM 的build 元素洗阿德 plugins 子元素中生命插件的使用，该例中用到的是 maven-source-plugin，其 groupId 为 org.apache.maven-plugins，artifactId 为 maven-source-plugin，version 为2.1.1.

在上述配置中，除了基本的插件坐标生命外，还有插件执行配置，execution 下的每个子元素用来配置一个任务。上例中配置了一个 id 为attach-source 的任务，通过 phase 配置，将其绑定到 verify 生命周期阶段上，再通过 global 配置指定要执行的插件目标





## 使用 Maven 构建 Web 应用

### Web 应用的目录结构

基于 Java 的 Web 应用，其标准的打包方式是 WAR。 WAR 与 JAR 类似，只不过它可以包含更多的内容，如 JSP 文件、Servlet，Java 类、Web.xml配置文件、依赖 JAR 包、静态 web 资源（如 HTML、CSS、Javascript文件）等。一个典型的 WAR 文件会有如下的目录结构：

```
- war/
 + META-INF/
 + WEB-INF/
 | + classes/
 | | + ServletA.class
 | | + config.properties
 | | + ...
 | |
 | + lib/
 | | + dom4j-1.4.1.jar
 | | + mail-1.2.1.jar
 | | + ...
 | |
 | + web.xml
 |
 + img/
 |
 + css/
 |
 + js/
 |
 + index.html
 + sample.jsp
```

一个 WAR 包下至少包含两个子目录：META-INF 和 WEB-INF。前者包含了一些打包元素据信息，我们一般不去关心；后者是 WAR 包的核心， WEB-INF 下必须包含一些个 Web 资源表述web见 web.xml，他的子目录 classes 包含所有该 Web 项目的类，而另外一个子目录 lib 则包含所有该 Web 项目的依赖 JAR 包，classes 和 lib 目录 都会再运行时被加入到 Classpath 中。除了 META-INF 和 WEB-INF 外，一般的 WAR 包 都会包含很多 Web 资源，如 html、jsp、css、js等等

同任何其他 Maven 项目一样， Maven 对 Web 下古墓的布局结构也有一个通用的约定。不过首先要注意的是，必须显式的指定打包方式为 war，如下：

```xml
<project>
	...
	<groupId>com.ken</group>
	<artifactId>WMS</artifact>
	<packaging>war<</packaging>
	<verion>1.0-SNAPSHOT</version>
	...
</project>
```

如果不显式地指出 packing，Maven 会使用默认的 jar 打包方式，从而导致 无法正确打包 Web 应用



Web 项目的类及资源文件同一般 JAR 项目一样，默认位置都是`src/main/java` 和 `src/main/resource` ，测试类及测试资源文件的默认位置是 `src/test/java` 和 `src/test/resources`.web 项目比较特殊的地方在于：它还有一个 Web 资源目录，其默认位置是 `src/main/webapp`。一个典型的 Web 项目的 Maven 目录如下：

```
+ project
  |
  + pom.xml
  |
  + src/
    + main/
    | + java/
    | | + ServletA.java
    | | + ...
    | |
    | + resources/
    | | + config.properties
    | | + ...
    | |
    | + webapp/
    |   + WEB-INF/
    |   | + web.xml
    |   |
    |   + img/
    |   |
    |   + css/
    |   |
    |   + js/
    |   |
    |   + index.html
    |   + sample.jsp
    + test/
      + java/
      + resources/
```

在 `src/main/webapp`目录下，必须包含一个子目录 WEB-INF ，该子目录海必须好汉 web.xml 文件。`src/main/webapp` 目录下的其他文件和目录包括 html 、 jsp、css、JavaScript等，它们与 WAR 包中的Web 资源完全一致

在使用 Maven 创建 Web 项目之前，必须首先理解这种 Maven 项目结构 和 WAR包结构的对应关系。有一点需要注意的是，WAR 包中有一个 lib 目录包含所有依赖的 jar 包，但 Maven 项目结构中没有这样的一个目录，这是因为依赖都配置在 POM 中， Maven 在用 WAR 方式打包的时候会根据 POM 的配置从本地仓库复制相应的 JAR 文件



### 修改默认目录结构

Maven 为我们提供了一致的项目目录配置（源文件、资源文件等），在自动构建项目的时候，Maven 会按照默认这个配置来执行操作。但是 Maven 规约的默认配置并不能够满足我们的需求，我们就需要手动修改项目的 Maven 配置，让项目能够很好的与 Maven 协同工作。

例如会有以下的需求：

* 配置多个源文件夹来管理我们项目的模块
* 配置多个资源文件夹管理我们项目模块的资源文件
* 有些遗留代码的一些资源文件放在源文件夹下，为了不改变原项目的结构，我们希望 Maven 能够从源文件中读取资源文件
* 希望将模块的源文件和资源文件放在同一个文件夹下，以方便开发和维护



要解决以上的许需求，可能会用到两个元素标签 `sourceDirdctory` 和 `resources`。

#### 配置源文件夹

当我们需要修改源文件夹的位置，或者是需要配置多个源文件夹的时候，可以在在 pom.xml 文件的 project/build 标签下添加 `sourceDirectory` 子标签，里面填写源文件的路径。

例如将源文件夹的路径修改为`src/main`

```xml
<project>
	...
	<build>
		...
		<sourceDirectory>src/main</sourceDirectory>
		...
	</build>
	...
</project>

```



#### 配置资源文件夹

当我们需要为项目配置多个资源文件夹，或者是修改资源文件夹位置的时候，可以在 pom.xml 文件的 project/build 元素下添加 `resources` 子元素，并且`resources` 元素下的每一个配置的 `resource` 代表一个资源文件夹路径。

例如：

```xml
<resources>  
	<resource>  
		<directory>src/resources</directory>  
	</resource>  
	<resource>  
		<directory>test</directory>  
	</resource>  
</resources>  
```



另外，Maven 默认情况下只会打包源文件夹下的所有的源文件，以及资源文件夹下的资源文件，开发过程中也会按照这样的规约。但是有些情况下，资源文件需要和 Java 文件放在同一个文件夹下，比如 Hibernate 的映射文件 `.hbm.xml`，还有 MyBatis 的映射文件 `*mapper.xml`。因此我们就需要对 Maven 的配置进行修改。

例如：

```xml
<build>
    <resources>
      <resource>
        <directory>src/main/resources</directory>
      </resource>
      <resource>
        <directory>src/main/java</directory>
        <includes>
            <include>**/*.xml</include>
        </includes>
      </resource>
    </resources>
</build>
```

同样的对于资源文件。我们需要在 `resources` 子元素中对其进行配置。上例中通过 include 标签，实现了将 `src/main/java` 路径下的所有 `.xml`文件加入到资源文件