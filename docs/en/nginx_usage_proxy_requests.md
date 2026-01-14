# Proxying Requests to Our Servers
Configure your Nginx server to proxy requests from search engine crawlers to SigmaAPI 
and serve pre-rendered HTML pages instead of the original client-side content. 
This setup eliminates the need to store rendered files locally and reduces disk usage on 
your infrastructure.

### Example of your nginx.conf

```nginx
http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile           on;
    keepalive_timeout  65;
    
    map $http_user_agent $is_webbot {
      default 0;
      "~*(?:\.net crawler|360spider|50\.nu|8bo crawler bot|aboundex|accoona|adldxbot|ahrefsbot|altavista|appengine-google|applebot|archiver|arielisbot|ask jeeves|auskunftbot|baidumobaider|baiduspider|becomebot|bingbot|bingpreview|bitbot|bitlybot|blitzbot|blogbridge|boardreader|botseer|catchbot|catchpoint bot|charlotte|checklinks|cliqzbot|clumboot|coccocbot|converacrawler|crawl-e|crawlconvera|dataparksearch|daum|deusu|discordbot|dotbot|duckduckbot|elefent|embedly|evernote|exabot|facebookbot|facebookexternalhit|meta-external|fatbot|fdse robot|feed seeker bot|feedfetcher|femtosearchbot|findlinks|flamingo_searchengine|flipboard|followsite bot|furlbot|fyberspider|gaisbot|galaxybot|geniebot|genieo|gigablast|gigabot|girafabot|gomezagent|gonzo1|googlebot|google sketchup|adsbot-google|google-structured-data-testing-tool|google-extended|developers\.google\.com/+/web/snippet|haosouspider|heritrix|holmes|hoowwwer|htdig|ia_archiver|idbot|infuzapp|innovazion crawler|instagram|internetarchive|iqdb|iskanie|istellabot|izsearch\.com|kaloogabot|kaz\.kz_bot|kd bot|konqueror|kraken|kurzor|larbin|leia|lesnikbot|linguee bot|linkaider|linkapediabot|linkedinbot|lite bot|llaut|lookseek|lycos|mail\.ru_bot|masidani_bot|masscan|mediapartners-google|metajobbot|mj12bot|mnogosearch|mogimogi|mojeekbot|motominerbot|mozdex|msiecrawler|msnbot|msrbot|netpursual|netresearch|netvibes|newsgator|ng-search|nicebot|nutchcvs|nuzzel|nymesis|objectssearch|odklbot|omgili|oovoo|oozbot|openfosbot|orangebot|orbiter|org_bot|outbrain|pagepeeker|pagesinventory|parsijoobot|paxleframework|peeplo screenshot bot|pinterest|plantynet_webrobot|plukkie|pompos|psbot|quora link preview|qwantify|read%20later|reaper|redcarpet|redditbot|retreiver|riddler|rival iq|rogerbot|saucenao|scooter|scrapy|scrubby|searchie|searchsight|seekbot|semanticdiscovery|seznambot|showyoubot|simplepie|simpy|sitelockspider|skypeuripreview|slackbot|slack-imgproxy|slurp|snappy|sogou|solofield|speedyspider|speedy spider|sputnikbot|stackrambler|teeraidbot|teoma|theusefulbot|thumbshots\.ru|thumbshotsbot|tineye|toweya\.com|toweyabot|tumblr|tweetedtimes|tweetmemebot|twitterbot|url2png|vagabondo|vebidoobot|viber|visionutils|vkshare|voilabot|vortex|votay bot|voyager|w3c_validator|wasalive\.bot|web-sniffer|websquash\.com|webthumb|whatsapp|whatweb|wire|wotbox|yacybot|yahoo|yandex|yeti|yisouspider|yodaobot|yooglifetchagent|yoozbot|yottaamonitor|yowedo|zao-crawler|zebot_www\.ze\.bz|zooshot|zyborgi|ai2bot|amazonbot|anthropic\.com|bard|bytespider|ccbot|chatgpt-user|claude-web|claudebot|cohere-ai|deepseek|diffbot|duckassistbot|gemini|gptbot|grok|mistralai|oai-searchbot|omgili|openai\.com|perplexity\.ai|perplexitybot|xai|youbot)" 1;
    }

    server {
        listen 80;
        listen 443 ssl;
        server_name seo.xclouds.dev;
        charset utf-8;
        ssl_certificate     /etc/letsencrypt/live/xclouds.dev/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/xclouds.dev/privkey.pem;
    
    
        gzip_static on;
        open_file_cache          max=1000 inactive=20s;
        open_file_cache_valid    30s;
        open_file_cache_min_uses 2;
        open_file_cache_errors   on;
        sendfile on;
        tcp_nopush on;
    
        large_client_header_buffers 4 32k;
    
        set $rootPath '/projects/xclouds/apps/site';
        set $index_page 'index.html';
        
        error_page 454 = @sigmaapi;  # CUSTOM 454 CODE FOR INTERNAL REDIRECT TO @sigmaapi
    
    
        location / {
          if ($is_webbot = 1) {
            return 454;
          }
          expires 0;
          root $rootPath;
          add_header Cache-Control "public, must-revalidate, proxy-revalidate";
          try_files /maintance.html $uri/$index_page $uri.html $uri;
        }
    
        location @sigmaapi {
          internal;
          set $renderer_domain "cdn-ru.sigmaapi.com";
          set $ssg_path $uri;
    
    
          proxy_pass_request_headers off;
          proxy_hide_header WWW-Authenticate;
          proxy_hide_header Set-Cookie;
          proxy_hide_header Cache-Control;
          proxy_set_header X-Auth-Token <YOUR_API_TOKEN>;
          proxy_set_header Host $renderer_domain;
          proxy_set_header User-Agent $http_user_agent;
          proxy_set_header Connection "close";
    
          proxy_http_version 1.0;
          resolver 1.1.1.1 8.8.4.4 8.8.8.8 1.0.0.1 valid=300s;
          resolver_timeout 15s;
    
          sendfile off;
          
          rewrite (.*) /v1/ssg/<YOUR_DOMAIN>$request_uri break;
          # rewrite (.*) /v1/ssg/example.com$request_uri break;
          proxy_pass https://$renderer_domain;
        }
    
        access_log  '/var/log/nginx/access_log' main buffer=16k;
        error_log   '/var/log/nginx/error_log';
    }
}
```
Below, we explain the key aspects of this configuration approach using this template as an example.

