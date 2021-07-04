# BRMS



目前主流的：

Drools、easyRule、QiExpress、Aviator



使用目的：

复杂项目开发以及其中随==外部条件不断变化的业务规则==，迫切需要分离商业决策者的商业决策逻辑和应用开发者的技术决策，并把这些商业决策放在中心数据库或其他统一的地方，让它们能在运行时（即商务时间）可以==**动态地管理和修改**==从而提供软件系统的柔性和适应性。规则正是应用于上述动态环境中的一种解决方法。



<img src="/Users/didi/Library/Application Support/typora-user-images/image-20201118154245758.png" alt="image-20201118154245758" style="zoom: 33%;" />





规则引擎由三部分组成：

- 事实（Fact）：就是用户输入的已经事实，可以理解为推理前的已知对象。
- LHS(Left Hand Side)：可以理解为规则执行需要满足的条件。
- RHS(Right Hand Sike)：可以理解为规则执行后的返回对象。



## Drools：

### 优点

- 基于Java的规则引擎，免费开源，以规则脚本的形式存放在文件中，使得规则的变更不需要修改代码重启机器就可以立即在线上环境生效。

- 可以通过使用springboot整合

- 社区活跃，最近一次更新在几周前。

- 规则的变更可以脱离开发人员。

- 可以快速做出响应。

- 支持动态加载（通过从数据库中读取数据来拼接字符串来生成drl）

  

### 缺点

- 学习成本较高，需要学习使用特定的语法。
- 引擎的设计和实现较为复杂。



为什么说可以脱离开发人员和快速响应？

运维同学或其他管理员需要对规则进行变更时，通过相应的前端页面对相应规则提交更改，被更改的数据会更新到数据库中，使得新规则可以快速生效。

除了drl文件以外，还可以使用xml文件和excel文件（决策表）来作为规则文件。



使用drl文件作为规则文件时，在调用的时候可以指明被使用的规则；



学习成本？

规则的编写主要是通过使用drl文件来编写。

需要使用java语法和部分的语法。



no-loop 防止循环

Lock-on-active  加锁

timer 定时器



1. 将规则写到drl中进行保存，程序运行过程中自行判断需要执行到哪一条。（简单，但是是否会存在不方便修改的问题？运维同学或管理员要如何对规则中的数值进行修改？）
2. 将规则模版写到drl中，将规则的具体数值和条件存入到数据库中，程序运行中去取对应的规则。（这样方便对规则进行修改，但是规则执行的过程是否会过于复杂？） 不会
3. 使用决策表来产出规则，可以通过使用excel、csv来生成，也可以使用字符串来动态生成。可以将文件放置在本地或者将其放在文件服务器，通过更新决策表后使用热部署或者使用redis间隔时间去刷新缓存来更新。

https://github.com/hongwen1993/fast-drools-spring-boot-starter/blob/master/README_CN.md

文档：https://drools.org/learn/documentation.html

总结用法：

- 制定规则文件可以（1）通过使用相应的语法编写drl，（2）使用决策表（xls，csv）来动态构建drl文件，（3）使用字符串动态的拼接编写drl；
- 调用方法，（1）可以将其放在程序运行的文件下（如Resource下）来根据classpath路径来寻找，（2）将其放在服务器中，通过使用绝对地址来寻址调用，（3）将规则文件放入数据库中，通过数据库来插取；
- 更新方法，（1）可以在java项目下来更新，但是这样需要停止项目，（2）对其文件进行更行，通过程序来定时读取，（3）对文件进行更新，使用redis进行定时扫描更新；





### 使用

文档：

https://docs.jboss.org/drools/release/7.46.0.Final/drools-docs/html_single/index.html#_droolslanguagereferencechapter

在创建好项目后，导入drools相关的依赖

```xml

<!--drools规则引擎-->
        <dependency>
            <groupId>org.drools</groupId>
            <artifactId>drools-core</artifactId>
            <version>7.6.0.Final</version>
        </dependency>
        <dependency>
            <groupId>org.drools</groupId>
            <artifactId>drools-compiler</artifactId>
            <version>7.6.0.Final</version>
        </dependency>
        <dependency>
            <groupId>org.drools</groupId>
            <artifactId>drools-templates</artifactId>
            <version>7.6.0.Final</version>
        </dependency>
        <dependency>
            <groupId>org.kie</groupId>
            <artifactId>kie-api</artifactId>
            <version>7.6.0.Final</version>
        </dependency>
        <dependency>
            <groupId>org.kie</groupId>
            <artifactId>kie-spring</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework</groupId>
                    <artifactId>spring-tx</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.springframework</groupId>
                    <artifactId>spring-beans</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.springframework</groupId>
                    <artifactId>spring-core</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.springframework</groupId>
                    <artifactId>spring-context</artifactId>
                </exclusion>
            </exclusions>
            <version>7.6.0.Final</version>
        </dependency>

```



