# Jupiter / Python Example

You can collect data for your projects from any website.
Sigma API supports JavaScript-heavy sites, with built-in protection bypass.

```python
import requests
import time

your_url = "https://github.com/abbasovalex/sigmaapi.com"
# add site to queue
response = requests.get("https://sigmaapi.com/v1/add?uri=%s" % your_url)
if response.status_code == 201:
    json = response.json()
    request_id = json["requestId"]
    
time.sleep(60)
    
# get source of the site from queue by request_id
response = requests.get("https://sigmaapi.com/v1/get?requestId=" + requestId)
if response.status_code == 200:
    data = response.json()
```

*Example screenshot of Jupiter:*
![Example usage from Jupiter Editor](/images/jupiter_example.png "Usage with Python over Jupiter")
