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
import time
import urllib.parse

your_url = urllib.parse.quote("https://github.com/abbasovalex/sigmaapi.com", safe='')
# add to queue
response = requests.get("https://sigmaapi.com/v1/add?uri=%s" % your_url,
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
import time

requestId = 'ff890ff7-90b3-4188-93d1-a10060bfd123'
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
| format    | string  | no       | By default, the method returns the page content in JSON format. If you pass `format=seo`, it will return the page’s HTML instead.                                                         |


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
| cookies    | string | A list of keys or an empty string.                                                                                                              |



#### Without X-Auth-Token
If, for some reason, it’s not convenient for you to pass the key through the header,
you can send it as a parameter `api_key=<YOUR_API_KEY>` in your API request.

_Example:_
```python
import requests
import time
import urllib.parse

your_url = urllib.parse.quote("https://github.com/abbasovalex/sigmaapi.com")
response = requests.get("https://sigmaapi.com/v1/add?uri=%s&api_key=%s" % (your_url, '<YOUR_API_KEY>'))
```
