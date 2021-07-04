



使用文档：

https://docs.jboss.org/drools/release/7.46.0.Final/drools-docs/html_single/index.html#_droolslanguagereferencechapter



### 依赖

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



### 工具类

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



### 简单例子

```java

package rule02;
import com.xzy.drools.entity.gameEntity;
rule "Underage"
  salience 15
  when
    $game : gameEntity()
    $d:Double($d==0)

  then
    $game.setName( "赛博朋克2077" );
    $game.setAge( 0 );
end

  
  @Data
public class gameEntity {

    private String name;

    private int age;
}
  
  
@Test
    public void test01() throws Exception {
        gameEntity game = new gameEntity();
        KieSession kieSession = KieSessionUtils.newKieSession("/Users/didi/IdeaProjects/spring-study/drools-03/src/main/resources/rules/rule02.drl");
        kieSession.insert(game);
        kieSession.insert(0d);
        kieSession.fireAllRules();
        System.out.println(game.toString());
    }

当when中的条件都满足时，就可以触发then
```

![image-20201126211530484](/Users/didi/Documents/Drools.assets/image-20201126211530484.png)

### function例子

```java
function String hello(String applicantName) {
    return "Hello " + applicantName + "!";
}

rule "Using a function"
  when
    // Empty
  then
    System.out.println( hello( "James" ) );
end
  
  
  //或者
  
import function my.package.applicant.hello;

rule "Using a function"
  when
    // Empty
  then
    System.out.println( hello( "James" ) );
end
  
```

<img src="/Users/didi/Documents/Drools.assets/image-20201126211459376.png" alt="image-20201126211459376" style="zoom: 50%;" />





### Query例子

```java
import com.xzy.drools.entity.gameEntity;
query "game more than 1 year"
    $game : gameEntity( age > 1 )
end
      
          @Test
    public void test04()throws Exception{
        KieSession kieSession = KieSessionUtils.newKieSession("/Users/didi/IdeaProjects/spring-study/drools-03/src/main/resources/rules/rule04.drl");
        gameEntity game1 = new gameEntity("lol",5);
        gameEntity game2 = new gameEntity("wow",10);
        kieSession.insert(game1);
        kieSession.insert(game2);
        QueryResults results = kieSession.getQueryResults( "game more than 1 year" );
        System.out.println(" game more than 1 year have " + results.size() );
        kieSession.fireAllRules();
    }
```







### 万能函数accumulate

语法结构：

```
accumulate( <source pattern>; <functions> [;<constraints>] )
```

例子：

```java
//所有订单的金额低于800时，触发规则
rule "order_0"
 
    when
        accumulate(
            Order(total>0.00, $total:total);
            $sum:sum($total);
            $sum <800.00
        )
    then
    System.out.println("销售业绩太差了");
end

```





### 通过使用GIFT对象存储服务来上传和获取drl文件

```bash

上传：  http://100.69.238.36:8000/resource/<namespace>/<key>

例如： curl http://10.90.28.42:8050/resource/anything/test.jpg -X POST -F filecontent=@test.jpg
curl http://10.88.128.229:8000/resource/businessdrl/rule01.drl -X POST -F 


查询、删除： http://100.69.238.34:8000/resource/<namespace>/<key>


查询： curl http://100.69.238.37:8000/resource/businessdrl/rule01.drl

删除： curl -X DELETE http://100.69.238.37:8000/resource/businessdrl/rule01.drl


下载：内网地址http://100.69.238.50/static/<namespace>/<key>


下载：curl http://10.90.28.42:8052/static/anything/test.jpg



默认域名 img-hxy011.didistatic.com, http://img-hxy011.didistatic.com/static/<namespace>/<key>
例如 http://img-hxy011.didistatic.com/static/hxy01/test.txt
如果配置了自己的子域名，这可以使用自己子域名，比如foundation.didistatic.com/static/foundation/test.jpg


/Users/didi/IdeaProjects/spring-study/drools-02/src/main/resources/rules/rule2.drl
```



通过上传时返回的JSON中或者使用查询服务可以获得文件的url

```json
{
    "creation_time": 1606815564,
    "download_url": "http://img-hxy021.didistatic.com/static/businessdrl/rule01.drl",
    "download_url_https": "https://img-hxy021.didistatic.com/static/businessdrl/rule01.drl",
    "file_size": 368,
    "md5": "df8add53c79357a87f48c06a5c965d79",
    "status": "success",
    "status_code": 200
}
```

通过GET请求这个url我们就可以获得其中的内容。





项目开始时：

先通过查询方法获取到rule.drl的信息

```json
http://100.69.238.37:8000/resource/businessdrl/rule.drl
```

得到JSON后截取中间的download_url部分

再通过这个部分去读取drl文件的内容。



需要注意的是gift服务刷新太慢，可能需要手动刷新？

http://wiki.intra.xiaojukeji.com/pages/viewpage.action?pageId=30559598#GIFTV2.0用户手册-代码示例

通过使用CDN来刷新文件内容。



所以每次上传完新文件以后，都需要手动去刷新一下才行，否则一直读取的都是旧数据。