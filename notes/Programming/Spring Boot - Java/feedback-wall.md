# 👥 Multi-User Feedback Wall

Link: <https://github.com/Bernardusz/feedback-wall.git>

> A public wall where users can post messages.

## 💻 Tech
- Spring Boot
- JdbcClient (Raw SQL)
- Thymeleaf Fragments
- PostgreSQL
- `@Transactional`

## 🌟 Goal
A public wall where users can post messages.

## ❓ Why
Learn Transactions

## 🐧 Post-Mortem
1. I learned about the `@Transactional` annotation and how it works.
2. I learned about AJAX Request, for a bit of interactivity
3. I learned about `service/` package, and how Controller should talk to Service, Service talks to Repository, and Repository talks to Database.
4. I learned about `@RestController` and `@RequestMapping` annotations.
5. I learned about Request DTO for sending Request over to the server.
6. Summary DTO for sending Response without everything over to the client.
7. And how ForeignKey doesn't automatically bring comments to Post, and we need PostDetails