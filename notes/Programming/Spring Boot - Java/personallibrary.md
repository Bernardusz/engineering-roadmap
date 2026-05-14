# 📚 Personal Library Manager

Link: <https://github.com/Bernardusz/personallibrary>

> A simple Spring Boot app, as my first project in the Spring Ecosystem

## 💻 Tech

- Spring Boot
- JdbcClient (Raw SQL)
- Thymeleaf
- PostgreSQL

## 🌟 Goal

Build a CRUD app where I can add books.

## ❓ Why

Get comfortable with Spring Boot

## Post-mortem:

1. I made my first Spring Boot app.
2. Model shouldn't exist, it should've been DTO and Entity that represents SQL Table. But for something like this, it's
   overkill
3. Repository is the only layer that talks to the Database, which I utilized JdbcClient, So I can write RAW SQL.
4. The last one is Controller, it handler your incoming Request, and by using the Repository
5. Model in Controller is the carrier of data between Controller, and later passes it to Thymeleaf Template
6. Remember Spring follows MVC (Model-View-Controller) model, so Model is `model/` (DTO and Entity), Controller is
   `controller/`, `repository/` is the Data Access Layer between Controller and Database, and View is `templates/` (
   Thymeleaf) sent via Controller.
7. I used raw SQL here via JdbcClient, so everytime I use the repository it executes a SQL command to my PostgreSQL
   database. I make sure to inject the command with ? instead of String Interpolation. SQL Injection NO NO 🐧
8. added GlobalExceptionHandler to handle errors by sending `error.html` with what went wrong, so it doesn't return a
   blank page

> And that's it 🐧💀