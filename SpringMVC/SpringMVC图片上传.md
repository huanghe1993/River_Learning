# SpringMVC图片上传与下载

[TOC]



## 1、SpringMVC上传图片在同一个服务器上

传统的项目图片是存放在Web工程的webRoot目录下的image文件夹下面的

- 好处是可以在jsp页面进行直接的访问，图片太多可以在Linux下进行映射盘符

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180505/j3DAHFhAcK.png?imageslim)

### 1.1、使用multipulate的方式

图片直接的上传到一个服务器上进行

```java
@Controller
public class UploadController {

    @RequestMapping(value = "/pic/upload")
    @ResponseBody
    public PictureResult imgUpload(@RequestParam MultipartFile uploadFile, HttpServletRequest request) throws IOException {

        //如果文件不为空，写入上传路径
        if(!uploadFile.isEmpty()) {
            //上传文件路径
            String path = request.getSession().getServletContext().getRealPath("static")+ File.separator+"images";
            //上传文件名
            String filename = uploadFile.getOriginalFilename();
            File filepath = new File(path,filename);
            //判断路径是否存在，如果不存在就创建一个
            if (!filepath.getParentFile().exists()) {
                filepath.getParentFile().mkdirs();
            }
            //  使用transferTo（dest）方法将上传文件写到服务器上指定的文件
            uploadFile.transferTo(new File(path + File.separator + filename));
            PictureResult result=new PictureResult();
            result.setError(0);
            result.setUrl("/static/images/"+filename);
            return result;
        } else {
            PictureResult result=new PictureResult();
            result.setError(1);
            result.setUrl(null);
            result.setMessage("上传失败");
            return result;
        }

    }

}
```

```java
public class PictureResult {

    private int error;
    private String url;
    private String message;

    public int getError() {
        return error;
    }

    public void setError(int error) {
        this.error = error;
    }

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

}

```

### 1.2、使用对象接收上传文件

​	在实际的项目中，很多时候上传的文件会作为对象的属性被保存。SpringMVC的处理也是非常简单的

```html
<html>
<head>
  <title>用户注册</title>
</head>
  <body>
    <h2>用户注册</h2>
    <form action="register" enctype="multipulate/form-data" method="post">
      <table>
        <tr>
        <td>用户名<td>
        <td><input type="text" name="username"/> 
        </tr>
        <tr>
        <td>请上传头像<td>
        <td><input type="file" name="image"/> 
        </tr>
        <td><input type="submit" value="注册"/> 
        </tr>
      </table>      
      </form>
  </body>
</html>
```

```java
public class User implements Serializable{
  private String username;
  private MultipartFile image;
}
```

```java
@RequestMapping("/register")
public String register(HttpServletRequest request,User user){
   //如果文件不为空，写入上传路径
        if(!uploadFile.isEmpty()) {
            //上传文件路径
            String path = request.getSession().getServletContext().getRealPath("static")+ File.separator+"images";
            //上传文件名
            String filename = uploadFile.getOriginalFilename();
            File filepath = new File(path,filename);
            //判断路径是否存在，如果不存在就创建一个
            if (!filepath.getParentFile().exists()) {
                filepath.getParentFile().mkdirs();
            }
            //  使用transferTo（dest）方法将上传文件写到服务器上指定的文件
            user.getImage().transferTo(new File(path + File.separator + filename));
            PictureResult result=new PictureResult();
            result.setError(0);
            result.setUrl("/static/images/"+filename);
            return result;
        } else {
            PictureResult result=new PictureResult();
            result.setError(1);
            result.setUrl(null);
            result.setMessage("上传失败");
            return result;
        }
}
```

## 2、多张图片的上传

```HTML
<body>  
<form action="${basePath}file/upload" method="post" enctype="multipart/form-data">  
    <label>用户名：</label><input type="text" name="name"/><br/>  
    <label>密 码：</label><input type="password" name="password"/><br/>  
    <label>头 像1</label><input type="file" name="file"/><br/>  
    <label>头 像2</label><input type="file" name="file"/><br/>  
    <input type="submit" value="提  交"/>  
</form>  
</body>  
```

```HTML
展示图片来个循环，以便显示多张图片
<body>  
<c:forEach items="${imagesPathList}" var="image">  
    <img src="${basePath}${image}"><br/>  
</c:forEach>  
</body>  
```

控制层代码如下：