创建一个Utils包用于存放与drools相关的Api

该类中主要是有关文件路径的获取，以及简化配置，不需要再配置xml文件

```java

import org.drools.decisiontable.InputType;
import org.drools.decisiontable.SpreadsheetCompiler;
import org.kie.api.KieBase;
import org.kie.api.KieServices;
import org.kie.api.builder.*;
import org.kie.api.event.rule.DebugRuleRuntimeEventListener;
import org.kie.api.io.Resource;
import org.kie.api.io.ResourceType;
import org.kie.api.runtime.KieContainer;
import org.kie.api.runtime.KieSession;
import org.kie.api.runtime.StatelessKieSession;
import org.kie.internal.io.ResourceFactory;
import org.kie.internal.utils.KieHelper;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.InputStream;
import java.util.List;

/**
 * @author xzy
 * @date 2020/11/24 3:43 下午
 * @describe
 * 封装有关drool的相关配置
 * 不需要再写kmodule.xml文件
 */

public class KieSessionUtils {
    private static KieSession kieSession;
    private static final String RULES_PATH = "rules/";

    private KieSessionUtils() {

    }
    /**
     * @description TODO(创建包含所有规则的对象)
     * @throws Exception
     * @return KieSession
     */
    public static KieSession getAllRules() throws Exception {
        try {
            disposeKieSession();
            KieServices kieServices = KieServices.Factory.get();
            KieFileSystem kieFileSystem = kieServices.newKieFileSystem();
            for (org.springframework.core.io.Resource file : new PathMatchingResourcePatternResolver().getResources("classpath*:" + RULES_PATH + "**/*.*")) {
                kieFileSystem.write(org.kie.internal.io.ResourceFactory.newClassPathResource(RULES_PATH + file.getFilename(), "UTF-8"));
            }
            final KieRepository kieRepository = KieServices.Factory.get().getRepository();
            kieRepository.addKieModule(new KieModule() {
                @Override
                public ReleaseId getReleaseId() {
                    return kieRepository.getDefaultReleaseId();
                }
            });
            KieBuilder kieBuilder = KieServices.Factory.get().newKieBuilder(kieFileSystem);
            kieBuilder.buildAll();
            kieSession = KieServices.Factory.get().newKieContainer(kieRepository.getDefaultReleaseId()).newKieSession().getKieBase().newKieSession();
            return kieSession;
        } catch (Exception ex) {
            throw ex;
        }
    }

    /**
     * @description TODO (快速新建KieSession)
     * @param classPath 绝对路径
     * @return KieSession 有状态
     */
    public static KieSession newKieSession(String classPath) throws Exception {
        KieSession kieSession = getKieBase(classPath).newKieSession();
        kieSession.addEventListener(new DebugRuleRuntimeEventListener());
        return kieSession;

    }

    /**
     * @description TODO (快速新建StatelessKieSession)
     * @param classPath 绝对路径
     * @return StatelessKieSession 无状态
     */
    public static StatelessKieSession newStatelessKieSession(String classPath) throws Exception {
        StatelessKieSession kiesession = getKieBase(classPath).newStatelessKieSession();
        return kiesession;

    }

    /**
     * @description TODO (清空对象)
     * @title disposeKieSession 重置KieSession
     * @return void
     */
    public static void disposeKieSession() {
        if (kieSession != null) {
            kieSession.dispose();
            kieSession = null;
        }
    }

    protected static KieBase getKieBase(String classPath) throws Exception {
        KieServices kieServices = KieServices.Factory.get();
        KieFileSystem kfs = kieServices.newKieFileSystem();
        //Resource resource = kieServices.getResources().newClassPathResource(classPath);
        Resource resource = ResourceFactory.newFileResource(classPath);
        resource.setResourceType(ResourceType.DRL);
        kfs.write(resource);
        KieBuilder kieBuilder = kieServices.newKieBuilder(kfs).buildAll();
        if (kieBuilder.getResults().getMessages(Message.Level.ERROR).size() > 0) {
            throw new Exception();
        }
        KieContainer kieContainer = kieServices.newKieContainer(kieServices.getRepository().getDefaultReleaseId());
        KieBase kBase = kieContainer.getKieBase();
        return kBase;
    }

    /**
     * 根据服务器真实路径下的xls文件生成drl文件内容
     */
    public static KieSession getKieSessionFromXLS(String realPath) throws FileNotFoundException {
        return createKieSessionFromDRL(getDRL(realPath));
    }

    // 把xls文件解析为String
    public static String getDRL (String realPath) throws FileNotFoundException {
        File file = new File(realPath); // 例如：C:\\abc.xls
        InputStream is = new FileInputStream(file);
        SpreadsheetCompiler compiler = new SpreadsheetCompiler();
        return compiler.compile(is, InputType.XLS);
    }

    // drl为含有内容的字符串
    public static KieSession createKieSessionFromDRL(String drl) {
        KieHelper kieHelper = new KieHelper();
        kieHelper.addContent(drl, ResourceType.DRL);
        Results results = kieHelper.verify();
        if (results.hasMessages(Message.Level.WARNING, Message.Level.ERROR)) {
            List<Message> messages = results.getMessages(Message.Level.WARNING, Message.Level.ERROR);
            for (Message message : messages) {
                System.out.println("Error: "+message.getText());
            }
            throw new IllegalStateException("Compilation errors were found. Check the logs.");
        }
        return kieHelper.build().newKieSession();
    }
}
```



