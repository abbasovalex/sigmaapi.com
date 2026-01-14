# Using ZIP Archives from the SigmaAPI UI with Your Own Nginx
The following instructions explain how to configure your Nginx server to serve rendered static files for Google, Yandex, and Telegram.
It’s very easy! You only need to add two code blocks and place the folder with the rendered files on your web server. That’s it.

1. Add the following block inside the ```http``` section:
```nginx
    map $http_user_agent $document_root {
        default /path/to/your/site; # ← Change on real path to your app
        "~*googlebot" /path/to/seo/static/files; # ← Rendered pages for Google
        "~*yandex" /path/to/seo/static/files; # ← Rendered pages for Yandex
        "~*telegrambot" /path/to/seo/static/files; # ← Rendered pages for Telegram
    }
```
2. Add the root directive inside the ```server``` section:

```nginx
    root $document_root;
```
_The root directive specifies the document root, which is the base directory where Nginx searches for files when serving requests._

3. Placing the static files to ```/path/to/seo/static/files```
<p class="warn">
<strong>Where can you get the rendered files?</strong><br>
You can download them as a ZIP archive from the <a href="#/rendering_via_ui">UI</a> and extract it to <code>/path/to/seo/static/files</code>

4. After placing the static files on your server and replacing the paths with the correct ones, check that your changes didn’t break the Nginx configuration by running: ```nginx -t```:
5. Restart Nginx: ```nginx -s reload```

#### See Live Example

We’ve prepared [a demo page](https://demo-seo-nginx.sigmaapi.com/) with work configuration:
```nginx
http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile           on;
    keepalive_timeout  65;
    
    map $http_user_agent $rootPathYourSite {
        default /projects/hive/apps/demo-seo-nginx;
        "~*googlebot" /projects/hive/apps/demo-seo-nginx/static;
        "~*yandex" /projects/hive/apps/demo-seo-nginx/static;
        "~*telegrambot" /projects/hive/apps/demo-seo-nginx/static;
    }

    server {
        listen 80;
        listen 443 ssl;
        server_name demo-seo-nginx.sigmaapi.com;
        charset utf-8;
        ssl_certificate     /etc/letsencrypt/live/sigmaapi.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/sigmaapi.com/privkey.pem;
    
        gzip_static on;
        open_file_cache          max=1000 inactive=20s;
        open_file_cache_valid    30s;
        open_file_cache_min_uses 2;
        open_file_cache_errors   on;
        sendfile on;
        tcp_nopush on;
    
        set $index_page 'index.html';
    
        location / {
            expires 0;
            root $rootPathYourSite; 
            add_header Cache-Control "public, must-revalidate, proxy-revalidate";
            try_files $uri/$index_page $uri.html $uri;
        }
    
        location ~* ^.+\.(html|xml|jpg|jpeg|gif|png|pdf|ico|swf|xap|css|js|svg|ttf|woff|eot|eot\?#iefix)$ {
            root $rootPathYourSite;
            expires 0;
            add_header Cache-Control "public, must-revalidate, proxy-revalidate";
        }
    
        access_log  '/var/log/access_log' main buffer=16k;
        error_log   '/var/log/error_log';
    }
}
```
If you visit the [page](https://demo-seo-nginx.sigmaapi.com/) as a regular user, you’ll see «Hello, world!»
But if you visit it as a Google, Yandex, or other search engine bot, you’ll see «This is a static page for SEO User-Agents». For example, you can check it using cURL.

```terminaloutput
$  curl https://demo-seo-nginx.sigmaapi.com
Hello, World!
$  curl -H "User-Agent: googlebot" https://demo-seo-nginx.sigmaapi.com
This is static page for SEO User-Agents
```