---
title: 网易云的MV下载
date: 2019-05-03 11:35:30
toc: true
categories: 工具
---

最近想把自己喜欢的MV放在个人站点上面，但是Mac客户端和网页版的网易云连个下载按钮都没有！好吧， 是时候自己动手…丰衣足食了，网页上能播放出来的肯定是能拿到的，最神奇最稳定的办法当然是开启录屏软件啦。但是本次为了播放效果，能不用最低端的方式就尽量不用啦！



很久没写过IO方面的东西了，现在拿这个例子来练练手：

![](https://s2.ax1x.com/2019/05/03/ENoX11.png)

我们要拿到的无非就是HTTP的请求链接和请求头信息：

![](https://s2.ax1x.com/2019/05/03/ENTenf.png)

```java
public static void main(String[] args) throws Exception{
        //请输入视频地址
        String url = "https://vodkgeyttp8.vod.126.net/cloudmusic/MCQ4IjAxICAwICEhICAgIQ==/294001/672888e4902fac1298f8e984182690c7.mp4?wsSecret=809e59d2b289d82917dedef9c05c64b2&wsTime=1556853453";
        
        File file = new File("/Users/tim/Desktop/a"+".mp4");
        //创建文件
        if(!file.createNewFile()) throw new RuntimeException("文件创建失败");
        
        HttpURLConnection con;
        FileOutputStream fs = null;
        InputStream is;
        BufferedInputStream bs = null;
        
        try {
            con = (HttpURLConnection) new URL(url).openConnection();
            con.setRequestProperty("User-Agent", "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.131 Safari/537.36");

            //输入流
            is = con.getInputStream();
            bs = new BufferedInputStream(is);
            fs = new FileOutputStream(file);
            byte[] bytes = new byte[1024 * 10];

            int line ;
            while((line = bs.read(bytes))!= -1){
                fs.write(bytes, 0, line);
                fs.flush();
            }

        } catch (Exception e) {
            e.printStackTrace();
        } finally{
            if(fs!= null){
                try {
                    fs.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(bs!=null){
                try {
                    bs.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

好了，首页的MV就是这样完成了下载，所以想要下载MV就是这么简单！	

![](https://s2.ax1x.com/2019/05/03/ENT6u6.png)