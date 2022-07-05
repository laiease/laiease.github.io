---
title:  编码规范
permalink: /docs/shares-code/
---

时间: 2022-07-05 
<br/>

### 1. 命名规范
<br/>
### 1.1. 函数命名规范
<br/>
#### 1.1.1. 命名的意义
&ensp;&ensp;函数命名要具体,空泛的命名没有意义。例如,processData()就不是一个好的命名,因为所有的方法都是对数据的处理,这样的命名并没有表明要做的事情,相比之下,validateUserCredentials()或者eliminateDuplicateRequests()就要好许多。   
<br/>
&ensp;&ensp;函数的命名要体现做什么,而不是怎么做。假如我们将雇员信息存储在一个栈中,现在要从栈中获取最近存储的一个雇员信息,那么getLatestEmployee()就比popRecord()要好,因为栈数据结构是底层实现细节,命名应该提升抽象层次、体现业务语义。合理的命名可以使你省掉记住“出栈”的脑力步骤,你只需要简单地说“取最近雇员的信息”。
   

#### 1.1.2 命名统一性

&ensp;&ensp;每个概念对应一个词,并且一以贯之。例如,fetch、retrieve、get、find和query都可以表示查询的意思,如果不加约定地给多个类中的同种查询方法命名,你怎么记得是哪个类中的哪个方法呢?同样,在一段代码中,同时存在manager、controller和handler,会令人感到困惑。因此在项目中,通常按照表如下所示的约定,保持命名的一致性。
  
<table>
  <thead>
    <tr>
      <td>操作</td>
      <td>函数名约定</td>
    </tr>          
  </thead>
  <tbody>
    <tr>
      <td>新增</td>
      <td>create</td>
    </tr>
     <tr>
      <td>添加</td>
      <td>add</td>
    </tr>
     <tr>
      <td>删除</td>
      <td>remove</td>
    </tr>
     <tr>
      <td>修改</td>
      <td>update</td>
    </tr>
     <tr>
      <td>查询(单个结果)</td>
      <td>get</td>
    </tr>
     <tr>
      <td>查询(多个结果)</td>
      <td>find</td>
    </tr>
     <tr>
      <td>分页查询</td>
      <td>page</td>
    </tr>
     <tr>
      <td>统计</td>
      <td>count</td>
    </tr>
  </tbody>
</table>

#####  1.1.3 使用对仗词
&ensp;&ensp;遵守对仗词的命名规则有助于保持一致性,从而提高代码的可读性。像first/last这样的对仗词就很容易理解;而像fileOpen()和fClose()这样的组合则不对称,容易使人迷惑。     
&ensp;&ensp;下面列出一些常见的对仗词组:
   
* add/remove 
* increment/decrement 
* open/close 
* begin/end 
* insert/delete 
* show/hide 
* create/destroy 
* lock/unlock 
* source/target 
* first/last 
* min/max 
* start/stop 
* get/set 
* next/previous 
* up/down 
* old/new

####  1.1.4 后置限定词
&ensp;&ensp;很多程序中会有表示计算结果的变量,例如总额、平均值、最大值等。如果你要用类似Total、Sum、Average、Max、Min这样的限定词来修改某个命名,那么记住把限定词加到名字的最后,并在项目中贯彻执行,保持命名风格的一致性。   
<br/>
&ensp;&ensp;这种方法有很多优点。首先,变量名中最重要的部分,即为这一变量赋予主要含义的部分应位于最前面,这样可以突出显示,并会被首先阅读到。其次,可以避免同时在程序中使用totalRevenue和revenueTotal 而产生的歧义。如果贯彻限定词后置的原则,我们就能收获一组非常优雅、具有对称性的变量命名,例如revenueTotal(总收入)、expenseTotal(总支出)、revenueAverage(平均收入)和expenseAverage(平均支出)。   
<br/>
&ensp;&ensp;需要注意的一点是Num这个限定词,Num放在变量名的结束位置表示一个下标,customerNum表示的是当前客户的序号。为了避免Num带来的麻烦,我建议用Count或者Total来表示总数,用Id表示序号。这样,customerCount表示客户的总数,customerId表示客户的编号。   
<br/>

