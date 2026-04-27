## Socket Basic - Day 1 🐧
> **Shalom!** 🐧 As usual, as a Highschool student, I'm tired 💀🐧. So let's get this soon ASAP.

### 1. "Everything is a file!" - Unix & Linux, probably 🐧💀
> So, throwback to [[System Calls (Syscalls)|System Calls]], you should already understand that whether you're talking to a disk, or a file, or your GPU 🐧💀 You use the same *lingua franca* of System Calls: `open()`, `read()`, `write()`,`close()` and maybe ` mmap()` but that's besides the point. Unix did this so how does programmer talk to many things remain the same. Whether it is a file, driver, and socket 🐧⚽.

### 2. "What is a server?" - Backend and Server, probably 🐧💀
> So we know that Backend lives in a **computer server**. Let's debunk this madness.

#### Server & Computer Server
> These are two distinct things. Computer server is a computer that hosts your Backend/server. It is the **Hardware**
> While Server is a program that serves other programs. It is the **Software** that is bind to a certain ports, ready to accept request

#### Socket & FD
> So, In Unix and Linux, everything is a file ([[System Calls (Syscalls)|System Calls]]). You write to a buffer. And the OS will handle the TCP/IP processing, breaking down what is in the buffer (data) to send the segments, adding addressing information through the network depending on the destination
> Socket is the one FD pointed to. Think of it like a bidirectional highway specialized for communication between client and the server. it manages communication between client and server. When you `write` to socket FD, it means you're dumping the data to the socket buffer.
> Then the OS will take that data, breaks it down following the TCP/IP protocol, and sends it down the next layer, which is located in the OS will tape the IP and Mac Address. Travels down the switch to the Router, to Modem and travels to the rest of the world.

Suddenly OSI layers make sense to me because of this 🐧💀

### 3. "How many people can I talk to?" - 🐧❓
> This is a quick section, because I am tired 🐧💀 So in IP Address (Which I learned in TKJ 💀🐧) you know that `127.0.0.1` is reserved for loopback IP, what is it? When you bind your program to this IP, it means it's bind to localhost, and limited to all the programs inside your host/PC
> While `0.0.0.0` means bind to all available network interfaces. It can also mean this host on the network. Which will make your server accesible for anyone in the network if they access your IP Address on the network and the port. Or in other terms: "I don't care whether they come from the front door (Network), backdoor (Ethernet), or even the window (VPN). As long as it's my room (Port), let them in." - 🐧💀

### 4. Java Socketing - ServerSocket & Socket
> So in C, we need to bind our process to a port and starts listening. But Java already handles that with `ServerSocket`. It automatically binds your program to a Port and listen, though to accept connection you need to do `socket.accept()`
> You must know that `.accept()` is a blocking call. Your program will halt until a client connects.
> And once a Client connect you will get a `Socket` object, resembling the FD for each connection the Socket maintains. It writes to the FD.

But how do we not run out of FD? Modern VPS can change their limit of FD from 1024 to 65,535 or even 1,048,576. And the size of each Socket is so small, you don't even need to worry.

BUT, You still need to close each connection once they're finished. Because File Descriptor leak is a real problem and "I'll close it later" might be the last words your server will ever hear 💀🐧

