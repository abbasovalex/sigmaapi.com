# Usage with Nginx
Sigma API helps you prerender your website, but you need to host the static files yourself.<br>
If you’re using Nginx and don’t know how to set it up, the following instructions will help you:

1. Add the following block inside the ```http``` section of your Nginx configuration: ```map $http_user_agent $document_root {}``` into ```http``` section of Nginx's config.
2. Add ```root $document_root;``` inside ```server``` section of your Nginx configuration.
3. Upload the folder with static files to your server, and replace ```/path/to/your/site``` and ```/path/to/seo/static/files``` with the actual paths.
4. Restart Nginx.

```nginx

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile           on;
    keepalive_timeout  65;
    
    map $http_user_agent $document_root {
        default /path/to/your/site;
        "~*googlebot" /path/to/seo/static/files;
        "~*yandex" /path/to/seo/static/files;
        "~*telegrambot" /path/to/seo/static/files;
    }
    
    server {
        listen 80;
        server_name example.com;
        root $document_root;
        
        location / {
            ...
        }

    }
}
```

**We’ve prepared a demo page.**
If you visit the [page](https://demo-seo-nginx.sigmaapi.com/) as a regular user, you’ll see «Hello, world!»
But if you visit it as a Google, Yandex, or other search engine bot, you’ll see «This is a static page for SEO User-Agents»

You can also check it using cURL.

```terminaloutput
$  curl https://demo-seo-nginx.sigmaapi.com
Hello, World!
$  curl -H "User-Agent: googlebot" https://demo-seo-nginx.sigmaapi.com
This is static page for SEO User-Agents
```