```java
@Controller  
@RequestMapping("/file")  
public class FileUploadController {  
      
    @RequestMapping(value="/upload",method=RequestMethod.POST)  
    private String fildUpload(Users users ,@RequestParam(value="file",required=false) MultipartFile[] file,  
            HttpServletRequest request)throws Exception{  
        //基本表单  
        System.out.println(users.toString());  
          
        //获得物理路径webapp所在路径  
        String pathRoot = request.getSession().getServletContext().getRealPath("");  
        String path="";  
        List<String> listImagePath=new ArrayList<String>();  
        for (MultipartFile mf : file) {  
            if(!mf.isEmpty()){  
                //生成uuid作为文件名称  
                String uuid = UUID.randomUUID().toString().replaceAll("-","");  
                //获得文件类型（可以判断如果不是图片，禁止上传）  
                String contentType=mf.getContentType();  
                //获得文件后缀名称  
                String imageName=contentType.substring(contentType.indexOf("/")+1);  
                path="/static/images/"+uuid+"."+imageName;  
                mf.transferTo(new File(pathRoot+path));  
                listImagePath.add(path);  
            }  
        }  
        System.out.println(path);  
        request.setAttribute("imagesPathList", listImagePath);  
        return "success";  
    }  
    //因为我的JSP在WEB-INF目录下面，浏览器无法直接访问  
    @RequestMapping(value="/forward")  
    private String forward(){  
        return "index";  
    }  
}  
```

## 3、文件下载

### 基于ResponseEntity实现

```java
@RequestMapping("/testHttpMessageDown")
public ResponseEntity<byte[]> download(HttpServletRequest request,@RequestParam("filename") String filename) throws IOException {
  	//下载文件路径
    String path=request.getSession().getServletContext().getRealPath("static")+ File.separator+"images";
    File file = new File(path+File.separator+filename);
    HttpHeaders headers = new HttpHeaders();
  	//下载显示的文件名，解决中文乱码问题
  	String downloadFielName=new String(file.getBytes("UTF-8"),"iso-8859-1"));
  	//通知浏览器以attachment(下载方式)打开图片
  	headers.setContentDispositionFormData("attchement",downloadFielName);
  	//application/octet-stream:二进制流数据
  	headers.setContentType(MediaType.APPLICATION_OCTET_STREAM)
    // HttpStatus.CREATED  201状态码
    ResponseEntity<byte[]> entity = new ResponseEntity<byte[]>(FileUtils.readFileToByteArray(file), headers, HttpStatus.CREATED);
    return entity;
}
```

## 4、分布式环境下图片上传到图片服务器

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180505/JI78bLl74A.png?imageslim)

需要安装nginx和配置ftp服务：

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180505/7EEEbaFmbf.png?imageslim)

使用java代码访问ftp服务器

使用apache的FTPClient工具访问ftp服务器。需要在pom文件中添加依赖

```xml
<dependency>
				<groupId>commons-net</groupId>
				<artifactId>commons-net</artifactId>
				<version>${commons-net.version}</version>
</dependency>

```

测试代码：

```java
public class FTPClientTest {

	@Test
	public void testFtp() throws Exception {
		//1、连接ftp服务器
		FTPClient ftpClient = new FTPClient();
		ftpClient.connect("192.168.25.133", 21);
		//2、登录ftp服务器
		ftpClient.login("ftpuser", "ftpuser");
		//3、读取本地文件
		FileInputStream inputStream = new FileInputStream(new File("G:\\web\\test.jpg"));
		//4、上传文件
		//1）指定上传目录,远程站点的路径（/home/ftpuser/www代表的就是192.168.25.133）
		ftpClient.changeWorkingDirectory("/home/ftpuser/www/images");
		//2）指定文件类型
		ftpClient.setFileType(FTPClient.BINARY_FILE_TYPE);
		//第一个参数：文件在远程服务器的名称
		//第二个参数：文件流
		ftpClient.storeFile("hello.jpg", inputStream);
		//5、退出登录
		ftpClient.logout();
	}
}
```

结果显示：

