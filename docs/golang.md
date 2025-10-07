# Usage with Go

You can collect data for your projects from any website.
Sigma API supports JavaScript-heavy sites, with built-in protection bypass.

```go
package main

import (
	"context"
	"encoding/json"
	"io"
	"log"
	"net/http"
	"time"
)

type FirstResponse struct {
	Url       string `json:"url"`
	Status    string `json:"status"`
	RequestId string `json:"requestId"`
}

type SecondResponse struct {
	Url        string `json:"url"`
	Status     string `json:"status"`
	SourcePage string `json:"sourcePage"`
}

func addToQueue(uri string) FirstResponse {

	req, err := http.NewRequest("GET", "https://sigmaapi.com/v1/add?uri="+uri, nil)
	if err != nil {
		log.Fatalf("Error creating request: %v", err)
	}

	// Execute the request
	client := &http.Client{Timeout: 10 * time.Second}
	resp, err := client.Do(req)
	if err != nil {
		log.Fatalf("Error performing request: %v", err)
	}
	defer resp.Body.Close()

	if resp.StatusCode != 201 {
		log.Fatalf("API returned non-OK status: %s", resp.Status)
	}

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatalf("Error reading response body: %v", err)
	}

	var response FirstResponse
	err = json.Unmarshal(body, &response)
	if err != nil {
		log.Fatalf("Error unmarshaling JSON: %v", err)
	}

	return response
}

func getSourceOfPage(requestId string) SecondResponse {
	req, err := http.NewRequest("GET", "https://sigmaapi.com/v1/get?requestId="+requestId, nil)
	if err != nil {
		log.Fatalf("Error creating request: %v", err)
	}

	// Execute the request
	client := &http.Client{Timeout: 10 * time.Second}
	resp, err := client.Do(req)
	if err != nil {
		log.Fatalf("Error performing request: %v", err)
	}
	defer resp.Body.Close()

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		log.Fatalf("Error reading response body: %v", err)
	}

	var response SecondResponse
	err = json.Unmarshal(body, &response)
	if err != nil {
		log.Fatalf("Error unmarshaling JSON: %v", err)
	}

	return response
}

func main() {

	yourURL := "https://github.com/abbasovalex/sigmaapi.com"
	r := addToQueue(yourURL)
	requestId := r.RequestId
	log.Println(requestId)

	// Create timeout context
	timeoutCtx, cancel := context.WithTimeout(context.Background(), time.Second*360)
	defer cancel()
	ticker := time.NewTicker(2 * time.Second)
	defer ticker.Stop()

	// Poll until completion or timeout
	for {
		select {
		case <-ticker.C:
			r := getSourceOfPage(requestId)
			if r.Status == "done" || r.Status == "failed" {
				log.Println(r.SourcePage)
				return
			}
		case <-timeoutCtx.Done():
			log.Println("Timed out. Try later.")
			return
		}
	}
}
```