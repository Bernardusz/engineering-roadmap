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