```java
import java.io.BufferedReader;
import java.io.FileWriter;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;

public class Main {
    public static void main(String[] args) throws Exception {
        try (ServerSocket serverSocket = new ServerSocket(8080)){
            while (true) {
                Socket clientSocket = serverSocket.accept();
                System.out.println("Client connected: " + clientSocket.getInetAddress());

                try(
                    BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
                    FileWriter fileWriter = new FileWriter("messages.txt", true);
                    PrintWriter printWriter = new PrintWriter(clientSocket.getOutputStream(), true);
                ){
                    String line;
                    while ((line = bufferedReader.readLine()) != null) {
                        System.out.println("Client: " + line);
                        fileWriter.write(line + "\n");
                        printWriter.println("Echo: " + line);

                    }
                }
            }
        }
        catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

We use BufferedLine so we can grab and read per line and not spamming the CPU with System Calls 🐧💀

We use PrintWriter so we don't need to flush manually nor forget it 🐧💀 and so that Error doesn't brick the request, and PrintWriter swallows the error and silently fail. Though it is actually bad. In a server you want to catch exception and severs the bad request immediately.

BUT, You remember `.accept()` is blocking, and after connected we immediately serve the first client. So if another client connect, they... will be put in a waiting list (50-128 slots) or ignored (Waiting room filled or client's own timeout) 💀🐧

That's it for day 1, I am busy as hell and tired as hell... And why there are so many penguin? 🐧🐧🐧

Penguin can stand alone. But Skull... somehow I feel it's incomplete without penguin 🐧💀

## HTTP Request - Day 2🐧📡 
> As usual, tired as hell 🐧💀And I woke up early, so today we'll be covering 2 materials... so letś get this finished ASAP.
### 1. TCP & HTTP - Request Structure 🐧📰
> So we've heard about HTTP request and TCP/IP, but what are they actually?

#### TCP/IP - The Handshake Protocol 🐧🤝🐧
> In Networking, TCP/IP is the protocol in which HTTP Is also a protocol... But they are different 🐧
> TCP/IP is the protocol in which handles how the data is sent (OSI Layer - Layer 4). It primarily is the protocol for breaking and sending/reading data safely

#### HTTP - The Languge Browser Understand 🐧🗣
> Now HTTP is also a protocol, but primarily lives on the 7th layer of OSI Layer; Application layer.
> It is the protocol in which the data will be structured so that the browser could understand.

### 2. "Can I order a `index.html` ?" - HTTP, Probably 🐧💀
> So RESTful API was intented to follow HTTP protocol as much as possible. Not the other way around. So `GET`, `POST`, statuses came from HTTP.

HTTP Request usually only comes in 3/4 sections:

1. The Request Line
	> Remember that the server is passive? This line tells your server what to do. You need to parse this line. IT includes a verb (GET/POST/PUT/DELETE), the thing you wanna get, and the HTTP version.
2. The Headers
	> These are like extra information for someone: "My house has a ... color fence" or "I am verified, let me in"
3. The Empty LIne
	> HTTP Protocol dictates that headers must end with a double new line; `\r\n\r\n`
4. Optional Body for `POST`/`PUT` and `GET` response.

This is only for `GET` and `DELETE`. for `POST` and `PUT` will be disscused later, because we need to have `Content-Length` and `Content-Type`

```plaintext
GET /index.html HTTP/1.1
Host: localhost:8080               
User-Agent: Mozilla/5.0...
                                   
[Optional Body/Payload]
```

```java
import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;
import java.nio.file.Files;
import java.nio.file.Path;

