# 🎫 Event Booking & Ticketing Engine - Week 1 & 2 of Spring Boot Kotlin
> An app that is used to book events to make sure booking stays strong during peak hours.

## 💻 Tech
- Kotlin
- Spring Boot
- Spring Data JPA + Hibernate & JdbcClient
- Virtual Threads
- Bruno
- PostgreSQL
- Maven
## 🌟 Goal
Let Hibernate manage the table creation and relationship mapping. While we handle the hard part with JDBC

## ❓ Why
Understand what ORM hides from us, and the trade-offs... Alongside a farewell project from JdbcClient 🐧💀

## 👣 Footnote
> I am late in finishing this project. And I am horrendously tired. So I am switching my target from Analog.js to learning Bruno for this week. Sorry for... my procrastination 🐧🙏

## What I learned 💀🐧
1. Spring Boot has a built-in Scheduler for CRON job for cleaning up the Database
2. Spring Data JPA is the abstraction on top of everything. Spring Data JPA is like Spring Boot on top of Spring Framework
3. JPA Specification is a specification. The annotation of `@Entity`, `@Id`, etc. It defined how relational database tables.
4. JPA Provider or Hibernate is the actual engine, the implementation of JPA Specification.
5. Hibernate takes your annotate entity classes and generate the necessary SQL queries dynamically.
6. Near the database is JDBC, this is the one that connects DB to your code.
7. Now one of the feature of ORM, so Spring Data JPA, is to translate database data into Object in a programming language.
8. Now, one quirk I understand in Spring JPA is that there is no Update. In Hibernate we have what we called Dirty checking. In `@Transactional` service method, JPA compared the Object you get from DB and its actual state now. If it is different Hibernate creates the SQL dynamically.
9. And in JPA you can directly LEFT JOIN in the Entity. So you can `@ManyToOne` or `@OneToMany` in Entity and eagerly JOIN or lazily fetch (source of N+1 🐧💀)
10. I am still a beginner in Bruno, and I tested my Spring Boot app optimistically. So... this time, I use Bruno as mock frontend only.
11. And only now I have realized, that in order for Spring Devtools to work, you need to be able to trigger the rebuild. All this time I use the maven wrapper on terminal instead of starting in the IntelliJ. That's why my code didn't restart. 