# 🔎 Secure Search & Filter Vault Engine
> An app to store your passwords safely, and the ability to search them.

Link: <https://github.com/Bernardusz/vault>

## 💻 Tech
- Spring Boot + Spring Security
- Virtual Threads
- JdbcClient (Raw SQL) + Dynamic SQL
- Analog.js (SSR + CSR + Nested Routes + Analog.js Signals + Analog.js Interceptors)
- PostgreSQL

## 🌟 Goal
- A product list with a search bar that filters as you type
- Learn how to protect my API in Spring Boot

## ❓ Why
Learn how to build dynamic queries with JdbcClient without the help of an ORM and learn the wall between Frontend and Database

## What I learned
1. Spring Boot "isn't a backend." It is an Inversion of Control machine. It's main job is Dependency Injection. Spring Boot Web is what transforms Spring Boot into a backend
2. Another reason Spring Boot is "slow" or has a bad cold start is because of its nature: an IoC and DI machine. It needs to scan annotations (which in itself doesn't do anything, just marks the functionality) and do mathematical and smart guess so you don't have to write verbose Spring XML configs 💀🐧
3. Bean is an Object managed by Spring Boot's IoC. You create one by either marking a Class with nearly all annotations in Spring Boot. Or giving annotations `@Bean` to a method that returns an Object. That instantiated Object is a Bean. So yep, Spring Boot is one massive Veggies soup chef 💀🐧
4. In Spring Boot, once Tomcat BEAN 🐧💀 Takes control, it doesn't even know anything about our BEANS 💀🐧
5. That's why Spring register a Filter inside Tomcat that basically says let us handle the intercepting before you do anything.
6. And then it execute the SecurityFilterChain inside your SecurityConfig.
7. And inside that SecurityFilterChain you can register your own Filter or middleware.
8. Spring Boot doesn't have a custom JWT starter, either you do session based or you do OAuth 2.0 with third party provider
9. Password hashing is always deterministic. Even if you 12346 turns into a massive long string, every instances of 123456 will always be that long string. This is a vunerability known as The Rainbow Table. That's where Salting is added on top of Hashing.
10. Encryption is different that Hashing. In my app Hashing is used for User password. Because hashing is always deterministic, you can hash it before storing it to DB and check whether user input turned into long unintelligible string is the same in the DB
11. Encryption can be Decrypted to return to its original value. Proven in TLS and SSL
12. The reason JWT and Encryption need a secret key is because of safety. When a hacker or malicious user tampers with the data, the signature will break. This secret key is used alongside your user id and a specific moving data (issued at, and JWT Id, etc) to make sure every JWT token is unique
13. But for Encryption we have IV. It is stored in the DB alongside your encrypted data. So that's why the same pass, encrypted (not hashed) is different. Even if you broke into the DB everything is encrypted. And if the hacker manages to get into your backend, .env, and DB.... just give up bro 💀🐧
14. JWT actually should have 2 tokens; Access Token and Refresh Token, in my implementation we only have Access Token with 10 hours life time (BAD 💀🐧).
15. So a bit of history. At the start of web when PHP is cool and the Backend returns HTML we use Session based Cookie and life was good, until the SPA kingdom arrives 🐧💀
16. We starts using SPA which makes Session based Cookie less preffered and gave rise to JWT because of its statelessness, so we have to use JWT. We use Authorization Bearer token, store it in Local Storage. Until we realize... SEO is bad and XSS is rampant because of Local Storage. 💀🐧
17. So we start creating a SERVER SIDE RENDERING JavaScript framework. And another bottleneck, localStorage doesn't exist in the server. So now we are back full circle to HtppOnly Cookie and TWO Servers for the Backend as a stateless JSON API and Frontend as an interactive and pretty server that SENDS HTML 🐧💀 Okay, in Next.js it does sends HTML and only hydrate client component to send as little JS as possible, but in Analog.js and Nuxt.js are Isomorphic, so the server will send HTML for SEO and in the end we hydrate the whole page anyway, turning them into SPA 💀🐧 Thoguh you can select partial hydration and use ISG
18. So from my nerding about history 🐧💀 You can see we now use Cookie based Auth to store JWT Access Token, this is done so that JavaScript can't access it (You need to send the whole header if you want to 💀🐧). After authentication (Authn) that cookie will be send with every request.
19. Spring Boot interpret that and you now can access `UserDetails` vian `@AuthenticationPrincipal` to access your user.
20. In a real secure JWT based arcitecture, we have a separate Table in DB to store Hashed Refresh Token. Usually this is also Salted. And when the Short lived Access Token dies, the frontend hits the endpoint for refresh. Your backend generates a new Access Token AND Refresh Token, sends them both to your client. 
21. GlobalExceptionHandler can either be Rest Exception Handler (`@RestControllerAdvice`) and sends a page (`@ControllerAdvice`)
22. You can create guards in Analog.js that will be 'subcribed' into when you visit the page. So Angular's router allows CanActivateFn functions to return a few different things: a boolean, a UrlTree, a Promise, or an Observable<boolean | UrlTree> that will be "subscribed" into.
23. And then what are Observable and its friends? Observable is a blueprint for a data stream that emits values asynchronously. So like a Promise but over a span of time.
24. You use pipe to chain events (like last week's explanation) but then we have `map()`, `tap()`, and `switchMap()`
25. `tap()` is like a middleware/filter that can't modify the data. It can look into the data but can't edit it in any way.
26. `map()` takes the data from Observable, runs it through a function to modify it and passes the new transformed value down the pipe.
27. Now `switchMap()` takes the data and allow you to do another asynchornous task and return a new Observable
28. Now to create an interceptor, which is just another Middleware/Filter like Spring Boot per request 💀🐧 You create an Interceptor and sign it in `app.config.ts` - `provideHttpClient([])` and from now on for every single Http request will pass through this.

## What I Went Through 🐧💀
1. I kept getting 403, no matter what and was frustrated 💀🐧 That's because Spring Boot by default masks error for safety. So that's why you need to specify you're developing and want to see the trace.
2. I also got 415, because every single data sent to Spring Boot Rest Controller via `@RequestBody` needs to be `application/json`.
3. Order of passing is important in Java 🐧💀 I had my password and description switched in my Service via wrong order, so my password is shown as desc and vice versa 💀🐧
4. Vite now has built in `tsconfigPaths: true` so now I can relax and use my `@` alias for src 🐧🐧🐧