### 1.2 类命名规范
&ensp;&ensp;类是面向对象中最重要的概念之一,是一组数据和操作的封装。对于一个应用系统,我们可以将类分为两大类:实体类和辅助类。    
<br/>
&ensp;&ensp;实体类承载了核心业务数据和核心业务逻辑,其命名要充分体现业务语义,并在团队内达成共识,如Customer、Bank和Employee等。     
<br/>
&ensp;&ensp;辅助类是辅佐实体类一起完成业务逻辑的,其命名要能够通过后缀来体现功能。例如,用来为Customer做控制路由的控制类CustomerController、提供Customer服务的服务类CustomerService、获取数据存储的仓储类CustomerRepository。   
<br/>
&ensp;对于辅助类,尽量不要用Helper、Util之类的后缀,因为其含义太过笼统,容易破坏SRP(单一职责原则)。    
<br/>
&ensp;比如对于处理CSV,可以这样写:    
```
      CSVHelper.parse(String) 
      CSVHelper.create(int[])
```
&ensp;但是我更建议将CSVHelper拆开:   
```
      CSVParser.parse(String)
      CSVBuilder.create(int[])
```


### 1.3 包名规范

&ensp;&ensp;包(Package)代表了一组有关系的类的集合,起到分类组合和命名空间的作用。在JavaScript的早期阶段,因为缺乏明确的分包机制,导致程序(特别是大型程序)很容易陷入混乱。     
<br/>
&ensp;&ensp;包名应该能够反映一组类在更高抽象层次上的联系。例如,有一组类Apple、Pear、Orange,我们可以将它们放在一个包中,命名为fruit。    
<br/>
&ensp;&ensp;包的命名要适中,不能太抽象,也不能太具体。此处以上面提到的水果作为例子,如果包名过于具体,比如Apple,那么Pear和Orange放进该包中就不恰当了;如果报名太抽象,称为Object,而Object无所不包,这就失去了包用来限定范围的作用。   
<br/>
   
### 1.4 模块名规范

&ensp;&ensp;这里说的模块(Module)主要是指Maven中的Module,相对于包来说,模块的粒度更大,通常一个模块中包含了多个包。    
<br/>
&ensp;&ensp;在Maven中,模块名就是一个坐标: <groupId, artifactId>。一方面, 其名称保证了模块在Maven仓库中的唯一性;另一方面,名称要反映模块在系统中的职责。例如,在COLA架构中,模块代表着架构层次,因此,对任何应该遵循COLA规范的应用都有着xxx-controller、xxx-app、xxx-domain和xxx-Infrastructure这4个标准模块。   
<br/>
<img width="278" alt="image" src="https://user-images.githubusercontent.com/5245347/177231695-3483092a-6a7c-42a6-9502-34d6654d261e.png">


### 1.5 场景举例说明 

**所有命名严禁使用拼音与英文混合的方式，更不允许直接使用中文拼音的方式。**

正例：`jinan / beijing / rmb 等国际通用的名称，可视同英文。`   
反例：`DaZhePromotion [打折] / getPingfenByName() [评分] / int 某变量 = 3`.  

**杜绝完全不规范的缩写，避免望文不知义：**

反例：condition"缩写"命名成 condi，此类随意缩写严重降低了代码的可阅读性。    

推荐动词在前，名词在后.  

```java
void importExcel(Excel excel); 
```

dao层可以省略名词。   

```java
public interface StudentMapper {
    int save(Student student);
   
    List<Student> findList();
   
    int edit(Student student);
}
```

service层要写操作的实体名。   

```java
public class StudentService{
    void saveStudent(Student student){}
    
    List<Student> findListStudent(){}
    
    void editStudent(Student student)(){}
}
```

* 属性名称

- boolean类型变量都不要加 `is`.  

* 方法参数

- 无论是 `controller` , `service` , `dao` 还是其他的代码，每个方法最多 `3` 个参数，如果超出 `3` 个参数的话，建议封装成 `javabean` 对象。   
- 不推荐直接使用 `JSONObject` , `Map<String,Object>` 等直接作为方法参数。  



### 2. 分层规范

#### 2.1. 整体分层

<table>
  <thead>
    <tr>
      <td>层次</td>
      <td>作用</td>
    </tr>          
  </thead>
  <tbody>
    <tr>
      <td>controller</td>
      <td>接口交互层，约定交互的输入输出要求，通过调用service获取数据并组装成mapping约定的格式返回数据</td>
    </tr> 
     <tr>
      <td>service</td>
      <td>业务逻辑层，负责业务的编排。service只是负责业务流水线的约定，service不具体负责业务逻辑的处理。service层的输入与输出不需要以controller层相一致，以便与提升复用性</td>
    </tr> 
     <tr>
      <td>entity</td>
      <td>实体对象层，他与数据库表相对应，entity不是数据容器，entity有自己的业务处理逻辑。通常entity是非Bean对象。</td>
    </tr> 
     <tr>
      <td>dao</td>
      <td>持久化层，负责将entity的数据持久化到硬盘中，同时也负责将硬盘的数据反序列化成为entity对象</td>
    </tr> 
  </tbody>