public class Main {
    public static void main(String[] args) throws Exception {
        try (ServerSocket serverSocket = new ServerSocket(8080)){
            while (true) {
                Socket clientSocket = serverSocket.accept();
                System.out.println("Client connected: " + clientSocket.getInetAddress());

                try(
                    BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
                    FileWriter fileWriter = new FileWriter("messages.txt", true);
                    PrintWriter printWriter = new PrintWriter(clientSocket.getOutputStream(), true);
                ){
                    String line;
                    while (!(line = bufferedReader.readLine()).isEmpty()) {
                        if (line.startsWith("GET")) {
                            String file = line.split(" ")[1].trim();

                            if (file.equals("/")){
                                file = "/index.html";
                            }
                            String fileName = file.substring(1);
                            Path path = Path.of("static/" + fileName);

                            if (Files.exists(path)) {
                                long fileSize = Files.size(path);

                                printWriter.println("HTTP/1.1 200 OK");
                                printWriter.println("Content-Type: text/html");
                                printWriter.println("Content-Length: " + fileSize);
                                printWriter.println();
                                printWriter.flush();

                                Files.copy(path, clientSocket.getOutputStream());
                            }
                            else{
                                printWriter.println("HTTP/1.1 404 Not Found");
                                printWriter.println();
                                printWriter.flush();
                            }

                        }

                    }
                }
            }
        }
        catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

A simple `GET` server in Java.

### 3. "Are things okay down there?" - Status Code 🐧🚒
> So you might see 2xx and 4xx, what are those? Those are statuses, basically saying;

| Status                    | Analogy                                                           | Explanation                                                                                                             |
| ------------------------- | ----------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| 200 OK                    | "I found the file, here it is."                                   | The server finds your file and gives it.                                                                                |
| 201 Created               | "Your request is created, sir."                                   | The server successfully created your file.                                                                              |
| 204 No Content            | "I did your request, but I don't know what to say"                | The server successfully processed the request, but there is no data to send back.                                       |
| 400 Bad Request           | "You're speaking Gibberish 🐧💀"                                  | The serve can't underatand your HTTP line, or your HTTP request is broken/malformed                                     |
| 401 Unauthorized          | "INTRUDER!" 🐧🔫💀                                                | You forgot or have wrong Key for access                                                                                 |
| 403 Forbidden             | "Boss, this area is restricted. Even for you"                     | The server recognize you, but you have no access to the specific file you're seraching.                                 |
| 404 Not Found             | "You are asking for what?"                                        | Basically, the file you want doesn't exists                                                                             |
| 405 Method Not Allowed    | "You're asking for pizza here, in a Hungarian restaurant?" 🐧🍕💀 | Basically, your server only handles specific verbs like GET, but you ask for delete.                                    |
| 500 Internal Server Error | "We're on fire" 🔥🐧💀                                            | Generic crash code. Before your server shuts down you should send this so the browser doesn't hang                      |
| 501 Not Implemented       | "We doesn´t have that menu yet, boss"                             | Basically when someone asks for a HEAD or OPTIONS or unimplemented features and you haven´t written that code down yet. |
| 503 Service Unavailable   | "We're down for renovation" or "We are busy, sir" 🐧🏗🔥          | Either your server is too busy, or the server is down for maintenance.                                                  |

### 4. Content-Length & Content-Type - Add this or fire 🐧💀
> Not fire exactly, put for POST/PUT and response in GET you need these 2, so the browser can render things correctly.

Content-Length is only how many bytes/how big is it, but Content-Type... oh boy 🐧💀

| Type       | MIME Type/Content-Type     | Description                         |
| ---------- | -------------------------- | ----------------------------------- |
| HTML       | `text/html`                | The structure of your page.         |
| CSS        | `text/css`                 | The styling and layout              |
| JavaScript | `application/javascript`          | The logic/interactivity             |
| PNG        | `image/png`                | Lossless images.                    |
| JPEG       | `image/jpeg`               | Standard photos                     |
| GIF        | `image/gif`                | Animated or simple images           |
| SVG        | `image/svg+xml`            | Vector graphics (XML based)         |
| JSON       | `application/json`         | The standard for sending data       |
| XML        | `application/xml`          | Older data format                   |
| Plain Text | `text/plain`               | Raw text with no formatting         |
| Raw Bytes  | `application/octet-stream` | Usually used for downloading files. |

So with everything out of the way, let's write a blocking, single-threaded GET/POST/PUT/DELETE server.

```java
import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;
import java.nio.file.Files;
import java.nio.file.Path;

public class Main {
    public static void main(String[] args) throws Exception {
        try (ServerSocket serverSocket = new ServerSocket(8080)) {
            while (true) {

                PrintWriter printWriter = null;
                try (
                    Socket clientSocket = serverSocket.accept();
                    BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
                ) {
                    System.out.println("Client connected: " + clientSocket.getInetAddress());
                    printWriter = new PrintWriter(clientSocket.getOutputStream(), true);
                    String line = bufferedReader.readLine();
                    if (line == null) continue;

                    String[] parts = line.split(" ");
                    String method = parts[0];
                    String path = parts[1];
                    if (path.equals("/")) {
                        path = "/index.html";
                    }
                    Path filePath = Path.of("static/", path.substring(1));

                    int contentLength = 0;
                    String header;

                    while (!(header = bufferedReader.readLine()).isEmpty()) {
                        if (header.startsWith("Content-Length:")) {
                            contentLength = Integer.parseInt(header.split(":")[1].trim());
                        }
                    }

                    switch (method) {
                        case "GET" -> handleGet(filePath, printWriter, clientSocket.getOutputStream());
                        case "POST" -> handlePost(filePath, contentLength, bufferedReader, printWriter);
                        case "PUT" -> handlePut(filePath, contentLength, bufferedReader, printWriter);
                        case "DELETE" -> handleDelete(filePath, printWriter);
                        default -> sendStatus(printWriter, "400 Bad Request");
                    }
                }
                catch (Exception e) {  // catch for inner try
                    if (printWriter != null){
                        printWriter.println("HTTP/1.1 500 Internal Server Error");
                        printWriter.println();
                        printWriter.flush();
                    }

                    e.printStackTrace();

                }
                finally {
                    if (printWriter != null){
                        printWriter.close();
                    }
                }

            }
        }
    }
    public static void handleGet(Path path, PrintWriter printWriter, OutputStream outputStream) throws IOException {
        if (Files.exists(path) && !Files.isDirectory(path)) {
            long fileSize = Files.size(path);
            String contentType = Files.probeContentType(path);

            printWriter.println("HTTP/1.1 200 OK");
            printWriter.println("Content-Type: " + contentType);
            printWriter.println("Content-Length: " + fileSize);
            printWriter.println();
            printWriter.flush();

            Files.copy(path, outputStream);
        } else {
            sendStatus(printWriter, "404 Not Found");
        }
    }

    private static void handlePost(Path path, int length, BufferedReader bufferedReader, PrintWriter printWriter) throws IOException {
        char[] body = new char[length];
        bufferedReader.read(body, 0, length);
        Files.writeString(path, new String(body));
        sendStatus(printWriter, "200 OK");
    }

    private static void handlePut(Path path, int length, BufferedReader bufferedReader, PrintWriter printWriter) throws IOException {
        boolean exists = Files.exists(path);
        char[] body = new char[length];
        bufferedReader.read(body, 0, length);

        Files.writeString(path, new String(body));
        sendStatus(printWriter, exists ? "200 OK" : "201 Created");
    }
    private static void handleDelete(Path path, PrintWriter printWriter) throws IOException {
        if (Files.deleteIfExists(path)){
            sendStatus(printWriter, "204 No Content");
        } else {
            sendStatus(printWriter, "404 Not Found");
        }
    }
    private static void sendStatus(PrintWriter writer, String status) {
        writer.println("HTTP/1.1 " + status);
        writer.println();
        writer.flush();
    }
}
```

Next section.... which is today 🐧💀 We'll learn about Multi-Threaded server, and SSL and HTTPS to after this. Let me rest 💀

## Security and Concurency - Day 3 🔒🐧
> Well... It's the same day anyway... let's start.

### 1. "No sniffing around!" - SSL & TLS, probably 🐧💀
> So we know that data travels in bytes through the network. But anyone can take it. With tools anyone can sniff a normal HTTP request and take your data.
> That's why SSL & TLS and by extension HTTPS exist.

SSL (Secure Sockets Layer) is old an technically deprecated, so we'll talk about TLS (Transport Layer Security) only. Even though it's called "SSL Certificate," it actually means TLS.

So think of TLS like a lock. Everyone know the key/password to lock it, but there is another key/password to unlock it. So that, hacker they can take the bytes, but it's locked. And they only have the key to lock, what's the use in that 🐧💀

1. First time you visit a website, the browser sends a list of supporter encryption algorithms and a random number to the server (like salting called Nonce - Number used once. So replay attacks won't happen)
2. Your server picks the strongest algorithm and sends back its Public Certificate, the public locks taht follows the X.509 standard everyone have alongside:
	1. The Public Key - Lock
	2. Domain - Who are you
	3. Issuer Name - Who signed the certificate
	4. Validity Period - How long is this certificate valid
	5. Serial Number - A unique identifier by Certificate Authority
	6. Digital Signature - A mathematical proof that the data hasn't been tampered with
3. The browser checks the certificate. As a note, `localhost` where our server ran isn't owned by anyone, so CA can't really know it's you 💀🐧.
4. The browser creates a Pre-Master secret, and encrypts it with you public key. Only your private key stored in your `.p12` in the server can unlock it.
5. Your Server unlocks the locked Pre-Master Secret.
6. Both sides now have a shared Session Key, now all your HTTP request is locked behind this TLS tunnel, making it an HTTP Secure (HTTPS) 🐧🐧

And as a note, to prevent Replay Attacks, this process is repeated many times. A new tab, refresh, server restart, you name it.

SIKE 🐧💀 I just realize I explaned the outdated RSA. In modern TLS, We use Diffie-Hellman. Let me (try to) explain it aight 💀

We have 6 Variables; 2 private and 4 public.

1. We start with 2 numbers everyone can see and agree upon. p, a very large prime number (modulus) and *g*, a base number.
2. Then Client picks a secret number *a*, but we'll call them and the same for server with *b*. These numbers never leave the machine, nor they need to due to a magic called *Math* 🐧💀 So these 2 are the Private Variables.
3. Both Server and Client send each other *A = g^a (mod p)*, meaning "the modululus of p to the base number to the power of a." The same with Server just with b.
4. Now here is the magic. Both perform one last forumula: *S = B^a (mod p)* and *S = A^b (mod p)* now they end with the same number that is the password of each HTTP request.

Do I understand? NO 💀🐧. But at least I get the hang of it. The reason this is more secure is that, if your server is hacked or someone compromised your .p12 file, or even worse push your .p12 file to GitHub... 🐧💀 Yeah.

But everything in Diffie-Hellman is temporary during handshake, so it's safer.

### 2. "Just add Virtual Threads!" - Concurency 🐧🐧🐧🐧
> This isn't really a section, because we have covered [[Process & Threads - Java|Threads & Virtual Threads]] already. So, now we'll implement Secure & Concurrent Mini-server.

```java
import javax.net.ssl.*;
import java.io.*;
import java.nio.file.Files;
import java.nio.file.Path;
import java.security.KeyStore;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Main {
    public static void main(String[] args) throws Exception {
        char[] password = "password".toCharArray();
        KeyStore keyStore = KeyStore.getInstance("PKCS12");
        try (FileInputStream fis = new FileInputStream("keystore.p12");) {
            keyStore.load(fis, password);
        }

        KeyManagerFactory keyManagerFactory = KeyManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm());
        keyManagerFactory.init(keyStore, password);

        SSLContext sslContext = SSLContext.getInstance("TLS");
        sslContext.init(keyManagerFactory.getKeyManagers(), null, null);

        SSLServerSocketFactory sslServerSocketFactory = sslContext.getServerSocketFactory();
        try (
            SSLServerSocket serverSocket = (SSLServerSocket) sslServerSocketFactory.createServerSocket(8443);
            ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();
        ) {
            while (true) {
                SSLSocket clientSocket = (SSLSocket) serverSocket.accept();
                executor.submit(() -> {
                    try(
                        clientSocket;
                        PrintWriter printWriter = new PrintWriter(clientSocket.getOutputStream(), true);
                        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
                    ){
                        try{
                            clientSocket.startHandshake();
                            handleRequest(clientSocket, bufferedReader, printWriter);
                        }
                        catch (IOException e){
                            send500(printWriter);
                        }
                    }
                    catch (IOException e){
                        e.printStackTrace();
                    }
                });
            }
        }
    }
    public static void handleRequest(SSLSocket clientSocket, BufferedReader bufferedReader, PrintWriter printWriter) throws IOException {
        System.out.println("Client connected: " + clientSocket.getInetAddress());
        String line = bufferedReader.readLine();
        if (line == null) return;

        String[] parts = line.split(" ");
        String method = parts[0];
        String path = parts[1];
        if (path.equals("/")) {
            path = "/index.html";
        }
        Path filePath = Path.of("static/", path.substring(1));

        int contentLength = 0;
        String header;

        while (!(header = bufferedReader.readLine()).isEmpty()) {
            if (header.startsWith("Content-Length:")) {
                contentLength = Integer.parseInt(header.split(":")[1].trim());
            }
        }

        switch (method) {
            case "GET" -> handleGet(filePath, printWriter, clientSocket.getOutputStream());
            case "POST" -> handlePost(filePath, contentLength, bufferedReader, printWriter);
            case "PUT" -> handlePut(filePath, contentLength, bufferedReader, printWriter);
            case "DELETE" -> handleDelete(filePath, printWriter);
            default -> sendStatus(printWriter, "400 Bad Request");
        }
    }

    private static void send500(PrintWriter out) {
        out.println("HTTP/1.1 500 Internal Server Error");
        out.println("Content-Type: text/html");
        out.println("Connection: close");
        out.println(); // The empty line that separates headers from body
        out.println("<html><body><h1>500 Internal Server Error</h1><p>Mini-Tomcat encountered a problem.</p></body></html>");
        out.flush();
    }

    public static void handleGet(Path path, PrintWriter printWriter, OutputStream outputStream) throws IOException {
        if (Files.exists(path) && !Files.isDirectory(path)) {
            long fileSize = Files.size(path);
            String contentType = Files.probeContentType(path);
            if (contentType == null) {
                if (path.toString().endsWith(".css")) contentType = "text/css";
                if (path.toString().endsWith(".js")) contentType = "application/javascript";
            }

            printWriter.println("HTTP/1.1 200 OK");
            printWriter.println("Content-Type: " + contentType);
            printWriter.println("Content-Length: " + fileSize);
            printWriter.println();
            printWriter.flush();

            Files.copy(path, outputStream);
        } else {
            sendStatus(printWriter, "404 Not Found");
        }
    }

    private static void handlePost(Path path, int length, BufferedReader bufferedReader, PrintWriter printWriter) throws IOException {
        StringBuilder body = new StringBuilder();
        int read;
        int totalRead = 0;
        while (totalRead < length && (read = bufferedReader.read()) != -1) {
            body.append((char) read);
            totalRead++;
        }
        Files.writeString(path, new String(body));
        sendStatus(printWriter, "200 OK");
    }

    private static void handlePut(Path path, int length, BufferedReader bufferedReader, PrintWriter printWriter) throws IOException {
        boolean exists = Files.exists(path);
        StringBuilder body = new StringBuilder();
        int read;
        int totalRead = 0;
        while (totalRead < length && (read = bufferedReader.read()) != -1) {
            body.append((char) read);
            totalRead++;
        }
        Files.writeString(path, new String(body));
        sendStatus(printWriter, exists ? "200 OK" : "201 Created");
    }
    private static void handleDelete(Path path, PrintWriter printWriter) throws IOException {
        if (Files.deleteIfExists(path)){
            sendStatus(printWriter, "204 No Content");
        } else {
            sendStatus(printWriter, "404 Not Found");
        }
    }
    private static void sendStatus(PrintWriter writer, String status) {
        writer.println("HTTP/1.1 " + status);
        writer.println();
        writer.flush();
    }
}
```

Now I am gonna be doing other assignments, I have tight time 🐧💀

---

Post-Mortem.

I messed up in the video, stuttered a lot and didn't fully understand the TLS implementation. I also messed up a bit in this notes:

1. It's TLS, not TSL. I did a Typo.
2. It's Loopback IP, not Callback. The reason it's called Loopback is because it never leaves the Device via Network Interface Card, rather it goes back to the machine.
3. The TLS handshake is outdated. What I explained was RSA Key Exchange. While not necessarily wrong, it is OUTDATED. I added explanation for Diffie-Hellman Key Exchange that uses Math as a way to securely send messages.
4. Minor typo, it is "application/octet-stream" not streat.
5. So my note about Swallowing Errors in `PrintWriter` is bad. We need to know what's wrong in real time and catch the exception.
6. JavaScript actually is an application datat type in MIME type, not text.
7. And the reason this is confusing is, I took OSI layer literally. Like word to word instead of using it like a map. That's why I am confused; "Why is switch lower the layer on layer 2 compared to router in layer 3 when we hit switch first before router 💀🐧?" or "Bro, to connect the PC to switch it travels through cables. It's layer 1 already 🐧💀"

I haven't understood my implementation in the last code, but I am gonna change it in my Mini-Tomcat implementation with the latest TLS and most robust 💀🐧.