# 📃 Wait-Less Task Board
> A simple Kanban board, utlizing Analog/Angular ecosystem.

## 💻 Tech
- Spring Boot
- Virtual Threads
- JdbcClient (Raw SQL)
- Analog.js
- PostgreSQL

## 🚩Constraint
Force a 2-second delay in Frontend, not SQL, service (Debouncing). I worded that one wrong 🐧💀

## 🌟 Goal
A public kanban board

## ❓ Why
Bridge the gap between Java 21 performance and Analog.js data fetching

## Post-Mortem

1. CORS issue still exist, as browser by default restrict outside request for Safety
2. In modern Spring Boot, we use Package by Feature so everything in relation to a feature is grouped neatly
3. I learned about how we need DTO in Java, as Java is a strictly typed language, and we need to map a DTO to incoming request
4. I learned about JdbcTemplate, and how powerful is that we can execute multiple SQL commands in one go to avoid multiple I/O to DB.
5. A single method in repository should only do one thing. Service is the orchestrator between these commands.
6. Controller should only handle incoming Request, and call the appropriate service method
7. I learned about Angular strcuture, and how my React/Next.js habit of {} for variable is affecting a lot 🐧💀
8. Routing inside (directoryName) doesn't affect the routing and only help to group features up.
9. I learned about "props" passing using input (Incoming) and output (Output - method)
10. Analog/Angular input/signal must be called like a function, because they... are 💀🐧? title() will return the value.
11. It's only now that I realize we only need Server Side Fetching, because we need to populate the website with data. Once it gets into the client, it's SPA
12. Unlike Next.js, we can use signal inside a "Server Component." That is because Analog.js follows Isomorphic model, where there is only HTML on the server, but once given to the client, HTML will hydrate. Giving a better DX overall.
13. I love the structure of Analog.js, it's nice. But because we only use Typescript and raw HTML for Angular templating, we don't get intellisense. I haven't checked the VS Code market place, but that's my state until now.
14. We should've used Server Side Form, but I encountered a problem along the way. So I used raw form and `handleSubmit()` function inside `task-update.component.ts`