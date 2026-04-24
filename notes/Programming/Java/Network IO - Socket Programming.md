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
> So, In Unix and Linux, everything is a file ([[System Calls (Syscalls)|System Calls]]). You write to a buffer, and the OS translates what is in the buffer to the tuple needed for Socket.
> Socket is the one FD pointed to. Think of it like a a mailing post office. You don't know how, you just put your packet (Data) in the post office (FD -> Socket) and write your address so your Network Card can send them.

Suddenly OSI layers make sense to me because of this 🐧💀

### 3. "How many people can I talk to?" - 🐧❓
> This is a quick section, because I am tired 🐧💀 So in IP Address (Which I learned in TKJ 💀🐧) you know that `127.0.0.1` is reserved for callback IP, what is it? When you bind your program to this IP, it means it's bind to localhost, and limited to all the programs inside your host/PC
> While `0.0.0.0` means non-routeable or empty IP Address. It can also mean this host on the network. Which will make your server accesible for anyone in the network if they access your IP Address on the network and the port. Or in other terms: "I don't care whether they come from the front door (Network), backdoor (Ethernet), or even the window (VPN). As long as it's my room (Port), let them in." - 🐧💀

### 4. Java Socketing - ServerSocket & Socket
> So in C, we need to bind our process to a port and starts listening. But Java already handles that with `ServerSocket`. It automatically binds your program to a Port and listen, though to accept connection you need to do `socket.accept()`
> You must know that `.accept()` is a blocking call. Your program will halt until a client connects.
> And once a Client connect you will get a `Socket` object, resembling the FD for each connection the Socket maintains. It writes to the FD.

But how do we not run out of FD? Modern VPS can change their limit of FD from 1024 to 65,535 or even 1,048,576. And it's so small, you don't even need to worry.

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

We use PrintWriter so we don't need to flush manually nor forget it 🐧💀 and so that Error doesn't brick the request, and PrintWriter swallows the error and silently fail.

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

HTTP Request usually only comes in 3 lines:

1. The Request Line
	> Remember that the server is passive? This line tells your server what to do. You need to parse this line. IT includes a verb (GET/POST/PUT/DELETE), the thing you wanna get, and the HTTP version.
2. The Headers
	> These are like extra information for someone: "My house has a ... color fence" or "I am verified, let me in"
3. The Empty LIne
	> HTTP Protocol dictates that headers must end with a double new line; `\r\n\r\n`

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
| JavaScript | `text/javascript`          | The logic/interactivity             |
| PNG        | `image/png`                | Lossless images.                    |
| JPEG       | `image/jpeg`               | Standard photos                     |
| GIF        | `image/gif`                | Animated or simple images           |
| SVG        | `image/svg+xml`            | Vector graphics (XML based)         |
| JSON       | `application/json`         | The standard for sending data       |
| XML        | `application/xml`          | Older data format                   |
| Plain Text | `text/plain`               | Raw text with no formatting         |
| Raw Bytes  | `application/octet-streat` | Usually used for downloading files. |

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