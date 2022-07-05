---
title:  编码规范
permalink: /docs/shares-code/
---

### Java 开发规范


#### 函数命名规范

##### 1. 命名的意义

$nbsp;$nbsp;$nbsp;$nbsp;函数命名要具体,空泛的命名没有意义。例如,processData()就不是一个好的命名,因为所有的方法都是对数据的处理,这样的命名并没有表明要做的事情,相比之下,validateUserCredentials()或者eliminateDuplicateRequests()就要好许多。    
$nbsp;$nbsp;$nbsp;$nbsp;函数的命名要体现做什么,而不是怎么做。假如我们将雇员信息存储在一个栈中,现在要从栈中获取最近存储的一个雇员信息,那么getLatestEmployee()就比popRecord()要好,因为栈数据结构是底层实现细节,命名应该提升抽象层次、体现业务语义。合理的命名可以使你省掉记住“出栈”的脑力步骤,你只需要简单地说“取最近雇员的信息”。
   

##### 2. 命名统一性

$nbsp;$nbsp;$nbsp;$nbsp;每个概念对应一个词,并且一以贯之。例如,fetch、retrieve、get、find和query都可以表示查询的意思,如果不加约定地给多个类中的同种查询方法命名,你怎么记得是哪个类中的哪个方法呢?同样,在一段代码中,同时存在manager、controller和handler,会令人感到困惑。因此在项目中,通常按照表如下所示的约定,保持命名的一致性。
  
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

#####  3. 使用对仗词
$nbsp;$nbsp;$nbsp;$nbsp;遵守对仗词的命名规则有助于保持一致性,从而提高代码的可读性。像first/last这样的对仗词就很容易理解;而像fileOpen()和fClose()这样的组合则不对称,容易使人迷惑。     
$nbsp;$nbsp;$nbsp;$nbsp;下面列出一些常见的对仗词组:
   
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

#####  4. 后置限定词
$nbsp;$nbsp;$nbsp;$nbsp;很多程序中会有表示计算结果的变量,例如总额、平均值、最大值等。如果你要用类似Total、Sum、Average、Max、Min这样的限定词来修改某个命名,那么记住把限定词加到名字的最后,并在项目中贯彻执行,保持命名风格的一致性。   
$nbsp;$nbsp;$nbsp;$nbsp;这种方法有很多优点。首先,变量名中最重要的部分,即为这一变量赋予主要含义的部分应位于最前面,这样可以突出显示,并会被首先阅读到。其次,可以避免同时在程序中使用totalRevenue和revenueTotal 而产生的歧义。如果贯彻限定词后置的原则,我们就能收获一组非常优雅、具有对称性的变量命名,例如revenueTotal(总收入)、expenseTotal(总支出)、revenueAverage(平均收入)和expenseAverage(平均支出)。   
$nbsp;$nbsp;$nbsp;$nbsp;需要注意的一点是Num这个限定词,Num放在变量名的结束位置表示一个下标,customerNum表示的是当前客户的序号。为了避免Num带来的麻烦,我建议用Count或者Total来表示总数,用Id表示序号。这样,customerCount表示客户的总数,customerId表示客户的编号。   


#### 类命名规范
$nbsp;$nbsp;$nbsp;$nbsp;类是面向对象中最重要的概念之一,是一组数据和操作的封装。对于一个应用系统,我们可以将类分为两大类:实体类和辅助类。    
$nbsp;$nbsp;$nbsp;$nbsp;实体类承载了核心业务数据和核心业务逻辑,其命名要充分体现业务语义,并在团队内达成共识,如Customer、Bank和Employee等。     
$nbsp;$nbsp;$nbsp;$nbsp;辅助类是辅佐实体类一起完成业务逻辑的,其命名要能够通过后缀来体现功能。例如,用来为Customer做控制路由的控制类CustomerController、提供Customer服务的服务类CustomerService、获取数据存储的仓储类CustomerRepository。   
   
$nbsp;$nbsp;$nbsp;$nbsp;对于辅助类,尽量不要用Helper、Util之类的后缀,因为其含义太过笼统,容易破坏SRP(单一职责原则)。    
$nbsp;$nbsp;$nbsp;$nbsp;比如对于处理CSV,可以这样写:    
```
      CSVHelper.parse(String) 
      CSVHelper.create(int[])
```
$nbsp;$nbsp;$nbsp;$nbsp;但是我更建议将CSVHelper拆开:   
```
      CSVParser.parse(String)
      CSVBuilder.create(int[])
```


#### 包名规范

$nbsp;$nbsp;$nbsp;$nbsp;包(Package)代表了一组有关系的类的集合,起到分类组合和命名空间的作用。在JavaScript的早期阶段,因为缺乏明确的分包机制,导致程序(特别是大型程序)很容易陷入混乱。     
$nbsp;$nbsp;$nbsp;$nbsp;包名应该能够反映一组类在更高抽象层次上的联系。例如,有一组类Apple、Pear、Orange,我们可以将它们放在一个包中,命名为fruit。    
$nbsp;$nbsp;$nbsp;$nbsp;包的命名要适中,不能太抽象,也不能太具体。此处以上面提到的水果作为例子,如果包名过于具体,比如Apple,那么Pear和Orange放进该包中就不恰当了;如果报名太抽象,称为Object,而Object无所不包,这就失去了包用来限定范围的作用。   
   
