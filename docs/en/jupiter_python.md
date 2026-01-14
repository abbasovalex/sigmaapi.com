# Jupiter / Python Example
Our API is simple and easy to use — it includes just a few methods. 

<p class="tip">
    You’ll need an api key to work with API. If you don’t have one, please <a href="https://sigmaapi.com/accounts/register/" target="_blank" rel="noopener">sign up</a> first.
</p>

#### https://sigmaapi.com/v1/add
Adds the target website to the rendering queue.
Processing time depends on the current system load and may take up to several hours, though we aim to keep it as fast as possible.

**Request example:**

```python
import requests
import urllib.parse

your_uri = urllib.parse.quote("https://github.com/abbasovalex/sigmaapi.com", safe='')
# add to queue
response = requests.get("https://sigmaapi.com/v1/add?uri=%s" % your_uri,
                        headers={'X-Auth-Token': '<YOUR_API_KEY>'})
if response.status_code == 201:
    json = response.json()
    request_id = json["requestId"]
```

**Request example with cookies:**

```python
import base64
import json
import requests
import urllib.parse


def encode_cookies(list_of_dict):
    b64 = base64.urlsafe_b64encode(json.dumps(list_of_dict).encode('utf-8'))
    return b64.decode('utf-8') # convert binary to string

your_uri = urllib.parse.quote("https://setcookie.net", safe='')
uri_cookies = [
    {
        "name": "my_cookie_1",
        "value": "tasty",
        "path": "/",
        "domain": "your_domain.com",
        "httpOnly": True, 
        "sameSite": "Lax", # can be None, Lax, Strict or emty string
        "secure": False,
        "expiry": 1766354057 # 
    },
    {
        "name": "my_cookie_2",
        "value": "tasty again",
        "path": "/",
        "domain": "your_domain.com",
        "secure": True,
        "expiry": 1796674794
    }
]
# add to queue
response = requests.get("https://sigmaapi.com/v1/add?uri=%s&cookies=%s" % (your_uri, encode_cookies(uri_cookies)),
                        headers={'X-Auth-Token': '<YOUR_API_KEY>'})
if response.status_code == 201:
    json = response.json()
    request_id = json["requestId"]
```

| Parameter         | Type    | Required | Description                                                                                                                                                                               |
|-------------------|---------|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| api_key           | string  | yes      | Your personal API key. If you don’t have one, please [sign up](https://sigmaapi.com/accounts/register/) first. You can skip this parameter if you pass the `X-Auth-Token` header instead. |
| uri               | string  | yes      | Site URL (with http or https)                                                                                                                                                             |
| rendering_timeout | number  | no       | Number of seconds to wait before capturing the page. Useful if your site needs time to load dynamic content.                                                                              |
| disable_js        | boolean | no       | If true, the page will be rendered with JavaScript disabled (emulates a browser without JS).                                                                                              |
| cookies           | string  | no       | If you want to pass cookies, you need to send them as a list encoded in Base64. Be careful — you must provide a Base64-encoded string, not binary data.                                   |


**Response example:**
```json
  {
    "uri":"https://sigmaapi.com/ru/",
    "status":"in queue",
    "requestId":"7466d7d6-0000-4000-ab00-01a000028497"
  }
```
        
| Parameter | Type   | Description                                                                                                                           |
|-----------|--------|---------------------------------------------------------------------------------------------------------------------------------------|
| uri       | string | Site URL (with http or https)                                                                                                         |
| status    | string | "in queue" means that your task has been successfully placed in the queue. To get the result, use the method`https://sigmaapi.com/v1/get` |
| requestId | string | A unique request key that you can use to check the task status and retrieve the result of the scraper or renderer.                |




#### https://sigmaapi.com/v1/get
This method displays the current status of your task in the queue.

**Request example:**

```python
import requests

requestId = '7466d7d6-0000-4000-ab00-01a000028497'
# get source of the site from queue by request_id
response = requests.get("https://sigmaapi.com/v1/get?requestId=" + requestId,
                        headers={'X-Auth-Token': '<YOUR_API_KEY>'})
if response.status_code == 200:
    data = response.json()
```

| Parameter | Type    | Required | Description                                                                                                                                                                               |
|-----------|---------|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| api_key   | string  | yes      | Your personal API key. If you don’t have one, please [sign up](https://sigmaapi.com/accounts/register/) first. You can skip this parameter if you pass the `X-Auth-Token` header instead. |
| requestId | string  | yes      | Use the requestId you received from the _/v1/add_ request                                                                                                                                 |
| format    | string  | no       | By default, the method returns the page content in JSON format. If you pass `format=seo` or `format=html`, it will return the page’s HTML instead.                                        |


**Response example:**

```json
  {
    "uri":"https://sigmaapi.com",
    "status":"done",
    "sourcePage":"",
    "cookies": ""
  }
```

| Parameter  | Type   | Description                                                                                                                                     |
|------------|--------|-------------------------------------------------------------------------------------------------------------------------------------------------|
| uri        | string | Site URL (with http or https)                                                                                                                   |
| status     | string | There are three statuses: "in queue", "fail", and "done". In case of an error, the status "fail" will be returned along with the error details. |
| sourcePage | HTML   | The page’s source code or an empty string.                                                                                                      |
| cookies    | string | A list of cookies or an empty string.                                                                                                           |



#### Without X-Auth-Token
If, for some reason, it’s not convenient for you to pass the key through the header,
you can send it as a parameter `api_key=<YOUR_API_KEY>` in your API request.

_Example:_
```python
import requests
import urllib.parse

your_uri = urllib.parse.quote("https://sigmaapi.com")
response = requests.get("https://sigmaapi.com/v1/add?uri=%s&api_key=%s" % (your_uri, '<YOUR_API_KEY>'))
```
