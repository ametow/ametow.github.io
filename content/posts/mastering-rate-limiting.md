---
title: "Rate limiting"
date: "2025-03-10"
description: "Rate limiting fundamentals and Go implementation"
summary: "Rate limiting fundamentals and Go implementation"
tags: ["golang", "rate limiting"]
categories: ["golang", "system-design"]
ShowToc: true
TocOpen: true
---

## 🌟 What is Rate Limiting?

Imagine you're at your favorite coffee shop. The barista can only make so many drinks per minute. If too many people order at once, they have to wait or come back later. ☕️ That's exactly how rate limiting works in the digital world!

Rate limiting is a technique used in software systems to **control the number of requests** a user, IP address, or service can send within a given time frame. This helps prevent spam, abuse, and overloading of resources. 💡

## 🎯 Why Do We Need Rate Limiting?

Rate limiting isn’t just some arbitrary restriction—it plays a **crucial role** in keeping online systems stable, secure, and fair. Here’s why it matters:

1. **🛑 Prevents Abuse & Spam** – Stops a single user from making too many requests and overwhelming the system.
2. **⚖️ Ensures Fair Usage** – Prevents a few users from hogging all the resources so that everyone gets a fair chance.
3. **💻 Protects Server Resources** – Helps prevent **server crashes** and keeps things running smoothly.
4. **🔒 Enhances Security** – Mitigates threats like **brute-force attacks** and **DDoS attacks** that can take down services.

### 🕵️‍♂️ Real-World Examples of Rate Limiting

- **Login Attempts** 🔐: Many websites limit login attempts to prevent hackers from guessing passwords.
- **API Requests** 📡: Services like Twitter, Google, and GitHub limit how many requests you can make per second.
- **Online Ticket Booking 🎟️**: Websites prevent bots from buying up all the tickets for concerts and events.

## 🔄 How Does Rate Limiting Work?

Rate limiting works by keeping track of recent requests. If a user sends **too many** within a short time, some requests get blocked.

There are a few common strategies:

### ⏳ Fixed Window

- Imagine a store that allows only **10 customers per hour**. If 10 people enter at 10 AM, no more are allowed in until 11 AM.
- Simple but can cause spikes in traffic at window resets.

### 📊 Sliding Window

- Like a rolling **queue** where we count requests over the last 60 seconds, regardless of when they occurred.
- More **precise** and **fair** than the fixed window.

### 💰 Token Bucket

- Users start with a **set number of tokens** (requests).
- Tokens regenerate at a fixed rate.
- A request can only be made if there are enough tokens available.
- This is often used for APIs to allow occasional bursts of requests.

## 💻 Implementing Rate Limiting in Golang 🎉

Now, let’s get our hands dirty with some Golang code! We will implement a simple rate limiter using a **map** to track requests per IP and a **queue** to manage time-based limits.

```go
package main

import (
	"fmt"
	"container/list"
)

type RequestLog struct {
	requests *list.List
}

func rateLimiter(timestamps []int64, ipAddresses []string, limit int, timeWindow int64) []int {
	requestLog := make(map[string]*RequestLog)
	result := make([]int, len(timestamps))

	for i, timestamp := range timestamps {
		ip := ipAddresses[i]
		if _, exists := requestLog[ip]; !exists {
			requestLog[ip] = &RequestLog{requests: list.New()}
		}
		log := requestLog[ip]

		// Remove outdated requests
		for log.requests.Len() > 0 {
			front := log.requests.Front()
			if front.Value.(int64) < timestamp-timeWindow {
				log.requests.Remove(front)
			} else {
				break
			}
		}

		// Check if we can accept the request
		if log.requests.Len() < limit {
			log.requests.PushBack(timestamp)
			result[i] = 1 // Accept request ✅
		} else {
			result[i] = 0 // Reject request ❌
		}
	}

	return result
}

func main() {
	timestamps := []int64{1600040547954, 1600040547957, 1600040547958}
	ipAddresses := []string{"127.105.232.211", "127.105.232.211", "127.105.232.211"}
	limit := 1
	timeWindow := int64(3)
	result := rateLimiter(timestamps, ipAddresses, limit, timeWindow)
	fmt.Println(result) // Output: [1, 0, 1]
}

```

### 🚀 How This Code Works:

1. **📝 Tracking Requests** – We store each IP's request timestamps in a queue.
2. **🗑️ Removing Old Requests** – We remove requests that are outside the allowed time window before processing new ones.
3. **✅ Accepting or ❌ Rejecting Requests** – If the number of requests in the time window is below the limit, we accept the request (`1`). Otherwise, we reject it (`0`).

## 🏆 Enhancements & Next Steps

If you want to make this **even better**, try:

1. **🌍 Using Redis** – For distributed rate limiting across multiple servers.
2. **📈 Adaptive Rate Limiting** – Adjust limits dynamically based on user behavior.
3. **🚧 Implementing Token Bucket Algorithm** – To allow burst requests.
4. **🕒 Logging & Monitoring** – Keep track of rate limit violations for security analysis.

## 🎉 Wrapping Up

Rate limiting is a **powerful tool** that keeps services running smoothly and fairly. Whether you're building an API, a website, or a login system, adding rate limiting can **protect your server** while ensuring a great user experience. 🏆

💡 **Now it's your turn!** How would you implement rate limiting in your own projects? Let’s discuss! 🚀