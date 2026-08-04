# 🖥 Dashboard & Relational notes
> A dashboard with Stat Cards

Link: <https://github.com/Bernardusz/character-stat>

## 💻 Tech
- Spring Boot
- Virtual Threads
- JdbcClient (Raw SQL) + SQL Aggregations + SQL `JOIN`s
- Analog.js (SSR + CSR)
- PostgreSQL

## 🚩Constraint
Complex Read Queries; Try to get as much data as possible with less query and Handle many-to-one relationships manually

## 🌟 Goal
A dashboard with Stat Cards

## ❓ Why
Mastering complex Raw SQL reads

## My Struggle and Limitations 🐧

1. Time: Last week I couldn't code because I was tasked with my school to go on a retreat and Eucharist seminar. Alongside I am the leader for my class Enterprenur team for the classmeet (We lost 🐧💀)
2. SSR is NOT Possible: For Profile to work we need localStorage which doesn't exist on the server.
3. Messy codebase: Tired and Sleep-depraved, I did some questionable arcitectural choice which is explained in detail in my video.
4. Naming: The second hardest thing in CS... And I can confirm 🐧💀

## What I learned

1. That aggeragtion Read deserves its own package by feature (dashboard)
2. RelationalQuery translates snake_case to camelCase.
3. For querying using JdbcClient, you don't need to always pass in a DTO and let them handle the work. You can pass in a lambda and aggregate the data yourself and finally returning the data based on a DTO.
4. Main constraint in Applications is not the backend, it is Database I/O and Networking. The more data that travels through the network, the slower it'll be. That's why N+1 Problems need to be solved by `JOIN`s
5. You have `@Input` directive for path param and query param. But in `app.config.ts` you must add the config: `provideFileRouter(withComponentInputBinding())`.
6. File name matters in File-based routing. Inside profiles/ to get a route /profiles I need to do `profiles/index.page.ts`. Though I feel like we should do `(profiles)/profiles.page.ts`
7. Unlike `input`, `output` doesn't have `.required` method to throw an error when it isn't passed. Which I learned the hard way when my button did nothing 💀🐧
8. When you use `HttpClient`, we have 3 important methods/functions: `pipe()`, `switchMap()`, and `subscribe()`
9. `pipe()` basically means, when I am called, do this also. And inside pipe, when we want do another request, we usually use `switchMap()` to return another http request.
10. When you return the result of HttpClient request, it is `Observable<T>`. And to actually fires the request, we need to use `subcribe()`. Inside subcribe you can pass in `{next: dataYouGet => {}, error: error => {}}` to execute what to do after you get the data.
11. Service should generally only be providing method for Http Request, and the page should be the one firing subcribe and aggregating data because the page is the one that understand what to do and what is the shape of the data. I did another questionable by putting subcribe inside IndexService.