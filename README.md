# springMvc2_upload
스프링 MVC 2편 - 백엔드 웹 개발 활용 기술 - 파일 업로드

# 파일업로드

HTML Form을 통한 파일 업로드는 아래와 같이 2가지 방식이있다.

1. application/x-www-urlencoded
2. multipart/form-data



### 1. application/x-www-urlencoded

 HTML 폼 데이터를 서버로 전송하는 가장 기본적인 방법이다.

<img width="644" alt="image-20220508135753815" src="https://user-images.githubusercontent.com/58017318/196014314-7400d514-7cf8-4dce-b48b-445cfd2de381.png">


**파일을 업로드** 하려면 파일은 문자가 아니라 **바이너리 데이터**를 **전송**해야 한다. <br>

그렇다면 아래와같이 text와 첨부파일을 전송하려면 어떻게 해야할까?

여기에서 이름과 나이도 전송해야 하고, 첨부파일도 함께 전송해야 한다. 문제는 **이름과 나이는 문자**로 전송하고, **첨부파일은 바이너리**로 전송해야 한다는 점이다. 여기에서 문제가 발생한다. **문자와 바이너리를 동시에 전송**해야 하는 상황이다.

```java
- 이름
- 나이
- 첨부파일
```



이 문제를 해결하기 위해 HTTP는 **multipart/form-data** 라는 전송 방식을 제공한다.

<img width="934" alt="image-20220508140132517" src="https://user-images.githubusercontent.com/58017318/196014317-bdd7d9bf-ec33-4ca5-8205-5f03f8ab1b2c.png">
multipart/form-data 방식은 **다른 종류**의 **여러 파일과 폼의 내용** 함께 **전송**할 수 있다. (그래서 이름이 multipart 이다.)


<img width="588" alt="image-20220508140252411" src="https://user-images.githubusercontent.com/58017318/196014319-acf3af9c-413a-4835-b780-a990c97e9da3.png">
폼의 입력 결과로 생성된 HTTP 메시지를 보면 각각의 전송 항목이 구분이 되어있다. <br>**Content-Disposition** 이라는 항목별 헤더가 추가되어 있고 여기에 부가 정보가 있다. <br>예제에서는 username , age , file1 이 각각 분리되어 있다.

	- 폼의 일반 데이터는 각 항목별로 문자가 전송되고
	- 파일의 경우 파일 이름과 **Content-Type**이 추가되고 **바이너리 데이터**가 전송된다.



-----

# 서블릿과 파일 업로드 1

```java
@Slf4j
@Controller
@RequestMapping("/servlet/v1")
public class ServletUploadControllerV1 {
  
    @GetMapping("/upload")
    public String newFile() {
        return "upload-form";
		}
}
```

```html
<form th:action method="post" enctype="multipart/form-data">
  <ul>
    <li>상품명 <input type="text" name="itemName"></li>
    <li>파일<input type="file" name="file" ></li> </ul>
  <input type="submit"/>
</form>
```

```properties
# application.properties
# 이 옵션을 사용하면 HTTP 요청 메시지를 확인할 수 있다.
logging.level.org.apache.coyote.http11=debug
```

<img width="297" alt="image-20220508163338874" src="https://user-images.githubusercontent.com/58017318/196014320-08f05bfb-67a7-4426-92c0-a6c16778e8b0.png">

**enctype="multipart/form-data"** 로 선언한 후, 데이터를 전송해보면 아래와 같이 찍힌다.

<img width="1139" alt="image-20220508164415085" src="https://user-images.githubusercontent.com/58017318/196014322-efb0f380-5459-4bc2-be67-72dd53cd3e9c.png">
> - **boundary**로 part들의 데이터가 전송되었음을 확인할 수 있다.
>
> - Content-Disposition: form-data; name="변수명"으로 찍히는데<br>file의 경우에는 **filename**과 **Content-Type**까지 함께 찍힌다.



.. 생략...

<img width="1178" alt="image-20220508164513305" src="https://user-images.githubusercontent.com/58017318/196014323-4322c825-dac9-49f5-9c33-a27e5af612b9.png">
```java
@PostMapping("/upload")
public String saveFileV1(HttpServletRequest request) throws ServletException, IOException {
  
  // request=org.springframework.web.multipart.support.StandardMultipartHttpServletRequest@3c3e6814
  // StandardMultipart 리퀘스트가 찍힌다.
  log.info("request={}", request);
  
  String itemName = request.getParameter("itemName");
  
  // itemName=상품명
  log.info("itemName={}", itemName);

  Collection<Part> parts = request.getParts();
  
  // parts=[org.apache.catalina.core.ApplicationPart@6b3c801e, org.apache.catalina.core.ApplicationPart@dc8954c]
  // 객체가 2개가 찍힌다.
  log.info("parts={}", parts);

  return "upload-form";
}
```



참고)

##### 업로드 사이즈 제한

```properties
#파일 하나의 최대 사이즈, 기본 1MB
spring.servlet.multipart.max-file-size=1MB 

# 멀티파트 요청 하나에 여러 파일을 업로드 할 수 있는데, 그 전체 합 기본 10MB
spring.servlet.multipart.max-request-size=10MB
```



##### **spring.servlet.multipart.enabled** 

##### spring.servlet.multipart.enabled=false

```log
request=org.apache.catalina.connector.RequestFacade@xxx
itemName=null
parts=[]
```

> spring.servlet.multipart.enabled 옵션을 끄면 서블릿 컨테이너는 멀티파트와 관련된 처리를 하지 않는다.<br>
> 그래서 결과 로그를 보면 request.getParameter("itemName") , request.getParts() 의 결과가 비어있다.



