# 🏥 Modern Healthcare & Medical Billing Audit System  - Week 3 - 6 of Spring Boot Kotlin
> An app that simulates an Enterprise Healthcare & Audit Compliance Platform. With Batch Data Processing, Strict AuthN & AuthZ, PDF and Excel reports

Link: <https://github.com/Bernardusz/healthcare-billing>

## 💻 Tech
- Kotlin
- Coroutines
- Spring Boot
- Spring Boot WebFlux
- Spring R2DBC
- `runBlocking{}` 💀🐧
- Analog.js
- PostgreSQL
- Gradle
- OpenPDF (PDF Generation)
- Apache POI (Excel Generation)
- Docker + Docker Compose (Pray for my limited Mobile Data 🙏🐧💀)
- Nginx (Not manual; rely on docs and AI for config)

## 🌟 Goal
Step up my game and try Data production for real software. Alongside trying Docker Compose and Nginx to containerize my apps (Not Spring's Support for Docker, but instead just pull the source code and type: `docker compose up -d --build`, and it runs)

## ❓ Why
Data should not be proprietary inside software; it should be distributable and shareable... For the Open Source movement... or Enterprise needs

## Post-Mortem
> I am gonna be transparent, I failed. Okay, I didn't finish this week's goal. I had competition and other matters I tended to before my learning project, and I just realized something, I am bad at time management. So... I didn't really learn anything new beside docker. What I learned really:

1. Docker container is a "fresh OS" so PNPM doesn't really work or give benefit except security and multiple container building if you use Docker volume and store deps there
2. Building a backend is... let's say for the least not an easy task. Early on in my Spring Boot journey a 2 weeks journey is manageable because I am still early on. Now I have became more busy, and let's say 2 weeks are not enough anymore
3. I really need to work with my planning. This looks manageable until assignment hit, competition hit, and everything just came crashing down.
4. WebFlux... Let's say it's overly complex. For app like this I should've gone with Spring MVC Tomcat and Virtual Threads. The learning phase has ended, Tomcat for speed and delivery, I love it.
5. Kotlin on Spring Boot feels modern, it definitly offer DX far better than Java. It's just I feel weird in it. It maybe a bias, but I prefer DX and explicitness over writing less line of code. I'll definitly be working with Kotlin in the future, especially when I want to do Jetpack Compose Multiplatform down the road, it's just I prefer Java for a standalone backend.
6. R2DBC acts similiarly to Hibernate, it auto generates query based on method name, but doesn't support Lazy Loading, which is fine and good for explicitness.
7. Mono represent a data to be fetched like Await variable. Flux represents a stream of data to be sent. Mono is replaced by Coroutine because Coroutine offloads Coroutine waiting for DB's work
8. And lastly, I appreciate abstraction more. Explicitness is one thing, but developer speed is another. When I have to manually wire up things when Virtual Threads can handle this... is exhausting. I appreciate explicitness in Business logic, not configuration.

At last, the backend works. But without a frontend, and I can't even use prod docker-compose.yaml because I'm on limited mobile data