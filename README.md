package com.zkn.imitate.tomcat.secondchapter.first;
 
import com.zkn.imitate.tomcat.secondchapter.Request;
import com.zkn.imitate.tomcat.secondchapter.Response;
import com.zkn.imitate.tomcat.secondchapter.StaticResourceProcessor;
import com.zkn.imitate.tomcat.utils.Constants;
import com.zkn.imitate.tomcat.utils.StringUtil;
 
import java.io.IOException;
import java.net.InetAddress;
import java.net.ServerSocket;
import java.net.Socket;
 
/**
 * Created by wb-zhangkenan on 2016/12/29.
 */
public class HttpServer {
 
    public static void main(String[] args){
 
        await();
    }
 
    private static void await() {
 
        ServerSocket serverSocket = null;
        try {
            boolean shutDown = false;
            //创建一个服务端
            serverSocket = new ServerSocket(8004,1, InetAddress.getByName("127.0.0.1"));
            while (!shutDown){
                //接收客户端请求
                Socket socket = serverSocket.accept();
                Request request = new Request(socket.getInputStream());
                request.parseRequest();//解析请求信息
                Response response = new Response(socket.getOutputStream());
                String uri = request.getUri();
                if(uri !=null && uri.startsWith("/favicon.ico")){
 
                }else if(!StringUtil.isEmpty(uri) && uri.startsWith("/static/")){
                    StaticResourceProcessor resouce = new StaticResourceProcessor();
                    resouce.process(request,response);//处理静态资源
                }else{
                    ServletProcessor servletProcessor = new ServletProcessor();
                    servletProcessor.process(request,response);//处理Servlet
                }
                socket.close();
                shutDown = Constants.SHUT_DOWN.equals(request.getUri());
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

public class Request implements ServletRequest {
    /**
     * 输入流
     */
    private InputStream inputStream;
    /**
     * uri
     */
    private String uri;
 
    public Request(InputStream inputStream) {
        this.inputStream = inputStream;
    }
 
    public void parseRequest(){
 
        BufferedReader br = new BufferedReader(new InputStreamReader(inputStream));
        StringBuffer sb = new StringBuffer();
        String str = null;
        try {
            while((str = br.readLine()) != null){
                if("".equals(str))
                    break;
                sb.append(str).append("\n");
            }
            System.out.println(sb.toString());
            uri = StringUtil.parserUri(sb.toString()," ");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

public class Response implements ServletResponse {
    /**
     * 输出流
     */
    private OutputStream outputStream;
    /**
     * 字符输出流
     */
    private PrintWriter printWriter;
 
    public Response(OutputStream outputStream) {
        this.outputStream = outputStream;
    }
 
    public void sendStaticResource(String path) {
 
        FileInputStream fis = null;
        try {
            File file = new File(Constants.WEB_PATH, path);
            if (file.exists() && !file.isDirectory()) {
                if (file.canRead()) {
                    fis = new FileInputStream(file);
                    int flag = 0;
                    byte[] bytes = new byte[1024];
                    while ((flag = fis.read(bytes)) != -1){
                        outputStream.write(bytes);
                    }
                }
            }else{
                PrintWriter printWriter = getWriter();
                //这里用PrintWriter字符输出流，设置自动刷新
                printWriter.write("HTTP/1.1 404 File Not Found \r\n");
                printWriter.write("Content-Type: text/html\r\n");
                printWriter.write("Content-Length: 23\r\n");
                printWriter.write("\r\n");
                printWriter.write("<h1>File Not Found</h1>");
                printWriter.close();
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if(fis != null)
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
        }
    }

public class StaticResourceProcessor {
 
    public void process(Request request,Response response){
 
        response.sendStaticResource(request.getUri());
    }
}
public class FirstServlet implements Servlet{
 
    public void init(ServletConfig servletConfig) throws ServletException {
 
    }
 
    public ServletConfig getServletConfig() {
        return null;
    }
 
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
 
        PrintWriter out = servletResponse.getWriter();
        out.println("Hello. Roses are red.");
    }
 
    public String getServletInfo() {
        return null;
    }
 
    public void destroy() {
 
    }
}
package com.zkn.imitate.tomcat.secondchapter.first;
 
import com.zkn.imitate.tomcat.secondchapter.Request;
import com.zkn.imitate.tomcat.secondchapter.Response;
import com.zkn.imitate.tomcat.utils.Constants;
import com.zkn.imitate.tomcat.utils.StringUtil;
 
import javax.servlet.Servlet;
import javax.servlet.ServletException;
import java.io.File;
import java.io.IOException;
import java.net.URL;
import java.net.URLClassLoader;
import java.net.URLStreamHandler;
 
/**
 * Created by wb-zhangkenan on 2016/12/29.
 */
public class ServletProcessor {
    /**
     * 处理请求信息
     * @param request
     * @param response
     */
    public void process(Request request, Response response){
 
        String str = request.getUri();
        String servletName = null;
        if(!StringUtil.isEmpty(str) && str.lastIndexOf("/") >= 0){
            servletName = str.substring(str.lastIndexOf("/")+1);
        }
        URLClassLoader classLoader = null;
        URL[] url = new URL[1];
 
        URLStreamHandler streamHandler = null;
        File classPath = new File(Constants.WEB_ROOT);
 
        try {
            //创建仓库
            String repository = (new URL("file", null, classPath.getCanonicalPath() + File.separator)).toString();
            url[0] = new URL(null,repository,streamHandler);
            classLoader = new URLClassLoader(url);//URL类加载器
 
            Class clazz = classLoader.loadClass(servletName);
            Servlet servlet = (Servlet) clazz.newInstance();
            servlet.service(request,response);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (ServletException e) {
            e.printStackTrace();
        }
    }
}
