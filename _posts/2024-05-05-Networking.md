---
title: Networking
date: 2024-05-05 12:00:00 -500
categories: [project,solo,study]
tags: [networking,cpp,buas]     # TAG names should always be lowercase
---

# Multiplayer networking: Sending/receiving data & API design

Hello! My name is Ayoub. I’m a 2nd year BUAS game developer student & for a while I’ve had an interest with multiplayer networking. Since that a big chunk of triple A games are online multiplayer games, it is in my that game developers should at least have an understanding how multiplayer games work.

For my school project, I decided to create a multiplayer game where players send messages to each other, move around in a lobby & players can play various minigames, all to learn how to create a multiplayer environment.

My ideas with this project were the following:

1. Get a better understanding how multiplayer games work.
2. Learn how to create a multiplayer framework.
3. Implement various mechanics/concepts to showcase what my framework can do.

Before starting the project, I had to make some design choices that dictates the framework.

# TCP or UDP?

The first choice that I had to make was what socket protocol I want to use: TCP or UDP. I decided to use UDP. Though TCP is more reliable with sending data & breaks down the data for you, it is slow & can even stall games to the point where it would be unplayable for the average user. Online multiplayer games are usually time-critical, in which you never want to use TCP. I want the data to be send as fast as possible. Though UDP is connectionless & unreliable, it is fast. I can mitigate those issues by implementing TCP features into my UDP implementation.

# P2P or Server-Client?

The second choice that I have to make is whether I want to use a peer-to-peer (P2P) or server-client network. With P2P, one client hast to act as a “server”, so the all the nodes are reliant to the network of that one node. If the network or system of the node is underperforming, it could cause latency issues or desynchronization. All peers are connected & are aware with each other, making the network a lot less secure & more vulnerable to exploits. In server-client, the server is authoritative, meaning that all clients need to connect & communicate with the server to gain the necessary data. This also makes the network more secure, since only the server is aware of all the connected client. Clients are not connected with each other. Clients are reliant to the server, however. If the server is down, nobody can play the game. In the end I chose server-client since it is more secure, synchronization more stable & that is the norm within the industry.

# The chat system

With my system design choices made, I can start with creating the multiplayer game. I started with creating a simple chat system. This gave me the opportunity to learn following things:

- How do I establish a connection between two sockets?
- How do I send data between sockets?
- With sending chat messages, how do I send one message to all clients connected to the server (a.k.a., how do I synchronize all clients with each other)?

First, I have to create a socket. Because I currently only want to run the project on my windows PC, I’m going to use Winsock for networking. This is also needed to use sockets.

First, I initialize Winsock.

CODE

With Winsock now initialized, I can now use sockets. When initializing sockets, you have to set up parameters such as the internet protocol we want to use & if the socket needs to be TCP or UDP.

CODE

When a packet is send to a socket, it is being send to a machine with an IP address & a specified port. For the server to receive packets from clients, the socket needs to be bound to a specified port & IP. sockaddr_in stores the IP address & port. The port I’ve chosen in this case is 8080, since ports below 1024 are already reserved.

CODE

With this setup, the socket can now send & receive data. For the client to send data, the client needs to call the sendto method with the data it wants to send & to the machine it wants to send the data to. I already know what the IP & the port is, so I can just fill the address & port of the server on intialization. With the server IP & port setup, I call the sendto method, fill the parameters with the data, size of the data & the server address & then the socket sends the data to the server.

CODE

The client is now sending the data to the server, but the server isn’t currently receiving the data. This is because the server isn’t calling the recvfrom method. The recvfrom method checks if there is a packet to receive. If there is, it grabs the data. One of the parameters in the recvfrom requires a sockaddr_in variable. If a packet is received, it fills the variable with the IP from the machine that send the data. Otherwise it keeps waiting until a packet is received (unless we enable nonblocking mode). It is important that the length parameter is equal or bigger than the size of the packet that the socket receives. Of the packet is bigger, than it is being ignored.

CODE

The server is now receiving data from the client. Great! Now the server just needs to send the data back client. With the packet that the server received, I just send that packet back to the client. As mentioned before, the address & port in the client variable is being set by recvfrom. So now I just do the same thing I did on the client side to the server.

CODE

And now on the client side, we call receive method.

CODE
GIFS

Great! Now the server send data back client. However, it only sends it to one client. If multiple clients would connect to the server & send a message, the message would only be send to the one that send the message, not to all connected client. Since UDP is connectionless, I need to implement a way for the server to know what clients are connected to the server.

The server needs to know the IP & port of the users that are connected. I created a struct that simply holds the IP & port of a client.

First, I created an enum to identify the packets.

CODE

Now, before a packet is being send, I add the packet ID at the start of the buffer so that the recipient can handle it accordingly.

CODE

On the server side, before handling the whole packet, it first checks the type of ID before handling it.

CODE

Now, whenever a client is sending a message to the server, the server sends the message to all the connected clients! With the basics, I can continue & expand this project by creating more features.

CODE

# Outputstream

But before continuing, I decided to take a step back & look at my project. Looking at my code, calculating the size of the buffer & adding data into the buffer is a bit tedious. I have to manually calculate if the added data isn’t overloading the buffer. Plus, with my current implementation I practically have to copy paste memcpy to get the data.

To resolve this, I created the OutputBufferStream class. The class holds a buffer where I can put the data I want to send to other sockets. It also holds the size of the of the buffer, which increases based on the data that is being added to the buffer.

CODE

Using this class, all I have to do is just call the Read method, add the data I want to add in the buffer to the parameter & let the class fill the data into the buffer.

CODE

# InputStream

For the input, I created a class called InputBufferStream, which is specifically made to read data. Just like the OuptutMemoryStream class, it holds the buffer data. Unlike it, however, it keeps track on the data that has been read. When reading the data, it is important that you read it in the order that the buffer is set, otherwise you’re going to get wrong/corrupted results.

CODE
CODE

# SocketUtility & UDPSocket

I’ve also created a socket utility script to simplify creating sockets, handling/reporting errors & initializing & closing winsock.

CODE

I also created a UDP socket class to make it easier to create a sock, bind & set it to non-blocking if desired. It utilizes the socket utility to report errors when something goes wrong.

CODE

To showcase an example, the code snippet below showcases an example where an socket is being created & bound to.

CODE

With all of this setup, this makes it easier to create extra features. For example, a lobby where players can move around.

GIF

Or even a (mini)game such as pong!

GIF

Network programming is… not easy, but I’ve most definitely enjoyed it. In the near future I’m going to expand my knowledge with networking, researching concepts such as rollback netcode, input prediction & anti-cheat such as input-validation.