**Step 1:**
```nginx
    map $http_user_agent $is_webbot {
      default 0;
      "~*(?:\.net crawler|360spider|50\.nu|8bo crawler bot|aboundex|accoona|adldxbot|ahrefsbot|altavista|appengine-google|applebot|archiver|arielisbot|ask jeeves|auskunftbot|baidumobaider|baiduspider|becomebot|bingbot|bingpreview|bitbot|bitlybot|blitzbot|blogbridge|boardreader|botseer|catchbot|catchpoint bot|charlotte|checklinks|cliqzbot|clumboot|coccocbot|converacrawler|crawl-e|crawlconvera|dataparksearch|daum|deusu|discordbot|dotbot|duckduckbot|elefent|embedly|evernote|exabot|facebookbot|facebookexternalhit|meta-external|fatbot|fdse robot|feed seeker bot|feedfetcher|femtosearchbot|findlinks|flamingo_searchengine|flipboard|followsite bot|furlbot|fyberspider|gaisbot|galaxybot|geniebot|genieo|gigablast|gigabot|girafabot|gomezagent|gonzo1|googlebot|google sketchup|adsbot-google|google-structured-data-testing-tool|google-extended|developers\.google\.com/+/web/snippet|haosouspider|heritrix|holmes|hoowwwer|htdig|ia_archiver|idbot|infuzapp|innovazion crawler|instagram|internetarchive|iqdb|iskanie|istellabot|izsearch\.com|kaloogabot|kaz\.kz_bot|kd bot|konqueror|kraken|kurzor|larbin|leia|lesnikbot|linguee bot|linkaider|linkapediabot|linkedinbot|lite bot|llaut|lookseek|lycos|mail\.ru_bot|masidani_bot|masscan|mediapartners-google|metajobbot|mj12bot|mnogosearch|mogimogi|mojeekbot|motominerbot|mozdex|msiecrawler|msnbot|msrbot|netpursual|netresearch|netvibes|newsgator|ng-search|nicebot|nutchcvs|nuzzel|nymesis|objectssearch|odklbot|omgili|oovoo|oozbot|openfosbot|orangebot|orbiter|org_bot|outbrain|pagepeeker|pagesinventory|parsijoobot|paxleframework|peeplo screenshot bot|pinterest|plantynet_webrobot|plukkie|pompos|psbot|quora link preview|qwantify|read%20later|reaper|redcarpet|redditbot|retreiver|riddler|rival iq|rogerbot|saucenao|scooter|scrapy|scrubby|searchie|searchsight|seekbot|semanticdiscovery|seznambot|showyoubot|simplepie|simpy|sitelockspider|skypeuripreview|slackbot|slack-imgproxy|slurp|snappy|sogou|solofield|speedyspider|speedy spider|sputnikbot|stackrambler|teeraidbot|teoma|theusefulbot|thumbshots\.ru|thumbshotsbot|tineye|toweya\.com|toweyabot|tumblr|tweetedtimes|tweetmemebot|twitterbot|url2png|vagabondo|vebidoobot|viber|visionutils|vkshare|voilabot|vortex|votay bot|voyager|w3c_validator|wasalive\.bot|web-sniffer|websquash\.com|webthumb|whatsapp|whatweb|wire|wotbox|yacybot|yahoo|yandex|yeti|yisouspider|yodaobot|yooglifetchagent|yoozbot|yottaamonitor|yowedo|zao-crawler|zebot_www\.ze\.bz|zooshot|zyborgi|ai2bot|amazonbot|anthropic\.com|bard|bytespider|ccbot|chatgpt-user|claude-web|claudebot|cohere-ai|deepseek|diffbot|duckassistbot|gemini|gptbot|grok|mistralai|oai-searchbot|omgili|openai\.com|perplexity\.ai|perplexitybot|xai|youbot)" 1;
    }
```
Here we detect the request source. When the request is made by a search engine crawler or an AI agent, the `$is_webbot` variable is set to `1`.

