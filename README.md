### Introduction
In this repository, we are going to implement a fully-functioning TCP chatroom using Python. We will have **one server** that hosts the room and **multiple clients** that connect to it and communicate with each other. 
### About TCP & Client Server Architecture
**Transmission Control Protocol** is a connection-oriented protocol for communications that helps in the exchange of messages between different devices over a network.

For implementing our chatroom, we will use the **client-server architecture**. This means that we will have multiple clients (the users) and one central server that hosts everything and provides the data for these clients.

---

### Setting up the Server ( server.py )
**1.** For setting up our base server we will need to import two libraries, namely `socket` and `threading`. `socket` library will be used for establishing and setting up the **network connection(s)** and the `threading` library is necessary for **performing various tasks at the same time**.

```python
import socket
import threading
```
**2.** The next task is to define our server data and to initialize our socket. We will need an IP address for the host and a free port number for our server. In this blog, we will use the address `127.0.0.1` i.e our **localhost** and the port **5500**. 

The port number is irrelevant but you have to make sure that the port you are using is **free and not reserved**. If you are running this chatroom on an actual server or a virtual machine, specify the IP-address of the chatroom server as the **host IP address** of the virtual machine or the server. 

> Check out [this list of reserved port numbers](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers) for more information.

```python
# Server Data
host = '127.0.0.1'
port = 5500
```
**3.** When we define our socket, we need to pass two parameters. These define the type of socket we want to use. The first one `(AF_INET)` indicates that we are using an internet socket rather than an unix socket. The second parameter stands for the protocol we want to use. `SOCK_STREAM` indicates that we are using TCP.

After defining the socket, we bind or attach it to our host and the specified port by passing a **tuple** that contains both values. We will then put our server into listening mode, so that it waits for clients to connect and send messages.. 
```python
# Start the Server
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind((host, port))
server.listen()
```
**4.** We will also create two empty lists, which we will use to store the connected clients and their nicknames later on.
```python
# Lists For Clients and Their Nicknames
clients = []
nicknames = []
```
**5.** Now we will define all the functions that are going to help us  in **broadcasting messages**. It will be sending a message to each client that is connected and therefore is present in the clients list.
```python
# Sending Messages To All Connected Clients
def broadcast(message):
    for client in clients:
        client.send(message)
```
**6.** Now we will create a `handle()` function. This function will be responsible for handling messages from the clients. This function will run in a while-loop. The function will accept a client as a parameter and handle it in an endless loop until an error occurs or client itself disconnects.
```python
# Handling Clients
def handle(client):
    while True:
        try:
            # Broadcasting Messages
            message = client.recv(1024)
            broadcast(message)
        except:
            # Removing And Closing Clients
            index = clients.index(client)
            clients.remove(client)
            client.close()
            nickname = nicknames[index]
            broadcast('{} left!'.format(nickname).encode('ascii'))
            nicknames.remove(nickname)
            break
```
**7.** Now we have to receive the message from the client and broadcast it to all connected clients. So when one client sends a message, everyone else can see this message via the `broadcast()` function. Now if for some reason there is an error with the connection to this client, we remove it and its nickname, close the connection and broadcast that this client has left the chat.
```python
# Receiving `Function
def receive():
    while True:
        # Accept Connection
        client, address = server.accept()
        print("Connected with {}".format(str(address)))

        # Request And Store Nickname
        client.send('NAME'.encode('ascii'))
        nickname = client.recv(1024).decode('ascii')
        nicknames.append(nickname)
        clients.append(client)

        # Print And Broadcast Nickname
        print("Nickname is {}".format(nickname))
        broadcast("{} joined!".format(nickname).encode('ascii'))
        client.send('Connected to server!'.encode('ascii'))

        # Start Handling Thread For Client
        thread = threading.Thread(target=handle, args=(client,))
        thread.start()
```
When we are ready to run our server, we will execute this receive function. Once a client is connected to the server it will send the string ‘NAME’ to it, which will tell the client that its nickname is requested. After that it waits for a response and appends the client with the respective nickname to the lists and start a thread for `handle()` function for that particular client. 

**8.** Now we can just run this function and our server is done.
```python
receive()
```

---

### Setting up the Client ( client.py )
Now we are going to implement our client. For this, we will again need to import the same libraries. 

```python
import socket
import threading
```

**1.** The first steps of the client are to choose a nickname and to connect to our server. We will need to know the exact address and the port at which our server is running. Instead of binding the data and listening, as a client we are connecting to an existing server.
```python
# Choosing Nickname
nickname = input("Choose your name for the chat : ")

# Connecting To Server
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(('127.0.0.1', 5500))
```

**2.** Now, a client needs to have two threads that are running at the same time. The first one will constantly receive data from the server and the second one will send our own messages to the server.
```python
# Listening to Server and Sending Nickname
def receive():
    while True:
        try:
            # Receive Message From Server
            # If 'NAME'
            message = client.recv(1024).decode('ascii')
            if message == 'NAME':
                client.send(nickname.encode('ascii'))
            else:
                print(message)
        except:
            # Close Connection When Error
            print("An error occured!")
            client.close()
            break
# Sending Messages To Server
def write():
    while True:
        message = '{}: {}'.format(nickname, input(''))
        client.send(message.encode('ascii'))
```


Again we have an endless while-loop here. It constantly tries to receive messages and to print them onto the screen. If the message is ‘NAME’ however, it doesn’t print it but it sends its nickname to the server.

The writing function is quite a short one. It also runs in an endless loop which is always waiting for an input from the user. Once it gets some, it combines it with the nickname and sends it to the server.

**3.** The last thing we need to do is to start two threads that run these two functions.
```python
# Starting Threads For Listening And Writing
receive_thread = threading.Thread(target=receive)
receive_thread.start()

write_thread = threading.Thread(target=write)
write_thread.start()
```
And now we are done. We have a fully-functioning server and working clients that can connect to it and communicate with each other. 

---

## Running the Chatroom
Let’s go for a test run. Just keep in mind that we always need to start the server first because otherwise the clients can’t connect to a non-existing host.

**Server Log :**
```
Connected with ('127.0.0.1', 4970)
Nickname is Rajat_One
Connected with ('127.0.0.1', 4979)
Nickname is Rajat_Two
```
**Client One Log :**
```
Choose your nickname: Rajat_One
Rajat_One joined!Connected to server!
Rajat_Two joined!
Hello
Rajat_One: Hello
Rajat_Two: Howdy!
nothing much
Rajat_One: nothing much
```
**Client Two Log :**
```
Choose your nickname: Rajat_Two
Rajat_Two joined!
Connected to server!
Rajat_One: Hello
Howdy!
Rajat_Two: Howdy!
Rajat_One: nothing much
```

---

### And we have successfully created a chatroom based on Transmission Control Protocol in Python! 🎉 🥳

**Follow me on Instagram :** https://instagram.com/rajatrajput.dev <br>
**Follow me on LinkedIn :** https://linkedin.com/in/rajatrajput2004
