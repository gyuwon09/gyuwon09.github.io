---
title: "TCP 통신을 통한 채팅 프로그램"
date: 2024-05-26 11:12:32
categories: [Research]
tag: [Research]
---

정확하고 안정적인 통신 프로토콜, TCP 통신을 이용해서 간단한 채팅 프로그램을 구현해보았습니다.

레포지토리 위치 : <a href="https://github.com/gyuwon09/tcp-socket-chatting">gyuwon09/tcp-socket-chatting</a>

# TCP란?
```
TCP(Transmission Control Protocol)은 네트워크 환경과 변동성을 고려하여 안정적인 통신을 제공하는 전송 계층 프로토콜입니다.
www, web, email, 파일 전송 등 주요 인터넷 어플리케이션은 TCP에 의존합니다.

TCP는 데이터 전송의 **신뢰성**을 보장하기 위해 연결 지향 방식으로 동작하며, 통신을 시작하기 전 송신자와 수신자 간의 연결을 설정합니다.
이 과정에서 데이터의 손실, 중복, 순서 변경과 같은 문제를 방지하여 정확한 정보 전달을 가능하게 합니다.

또한 TCP는 전송 중 오류가 발생하면 이를 감지하고 재전송을 수행하며,  
수신자의 처리 속도에 맞춰 전송량을 조절하는 **흐름 제어(Flow Control)** 기능을 제공합니다.  
네트워크 혼잡 상황에서는 전송 속도를 조절하는 **혼잡 제어(Congestion Control)**를 통해 전체 네트워크의 안정성을 유지합니다.

이러한 특징으로 인해 TCP는 실시간성보다는 **정확성과 신뢰성이 중요한 서비스**에 적합하며,  
대표적으로 웹 통신(HTTP/HTTPS), 이메일(SMTP, POP3, IMAP), 파일 전송(FTP) 등에서 널리 사용됩니다.
```

tcp-socket-chatting에는 client.py와 server.py 파일이 존재합니다.<br>
client.py는 사용자로부터 텍스트를 입력받아 서버로 전송하는 역할을 수행하며,<br>
server.py는 클라이언트로부터 텍스트를 전달받아 다른 사용자에게 메시지를 전송하는 역할을 수행합니다.

# client.py
```
import socket
import threading

host = 'localhost'
port = 9999

class tcpclient():
        def __init__(self):
                pass

        def connect(self):
                #client socket create
                client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
                client_socket.connect((host, port))
                print(f"[*] socket connected {host}:{port}")
                #set to receive message from server to threading
                server_receiver = threading.Thread(target=self.receive_chat, args=(client_socket,))
                server_receiver.start()
                #set to send message from server to threading
                server_sender = threading.Thread(target=self.send_chat, args=(client_socket,))
                server_sender.start()

        def receive_chat(self,client_socket):
                while True:
                        try:
                                message = client_socket.recv(1024).decode('utf-8')
                                print(f"[R]: {message} ")
                        except ConnectionResetError:
                                print("server disconnected")
                                break

        def send_chat(self,client_socket):
                while True:
                        try:
                                message = input("")
                                if message == 'quit':
                                        break
                                client_socket.send(message.encode('utf-8'))
                        except:
                                client_socket.send("client Error".encode('utf-8'))
                                break

chatting_client = tcpclient()
chatting_client.connect()
```

# server.py
```
import socket
import threading
import os

class tcpserver():
        def __init__(self):
                pass

        def open(self):
                host = os.getenv('host')
                port = int(os.getenv('port'))
                #tcp socket create
                server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM) 
                #Set the socket option to allow reuse of an already used address
                server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1) 
                server_socket.bind((host, port))
                server_socket.listen()
                print(f"[*] ip : {host} port : {port}")
                while True:
                        client_socket, addr = server_socket.accept()
                        print(f"[+] {addr} connected.")
                        #set to receive message from client to thread
                        client_receive = threading.Thread(target=self.receive_chat, args=(client_socket,))
                        client_receive.start()
                        #set to send message from client to thread
                        client_sender = threading.Thread(target=self.send_chat, args=(client_socket,))
                        client_sender.start()

        def receive_chat(self,client_socket):
                while True:
                        try:
                                message = client_socket.recv(1024).decode('utf-8')
                                print(f"[R]: {message}")
                        except ConnectionResetError:
                                print("\nclient disconnected")
                                break
                client_socket.close()

        def send_chat(self,client_socket):
                while True:
                        try:
                                response = input('')
                                client_socket.send(response.encode('utf-8'))
                        except:
                                client_socket.send("Error 404".encode('utf-8'))
                                break
                client_socket.close()

chatting_server = tcpserver()
chatting_server.open()
```