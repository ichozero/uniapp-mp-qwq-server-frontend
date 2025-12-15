# 部署指南：将 UniApp Web 前端发布到 https://sh-aliyun2.vincentzyu233.cn/qwq-server

本指南涵盖从本地打包、上传到云服、服务器解压、以及修改 Nginx 配置以在现有域名下添加新应用的完整流程。

**目标：**
- **访问地址:** `https://sh-aliyun2.vincentzyu233.cn/qwq-server`
- **利用现有域名和SSL证书**

## 1. 本地打包（7-Zip）

此步骤不变。确保 `web` 目录已准备好。

```powershell
# 切换到包含 web 目录的构建输出目录
Set-Location "G:\GGames\Minecraft\shuyeyun\qq-bot\uniapp-webui\uniapp-mp-qwq-server-frontend\unpackage\dist\build"

# 使用 7-Zip 重新打包（存在则覆盖）
7z a -tzip -y web.zip .\web\*
```

## 2. 上传到服务器（scp）

此步骤不变。将 `web.zip` 上传到服务器。

```powershell
# 将 web.zip 上传到服务器 /data/mp_qwq_frontend 目录
scp -P 22 ./web.zip root@sh-aliyun2.vincentzyu233.cn:/data/mp_qwq_frontend/
```

## 3. 服务器解压与准备工作（ssh）

此步骤不变。解压并设置文件权限。

```bash
ssh root@101.132.131.209

# 确保目录存在
mkdir -p /data/mp_qwq_frontend
cd /data/mp_qwq_frontend

mkdir web
unzip -o web.zip -d web

# 设置所有者为 nginx 运行用户
chown -R www-data:www-data /data/mp_qwq_frontend/web
```

## 4. 现在的nginx配置：

```shell
cat /etc/nginx/sites-available/services
```

```shell
# ===================== sh-aliyun2.vincentzyu233.cn =====================
# HTTP 服务配置 (保持不变，但增加 http 重定向到 https)
server {
    listen 80;
    listen [::]:80;
    server_name sh-aliyun2.vincentzyu233.cn;

    # 强制将所有 HTTP 请求重定向到 HTTPS
    # 对于 koishi-market-proxy，我们不需要额外的 HTTP 配置，只需重定向到 HTTPS
    return 301 https://$host$request_uri;
}

# HTTPS 服务配置 (主配置)
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name sh-aliyun2.vincentzyu233.cn;

    # Certbot-managed SSL configuration
    ssl_certificate /etc/letsencrypt/live/sh-aliyun2.vincentzyu233.cn/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/sh-aliyun2.vincentzyu233.cn/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    # ================== 🌟 新增 koishi-market-proxy 配置 🌟 ==================
    # 目标：https://sh-aliyun2.vincentzyu233.cn/koishi-market-proxy/market
    #       -> http://127.0.0.1:51214/market
    location /koishi-market-proxy/ {
        # 使用 rewrite 移除路径前缀 /koishi-market-proxy/
        # $1 会捕获 (.*) 的内容，例如 /market 或 /health
        # last 表示继续处理下一个 location 块或当前 location 块
        rewrite ^/koishi-market-proxy/(.*)$ /$1 last;
    }

    # 由于上面的 rewrite 已经将 URI 改成了 /market 或 /health 等，
    # 我们需要一个新的 location 块来捕获这些请求并进行代理
    location / {
        # 假设 frp 目标服务监听在 127.0.0.1:51214
        proxy_pass http://127.0.0.1:51214;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 错误处理：如果内部服务没启动，可以返回一个友好的错误
        # error_page 502 /502.html;
    }
    # =====================================================================


    # ------------------ 保持原有的 FastAPI API 反向代理配置 ------------------
    location /qs/ {
        proxy_pass http://127.0.0.1:8326/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /mpbackend/ {
        proxy_pass http://127.0.0.1:8416/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location = /qwq-server {
        return 301 /qwq-server/;
    }

    location /qwq-server/ {
        alias /data/mp_qwq_frontend/web/;
        index index.html;
        try_files $uri $uri/ /qsfrontend/index.html;
    }

    location /qsfrontend/ {
        alias /data/mp_qwq_frontend/web/;
        index index.html;
        try_files $uri $uri/ /qsfrontend/index.html;
    }
}

# ===================== mc.vincentzyu233.cn =====================
# (以下 mc.vincentzyu233.cn 的配置保持不变)
# HTTP 服务配置
server {
    listen 80;
    listen [::]:80;
    server_name mc.vincentzyu233.cn;

    # 强制跳转 HTTPS
    return 301 https://$host$request_uri;
}

# HTTPS 服务配置
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name mc.vincentzyu233.cn;

    ssl_certificate /etc/letsencrypt/live/mc.vincentzyu233.cn/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/mc.vincentzyu233.cn/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    # 直接指向前端目录
    location /qwq-server/ {
        alias /data/mp_qwq_frontend/web/;
        index index.html;
        try_files $uri $uri/ /index.html;
    }
}
root@iZgmp2m2rimd61Z:/data/mp_qwq_frontend#

```



**应用配置**

在服务器上执行以下命令来应用新的配置：

```bash
# 检查语法
nginx -t

# 重载配置
systemctl reload nginx
```

## 5. 验证访问

现在，SSL证书申请步骤已经不再需要。直接在浏览器中访问新的URL：

- **https://sh-aliyun2.vincentzyu233.cn/qwq-server**

如果页面和资源（如图片、CSS）都加载正常，那么部署就成功了！

## 6. 后续更新流程

此步骤不变，依然是推荐的零停机更新方式。

```bash
# 上传新的 web.zip 后：
cd /data/mp_qwq_frontend
rm -rf web_new
mkdir -p web_new
unzip -o web.zip -d web_new
mv web web_bak_$(date +%Y%m%d%H%M%S)
mv web_new/web web
chown -R www-data:www-data /data/mp_qwq_frontend/web
# 验证无误后，清理旧备份目录
rm -rf web_bak_*
```

— 完 —
