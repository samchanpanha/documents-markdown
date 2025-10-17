# Docker Deployment Nginx Detailed Manual

**Original** **AIGap** [Source Code Jumping](javascript:void(0);)

 *October 4, 2025, 22:10*

**Before we begin the Nginx image pull and deployment operation, let's briefly clarify the core value of Nginx and the advantages of Docker deployment - this will help you more clearly understand the meaning of the subsequent operation.**

## About Nginx: Core Features and Values

**Nginx is a lightweight, high-performance HTTP server and reverse proxy server that is one of the most commonly used components of enterprise Web services today. Its core role can be summarized in four broad categories:**

* **·**  **Static service ** **: directly build static web page service （ such as official website , product documentation , front-end packaged single-page application ） , support efficient processing of static resources （ HTML , CSS , JS , pictures , etc . ） ；**
* **·**  **Reverse proxy ** **: as the " intermediary layer " between the client and the back-end service , forward the request to the back-end API or application server （ such as Tomcat , Node.js ） , hide the real service address , improve security ；**
* **·**  **Load balancing ** **: When facing high concurrency , the request is evenly distributed to multiple back-end machines to avoid single-point server overload and ensure service stability .**
* **·**  **SSL Terminal ** **: Unified processing of HTTPS encryption and decryption, management of SSL certificates, without the need for back-end services to separately configure HTTPS, simplifying the overall architecture.**

**Its biggest feature is its**  **lightness （ low resource consumption ） , stability （ low failure rate ） and high concurrency （ a single instance can support tens of thousands of concurrent connections ） ** **, which makes it a " standard " service component for SMEs to large Internet companies .**

## Why deploy Nginx with Docker? Core strengths