</table>


#### 2.2.  controller

命名：根据交互的展示页面来命名。   
职责：请求参数的校验（非业务逻辑的），返回数据组合、拼装，*controller层可以直接对持久化层的数据做展示性响应*。

*正例：*

```java
/**
 * 系统配置Controller
 */
@RestController
@RequestMapping("systemConfig")
public class SystemConfigController{
    @PostMapping("/save")
    public Response saveSystemConfig(@RequestBody SystemConfigVO configVO) {
        configDTO.validateXXX(); //请求参数的校验
        systemConfigService.saveSystemConfig(configVO);
        return Response.ok();
    }
}
```

1.不推荐使用 rest 命名 url， 只使用 `get/post` 方法。url 命名上规范如下：

```
/业务模块/子模块/动作
```

*正例：*

```
GET   /department/get?id=1  查询某个部门详细信息
POST  /department/save      添加部门
POST  /department/edit      更新部门
POST  /department/remove    删除部门
```

*反例：*

```
GET     /department/{id}    查询某个部门详细信息
POST    /department         添加部门
PUT     /department         更新部门
DELETE  /department/{id}    删除部门
```

2.响应数据的类型格式设置等。

*正例：*

```java
@GetMapping
public void downloadFile(String filePath, HttpServletResponse response) {
    File file = fileService.getFile(filePath);
    //使用流的形式下载文件
    InputStream is = null;
    OutputStream os = null;

    response.reset();
    response.setContentType("application/octet-stream");
    response.setCharacterEncoding("utf-8");
    response.setContentLength((int) file.length());
    response.addHeader("Content-Disposition", "attachment;filename=" + URLEncoder.encode(file.getName(), "UTF-8"));
	// ...
}
```

3.不做任何的业务逻辑操作。

*反例：*

```java
@PostMapping("/edit")
@Transactional
public Response editUser(UserVO userVO) {

    User user = userService.getUser(userVO.getId());
    if (user == null) {
        return Response.buildFailure("xxx","用户不存在");
    }

    List<Role> roleList = userVO.getRoleList();
    userRoleService.edit(roleList);
    
    userService.editUser(userVO);

    return Response.buildSuccess();
}
```

4.对于试图数据对象(VO)，推荐使用内部类。  

```
@Setter
@Getter
public class UserVO{

   private String name;
   private String department;
   private List<Role> roles;
   
   @Setter
   @Getter
   public staic class Role {
      private String id;
      private String name;
   }
}

```

#### 2.3. service

命名：以约定的业务模块命名。    

职责：业务编排，service只是负责业务流水线的约定，service不具体负责业务逻辑的处理。service层的输入与输出不需要以controller层相一致，以便与提升复用性。业务逻辑在entity层编写。

*正例：*

```java
public CountResult analysisTemporary(Temporary temporary) {
    // -------------- 1、解析地址信息 ------------------------
    List<Document> documentList = addressResolver.resolveAddress(temporaryList);
    // -------------- 2、保存档案库信息 -------------------------------
    documentMapper.saveBatch(documentList);
    // -------------- 3、更新临时库信息 -------------
    temporaryMapper.updateBatch(temporaryList);
    // -------------- 4、统计失败和成功的数据 --------------------
    return new CountResult(successCount, errorCount);
}
```

1.不推荐写service接口然后写实现类。(由于大部分service都不具备重写需求)

```java
@Service
public class StudentServiceImpl implements StudentService{
    // ......
} 
```

2.合理拆分service文件，如果业务较大，请拆分为多个service。

3.避免将controller层某些参数传递到service 中。

*反例：*

```java
public void downloadImage(HttpServletResponse response, Image image) throws IOException {
    Image image = imageMapper.getById(image.getId());
    InputStream in = fileService.download(image.getOriginalUrl());
    BufferedOutputStream out = new BufferedOutputStream(response.getOutputStream());
    ......
}
```

4.禁止使用 MyBatis-Plus 的 Wrapper 条件构建器在service层拼接sql。

*反例：*

```java
public List<Folder> findChildFolder(String folderId, List<String> folderIds) {
    return folderMapper.selectList(new LambdaQueryWrapper<Folder>().select(Folder::getId).eq(Folder::getPid, folderId));
}
```

5.禁止返回值是 Controller 层里响应给前端的返回类型。