之后可以通过该工具类直接获取需要使用的drl文件的路径；



在resources目录下创建一个rule包，用于存放drl文件

<img src="/Users/didi/Documents/BRMS.assets/image-20201124150258342.png" alt="image-20201124150258342" style="zoom:50%;" />



```java
package rule01;
//generated from Decision Table
// rule values at B9, header at B4
//一个简单的使用例子

rule "rule_01"
	no-loop true
	when
		$d:Double($d==0);
	then
		System.out.println("这是rule111111111111");
end

// rule values at B10, header at B4
rule "rule_02"
	no-loop true
	when
		$d:Double($d==999);
	then
		System.out.println("这是rule222222222222");
end
  
  
    @Test
    public void test01() throws Exception {
        KieSession kieSession = KieSessionUtils.newKieSession("/Users/didi/IdeaProjects/spring-study/drools-03/src/main/resources/rules/rule01.drl");
        kieSession.insert(999d);
        kieSession.fireAllRules();
    }
}
```



<img src="/Users/didi/Documents/BRMS.assets/image-20201124155922690.png" alt="image-20201124155922690" style="zoom:50%;" />



|    关键字名称    |                             作用                             |
| :--------------: | :----------------------------------------------------------: |
|     package      |                     当前规则文件的标示ID                     |
|      import      |            所引用的包名，可引用多个和java语法相同            |
|  global（可选）  |                         声明全局变量                         |
|  query（可选）   | 查询，再DRL中添加查询定义，然后在java代码中获得匹配的结果<img src="/Users/didi/Documents/BRMS.assets/image-20201124164619330.png" alt="image-20201124164619330" style="zoom:25%;" /> |
| declear（可选）  | 在进入LHS前，提前声明所需对象和变量。<img src="/Users/didi/Documents/BRMS.assets/image-20201124163625430.png" alt="image-20201124163625430" style="zoom:30%;" />，甚至可以在drl文件中声明一个类，或者继承java代码中已经写好的类。 |
| function（可选） | 可以引用java程序中已经写好的函数<img src="/Users/didi/Documents/BRMS.assets/image-20201124163247765.png" alt="image-20201124163247765" style="zoom:30%;" /> |
| salience（可选） |            表明当前规则的优先级，值越大优先级越高            |
|   agenda-group   | 声明当前规则所在组，一个组可以有多个规则，但是该组中的多个规则中，只会有一个规则被执行，执行完以后，就不会再执行其他规则。 |
|    rule + ""     |                        该规则的标示ID                        |
|       when       |                       LHS，评估语句块                        |
|       then       |                       RHS，结果语句块                        |
|       end        |                       表明当前规则结束                       |
|      global      |      声明一个全局变量，该变量在当前规则文件中都可以生效      |
|                  |                                                              |
|                  |                                                              |
|                  |                                                              |



## EasyRules



