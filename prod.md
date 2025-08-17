# 部署指南：将 UniApp Web 前端发布到 https://sh-aliyun2.vincentzyu233.cn/qwq-server

本指南涵盖从本地打包、上传到云服、服务器解压、以及修改 Nginx 配置以在现有域名下添加新应用的完整流程。

**新目标：**
- **访问地址:** `https://sh-aliyun2.vincentzyu233.cn/qwq-server`
- **利用现有域名和SSL证书**

## 1. 本地打包（7-Zip）

此步骤不变。确保 `web` 目录已准备好。

```powershell
# 切换到包含 web 目录的构建输出目录
Set-Location "G:\GGames\Minecraft\shuyeyun\qq-bot\uniapp-webui\uniapp-mp-qwq-server-frontend-v0.1.0\unpackage\dist\build"

# 使用 7-Zip 重新打包（存在则覆盖）
7z a -tzip -y web.zip .\web\*
```

## 2. 上传到服务器（scp）

此步骤不变。将 `web.zip` 上传到服务器。

```powershell
# 将 web.zip 上传到服务器 /data/mp_qwq_frontend 目录
scp -P 22 `
  "G:\GGames\Minecraft\shuyeyun\qq-bot\uniapp-webui\uniapp-mp-qwq-server-frontend-v0.1.0\unpackage\dist\build\web.zip" `
  root@101.132.131.209:/data/mp_qwq_frontend/
```

## 3. 服务器解压与准备工作（ssh）

此步骤不变。解压并设置文件权限。

```bash
ssh root@101.132.131.209

# 确保目录存在
mkdir -p /data/mp_qwq_frontend
cd /data/mp_qwq_frontend

# 解压
unzip -o web.zip

# 设置所有者为 nginx 运行用户
chown -R www-data:www-data /data/mp_qwq_frontend/web
```

## 4. Nginx 站点配置 (新方案)

我们将修改 `/etc/nginx/sites-available/services` 文件，在 `server_name sh-aliyun2.vincentzyu233.cn;` 的 `server` 块中，添加一个新的 `location` 来处理 `/qwq-server` 路径的请求。

**将你的 `/etc/nginx/sites-available/services` 文件更新为以下完整内容：**

```nginx

# ===================== sh-aliyun2.vincentzyu233.cn =====================
server {
    # 将所有 HTTP 请求重定向到 HTTPS
    listen 80;
    server_name sh-aliyun2.vincentzyu233.cn;
    return 301 https://$host$request_uri;
}


server {
    # 启用 HTTPS
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name sh-aliyun2.vincentzyu233.cn;

    # Certbot-managed SSL configuration
    ssl_certificate /etc/letsencrypt/live/sh-aliyun2.vincentzyu233.cn/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/sh-aliyun2.vincentzyu233.cn/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    # ------------------ FastAPI API 反向代理配置 ------------------
    # 通过 /qs/ 路径访问你的 FastAPI 服务
    location /qs/ {
        # 转发到 FRP 客户端暴露的本地 8326 端口
        proxy_pass http://127.0.0.1:8326/;

        # 设置请求头
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # 路径 /mpbackend/
    location /mpbackend/ {
        # 将请求转发到 FRP 映射的本地 8416 端口
        proxy_pass http://127.0.0.1:8416/;

        # 设置请求头
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # 访问 /qwq-server 时，自动在末尾添加斜杠
    location = /qwq-server {
        return 301 /qwq-server/;
    }

    # 处理 /qwq-server/ 路径下的所有请求
    location /qwq-server/ {
        alias /data/mp_qwq_frontend/web/;
        index index.html;
        # 对于单页应用(SPA)，确保刷新页面时能正确找到 index.html
        try_files $uri $uri/ /qsfrontend/index.html;
    }

    # 静态资源路径 /qsfrontend/
    location /qsfrontend/ {
        alias /data/mp_qwq_frontend/web/;
        index index.html;
        try_files $uri $uri/ /qsfrontend/index.html;
    }

}

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