*正例：*

```java
public class StudentService{
    public List<Student> findListStudent(Student student) {
        // ......
        return studentList;
    }
}
```

*反例：*

```java
public class StudentService{
    public ResponseEntity<List<Student>> findListStudent(Student student) {
        // ......
        return ResponseEntity.ok(studentList);
    }
}
```

#### 2.4. dao

命名： 和数据库表一致。  
职责： 负责将entity的数据保存到数据库中，也负责将数据反序列化成为entity对象。
1.建议dao层方法命名尽量以sql语义命名，避免与业务关联。

*正例：*

```java
List<Student> findListByClassId(@Param("classId") Long classId);
```

*反例：*

```java
List<Student> getData(@Param("classId") Long classId);
```

2.建议不要直接在 mybatis xml 中写死常量，应从 dao 中传入到 xml 中。

*正例：*

```java
int count(@Param("status") Integer status);
```

对应的mapper.xml

```xml
<select id="count" resultType="integer">
    select
    count(1)
    from student
    where
    status = #{status}
</select>
```

*反例*

```java
int count();
```

对应的mapper.xml

```xml
<select id="count" resultType="integer">
    select
    count(1)
    from student
    where
    status = 1
</select>
```

#### 2.5. entity

命名： 建议与数据库表保持一致。   
职责： 业务逻辑的执行处理，entity不是数据容器，entity有自己的业务处理逻辑。可遵循Domain规范（若了解领域驱动设计思想）

正例：

```java
public class StudentEntity{
    int status;
    int deleted;

    public boolean isEnable(){
        return status == 1;
    }

    public boolean isEffective(){
        return status == 1 && deleted = 0;
    }
    
}
```

### 3. 异常定义规范

异常分类两个种：返回给前端错误信息的 `LocalMessageException` ，以及业务后台出现的异常的 `Exception` 。

