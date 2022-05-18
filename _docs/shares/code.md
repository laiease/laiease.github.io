---
title:  编码规范
permalink: /docs/shares-code/
---

### Java 开发规范

#### 命名规范

**所有命名严禁使用拼音与英文混合的方式，更不允许直接使用中文拼音的方式。**

正例：`jinan / beijing / rmb 等国际通用的名称，可视同英文。`

反例：`DaZhePromotion [打折] / getPingfenByName() [评分] / int 某变量 = 3`

**杜绝完全不规范的缩写，避免望文不知义：**

反例：AbstractClass"缩写"命名成 AbsClass；condition"缩写"命名成 condi，此类随意缩写严重降低了代码的可阅读性。

#####  1. 方法名称

**常见方法的前缀**

- 导入 `import` 
- 导出 `export` 
- 添加 `save`
- 修改 `edit` 
- 删除 `remove` 
- 查询列表 `findList` 
- 分页查询 `findPage` 
- 查询单个 `get`
- 统计值  `count`

推荐动词在前，名词在后

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

##### 2. 属性名称

- boolean类型变量都不要加 `is`

##### 3. 方法参数

- 无论是 `controller` , `service` , `dao` 还是其他的代码，每个方法最多 `3` 个参数，如果超出 `3` 个参数的话，建议封装成 `javabean` 对象。
- 不推荐直接使用 `JSONObject` , `Map<String,Object>` 等直接作为方法参数。

#### 分层规范

##### 1. 整体分层

- controller
- service
- dao
- entity

##### 2.  controller

命名：根据前台的展示页面命名。

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

命名：以业务模块命名。

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

1.不推荐写service接口然后写实现类。

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

命名：和数据库表一致。

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

业务逻辑或者计算

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