![mark](http://ozxf77u6w.bkt.clouddn.com/blog/180505/4bgJ2IJBe8.png?imageslim)

使用工具类的测试代码：

```java
@Test
	public void ftpUtilTest() throws Exception{
		FileInputStream inputStream = new FileInputStream(new File("G:\\web\\test.jpg"));
		String host="192.168.25.133";
		int port=21;
		String username="ftpuser";
		String password="ftpuser";
		String basePath="/home/ftpuser/www/images";
		String filePath="/2018/04/23";
		String filename="hello1.jpg";
		InputStream input=inputStream;
		FtpUtil.uploadFile(host, port, username, password, basePath, filePath, filename, input);
	}
```

**SpringMVC中使用ftp进行图片上传**

ftp服务器配置文件：

```properties
#ftp
FTP_ADDRESS=192.168.25.133
FTP_PORT=21
FTP_USERNAME=ftpuser
FTP_PASSWORD=ftpuser
FTP_BASE_PATH=/home/ftpuser/www/images


#picture
# server base url
IMAGE_BASE_URL=http://192.168.25.133/images
```

```java
@Service
public class PictureServiceImpl implements PictureService {

	@Value("${FTP_ADDRESS}")
	private String FTP_ADDRESS;
	
	@Value("${FTP_PORT}")
	private Integer FTP_PORT;
	
	@Value("${FTP_USERNAME}")
	private String FTP_USERNAME;
	
	@Value("${FTP_PASSWORD}")
	private String FTP_PASSWORD;
	
	//ftp文件存放的路径，其中www代表的就是根路径
	@Value("${FTP_BASE_PATH}")
	private String FTP_BASE_PATH;
	
	@Value("${IMAGE_BASE_URL}")
	private String IMAGE_BASE_URL;
	
	
	
	
	@Override
	public Map<String, Object> uploadPicture(MultipartFile uploadFile) {
		HashMap<String, Object> resultMap= new HashMap<>();
		//生成一个新的文件名
		//去原始文件的名字
		String oldName = uploadFile.getOriginalFilename();
		//生产新的文件名,使用UUID(或者是时间)，这里使用的是IDUtils
		String newName=IDUtils.genImageName();
		newName = newName + oldName.substring(oldName.lastIndexOf("."));
		//图片上传
		//上传的相对路径,这里需要导入Joda-time包
		String filePath=new DateTime().toString("/yyyy/MM/dd");
		//获取输入流
		InputStream input=null;
		try {
			input = uploadFile.getInputStream();
			boolean result = FtpUtil.uploadFile(FTP_ADDRESS, FTP_PORT, FTP_USERNAME, FTP_PASSWORD, FTP_BASE_PATH, filePath, newName, input);
			//返回结果(使用map)
			if (!result) {
				resultMap.put("error", 1);
				resultMap.put("message", "文件上传失败");
				return resultMap;
			}
			resultMap.put("error", 0);
			resultMap.put("url", IMAGE_BASE_URL+filePath+"/"+newName);
			return resultMap;
		}catch (IOException e) {
			resultMap.put("error", 1);
			resultMap.put("message", "文件上传发生异常");
			return resultMap;
		}
	}

}
```

Controller的代码：

功能：接收页面传递过来的图片。调用service上传到图片服务器。返回结果。

参数：MultiPartFileuploadFile

返回值：返回json数据，应该返回一个pojo，PictureResult对象

```java
/** 
* @author 作者 E-mail: River 735597346@qq.com
* @version 创建时间：2018年4月23日 下午10:01:57 
* 类说明:图片上传，接收MultiPartFile对象。调用Service上传图片返回jSON
*/
@Controller
public class PictureController {

	@Autowired
	private PictureService pictureService;
	
	
	/**
	 * 一个小的知识点，如果说是@ResponseBody返回的是对象直接的转化为json格式输出 Content-Type:Json，
	 * 如果返回的是字符串就直接的输出  Content-Type:text/plain
	 * @param uploadFile
	 * @return
	 */
	@RequestMapping("/pic/upload")
	@ResponseBody
	public String upload(MultipartFile uploadFile) {
		Map<String, Object> result = pictureService.uploadPicture(uploadFile);
		//为了保证兼容性，需要把Reult对象转化为Json格式的字符串
		String json = JsonUtils.objectToJson(result);
		return json;
	}
```



## 5、图片上传工具类：

图片文件命名的工具类：

```java
package com.taotao.common.utils;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;

import org.apache.commons.net.ftp.FTP;
import org.apache.commons.net.ftp.FTPClient;
import org.apache.commons.net.ftp.FTPFile;
import org.apache.commons.net.ftp.FTPReply;

/**
 * ftp上传下载工具类
 * <p>Title: FtpUtil</p>
 * <p>Description: </p>
 * <p>Company: www.itcast.com</p> 
 * @author	黄河
 * @date	2018年4月293日下午16:49:51
 * @version 1.0
 */
public class FtpUtil {

	/** 
	 * Description: 向FTP服务器上传文件 
	 * @param host FTP服务器hostname 
	 * @param port FTP服务器端口 
	 * @param username FTP登录账号 
	 * @param password FTP登录密码 
	 * @param basePath FTP服务器基础目录
	 * @param filePath FTP服务器文件存放路径。例如分日期存放：/2015/01/01。文件的路径为basePath+filePath
	 * @param filename 上传到FTP服务器上的文件名 
	 * @param input 输入流 
	 * @return 成功返回true，否则返回false 
	 */  
	public static boolean uploadFile(String host, int port, String username, String password, String basePath,
			String filePath, String filename, InputStream input) {
		boolean result = false;
		FTPClient ftp = new FTPClient();
		try {
			int reply;
			ftp.connect(host, port);// 连接FTP服务器
			// 如果采用默认端口，可以使用ftp.connect(host)的方式直接连接FTP服务器
			ftp.login(username, password);// 登录
			reply = ftp.getReplyCode();
			if (!FTPReply.isPositiveCompletion(reply)) {
				ftp.disconnect();
				return result;
			}
			//切换到上传目录
			if (!ftp.changeWorkingDirectory(basePath+filePath)) {
				//如果目录不存在创建目录
				String[] dirs = filePath.split("/");
				String tempPath = basePath;
				for (String dir : dirs) {
					if (null == dir || "".equals(dir)) continue;
					tempPath += "/" + dir;
					if (!ftp.changeWorkingDirectory(tempPath)) {
						if (!ftp.makeDirectory(tempPath)) {
							return result;
						} else {
							ftp.changeWorkingDirectory(tempPath);
						}
					}
				}
			}
			//设置上传文件的类型为二进制类型
			ftp.setFileType(FTP.BINARY_FILE_TYPE);
			//上传文件
			if (!ftp.storeFile(filename, input)) {
				return result;
			}
			input.close();
			ftp.logout();
			result = true;
		} catch (IOException e) {
			e.printStackTrace();
		} finally {
			if (ftp.isConnected()) {
				try {
					ftp.disconnect();
				} catch (IOException ioe) {
				}
			}
		}
		return result;
	}
	
	/** 
	 * Description: 从FTP服务器下载文件 
	 * @param host FTP服务器hostname 
	 * @param port FTP服务器端口 
	 * @param username FTP登录账号 
	 * @param password FTP登录密码 
	 * @param remotePath FTP服务器上的相对路径 
	 * @param fileName 要下载的文件名 
	 * @param localPath 下载后保存到本地的路径 
	 * @return 
	 */  
	public static boolean downloadFile(String host, int port, String username, String password, String remotePath,
			String fileName, String localPath) {
		boolean result = false;
		FTPClient ftp = new FTPClient();
		try {
			int reply;
			ftp.connect(host, port);
			// 如果采用默认端口，可以使用ftp.connect(host)的方式直接连接FTP服务器
			ftp.login(username, password);// 登录
			reply = ftp.getReplyCode();
			if (!FTPReply.isPositiveCompletion(reply)) {
				ftp.disconnect();
				return result;
			}
			ftp.changeWorkingDirectory(remotePath);// 转移到FTP服务器目录
			FTPFile[] fs = ftp.listFiles();
			for (FTPFile ff : fs) {
				if (ff.getName().equals(fileName)) {
					File localFile = new File(localPath + "/" + ff.getName());

					OutputStream is = new FileOutputStream(localFile);
					ftp.retrieveFile(ff.getName(), is);
					is.close();
				}
			}

			ftp.logout();
			result = true;
		} catch (IOException e) {
			e.printStackTrace();
		} finally {
			if (ftp.isConnected()) {
				try {
					ftp.disconnect();
				} catch (IOException ioe) {
				}
			}
		}
		return result;
	}
	
	public static void main(String[] args) {
		try {  
	        FileInputStream in=new FileInputStream(new File("D:\\temp\\image\\gaigeming.jpg"));  
	        boolean flag = uploadFile("192.168.25.133", 21, "ftpuser", "ftpuser", "/home/ftpuser/www/images","/2015/01/21", "gaigeming.jpg", in);  
	        System.out.println(flag);  
	    } catch (FileNotFoundException e) {  
	        e.printStackTrace();  
	    }  
	}
}
```

名称工具类：

```java
package com.taotao.common.utils;

import java.util.Random;

/** 
* @author 作者 E-mail: River 735597346@qq.com
* @version 创建时间：2018年4月23日 下午9:31:25 
* 类说明:各种id生成策略
*/
public class IDUtils {

	/**
	 * 图片名生成
	 */
	public static String genImageName() {
		//取当前时间的长整形值包含毫秒
		long millis = System.currentTimeMillis();
		//long millis = System.nanoTime();
		//加上三位随机数
		Random random = new Random();
		int end3 = random.nextInt(999);
		//如果不足三位前面补0
		String str = millis + String.format("%03d", end3);
		
		return str;
	}
	
	/**
	 * 商品id生成
	 */
	public static long genItemId() {
		//取当前时间的长整形值包含毫秒
		long millis = System.currentTimeMillis();
		//long millis = System.nanoTime();
		//加上两位随机数
		Random random = new Random();
		int end2 = random.nextInt(99);
		//如果不足两位前面补0
		String str = millis + String.format("%02d", end2);
		long id = new Long(str);
		return id;
	}
	
	public static void main(String[] args) {
		for(int i=0;i< 100;i++)
		System.out.println(genItemId());
	}
}


```