**Deploying Nginx in traditional ways (such as through** `<span leaf="" class="js_darkmode__text__42">yum</span>`/ `<span leaf="" class="js_darkmode__text__45">apt</span>`Installation and source code compilation） often face **inconsistent environment, dependency conflicts, poor configuration isolation and tedious migration **problems （for example: Nginx running normally on the development machine fails to start on the production server due to different system library versions；When multiple services are mixed, Nginx configuration errors can affect other applications.Docker deployment can perfectly solve these pain points, the core advantages are as follows:

1. **1.**  **Environment is absolutely consistent ** **: The Nginx image has packaged all the runtime dependencies （system libraries，Configuration template, basic environment）, regardless of development machine, test machine or production server, as long as it can run Docker, Nginx can "out of the box", completely avoid "local can run, online error"；**
2. **2.**  **Lightweight and efficient ** **: Docker containers are process-level isolation , consume 80 % less resources than virtual machines , and Nginx containers only take**  **seconds to start ** **, and can flexibly limit CPU / memory usage to avoid wasting resources ；**
3. **3 .**  **Service isolation security ** **: Nginx container is completely isolated from the host and other services （ such as MySQL , Redis ） . Even if the Nginx configuration is wrong or crashes , it will not affect other applications , reducing the risk of failure spread ；**
4. **4.**  **Fast iteration and rollback ** **: Updating Nginx only requires pulling a new image and restarting the container （within 10 seconds）；If there is a problem with the new version, delete the new container and start the old version of the image to roll back, which is more than 10 times more efficient than the traditional "uninstall-reinstall" deployment；**
5. **5 .**  **Simplify operations and maintenance management ** **: by** `<span leaf="" class="js_darkmode__text__85">docker</span>`Order or `<span leaf="" class="js_darkmode__text__88">docker-compose</span>`One-click implementation of Nginx start and stop, log view, status monitoring (such as `<span leaf="" class="js_darkmode__text__91">docker logs nginx-web</span>`Look directly at the logs) and beginners can get started quickly.

## 1. View Nginx Images

**You can** **search for **the Nginx image page at :
👉 https://xuanyuan.cloud/r/library/nginx

**You'll see a variety of pull-up methods on the page, and we'll explain each one below.**

## 2. Download Nginx Image

### 2.1 Method of pulling using Xuan Mirror login verification

```
docker pull docker.xuanyuan.run/library/nginx:latest
```

### 2.2 Rename after Ryder

```
docker pull docker.xuanyuan.run/library/nginx:latest \
&& docker tag docker.xuanyuan.run/library/nginx:latest library/nginx:latest \
&& docker rmi docker.xuanyuan.run/library/nginx:latest
```

#### Dxplaination:

* **•** `<span leaf="" class="js_darkmode__text__124">docker pull</span>`: Accelerate the pull of the mirror from the image of the Harvey
* **•** `<span leaf="" class="js_darkmode__text__130">docker tag</span>`: Rename the mirror to the official standard name `<span leaf="" class="js_darkmode__text__133">library/nginx:latest</span>`The subsequent run commands are more concise.
* **•** `<span leaf="" class="js_darkmode__text__139">docker rmi</span>`Remove temporary image tags to avoid taking up extra storage space

### 2.3 Use login-free access (recommended)

**Basic pull command:**

```
docker pull xxx.xuanyuan.run/library/nginx:latest
```

**Full command with rename:**

```
docker pull xxx.xuanyuan.run/library/nginx:latest \
&& docker tag xxx.xuanyuan.run/library/nginx:latest library/nginx:latest \
&& docker rmi xxx.xuanyuan.run/library/nginx:latest
```

#### Dxplaination:

**The login-free method does not require the configuration of account information and can be used directly by novices; Mirror content and** `<span leaf="" class="js_darkmode__text__160">docker.xuanyuan.run</span>`The source is exactly the same, only the pulling address is different.

### 2.4 The official way to connect

**If the network can connect directly to the Docker Hub, or has configured the Xuanyuan image accelerator, you can directly pull the official image:**

```
docker pull library/nginx:latest
```

### 2.5 See if the mirror pulled successfully

```
docker images
```

**If you output something like the following, the mirror download was successful:**

```
REPOSITORY          TAG       IMAGE ID       CREATED        SIZE
library/nginx       latest    f652ca386ed1   2 weeks ago    142MB
```

## 3. Deploy Nginx

**The following use the downloaded ones** `<span leaf="" class="js_darkmode__text__185">library/nginx:latest</span>`Mirroring offers three deployment options that can be selected based on the scene.

### 3.1 Quick deployment (the simplest way)

**Suitable for testing or temporary use, the command is as follows:**

```
# 启动 Nginx 容器，命名为 nginx-test
# 宿主机 80 端口映射到容器 80 端口（Nginx 默认端口）
docker run -d --name nginx-test -p 80:80 library/nginx:latest
```

#### Core parameter explaination:

* **•** `<span leaf="" class="js_darkmode__text__205">--name nginx-test</span>`: Specify a name for the container for subsequent management (e.g. stop, restart)
* **•** `<span leaf="" class="js_darkmode__text__211">-p 80:80</span>`: Port map, format "host port: container port"
* **•** `<span leaf="" class="js_darkmode__text__217">-d</span>`: Run a container in the background

#### How to verify:

**Browser access** `<span leaf="" class="js_darkmode__text__224">http://服务器IP</span>`, should display the official Nginx welcome page.

### 3.2 Mount a directory (recommended method, suitable for actual projects)

**By mounting the host directory, you can achieve "configuration persistence," "log separation," and "web file independent management," as follows:**

#### Step 1: Create a host directory

```
# 一次性创建 html（网页）、conf（配置）、logs（日志）三个目录
mkdir -p /data/nginx/{html,conf,logs}
```

#### Step 2: Prepare to test the web page

```
# 向 html 目录写入测试内容
echo "Hello from Xuanyuan Nginx!" > /data/nginx/html/index.html
```

#### Step 3: Launch the container and mount the directory

```
docker run -d --name nginx-web \
  -p 80:80 -p 443:443 \  # 映射 HTTP(80) 和 HTTPS(443) 端口
  -v /data/nginx/html:/usr/share/nginx/html \  # 网页目录挂载
  -v /data/nginx/conf:/etc/nginx/conf.d \      # 配置目录挂载
  -v /data/nginx/logs:/var/log/nginx \        # 日志目录挂载
  library/nginx:latest
```

#### Directory map instructions:

| **Host Directory**                                                 | **Directory within a container**                                        | **use**                              |
| ------------------------------------------------------------------------ | ----------------------------------------------------------------------------- | ------------------------------------------ |
| `<span leaf="" class="js_darkmode__text__287">/data/nginx/html</span>` | `<span leaf="" class="js_darkmode__text__290">/usr/share/nginx/html</span>` | **Store web files (e.g. HTML, CSS)** |
| `<span leaf="" class="js_darkmode__text__297">/data/nginx/conf</span>` | `<span leaf="" class="js_darkmode__text__300">/etc/nginx/conf.d</span>`     | **Store Nginx configuration files**  |
| `<span leaf="" class="js_darkmode__text__307">/data/nginx/logs</span>` | `<span leaf="" class="js_darkmode__text__310">/var/log/nginx</span>`        | **Storing access logs, error logs**  |

#### Step 4: Example configuration file

**Create a new Nginx base profile** `<span leaf="" class="js_darkmode__text__319">/data/nginx/conf/default.conf</span>`：

```
server {
    listen       80;          # 监听 80 端口（HTTP）
    server_name  _;           # 匹配所有域名（可替换为实际域名，如 example.com）

    root   /usr/share/nginx/html;  # 网页根目录
    index  index.html;             # 默认首页

    access_log  /var/log/nginx/access.log;  # 访问日志路径
    error_log   /var/log/nginx/error.log;   # 错误日志路径

    # 处理请求的核心规则
    location / {
        try_files $uri $uri/ =404;  # 尝试访问文件/目录，不存在则返回 404
    }
}
```

#### Restart the container after configuration update

**After you modify the configuration file, you need to restart the container for the configuration to take effect:**

```
docker restart nginx-web
```

### 3.3 docker-compose deployment (suitable for enterprise scenarios)

**adopt** `<span leaf="" class="js_darkmode__text__392">docker-compose.yml</span>`Unified management of container configurations supports one-click start / stop, as follows:

#### Edit: Create docker-compose.yml file

```
version: '3'  # 指定 docker-compose 语法版本
services:
  nginx:
    image: library/nginx:latest  # 使用的镜像
    container_name: nginx-service  # 容器名称
    ports:
      - "80:80"   # HTTP 端口映射
      - "443:443" # HTTPS 端口映射
    volumes:
      - ./html:/usr/share/nginx/html   # 网页目录（相对路径，与 yml 同目录）
      - ./conf:/etc/nginx/conf.d       # 配置目录
      - ./logs:/var/log/nginx          # 日志目录
    restart: always  # 容器退出后自动重启（保障服务可用性）
```

#### Step 2: Launch the service

**in** `<span leaf="" class="js_darkmode__text__465">docker-compose.yml</span>`The directory in question executes:

```
docker compose up -d
```

#### Additional note:

* **· To modify the configuration / web page, directly operate in the current directory** `<span leaf="" class="js_darkmode__text__477">html</span>`、 `<span leaf="" class="js_darkmode__text__480">conf</span>`Just in a folder.
* **· Stop service command:** `<span leaf="" class="js_darkmode__text__486">docker compose down</span>`
* **· Check the status of the service:** `<span leaf="" class="js_darkmode__text__491">docker compose ps</span>`

## 4 Verification of results

**Verify that the Nginx service is working properly by:**

1. **1.**  **Browser validation ** **:**
   access `<span leaf="" class="js_darkmode__text__505">http://服务器IP</span>`Please display the test content that was previously written ( `<span leaf="" class="js_darkmode__text__508">Hello from Xuanyuan Nginx!</span>`) or Nginx Welcome Page.
2. **2 .**  **View the container status ** **:**

   ```
   docker ps
   ```

   **if** `<span leaf="" class="js_darkmode__text__522">STATUS</span>`The column shows `<span leaf="" class="js_darkmode__text__525">Up</span>`That means the container is running properly.
3. **3 .**  **View container logs ** **:**
   with `<span leaf="" class="js_darkmode__text__535">nginx-web</span>`Containers as an example (if used) `<span leaf="" class="js_darkmode__text__538">docker-compose</span>`That would be: `<span leaf="" class="js_darkmode__text__541">nginx-service</span>`）：

   ```
   docker logs nginx-web
   ```

   **No error message is reported, which means the service is started normally.**

## 5. Frequently Asked Questions

### 5.1 Can't access the web page?

**Directions of the investigation:**

* **·**  **Security Group ** **: Check whether the cloud server security group allows the 80 （ HTTP ） and 443 （ HTTPS ） ports**
* **·**  **Firewall ** **: Check the server 's local firewall , such as** `<span leaf="" class="js_darkmode__text__568">ufw</span>`or `<span leaf="" class="js_darkmode__text__571">firewalld</span>`) Whether the corresponding port is open

  * **•** `<span leaf="" class="js_darkmode__text__578">ufw</span>`Open ports: `<span leaf="" class="js_darkmode__text__581">sudo ufw allow 80/tcp && sudo ufw allow 443/tcp</span>`
  * **•** `<span leaf="" class="js_darkmode__text__586">firewalld</span>`Open ports: `<span leaf="" class="js_darkmode__text__589">sudo firewall-cmd --add-port=80/tcp --permanent && sudo firewall-cmd --add-port=443/tcp --permanent && sudo firewall-cmd --reload</span>`
* **·**  **Port conflict ** **: Execute** `<span leaf="" class="js_darkmode__text__597">netstat -tuln | grep 80</span>`Check whether port 80 is being occupied by other processes, and if so, replace the host port (e.g. `<span leaf="" class="js_darkmode__text__600">-p 8080:80</span>`）

### 5.2 How do I enable HTTPS?

1. **1.**  **Prepare the certificate ** **: Get the SSL certificate file （ usually contains** `<span leaf="" class="js_darkmode__text__612">fullchain.pem</span>`Certificate chains and `<span leaf="" class="js_darkmode__text__615">privkey.pem</span>`Private key), placed in the host directory (e.g. `<span leaf="" class="js_darkmode__text__618">/data/nginx/certs</span>`）。
2. **2 .**  **Mount the certificate directory ** **: add the certificate mount parameter to the container start command :**
   ```
   -v /data/nginx/certs:/etc/nginx/certs
   ```
3. **3.**  **Modify the Nginx configuration ** **: Update** `<span leaf="" class="js_darkmode__text__636">default.conf</span>`, Add HTTPS listening rules:
   ```
   server {
       listen 443 ssl;  # 监听 443 端口（HTTPS）
       server_name 你的域名;  # 替换为实际域名（如 example.com）

       # 证书路径（容器内路径，对应宿主机 /data/nginx/certs）
       ssl_certificate     /etc/nginx/certs/fullchain.pem;
       ssl_certificate_key /etc/nginx/certs/privkey.pem;

       root   /usr/share/nginx/html;
       index  index.html;
   }
   ```
4. **4 .**  **Restart the container ** **:** `<span leaf="" class="js_darkmode__text__677">docker restart nginx-web</span>`

### 5.3 What to do if the log file is too large?

* **·**  **Option 1 : Cutting logs with logrotate ** **（ recommended ） :**
  Create on a host machine `<span leaf="" class="js_darkmode__text__689">/etc/logrotate.d/nginx</span>`Configure the file to set the log cutting rules (e.g. by day and kept for 7 days):
  ```
  /data/nginx/logs/*.log {
      daily
      rotate 7
      compress
      delaycompress
      missingok
      notifempty
      create 0640 root root
  }
  ```
* **·**  **Option 2 : Log collection system ** **:**
  With tools such as ELK Stack (Elasticsearch + Logstash + Kibana) or Loki, centralized log collection, storage and analysis can be achieved.

### 5.4 The time zone inside the container is not correct?

**When launching a container, go through** `<span leaf="" class="js_darkmode__text__714">-e</span>`Parameter Set Time Zone (in Shanghai Time Zone) `<span leaf="" class="js_darkmode__text__717">Asia/Shanghai</span>`Examples:

```
docker run -d -e TZ=Asia/Shanghai \
  --name nginx-web \
  -p 80:80 -p 443:443 \
  -v /data/nginx/html:/usr/share/nginx/html \
  -v /data/nginx/conf:/etc/nginx/conf.d \
  -v /data/nginx/logs:/var/log/nginx \
  library/nginx:latest
```

## ending

**By now, you've mastered the entire process of Nginx image pulling and Docker deployment based on archetype - from image download validation, to deployment practices for different scenarios, to troubleshooting, with complete commands and instructions for each step. For beginners, it is recommended to familiarize yourself with the process from "fast deployment," then try the "directory mount" solution to understand the meaning of persistence configuration, and finally move to "docker-compose" management depending on your requirements.**

**In actual use, if you encounter problems that are not covered by the document, you can combine them** `<span leaf="" class="js_darkmode__text__735">docker logs 容器名</span>`Check the log location reasons, or refer to the official Nginx documentation for additional learning. With in-depth practice, you can also further explore Nginx's reverse proxy, negative load balancing, HTTPS advanced configuration and other functions based on this article, so that Nginx can better support your business needs.
