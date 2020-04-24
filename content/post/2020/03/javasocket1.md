---
title: "小试Java中的socket通信"
date: 2020-03-02T12:23:51+08:00
draft: false
tags: ["java"]
---

socket是一种基于TCP/IP的封装，本文模拟了java里简单的ServerSocket和客户端Socket的通信。

## 客户端：

```java

import java.io.*;
import java.net.Socket;
import java.util.Scanner;

/**
 * @author lei
 */
public class ClientDemo {
    public static void main(String[] args) {
        System.out.println("Client Run...");
        Scanner scanner = new Scanner(System.in);
        System.out.print("请输入你要发送的消息：");
        //循环接受键盘录入，创建连接到服务器的socket，发送定长数据包
        while (scanner.hasNextLine()) {
            String message = scanner.nextLine();
            byte[] bytes = message.getBytes();
            int length = bytes.length + 5;

            //先发送，再接收返回的消息
            Socket socket = null;
            OutputStream outputStream = null;
            DataOutputStream dataOutputStream = null;
            InputStream inputStream = null;
            DataInputStream dataInputStream = null;
            try {
                socket = new Socket("127.0.0.1", 6666);
                outputStream = socket.getOutputStream();
                dataOutputStream = new DataOutputStream(outputStream);
                dataOutputStream.writeByte(1);
                dataOutputStream.writeInt(length);
                dataOutputStream.write(bytes);
                dataOutputStream.flush();
                
                //接收服务器的回报消息
                socket.shutdownOutput();
                inputStream = socket.getInputStream();
                dataInputStream = new DataInputStream(inputStream);
                String s = dataInputStream.readUTF();
                System.out.println(s);
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                if (dataOutputStream != null) {
                    try {
                        dataOutputStream.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
                if (outputStream != null) {
                    try {
                        outputStream.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
                if (dataInputStream != null) {
                    try {
                        dataInputStream.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
                if (inputStream != null) {
                    try {
                        inputStream.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
                if (socket != null) {
                    try {
                        socket.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
        }


    }
}
```

## 服务器主线程:

```java
import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * @author lei
 */
public class ServerDemo {
    public static void main(String[] args) throws IOException {
        System.out.println("Server run");
        ServerSocket serverSocket = new ServerSocket(6666);
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        while (true) {
            Socket socket = serverSocket.accept();
            Runnable task = new HandleSocket(socket);
            executorService.submit(task);
        }
    }
}
```

## 服务器子线程：

```java

import java.io.*;
import java.net.Socket;

/**
 * @author lei
 */
public class HandleSocket implements Runnable {
    private Socket socket;

    public HandleSocket(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        InputStream inputStream = null;
        DataInputStream dataInputStream = null;
        OutputStream outputStream = null;
        DataOutputStream dataOutputStream = null;
        try {
            inputStream = socket.getInputStream();
            dataInputStream = new DataInputStream(inputStream);
            byte b = dataInputStream.readByte();
            int len = dataInputStream.readInt();
            byte[] buf = new byte[len - 5];
            dataInputStream.readFully(buf);
            //此处不要输入一下代码，应该在最后关闭，否则会报错：Socket is closed.
            // dataInputStream.close();
            //inputStream.close();

            socket.shutdownInput();
            outputStream = socket.getOutputStream();
            dataOutputStream = new DataOutputStream(outputStream);
            dataOutputStream.writeUTF("服务器收到字节数：" + len);
            String s = new String(buf);
             System.out.println("收到" + socket.getInetAddress() + ":" + socket.getPort() + "消息：");
            System.out.println("数据类型：" + b + "\n数据长度：" + len + "\n内容：" + s);
            } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (dataInputStream != null) {
                try {
                    dataInputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (inputStream != null) {
                try {
                    inputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            try {
                if (dataOutputStream != null) {
                    dataOutputStream.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if (outputStream != null) {
                    outputStream.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if (socket != null) {
                    socket.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }

        }
    }
}
```

参考资料： 
[Java socket详解,看这一篇就够了](https://www.jianshu.com/p/cde27461c226)