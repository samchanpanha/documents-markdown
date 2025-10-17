# Build a home asset management platform with NAS!

**Original** **kiwi007** [KIWIWIKI](javascript:void(0);)

 *September 28, 2025, 20:20*

### preface

**This need stems from a previous experience. One day, when I was ready to wash the dishes after dinner, I found that the washing block in the cupboard was gone, and I later found no inventory in the various lockers in my home. But I remember seeing a few boxes in stock the previous month. Another thing is that I was preparing to apply with Fortune to replace the stove brackets, but found that the serial code of the equipment was not recorded, so the application was not successful in time. I need to use my home NAS to deploy a home asset (device) management platform to manage these device consumables records, so that I can easily view and call at any time.**
Looking for a few days on the Internet, I saw the bloggers recommended two software, one is homebox, one is grocy, I am both deployed, with a few days, personally prefer homebox (visual party). But homebox has two missing, one is no Chinese interface, one is no mobile APP. The problem with this Chinese interface has been searching online for a few days, and finally found a project with Chinese, so share it here.
Project address: https://github.com/sysadminsmedia/homebox
**Special thanks to: sysadminsmedia**

**The deployment environment is still the Green Union DXP4800**

### Deployment Process

**1. New Homebox Paper Clip**
Create a homebox folder in the Docker folder of the NAS as a project directory
![Writing Clip.png](https://mmbiz.qpic.cn/sz_mmbiz_png/NqDgAy6bPVia9xaewicwAzUlaAibZZPupOCRKqL4rQjymGo6hRtj3ZUvEBD7TJYbajepWe8wNo7xv4keicibgPx3sag/640?wx_fmt=png&from=appmsg)

**2. Building a project on Docker**
In Docker- Projects - Creating Projects
![Project.png](https://mmbiz.qpic.cn/sz_mmbiz_png/NqDgAy6bPVia9xaewicwAzUlaAibZZPupOC1fGbLQwvToXS8hSW4n77MBVKvDzgUANNn91hE7mRmqjpfz8FduGkfw/640?wx_fmt=png&from=appmsg)

**The docker-compose code is as follows:**

<pre class="js_darkmode__12 js_darkmode__text__29"><section><span class="js_darkmode__13"></span><span class="js_darkmode__14"></span><span class="js_darkmode__15"></span></section><section class="js_darkmode__text__30"><code class="js_darkmode__text__31"><span leaf="" class="js_darkmode__text__32">services:</span><span leaf=""><br/></span><span leaf="" class="js_darkmode__text__33">  homebox:</span><span leaf=""><br/></span><span leaf="" class="js_darkmode__text__34">    image: sysadminsmedia/homebox:latest</span><span leaf=""><br/></span><span leaf="" class="js_darkmode__text__35">    container_name: homebox</span><span leaf=""><br/></span><span leaf="" class="js_darkmode__text__36">    restart: always</span><span leaf=""><br/></span><span leaf="" class="js_darkmode__text__37">    environment:</span><span leaf=""><br/></span><span leaf="" class="js_darkmode__text__38">    - HBOX_LOG_LEVEL=info # 日志级别</span><span leaf=""><br/></span><span leaf="" class="js_darkmode__text__39">    - HBOX_LOG_FORMAT=text # 日志格式</span><span leaf=""><br/></span><span leaf="" class="js_darkmode__text__40">    - HBOX_WEB_MAX_UPLOAD_SIZE=10 # 最大上传文件大小10MB</span><span leaf=""><br/></span><span leaf="" class="js_darkmode__text__41">    volumes:</span><span leaf=""><br/></span><span leaf="" class="js_darkmode__text__42">      - ./:/data/ # 冒号左侧可自定义</span><span leaf=""><br/></span><span leaf="" class="js_darkmode__text__43">    ports:</span><span leaf=""><br/></span><span leaf="" class="js_darkmode__text__44">      - 3100:7745 # 冒号左侧端口号可自定义修改</span><span leaf=""><br/></span></code></section></pre>

**We talked about Docker acceleration before, and under normal circumstances, the project is built in minutes.**

**3. Reverse proxy**
Go lucky to do a reverse agent, the web address agent out, in order to be able to access the external network, there is no need to use http://nasip:3100 Direct access.
![Reverse Agent.png](https://mmbiz.qpic.cn/sz_mmbiz_png/NqDgAy6bPVia9xaewicwAzUlaAibZZPupOCGQarGMvhAXVoNJdrbz78RrM1Xfj7wv8B7yN2YaOhpribgmtpptib0PyQ/640?wx_fmt=png&from=appmsg)

**4. External Internet access**
Use the external network you set up to access, register and log in.
![Register.png](https://mmbiz.qpic.cn/sz_mmbiz_png/NqDgAy6bPVia9xaewicwAzUlaAibZZPupOCk2gBmp22tEFs4tsfXibYsAZeQWhut24kAm3kthtWtliaAt0ibc32Uicjng/640?wx_fmt=png&from=appmsg)![Home 2.png](https://mmbiz.qpic.cn/sz_mmbiz_png/NqDgAy6bPVia9xaewicwAzUlaAibZZPupOC3oTu8rkn86R31sEK3OKQoqkH4LZkJicfX7vYCWpQia5CfY2Ia8Ffd2Ow/640?wx_fmt=png&from=appmsg)

**There are a lot of beautiful themes in the user settings, and you can choose what you like to set up. The operation of the whole project was relatively simple and efficient. The Chinese interface is not very well explained, and everyone can explore it on their own.**
![](https://mmbiz.qpic.cn/sz_mmbiz_png/NqDgAy6bPVia9xaewicwAzUlaAibZZPupOCUIVw6L75MEHNHnOf8oPkXLqq7j3bD5KcXcnhQhNR1cIr7JKmiah9AXQ/640?wx_fmt=png&from=appmsg)

**The device's page can not only enter information such as model, price, brand, serial number and so on, but also upload pictures, instructions, invoices and other accessories, which are very comprehensive.**
Users with this demand quickly deploy a homebox, establish a private asset management platform, and register and manage the equipment and consumables at home. You are also welcome to leave a message to discuss, recommend more projects, or whether there is a homebox app.