- LocalMessageException （uncheckexception）
  - 用在controller层返回给前台提示。
  - 需要包含 错误码 `errorCode` 和**给用户展示的**错误提示信息 `errorMessage`。 [LocalMessageException](https://github.com/codingapi/springboot-framework)支持国际化 
- Exception （checkexception）
  - 用在service层以及以下，用于找到出错的位置。
  - 需要包含**给程序员调试的**错误提示信息和堆栈信息。

&ensp;&ensp;在Service层以下出现的异常，都是业务异常。这些异常都是checkexception，需要try catch完成异常的处理。当service层的异常传递到controller层时就需要返回LocalMessageException。       
&ensp;&ensp;LocalMessageException是给用户提示的异常，不允许展示程序内部异常堆栈信息，要给用户以友好的提示。尽量让用户清楚如何解决该问题。


* 1.应该抛出异常的的情况

1.无法通过参数检查。  
2.可能导致以后的相关功能出错。  
3.本方法没有处理异常的能力，让调用方处理。即：将底层的异常继续往上抛  

* 2.什么时候自定义异常类

1.出现特定于业务的异常状态时候。
比如，在超出自定义时间范围内时，创建并且抛出一个`InvalidDateRangeException`。
2.错误处理需要定制信息以提供给与用户（开发人员或者最终用户）。

* 3.异常场景举例说明


反例
```java
     
    int getUserId(String username){
        User user =  userDao.getUserByUserName(username);
        if(user==null){
            return 0;
        }
        return user.getId();
    }

```


正例 
```java
     
    int getUserId(String username){
        User user =  userDao.getUserByUserName(username);
        if(user==null){
            throw new UserNotFundException(String.format("can't find user :%s",username));
        }
        return user.getId();
    }


```

 

### 4. 日志记录规范
<br/>
#### 4.1. 日志规范

&ensp;&ensp;日志的重要性很容易被开发人员忽视,写好程序的日志可以帮助我们大大减轻后期维护的压力。在实际工作中,开发人员往往迫于时间压力,认为写日志是一件非常烦琐的事情,往往没有足够的重视,导致日志文件管理混乱、日志输出格式不统一,结果在出现故障时影响工作效率。开发人员应在一开始就养成良好的撰写日志的习惯,并在实际的开发工作中为写日志预留足够的时间。       
<br/>
&ensp;&ensp;在打印日志时,要特别注意日志输出级别,这是系统运维的需要。详细的日志输出级别分为OFF、FATAL、ERROR、WARN、INFO、DEBUG、ALL或者自定义的级别。我认为比较有用的4个级别依次是ERROR、WARN、INFO和DEBUG。通常这4个级别就能够很好地满足我们的需求了。    

* 日志分级

1. `fatal` - 严重的，造成服务中断的错误；

2. `error` - 其他错误运行期错误；

3. `warn` - 警告信息，如程序调用了一个即将作废的接口，接口的不当使用，运行状态不是期望的但仍可继续处理等；

4. `info` - 有意义的事件信息，如程序启动，关闭事件，收到请求事件等；记录给运维人员查看的信息。

5. `debug` - 记录详细的业务处理到哪一步了，以及当前的变量状态；记录给开发人员调试信息。*比如方法的参数，返回的结果等。*

6. `trace` - 更详细的跟踪信息；


#### 4.2. ERROR级别

&ensp;&ensp;ERROR表示不能自己恢复的错误,需要立即被关注和解决。例如,数据库操作错误、I/O错误(网络调用超时、文件读取错误等)、未知的系统错误(NullPointerException、OutOfMemoryError等)。对于ERROR,我们不仅要打印线程堆栈,最好打印出一定的上下文(链路TraceId、用户Id、订单Id、外部传来的关键数据),以便于排查问题。    
<br/>
&ensp;&ensp;ERROR要接入监控和报警系统。ERROR需要人工介入处理,及时止损,否则会影响系统的可用性。当然也不能滥用ERROR,否则就会出现“狼来了”的情况。我在实际工作中曾碰到过系统每天会发出上千条错误报警的情况,导致根本没有人看报警内容,在真正出现问题时,也没有人关注,从而引发线上故障。因此,一定要做好ERROR输出的场景定义和规范,再配合监控治理,双管齐下,确保线上系统的稳定。   


#### 4.3. WARN级别

&ensp;&ensp;对于可预知的业务问题,最好不要用ERROR输出日志,以免污染报警系统。例如,参数校验不通过、没有访问权限等业务异常,就不应该用ERROR输出。    
<br/>
&ensp;&ensp;需要注意的是,在短时间内产生过多的WARN日志,也是一种系统不健康的表现。因此,我们有必要为WARN配置一个适当阈值的报警, 比如访问受限WARN超过100次/分,则发出报警。这样在WARN日志过于频繁时,我们能及时收到系统报警,去跟进用户问题。例如,如果是产品设计上有缺陷导致用户频繁出现操作卡点,可以考虑做一下流程或者产品上的优化。   
<br/>
#### 4.4. INFO级

&ensp;&ensp;INFO用于记录系统的基本运行过程和运行状态。
<br/>
&ensp;&ensp;通常来说,优先根据INFO日志可初步定位,主要包括系统状态变化日志、业务流程的核心处理、关键动作和业务流程的状态变化。适当的INFO可以协助我们排查问题,但是切忌把INFO当成DEBUG使用,这样会导致记录的数据过多,一方面影响系统性能,日志文件增长过快, 消耗不必要的存储资源;另一方面也不利于阅读日志文件。   
<br/>
#### 4.5. DEBUG级别

&ensp;&ensp;DEBUG是输出调试信息,如request/response的对象内容。在输出对象内容时,要覆盖Object的toString方法,否则输出的是对象的内存地址,就起不到调试的作用了。    
<br/>
&ensp;&ensp;通常在开发和预发环境下,DEBUG日志会打开,以方便开发和调试。而在线上环境,DEBUG开关需要关闭,因为在生产环境下开启DEBUG会导致日志量非常大,其损耗是难以接受的。只有当线上出现bug或者棘手的问题时,才可以动态地开启DEBUG。为了防止日志量过大,我们可以采用分布式配置工具来实现基于requestId判断的日志过滤,从而只打印我们所需请求的DEBUG日志。    
<br/>
#### 4.6. 使用要求说明

* 日志使用
&ensp;&ensp;可直接使用日志系统（Log4j、Logback）中的API，而应依赖使用日志框架SLF4J中的API，使用门面模式的日志框架，有利于维护和各个类的日志处理方式统一。

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

private static final Logger log = LoggerFactory.getLogger(Test.class);
```

* 使用占位符 `{}` 拼接字符串，而不是使用加号 `+` 拼接

```java
public void function(String param) {
    log.debug("function start, input param: {}", param);
}
```

* 禁止使用 `exception.printStackTrace()`

* 异常信息应该包括两类信息：案发现场信息和异常堆栈信息。如果不处理，那么往上抛。

```java
logger.error(各类参数或者对象toString + "_" + e.getMessage(), e);
```

* 禁止出现 `System.print`(包括System.out.println和System.err.println)语句。



