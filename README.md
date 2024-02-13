<!----- Conversion time: 1.019 seconds.


Using this Markdown file:

1. Cut and paste this output into your source file.
2. See the notes and action items below regarding this conversion run.
3. Check the rendered output (headings, lists, code blocks, tables) for proper
   formatting and use a linkchecker before you publish this page.

Conversion notes:

* Docs to Markdown version 1.0β17
* Wed Sep 18 2019 01:22:59 GMT-0700 (PDT)
* Source doc: https://docs.google.com/open?id=13Bwj-zrzPHWxDyeuZUzSwTNSqtZj9FI-spwD9tnhUTA
----->


# Простейшие TCP live stream video server и client

## Цель работы

Познакомиться с приемами работы с сетевыми сокетами в языке программирования Python.

## Задания для выполнения



Today we are going to live stream video using socket programming and OpenCV in python. where we will extracting video of host webcam and then send it to the client. thus establishing a video connection between server and client.

Socket programming:
-------------------

First, What is a **Socket**? Sockets allow communication between two different processes on the same or different machines, To be more precise, it's a way to talk to other computers using standard Unix file descriptors. Here a socket works much like a low-level file descriptor as commands such as read() and write() work with sockets in the same way they do with files and pipes.

**Socket programming** is a way of connecting two nodes on a network to communicate with each other. One socket(node) listens on a particular port at an IP, while other socket reaches out to the other to form a connection. Server forms the listener socket while client reaches out to the server.

![](np10-9.png)

Creating the Host Side Setup:
-----------------------------

We will create a socket, get host name, host IP and print them to check retrieval:

    # Importing the libraries
    import socket, cv2, pickle, struct, imutils
    # Create Socket
    server_socket = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    host_name  = socket.gethostname()
    host_ip = socket.gethostbyname(host_name)
    print('HOST IP:',host_ip)
    port = 9999
    socket_address = (host_ip,port)
    

Here we made a socket instance and passed it two parameters. The first parameter is AF\_INET and the second one is SOCK\_STREAM. AF\_INET refers to the address family ipv4. The SOCK\_STREAM means connection oriented TCP protocol.

**bind() method**: A server has a bind() method which binds it to a specific ip and port so that it can listen to incoming requests on that IP and port.

**listen() method**: A server has a listen() method which puts the server into listen mode.

    # Socket Bind
    server_socket.bind(socket_address)
    # Socket Listen
    server_socket.listen(5) //5 here means that 5 connections are kept waiting if the server is busy and if a 6th socket trys to connect then the connection is refused.
    
    print("LISTENING AT:",socket_address)
    

**imutils.resize()** :-The resize function of imutils maintains the aspect ratio and provides the keyword arguments width and height so the image can be resized to the intended width/height while maintaining aspect ratio and ensuring the dimensions of the image do not have to be explicitly computed by the developer.

**pickle.dump()** :-The dump () method of the pickle module in Python, converts a Python object hierarchy into a byte stream.

**struct.pack ()** :-This is used to pack elements into a Python byte-string (byte object).

    # Socket Accept
    while True:
        client_socket,addr = server_socket.accept()
        print('GOT CONNECTION FROM:',addr)
        if client_socket:
            vid = cv2.VideoCapture(0)
            while(vid.isOpened()):
                img,frame = vid.read()
                frame = imutils.resize(frame,width=320)
                a = pickle.dumps(frame)
                message = struct.pack("Q",len(a))+a
                client_socket.sendall(message)
    
                cv2.imshow('TRANSMITTING VIDEO',frame)
                key = cv2.waitKey(1) & 0xFF
                if key ==ord('q'):
                    client_socket.close()
                    break
    cv2.destroyAllWindows()
    

Creating the Client Side Setup:

As we did in host side:

    import socket, cv2, pickle, struct
    # create socket
    client_socket = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    host_ip = '192.168.56.1' #IP Address of Host to be Entered 
    port = 9999
    client_socket.connect((host_ip,port))
    data = b""
    payload_size = struct.calcsize("Q")
    

After connecting to the host, we will unpack the message received. The byte stream of a pickled Python object can converted back to a Python object using the pickle.load () method.

    while True:
        while len(data) < payload_size:
            packet = client_socket.recv(4*1024) 
            if not packet: break
            data+=packet
        packed_msg_size = data[:payload_size]
        data = data[payload_size:]
        msg_size = struct.unpack("Q",packed_msg_size)[0]
        while len(data) < msg_size:
            data += client_socket.recv(4*1024)
        frame_data = data[:msg_size]
        data  = data[msg_size:]
        frame = pickle.loads(frame_data)
        cv2.imshow("RECEIVING VIDEO",frame)
        key = cv2.waitKey(1) & 0xFF
        if key  == ord('q'):
            break
    client_socket.close()
    cv2.destroyAllWindows()
    

So we are ready with the code of both client and host side. First we will run the host side code, which will make host under listening mode and then we will run the client size code and client will establish the connection.