**Step 2:**

```nginx
location @sigmaapi {
  internal;
  set $renderer_domain "cdn-ru.sigmaapi.com";
  set $ssg_path $uri;

  proxy_pass_request_headers off;
  proxy_hide_header WWW-Authenticate;
  proxy_hide_header Set-Cookie;
  proxy_hide_header Cache-Control;
  proxy_set_header X-Auth-Token <YOUR_API_TOKEN>;
  proxy_set_header Host $renderer_domain;
  proxy_set_header User-Agent $http_user_agent;
  proxy_set_header Connection "close";

  proxy_http_version 1.0;
  resolver 1.1.1.1 8.8.4.4 8.8.8.8 1.0.0.1 valid=300s;
  resolver_timeout 15s;

  sendfile off;
  
  rewrite (.*) /v1/ssg/<YOUR_DOMAIN>$request_uri break;
  # rewrite (.*) /v1/ssg/example.com$request_uri break;
  proxy_pass https://$renderer_domain;
}
```
Here we define a named location block to process requests from search engine crawlers and AI agents. Make sure to replace `<YOUR_API_TOKEN>` and `<YOUR_DOMAIN>`, and keep the rest unchanged.

**Step 3:**
```nginx
error_page 454 = @sigmaapi;  # CUSTOM 454 CODE FOR INTERNAL REDIRECT TO @sigmaapi
```

Here we route requests from search engine bots and AI agents to the `location @sigmaapi {}`, you need the following directive:

**Step 4:**
```nginx
  if ($is_webbot = 1) {
    return 454;
  }
```
It's important to set `if` directive inside the main location block. It evaluates the `$is_webbot` variable and routes the request to `location @sigmaapi {}` when the request comes from a search bot or an AI agent.


### Common pitfalls

#### Wrong YOUR_DOMAIN
Set `<YOUR_DOMAIN>` to match exactly the domain used in the sitemap configured in the UI (see screenshot below).
For example, if your sitemap is `https://www.departures-international.com/sitemap`, then the domain must be `departures-international.com`.
Please note: do not include `http://` or `https://` at the beginning, and do not add a trailing `/` at the end.

```nginx
rewrite (.*) /v1/ssg/departures-international.com$request_uri break;
```


![UI of Scheduler](/images/domain_examples.png)

#### Wrong YOUR_API_TOKEN
<p class="tip">
    You’ll need an api key to work with API. If you don’t have one, please <a href="https://sigmaapi.com/accounts/register/" target="_blank" rel="noopener">sign up</a> first.
</p>

`<YOUR_API_TOKEN>` should be copied from our [UI](https://sigmaapi.com/settings/seo_rendering/).
Make sure it does not contain any spaces or line breaks.
![UI of Scheduler](/images/api_key_example.png)

```nginx
proxy_set_header X-Auth-Token 0000d5bd-1111-2222-3333-555589392dcd;
``` 