#### 模块名规范

$nbsp;$nbsp;$nbsp;$nbsp;这里说的模块(Module)主要是指Maven中的Module,相对于包来说,模块的粒度更大,通常一个模块中包含了多个包。    
$nbsp;$nbsp;$nbsp;$nbsp;在Maven中,模块名就是一个坐标: <groupId, artifactId>。一方面, 其名称保证了模块在Maven仓库中的唯一性;另一方面,名称要反映模块在系统中的职责。例如,在COLA架构中,模块代表着架构层次,因此,对任何应该遵循COLA规范的应用都有着xxx-controller、xxx-app、xxx-domain和xxx-Infrastructure这4个标准模块。   

 <img width="278" alt="image" src="https://user-images.githubusercontent.com/5245347/177231695-3483092a-6a7c-42a6-9502-34d6654d261e.png">


#### 场景举例说明 

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



#### 分层规范

##### 1. 整体分层

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


##### 2.  controller

命名：根据交互的展示页面来命名。

职责：请求参数的校验（非业务逻辑的），返回数据组合、拼装。

*正例：*

```java
/**
 * 系统配置Controller
 */
@RestController
@RequestMapping("systemConfig")
public class SystemConfigController{
    @PostMapping("/save")
    public Response saveSystemConfig(@RequestBody SystemConfigDTO configDTO) {
        configDTO.validateXXX(); //请求参数的校验
        systemConfigService.saveSystemConfig(configDTO);
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
GET  	/department/get?id=		查询某个部门详细信息
POST 	/department/save		添加部门
POST 	/department/edit		更新部门
POST 	/department/remove		删除部门
```

*反例：*

```
GET		/department/{id}		查询某个部门详细信息
POST	/department				添加部门
PUT		/department				更新部门
DELETE	/department/{id}		删除部门
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
public Response editUser(UserDTO userDTO) {

    User user = userService.getUser(userDTO.getId());
    if (user == null) {
        return Response.buildFailure("xxx","用户不存在");
    }

    List<Role> roleList = userDTO.getRoleList();
    userRoleService.edit(roleList);
    
    userService.editUser(userDto);

    return Response.buildSuccess();
}
```

##### 3. service

命名：以约定的业务模块命名。

职责：业务编排，业务逻辑在entity层编写。

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

##### 4. dao

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

##### 5. entity

命名： 建议与数据库表保持一致，可遵循Domain规范（若了解领域驱动设计思想）。  
职责： 业务逻辑的执行处理

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

#### 异常定义规范

异常分类两个基本类：返回给前端错误信息的 `MessageException` ，后台出现异常的 `BaseException` 。

- MessageException
  - 用在controller层返回给前台提示。
  - 需要包含 错误码 `errorCode` 和**给用户展示的**错误提示信息 `errorMessage`。
- BaseException
  - 用在service层以及以下，用于找到出错的位置。
  - 需要包含**给程序员调试的**错误提示信息和堆栈信息。

##### 1.  应该抛出异常的的情况

1.无法通过参数检查。

2.可能导致以后的相关功能出错。

3.本方法没有处理异常的能力，让调用方处理。

##### 2.  什么时候自定义异常类

1.出现特定于业务的异常状态时候。

比如，在超出自定义时间范围内时，创建并且抛出一个`InvalidDateRangeException`。

2.错误处理需要定制信息以提供给与用户（开发人员或者最终用户）。

#### 日志记录规范

##### 1. 需要记录日志的情况

- 登录成功和失败
- 注销
- 密码更改和重置
- 创建和删除用户
- 授权失败
- 访问级别更改
- catch中

此外，还应该记录可以指向特定攻击的指标。

- 输入验证错误：识别导致 XSS 攻击的脚本标签，或 UNION 等指示 SQL 注入的参数
- HTTP 4xx 和 5xx 响应

##### 2. 需要记录的内容

- 调用的方法名
- 方法参数
- 状态 （成功或者失败）
- 操作时间

##### 3. 记录规范

1.日志分级

1. `fatal` - 严重的，造成服务中断的错误；

2. `error` - 其他错误运行期错误；

3. `warn` - 警告信息，如程序调用了一个即将作废的接口，接口的不当使用，运行状态不是期望的但仍可继续处理等；

4. `info` - 有意义的事件信息，如程序启动，关闭事件，收到请求事件等；记录给运维人员查看的信息。

5. `debug` - 记录详细的业务处理到哪一步了，以及当前的变量状态；记录给开发人员调试信息。*比如方法的参数，返回的结果等。*

6. `trace` - 更详细的跟踪信息；

2.可直接使用日志系统（Log4j、Logback）中的API，而应依赖使用日志框架SLF4J中的API，使用门面模式的日志框架，有利于维护和各个类的日志处理方式统一。

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

private static final Logger log = LoggerFactory.getLogger(Test.class);
```

3.使用占位符 `{}` 拼接字符串，而不是使用加号 `+` 拼接

```java
public void function(String param) {
    log.debug("function start, input param: {}", param);
}
```

4.禁止使用 `exception.printStackTrace()`

5.异常信息应该包括两类信息：案发现场信息和异常堆栈信息。如果不处理，那么往上抛。

```java
logger.error(各类参数或者对象toString + "_" + e.getMessage(), e);
```

6.禁止出现 `System.print`(包括System.out.println和System.err.println)语句。
