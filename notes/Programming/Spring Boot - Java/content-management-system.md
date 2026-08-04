# 🕵️‍♂️ Social Media Feed
> An app that has CRUD feature for Post and Comment (A CMS)

Link: <https://github.com/Bernardusz/content-management-system>

## 💻 Tech
- Spring Boot
- JdbcClient
- Virtual Threads
- Spring Security
- Analog.js
- PostgreSQL
- Rotating Refresh Token

## 🌟 Goal
Master Rotating Refresh Token

## ❓ Why
Understand what ORM hide from us, and the tradeoffs

## What I Learned
1. Access Token are supposed to be short lived, and Refresh Token are long live and often rotated
2. You should use @PreAuthorize, but I learnt too late 💀🐧
3. JwtAuthenticationFilter takes the data from the Access Token, and populates the Security Context holder
4. SecurityContextHolder is actually a ThreadLocal. It behaves like a Singleton but is unique per Thread. So for Coroutines, it is antirely different story
5. Analog.js automatically configure load and action inside .server.ts. It acts like a route your frontend will call via form or injectLoad.
6. I just realized, not all pages want to be SSRed, every user's page is unique, and sometimes the best way is to let SPA do its job.
7. Analog.js relies on Vite. So sometimes caches of Vite will still hold on to the old files' name or deleted files. So deleting caches sometimes is the way to go. 