---
title:  编码规范
permalink: /docs/shares-code
---

# Java 开发规范

> *v1.0.1 2022/05/16*

## 分层规范

### 1. 整体分层

- controller
- service
- dao
- entity

### 2.  controller

职责：参数校验，返回数据组合、拼装

1. 命名：前台的展示页面命名。不推荐使用 rest 命名 url， 只使用 `get/post` 方法。url 命名上规范如下：

   ```
   /业务模块/子模块/动作
   ```

   正例：

   ```
   GET  /department/get?id=	   查询某个部门详细信息
   POST /department/save          添加部门
   POST /department/edit          更新部门
   POST /department/remove  	   删除部门
   ```

2. 响应数据的类型格式设置等。

   ```java
   public void exportCheckData(String filePath, HttpServletResponse response) {
       String excelFilePath = documentService.exportCheckData(document);
       //使用流的形式下载文件
       InputStream is = null;
       OutputStream os = null;
       File file = new File(excelFilePath);
   
       response.reset();
       response.setContentType("application/octet-stream");
       response.setCharacterEncoding("utf-8");
       response.setContentLength((int) file.length());
       response.addHeader("Content-Disposition", "attachment;filename=" + URLEncoder.encode(file.getName(), "UTF-8"));
   	// ...
   }
   ```

3. 不做任何的业务逻辑操作。

### 3.  service

1. 命名：以业务模块命名。

2. 职责：业务编排，业务逻辑在entity层编写。

   正例：

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

   

3. 不推荐写service接口然后写实现类。

4. 合理拆分service文件，如果业务较大，请拆分为多个service。

5. 避免将controller层某些参数传递到service 中。

   反例：

   ```java
   public void downloadImage(HttpServletResponse response, Image image) throws IOException {
       Image image = imageMapper.getById(image.getId());
       InputStream in = fileService.download(image.getOriginalUrl());
       BufferedOutputStream out = new BufferedOutputStream(response.getOutputStream());
       ......
   }
   ```

6. 禁止使用 MyBatis-Plus 的 Wrapper 条件构建器在service层拼接sql。

7. 禁止返回值是 Controller 层里响应给前端的返回类型。

   正例：

   ```java
   public class StudentService{
       public List<Student> findListStudent(Student student) {
           // ......
           return studentList;
       }
   }
   ```

   

   反例：

   ```java
   public class StudentService{
       public ResponseEntity<List<Student>> findListStudent(Student student) {
           // ......
           return ResponseEntity.ok(studentList);
       }
   }
   ```

### 4. dao

1. 命名：和数据库表一致。
2. 建议不要直接在 mybatis xml 中写死常量，应从 dao 中传入到 xml 中。

### 5. entity

业务逻辑或者计算

## 命名规范

**所有命名严禁使用拼音与英文混合的方式，更不允许直接使用中文拼音的方式。**

正例：`jinan / beijing / rmb 等国际通用的名称，可视同英文。`

反例：`DaZhePromotion [打折] / getPingfenByName() [评分] / int 某变量 = 3`

**杜绝完全不规范的缩写，避免望文不知义：**

反例：AbstractClass"缩写"命名成 AbsClass；condition"缩写"命名成 condi，此类随意缩写严重降低了代码的可阅读性。

### 1. 方法名称

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

service层要写操作的实体名，dao层可以省略。

动词在前，名词在后。

正例：

```java
public interface StudentMapper {
    int save(Student student);
   
    List<Student> findList();
   
    int update(Student student);
}

public class StudentService{
    void saveStudent(Student student){}
    
    List<Student> findListStudent(){}
    
    void updateStudent(Student student)(){}
}
```

### 2. 属性名称

- boolean类型变量都不要加 `is`

### 3. 方法参数

- 无论是 `controller` , `service` , `dao` 还是其他的代码，每个方法最多 `3` 个参数，如果超出 `3` 个参数的话，建议封装成 `javabean` 对象。
- 不推荐直接使用 `JSONObject` , `Map<String,Object>` 等直接作为方法参数。

## 异常定义规范

异常分类两个基本类：返回给前端错误信息的 `MessageException` ，后台出现异常的 `BaseException` 。

- MessageException
  - 用在controller层返回给前台提示。
  - 需要包含 错误码 `errorCode` 和**给用户展示的**错误提示信息 `errorMessage`。

- BaseException
  - 用在service层以及以下，用于找到出错的位置。
  - 需要包含**给程序员调试的**错误提示信息和堆栈信息。


## 日志记录规范

1. 日志分级

   1. `fatal` - 严重的，造成服务中断的错误；

   2. `error` - 其他错误运行期错误；

   3. `warn` - 警告信息，如程序调用了一个即将作废的接口，接口的不当使用，运行状态不是期望的但仍可继续处理等；

   4. `info` - 有意义的事件信息，如程序启动，关闭事件，收到请求事件等；记录给运维人员查看的信息。

   5. `debug` - 记录详细的业务处理到哪一步了，以及当前的变量状态；记录给开发人员调试信息。*比如方法的参数，返回的结果等。*

   6. `trace` - 更详细的跟踪信息；

2. 可直接使用日志系统（Log4j、Logback）中的API，而应依赖使用日志框架SLF4J中的API，使用门面模式的日志框架，有利于维护和各个类的日志处理方式统一。

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

private static final Logger log = LoggerFactory.getLogger(Test.class);
```

3. 使用占位符 `{}` 拼接字符串，而不是使用加号 `+` 拼接

```java
public void function(String param) {
    log.debug("function start, input param: {}", param);
}
```

4. 禁止使用 `exception.printStackTrace()`

5. 异常信息应该包括两类信息：案发现场信息和异常堆栈信息。如果不处理，那么往上抛。

```java
logger.error(各类参数或者对象toString + "_" + e.getMessage(), e);
```

6. 禁止出现 `System.print`(包括System.out.println和System.err.println)语句。