https://github.com/j-easy/easy-rules

文档：https://www.jianshu.com/p/39fa0475643a

### 	特点：

- 轻量级，学习成本低
- 基于POJO
- 集成了mevl表达式
- 可以通过yml来编写规则文件，但是规则复杂的话可读性不高，还可以使用代码来对规则进行编写，但代码编写的可读性也不高。

### 缺点：

- 通过java来编写规则的话，对于规则的修改不友好。需要进入对应代码来修改
- 适用于极简单的规则。对于复杂规则很难实现

<img src="/Users/didi/Documents/image-20201123142459459.png" alt="image-20201123142459459" style="zoom:50%;" />





## Aviator

Aviator的设计目标是轻量级和高性能，当然, Aviator的语法是受限的, 它不是一门完整的语言, 而只是语言的一小部分集合。Aviator的实现思路与其他轻量级的求值器很不相同, 其他求值器一般都是通过解释的方式运行, 而Aviator则是直接将表达式编译成Java 字节码, 交给JVM去执行。Aviator支持大部分运算操作符, 包括算术操作符、关系运算符、逻辑操作符、位运算符、正则匹配操作符(`=~`)、三元表达式(`?:`), 并且支持操作符的优先级和括号强制优先级, 具体请看后面的操作符列表, 支持自定义函数.

https://github.com/killme2008/aviatorscript

文档：https://www.yuque.com/boyan-avfmj/aviatorscript/cpow90

### 特点：

- 开源，社区活跃，仍在维护中，有较为完整的中文文档
- 可以搭配相应的idea插件来使用（https://github.com/yanchangyou/aviatorscript-ideaplugin）
- 支持java语言中大部分的运算符，自定义变量，动态类型（类型随着赋值而改变），循环语句，条件语句，lambda表达式；
- 支持多个规则文件间的引用
- 定位是一个表达式引擎，如今是一个通用的脚本语言，并不是单纯为规则而生

### 缺点：

- 由个人维护，对于新版本的使用可能会存在bug，需要考虑稳定性。

- 不支持将对象类型作为参数传入。

- 不像其他规则引擎，Aviator无法在一个av文件中编写多个规则。

  



## QlExpress

https://github.com/alibaba/QLExpress

### 特点：

- ali开源的，高效稳定
- 使用普通的java语法，符合我们的技术栈
- 支持脚本定义function
- 可以拓展操作符或者自定义Operator，如：

```java
runner.addOperatorWithAlias("如果", "if",null);
runner.addOperatorWithAlias("则", "then",null);
runner.addOperatorWithAlias("否则", "else",null);

exp = "如果  (语文+数学+英语>270) 则 {return 1;} 否则 {return 0;}";
DefaultContext<String, Object> context = new DefaultContext<String, Object>();
runner.execute(exp,context,null,false,false,null);
```





### 缺点：

- 社区不活跃，文档不完整，学习成本高，需要看源码
- 不支持异常捕获（try，catch），不支持lambda表达式，不支持for循环







## Urule

开源版社区不活跃，最后的更新在两年前。

商用版暂时不考虑。



## VisualRules

只有商用版。





## 总结

1. 不考虑使用环境时，推荐Drools，EasyRules，Aviator，原因是这三个的社区都比较活跃，开发者仍在积极维护。
2. 如果只考虑到非常轻量的使用，或者说需要处理的逻辑十分简单的时候，推荐使用EasyRules，因为它的学习成本低，使用也非常简便，可以轻松的处理较为简单的逻辑。
3. 如果涉及到较为复杂的逻辑处理，就推荐使用Drools和Aviator，但这两个也各有特点。
4. Drools可以很好的处理复杂的逻辑任务，可以很好的实现业务逻辑与代码分离，可以将规则文件与项目进行分离，在外部对规则文件进行更新，甚至使用xls、csv等决策表或字符串来动态的编写规则文件，使得规则的更新变得十分简单。但它需要编写一定的配置文件，使得Drools较Aviator相比会略显沉重。
5. 使用Aviator也可以很好很高校的处理复杂的逻辑任务，实现业务逻辑与代码的分离。但若要使用该组件需要寻找相对稳定的版本，而不是找到最新版本，例如我在调研时使用了最新的版本，其中就出现了问题使得程序无法运行，在降低版本后才解决。在查看文档时未提及可以使用对象类型作为参数进行逻辑处理，这也是较大的一个缺陷。