##### spring.servlet.multipart.enabled=false

> ```log
> request=org.springframework.web.multipart.support.StandardMultipartHttpServletRequest
> itemName=Spring
> parts=[ApplicationPart1, ApplicationPart2]
> ```

> request.getParameter("itemName") 의 결과도 잘 출력되고, request.getParts() 에도 요청한 두 가지 멀티파트의 부분 데이터가 포함된 것을 확인할 수 있다. 이 옵션을 켜면 복잡한 멀티파트 요청을 처리해서 사용할 수 있게 제공한다.
>
> 로그를 보면 HttpServletRequest 객체가 <br>RequestFacade => StandardMultipartHttpServletRequest 로 변한 것을 확인할 수 있다.

-------

# 서블릿과 파일 업로드2



application.properties에 아래 내용을 추가하자

```
file.dir=/Users/geumbit/study/springmvcPart2/file/
```

1. 꼭해당경로에실제폴더를미리만들어두자.
2. application.properties 에서 설정할 때 마지막에 / (슬래시)가 포함된 것에 주의하자.



또한, Controller에서 **file.dir**값을 가져와보자

```
@Value("${file.dir}")
private String fileDir;
```



Controller

```java
@GetMapping("/upload")
public String newFile(){
  return "upload-form";
}

@PostMapping("/upload")
public String saveFile1(HttpServletRequest request) throws ServletException, IOException {
  log.info("request={}", request);

  String itemName = request.getParameter("itemName");
  log.info("itemName={}", itemName);

  Collection<Part> parts = request.getParts();
  log.info("parts={}", parts);

  for (Part part : parts) { // multipart/form-data 안에 또 part별 header가 존재한다.
    log.info("==== PART ====");
    log.info("name={}", part.getName());

    Collection<String> headerNames = part.getHeaderNames();
    for (String headerName : headerNames) {
      log.info("header{}: {}", headerName, part.getHeader(headerName));
    }

    // 편의메서드
    //Content-Disposition
    log.info("submittedFilename={}", part.getSubmittedFileName());
    log.info("size={}", part.getSize());

    // 데이터 읽기
    InputStream inputStream = part.getInputStream();
    String body = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
    log.info("body={}", body);
    
    // 파일에 저장하기
    if(StringUtils.hasText(part.getSubmittedFileName())){
      String fullPath = fileDir + part.getSubmittedFileName();
      log.info("파일저장 fullPath={}", fullPath);
      part.write(fullPath);
    }
  }

  return "upload-form";
}
```



<img width="826" alt="image-20220508174651149" src="https://user-images.githubusercontent.com/58017318/196014650-398c0ccb-ef46-4e39-854b-3956ed1196d8.png">

공통

| 함수                              | 내용                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| request                           | request=org.springframework.web.multipart.support.StandardMultipartHttpServletRequest@6b15a6c |
| request.getParameter("itemName"); | SangPumName                                                  |
| request.getParts();               | [org.apache.catalina.core.ApplicationPart@66042cbe<br>, org.apache.catalina.core.ApplicationPart@1f4a5db3] |



```java
for (Part part : parts) 
```

| 함수                                                         | itemName                                              | File                                                         |
| ------------------------------------------------------------ | ----------------------------------------------------- | ------------------------------------------------------------ |
| part.getName()                                               | itemName                                              | file                                                         |
| log.info("header{}: {}", headerName, part.getHeader(headerName)) | headercontent-disposition: form-data; name="itemName" | headercontent-disposition: form-data; name="file"; filename="Unknown.jpeg" |
| part.getSubmittedFileName()                                  | submittedFilename=null                                | submittedFilename=Unknown.jpeg                               |
| log.info("size={}", part.getSize());                         | size=11                                               | size=59426                                                   |
| StreamUtils.copyToString(part.getInputStream(), StandardCharsets.UTF_8); | SangPumName                                           | ����JFIFHH��C~~~~~                                           |



**파일저장된 결과**
<img width="354" alt="image-20220508175111964" src="https://user-images.githubusercontent.com/58017318/196014651-0656b50e-ffbe-4eae-921d-3d50968da076.png">





**Part 주요 메서드**

- part.getSubmittedFileName() : 클라이언트가 전달한 파일명 
- part.getInputStream(): Part의 전송 데이터를 읽을 수 있다. 
- part.write(...): Part를 통해 전송된 데이터를 저장할 수 있다.

------

# 스프링과 파일 업로드

스프링은 **MultipartFile**이라는 인터페이스로 멀티파트 파일을 매우 편리하게 지원한다.

```java
// Controller에서 @RequestParam을 사용하여 MultipartFile Type으로 파일을 받을 수 있다.
@RequestParam MultipartFile file
```

```java
// file로 파일명을 바로 받을 수 있다.
file.getOriginalFilename()
```

```java
// 파일을 업로드 할 수 있다.
file.transferTo(new File(fullPath));
```



완성

```java
@Slf4j
@Controller
@RequestMapping("/spring")
public class SpringUploadController {
    @Value("${file.dir}")
    private String fileDir;

    @GetMapping("/upload")
    public String newFile(){
        return "upload-form";
    }

    @PostMapping("/upload")
    public String saveFile(@RequestParam String itemName,
                           @RequestParam MultipartFile file,
                           HttpServletRequest request) throws IOException {
        log.info("request={}", request);
        log.info("itemName={}", itemName);
        log.info("MultipartFile={}", file);

        if(!file.isEmpty()){
            String fullPath = fileDir + file.getOriginalFilename();
            log.info("파일 저장 fullPath={}", fullPath);
            file.transferTo(new File(fullPath));
        }

        return "upload-form";
    }
}
```







