---
title: 一次简单的elasticsearch使用
date: 2019-10-30 14:52:29
categories: elasticsearch
# img: http://pzpoejx7j.bkt.clouddn.com/systemstructure.png
---

## System structure
![System structure](http://pzpoejx7j.bkt.clouddn.com/systemstructure.png)
+ First, main thread monitors the bif data folder backup. 
+ When the file directory is updated，the live thread of pool will work for unzipping zip file and extracting all XML files then resolving XML files to json files.
+ Put all json files to elasticsearch.
+ Visualize data of elasticsearch by Grafana.

## Monitor the folder
Use Java class: **WatchService**
<!--more-->
This class allows you to monitor changes to files in the operating system in real time, including creating, updating, and deleting events.

the core code:
```java
import lombok.extern.slf4j.Slf4j;
import micro.trend.wh.customerinsightsystem.service.MyWatchService;

import java.io.File;
import java.io.IOException;
import java.nio.file.*;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

@Slf4j
public class FileWatchTask implements Runnable {
    private String fileDirectory;
    private MyWatchService myWatchService;

    public FileWatchTask(String fileDirectory,MyWatchService myWatchService) {
        this.fileDirectory = fileDirectory;
        this.myWatchService=myWatchService;
    }

    public void run() {
        WatchService watchService = null;
        ExecutorService executorService=Executors.newFixedThreadPool(20);
        try {
            //获取当前文件系统的WatchService监控对象
            watchService = FileSystems.getDefault().newWatchService();
        } catch (IOException e) {
            log.error(e.getMessage());
        }
        try {
            //获取文件目录下的Path对象注册到 watchService中。
            //监听的事件类型，有创建，删除，以及修改
            Paths.get(fileDirectory)
                    .register(watchService, StandardWatchEventKinds.ENTRY_CREATE, StandardWatchEventKinds.ENTRY_DELETE,
                            StandardWatchEventKinds.ENTRY_MODIFY);
        } catch (IOException e) {
            log.error(e.getMessage());
        }
        while (true) {
            WatchKey key = null;
            try {
                //获取可用key.没有可用的就wait
                key = watchService.take();
            } catch (InterruptedException e) {
                log.error(e.getMessage());
            }
            for (WatchEvent<?> event : key.pollEvents()) {
                //todo
                System.out.println("the folder is updated!")
            }
            //重置，这一步很重要，否则当前的key就不再会获取将来发生的事件
            boolean valid = key.reset();
            //失效状态，退出监听
            if (!valid) {
                break;
            }
        }
    }
}
```
## Unzip File

core code:
```java
import lombok.extern.slf4j.Slf4j;

import java.io.*;
import java.nio.charset.Charset;
import java.util.Enumeration;
import java.util.zip.ZipEntry;
import java.util.zip.ZipFile;

@Slf4j
public class UnZip {
    public  static void unZipFiles(String zipPath,String dstPath) throws IOException{
        unZipFiles(new File(zipPath),dstPath);
    }
    /*
    * unzip file to specified path
    * @param zipFile: the file that is need to unzip
    * @param dstPath: destination of unzipping
    * */
    public static void unZipFiles(File zipFile,String dstPath) throws IOException {
        ZipFile zip=null;
        try{
            zip=new ZipFile(zipFile, Charset.forName("GBK"));// solving for chinese grabled
        }catch (FileNotFoundException e){
            try{
                Thread.currentThread().sleep(10000);
            }catch (InterruptedException sleepErr){
                sleepErr.printStackTrace();
                log.error(sleepErr.getMessage());
            }
            zip=new ZipFile(zipFile, Charset.forName("GBK"));// solving for chinese grabled
        }
        String name=zip.getName().substring(zip.getName().lastIndexOf('\\')+1,zip.getName().lastIndexOf('.'));

        File pathFile=new File(dstPath+name);
        if(!pathFile.exists()){
            pathFile.mkdirs();
        }

        for(Enumeration<? extends ZipEntry> entries=zip.entries();entries.hasMoreElements();){
            ZipEntry entry=(ZipEntry) entries.nextElement();
            String zipEntryName=entry.getName();
            InputStream in=zip.getInputStream(entry);
            String outPath=(dstPath+name+"/"+zipEntryName).replaceAll("\\*","/");
            // Judge whether the path exist
            File file=new File(outPath.substring(0,outPath.lastIndexOf('/')));
            if(!file.exists()){
                file.mkdirs();
            }
            //judge whether the full path is directory
            if(new File(outPath).isDirectory()){
                continue;
            }

            FileOutputStream out=new FileOutputStream(outPath);
            byte[] buf1=new byte[1024];
            int len;
            while((len=in.read(buf1))>0){
                out.write(buf1,0,len);
            }
            in.close();
            out.close();
        }
        log.info(zipFile.getName()+"unzipped finished!");
        return ;
    }
}
```

## Resolve XML file to Json file


core code:
```java
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.TypeReference;
import lombok.extern.slf4j.Slf4j;
import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;

import java.io.*;
import java.math.BigInteger;
import java.util.LinkedHashMap;
import java.util.Map;

@Slf4j
public class Dom4jResolveXml {
    String xmlDir=null;
    String jsonFileDir=null;

    public Dom4jResolveXml(String xmlDir, String jsonFileDir) {
        this.xmlDir = xmlDir;
        this.jsonFileDir=jsonFileDir;
    }

    public void resolveXML(){
        try{

            InputStream inputStream=new FileInputStream(xmlDir);
            SAXReader saxReader=new SAXReader();
            Document document=saxReader.read(xmlDir);
            Element bifElement=document.getRootElement();
//            System.out.println("root name:"+bifElement.getName());
//            System.out.println("root element attribute count:"+bifElement.attributeCount());
//            System.out.println("root element value:"+bifElement.attributeValue("bif"));
            Element productElement=bifElement.element("Product");
            Element trafficMatrixElement=productElement.element("TrafficMatrix");
            if(trafficMatrixElement==null){
                log.info("the xml file: "+xmlDir+" has no traffixMatrix node.");
                return ;
            }
            String trafficMatrixText=trafficMatrixElement.getTextTrim();
            Element acElement=productElement.element("AC");
            String acText=acElement.getTextTrim();
            Element guidElement=productElement.element("GUID");
            String guidText=guidElement.getText();
//            System.out.println(trafficMatrixText);
//            System.out.println("*************************");

            StringBuilder jsonSB=new StringBuilder(trafficMatrixText);

//            jsonSB.insert(1,"\"AC\":"+"\""+acText+"\","+"\"GUID\":\""+guidText+"\",\"cpuinfo\":{");
//            jsonSB.append("}");

            trafficMatrixText=jsonSB.toString();
            synchronized (this.getClass()){
                jsonResolve(trafficMatrixText,acText,guidText);
            }
//            System.out.println(trafficMatrixText.substring(0,1));
//            outJsonToFile(trafficMatrixText);
        }catch (FileNotFoundException e){
            e.printStackTrace();
        }catch (DocumentException e){
            e.printStackTrace();
        }
    }


    public void jsonResolve(String jsonOrigin,String acInfo,String guidInfo){
        String idTxtFileName=PropertiesUtils.getValueByKey("config.properties","idTxtFileName");
        String idStr=ReadAndWriteFile.readFile(idTxtFileName);
        BigInteger bigInteger=new BigInteger(idStr);

//        JSONObject jsonObject= JSON.parseObject(jsonOrigin);//disorder
        LinkedHashMap<String, String> jsonMap = JSON.parseObject(jsonOrigin, new TypeReference<LinkedHashMap<String, String>>() {
        });
        StringBuilder sb=new StringBuilder();
        JSONObject cpuObject=null;
        String jsonFileNameNOTDir=null;
        for(Map.Entry<String,String> entry:jsonMap.entrySet()){
//            System.out.println("key:"+entry.getKey()+" value:"+entry.getValue().toString());
//            {"index":{"_index":"acinfoindex","_id":5}}
            sb.append("{\"index\":{\"_index\":\"acinfoindex\",\"_id\":"+bigInteger+"}}\n");
            bigInteger=bigInteger.add(BigInteger.valueOf(1));
            sb.append("{");
            sb.append("\"AC\":\""+acInfo+"\",");
            sb.append("\"GUID\":\""+guidInfo+"\",");
            sb.append("\"TIME\":\""+entry.getKey()+"\",");
            sb.append("\"CPUINFO\":");
            sb.append("{");
            LinkedHashMap<String,String> cpuMap=JSON.parseObject(entry.getValue(),new TypeReference<LinkedHashMap<String, String>>(){});
            for(Map.Entry<String,String> cpuEntry:cpuMap.entrySet()){
//                System.out.println(cpuEntry.getKey()+"*******"+cpuEntry.getValue());
                if(cpuEntry.getValue()==null||cpuEntry.getValue().equals("")){
                    cpuEntry.setValue("0");
                }else{
                    sb.append("\""+cpuEntry.getKey()+"\":"+cpuEntry.getValue()+",");
                }
            }
            sb.deleteCharAt(sb.length()-1);
            sb.append("}");
//            sb.append(entry.getValue());
            sb.append("}");
            sb.append("\n");
        }
        jsonFileNameNOTDir=acInfo+"_"+guidInfo+bigInteger+".json";
        ReadAndWriteFile.writeFile(idTxtFileName,bigInteger.toString());
        outJsonToFile(sb.toString(),jsonFileNameNOTDir);
    }
    /*
     * output json string to file
     * */
    public void outJsonToFile(String jsonStr,String jsonFileNameNeNOTDir){
        byte[] buff=new byte[]{};

        FileOutputStream fileOutputStream=null;
        File file=new File(jsonFileDir+jsonFileNameNeNOTDir);
        if(!file.getParentFile().exists()){
            file.getParentFile().mkdirs();
        }
        try{
            buff=jsonStr.getBytes();
            fileOutputStream=new FileOutputStream(file);
            log.info("output file path:"+jsonFileDir);
            fileOutputStream.write(buff,0,buff.length);
            log.info("output json data to file success!");
            SSH2Util ssh2Util = new SSH2Util(PropertiesUtils.getValueByKey("config.properties","ip"), PropertiesUtils.getValueByKey("config.properties","usrName"),PropertiesUtils.getValueByKey("config.properties","pwd"), 22);
            ssh2Util.putFile(jsonFileDir, jsonFileNameNeNOTDir,PropertiesUtils.getValueByKey("config.properties","serverDir"));
        }catch (IOException e){
            log.error(e.getMessage());
        }catch (Exception e){
            log.error(e.getMessage());
        } finally {
            if(fileOutputStream!=null){
                try{
                    fileOutputStream.close();
                }catch (IOException eS){
                    log.error(eS.getMessage());
                }
            }
        }
    }
}
```
Beacuse the program need to record the index id, so I just write the id to the txt, so when build each json need to read the txt and increase the id. In order to speed up, multi-threaded data is used. We need to use lock avoiding open txt file failed.

## Elasticsearch
install guid:https://www.howtoing.com/how-to-install-elasticsearch-logstash-and-kibana-elastic-stack-on-centos-7

index:
```bash
curl -X PUT "localhost:9200/acinfoindex?pretty" -H 'Content-Type: application/json' -d'
{
  "mappings":{
    "info":{
      "properties":{
        "AC":{"type":"keyword"},
        "GUID":{"type":"keyword"},
        "TIME":{"type":"date",
          "format": "epoch_second"
          },
        "CPUINFO":{
            "properties":{
                "auth_simp_n":{"type": "integer"},
                "auth_ntlm_n":{"type": "integer"},
                "th_inc":{"type": "integer"},
                "th_net_in_traffic":{"type": "integer"},
                "auth_krb5_n":{"type": "integer"},
                "usg_mem":{"type": "integer"},
                "th_net_out_traffic":{"type": "integer"},
                "auth_succ_n":{"type": "integer"},
                "usg_disk_io":{"type": "integer"},
                "th_inc_http2":{"type": "integer"},
                "th_inc_https":{"type": "integer"},
                "th_inc_http":{"type": "integer"},
                "auth_n":{"type": "integer"},
                "usg_cpu":{"type": "integer"}
            }
            
        } 
    }
    }   
  }
}
'
```

and then put data to elasticsearch

```python
import os
import pyinotify

multi_event=pyinotify.IN_CREATE|pyinotify.IN_ACCESS|pyinotify.IN_CLOSE_WRITE
wm=pyinotify.WatchManager()

class MyEventHandler(pyinotify.ProcessEvent):
    def process_IN_CREATE(self,event):
        print('CREATE',event.pathname)
    def precess_IN_ACCESS(self,event):
        print('ACCESS',event.pathname)
    def process_IN_CLOSE_WRITE(self,event):
        print('CLOSE_WRITE',event.pathname)
        output = os.popen("curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/acinfoindex/info/_bulk?pretty' --data-binary @"+event.pathname)

handler=MyEventHandler()

notifier=pyinotify.Notifier(wm,handler)

wm.add_watch('/home/huanhuan/data',multi_event)
notifier.loop()
```

## Grafana
 refer to:

http://docs.flycloud.me/docs/ELKStack/elasticsearch/other/grafana.html

https://www.aiprose.com/blog/26
