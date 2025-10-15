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