# Json



## json格式：

```json
{
  "code":200,
  "message":"OK",
  "data":{
    
  }
}
```

- Code:返还状态码

- message：返回信息的描述

- data：返回值



## 定义JavaBean字段

### 定义状态码枚举类

```java

@Data
public enum ResultStatus{
  SUCCESS(HttpStatus.BAD_REQUEST,400,"Bad Request"),
  BAD_REQUEST(HttpStatus.BAD_REQUEST,400,"Bad Request"),
  INTERNAL_SERVER_ERROR(HttpStatus.INTERNAL_SERVER_ERROR,500,"Internal Server Error"),;
  
  //返回的HTTP状态码，符合http请求
  private HttpStatus httpStatus;
  //业务异常码
  private Integer code;
  //业务异常信息描述
  private String message;
  
  ResultStatus(HttpStatus httpStatus,Integer code,String message){
    this.httpStatus = httpStatus;
    this.code = code;
    this.message = message;
  }
}
```



### 定义返回体类

```java
@Data
public class Result<T>{
  //业务错误码
  private Integer code;
  //信息描述
  private String message;
  //返回参数
  private T data;
  
  private Result(ResultStatus resultStatus, T data){
    this.code = resultStatus.gerCode();
    this.message = resultStatus.getMessage();
    this.data = data;
  }
  
  //业务成功返回业务代码和描述信息
  
  public static Result<Void> success(){
    return new Result<Void>(ResultStatus.SUCCESS,null);
  }
  
  //业务成功返回业务代码，描述和返回的参数
  public static <T> Result<T> success(T data){
    return new Result<T>(ResultStatus.SUCCESS,data);
  }
  
    //业务成功返回业务代码，描述和返回的参数
  public static <T> Result<T> success(ResultStatus resultStatus,T data){
    if(resultStatus == null){
      return success(data);
    }
    return new Result<T>(resultStatus,data);
  }
  
  //业务异常返回业务代码和描述信息
  public static <T> Result<T> failure(){
  	return new Result<T>(ResultStatus.INTERNAL_SERVER_ERROR,null);
  }
  
    //业务异常返回业务代码,描述和描述信息
  public static <T> Result<T> failure(){
    return failure(resultStatus,null);
  }
    //业务异常返回业务代码，描述和描述信息
  public static <T> Result<T> failure(){
    if(resultStatus == null){
      return new Result<T>(ResultStatus.INTERNAL_SERVER_ERROR,null);
    }
    return new Result<T>(resultStatus,data);
  }
}
```



### Result实体返回测试

```java
@RestController
@RequestMapping("/hello")
public class HelloController{
  private static final HashMap<String,Object> INFO;
  
  static{
    INFO = new HashMap<>();
    INFO.put("name","galaxy");
    INFO.put("age","70");
  }
  
  @GetMapping("/hello")
  public Map<String,Object>hello(){
    return INFO;
  }
  
  @GetMapping("/result")
  @ResponseBody
  public Result<Map<String,Object>> helloResult(){
    return Result.success(INFO);
  }
}
```

