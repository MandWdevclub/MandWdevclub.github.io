---
layout: post
title: Contextualizing Sockets - UDP sockets
author: Edward McEnrue
date: 2017-01-21
updated: 2017-01-21
headerImg: ""
headingTextColor: "#FEFEFE"
categories: UDP sockets networking
---

This post is partially a follow up on the flask endpoint tutorials. I
realized that without the context of how flask works internally, it will probably
seem like black magic even if flask is simple. This post/workshop will hopefully
dispel any confusion by providing the context you need to succeed!

## UDP Sockets

### Post Contents

###### 1. Overview

###### 2. UDP and the Interwebz

###### 3. Coding the Client

###### 4. Coding the Server

###### 5. Testing it out on Digital Ocean

#### 1. Overview

Have you been hearing your friends talk about sockets, UDP, or networking lately
and asked yourself, "what the hell are sockets?" Then this is the workshop for you!

Rather than use any framework to create a very, very simple server, today you'll
use just what your computer's OS provides to you for interacting with the internet. 
What is this resource that the OS provides? It's called a "socket".
Sockets are used by pretty much everything connected to the internet. They are the
object/resource that developers use to send a string of characters to another computer. That
may sound abstract, but we'll see how they work concretely soon enough.

In addition to sockets, I'll discuss UDP (User datagram protocol) and how it relates
to sockets and the internet in general, and why a 'protocol' would matter when
all we're doing is trying to send some data over the internet.

Enough talk, let us begin!

#### 2. UDP and the Interwebz

First off, there are two terms that will be covered in this part of the workshop:
UDP (User Datagram Protocol) and sockets. The fact that UDP has the word "protocol"
in it may at first be intimidating until you realize that a protocol is simply
a way of doing things. For example, the protocol for starting a car is to get into
the car, put your keys in the ignition and turn the key. See, not so bad.

So what is UDP then? This protocol is a way of reading a series of bytes,
and determining how to construct something meaningful from it. It just so happens
that this series of bytes came from another computer, AND this series of bytes
passed through a socket at one point of time, which is what put the series of bytes into the format they
are currently in. The most important part of this protocol is that it is "connectionless".
That means when you send data to another computer, the packets may not arrive in the same
order of transmission, or the data may not arrive at all (it might get dropped by some
routing machine somewhere). You may say, "well UDP is total crap then." BUT it turns out
that by enforcing a minimal amount of constraints on the way your data is handled over the internet,
it makes the transmission of your data SUPER ULTRA fast. Another protocol, TCP, is a more constrained, but more reliable protocol, and it's what many modern
websites use to send you HTTP communication when you're browsing websites. This post is about UDP, because I think it's easier to understand, and UDP is used frequently for many applications where reliability isn't paramount, but speed is.

Ok, very cool, but how does this information help you? Great question! It helps, because now that you understand the background of UDP, you can understand what a socket configured to use UDP is doing. In other words, sockets can be
configured to say, "the data that I'm sending is using the protocol UDP". In fact,
you will be doing that very soon. Once you configure a socket object to use a protocol,
then you can give it an IP address and a port number (the way to identify where to send data),
or just a port number (the way to identify where to receive data on your computer),
and then you can give it some data, and then tell the socket to send that data. The OS, your wifi device, and the internet will handle the rest. Pretty cool! IF you reached this point, and you're still confused, that's OK, you don't
need to know how the entire internet works in order to use sockets!


#### 3. Coding the Client

Up until now, we didn't code anything (lame, right?) but we will now. I've chosen
java for this lesson, since golang and C++ are a bit heavy, and python is a bit
too abstracted.

In any case, the client is the code that will be running on your laptop. We'll start with
the basic java file and main method with some logic:


In UDPSocketsClientExample.java we have


```
public class UDPSocketsClientExample {

    public static void main(String[] args){

        UDPClient client = new UDPClient();
        Scanner cmdLine = new Scanner(System.in);

        while(true){
            String message = cmdLine.nextLine();
            client.run(message);
        }
    }
}
```


And in UDPClient.java we have



```
public class UDPClient {

    public void run(String message) {

    }
}

```


Essentially, what we want to do is fill in the run method to use a DatagramSocket
object (java's version of a socket configured for UDP, less work for you!). Likewise, when we pass in a string to this method from the command line, we want the DatagramSocket to send it to another computer at a
destination IP address and port. That's a lot simpler than it sounds,
so lets do that now:



```
public class UDPClient {

    public static final int UDP_SERVER_PORT = 8000; //TODO this can be chosen by you, computers have lots of ports
    public static final String SERVER_ADDRESS = "1-800-588-2300 Empire Today"; //TODO change this once you're on the second post, it is the server's IP address

    public void run(String message) {
        DatagramSocket socket = null;

        try{
            /**
             * The routing machines in between your client machine and your digital ocean server need an IP address
             * to know how/where to route/send the Datagram
             */
            InetAddress addr = InetAddress.getByName(SERVER_ADDRESS);

            /**
             * Since nobody is sending this client socket data, we don't need to specify a port
             */
            socket = new DatagramSocket();

            //DatagramPacket's are constructed with byte arrays, not strings
            byte[] buf = message.getBytes();

            /**
             * Now the routing nodes have the data, the IP address, and a port at that IP address
             */
            DatagramPacket data = new DatagramPacket(buf, buf.length, addr, UDP_SERVER_PORT);

            //Now, all we have to do as the client is send our data
            socket.send(data);

        } catch (SocketException e) {
            e.printStackTrace();
        } catch (UnknownHostException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(socket != null){ //clean up after yourself
                socket.close();
            }
            socket = null;
        }
    }
}
```


As promised, the first two variables declared are the port and IP address of your
server, which your client will send data to (don't forget to replace this). We then create our DatagramSocket.
Next we create a byte array from our message string, and we package it in a
DatagramPacket like so:


```
DatagramPacket data = new DatagramPacket(buf, buf.length, addr, UDP_SERVER_PORT);
```


A DatagramPacket is java's way of interpreting data that is packaged for UDP. This is nice, because we don't need to know the internals of UDP, just what benefits it provides, and what objects are needed to take advantage of them.
Finally, we send the data from the socket, and once that's complete, we close the
socket. "Wait, that was it?!" Yup, you just sent data with no flask, no node, just
with sockets.


#### 4. Coding the Server

But wait, how do you know that I'm not just pranking you and that
sockets, like unicorns, are not just a figment of my imagination? To make sure I'm
not a youtube prank master, we need to create another socket that is listening/receiving at the IP
address and the port we sent data to. Again we'll create two files:

In UDPSocketsServerExample.java we'll put


```
public class UDPSocketsServerExample {

    public static void main(String[] args){
        run();
    }

    /**
     * The run function does two things:
     * 1: Creates the class which has the listening 'server' socket
     * 2: starts an infinite loop to, so that we can receive multiple datagram packets
     *
     * Why do we need this infinite while loop? Your server will receive data, and then destroy the socket, but
     * what if there's more data being sent to your server?
     */
    private static void run(){

        UDPReceiver receiver = new UDPReceiver();

        while(true){
            receiver.run();
        }
    }
}

```


And in UDPReceiver.java we'll put



```
public class UDPReceiver {

    public static final int UDP_RECEIVER_PORT = 8000; //later on, if this port isn't open, use "lsof -i:8000" on linux and kill <PID>

    public void run() {


    }
}
```


This time, we'll put our run method in an infinite loop, since we'll want to be
able to receive data, print it out, and then be ready to receive more data. Again,
this will be super simple, since all we're changing in our socket code is that we're
receiving rather than sending as shown below:


```
import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.SocketException;

public class UDPReceiver {

    public static final int UDP_RECEIVER_PORT = 8000; //if this port isn't open, use "lsof -i:8000" on linux and kill <PID>

    public void run() {
        DatagramSocket socket = null;

        try{
            /*
             * sockets are associated with a port, since that's how we can differentiate different
             * connections on the machine you run this program on. Technically, you could create this
             * object without passing in a port and it would just find an open port. In any case, the
             * socket will "bind" to a port.
             */
            socket = new DatagramSocket(UDP_RECEIVER_PORT);


            byte[] buf = new byte[100];
            /**
             * At a high level, Datagrams are the package of 1's and 0's that will be sent over the internet when using UDP
             *
             * Datagrams are typically structured in header and payload sections.
             * Datagrams provide a "connectionless communication service"
             *
             * what does connectionless communication mean? The delivery, arrival time, and order of arrival of datagrams
             * need not be guaranteed by the network. However, UDP has less overhead than TCP because of this.
             *
             * In other words, "send and forget"
             */
            DatagramPacket packet = new DatagramPacket(buf, buf.length);


            System.out.println("now listening for datagrams");

            /**
             * Receive will wait for a datagram to be received by our socket listening to our port.
             *
             * You would want to have configured this socket with some of its other functions before calling .receive
             */
            socket.receive(packet);


            /**
             * We can reconstruct the bytes that were sent by just using String
             */
            String data = new String(packet.getData(), 0, packet.getLength());

            System.out.println("received the following datagram:");
            System.out.println(data);

        } catch (SocketException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {

            /**
             * Always close your sockets, so that you don't have any port binding conflicts on the next iteration
             * of the while loop that is listening for UDP packets.
             */
            if (socket != null){
                socket.close();
            }
            socket = null;
        }

    }
}
```


Much of this file should look familiar. Again, we're creating a socket that is
configured to use UDP so our socket knows how to interpret our incoming data. We
create a DatagramPacket to put the incoming data into, and finally we wait by
saying ```socket.receive(packet)```. Once something is sent to this socket, we'll
convert the data into a string and print it out. IZI PIZI


#### 4. Testing it out on Digital Ocean

For those of you who don't want to use digital ocean, you can just use the IP
address which always points to your own computer: "127.0.0.1". Just have your server running
on the same port where you're sending your data, and make sure you change the IP
string variable.

For the digital ocean lovers, first you'll want to create a droplet with linux,
then you'll want to install java on it by following the super quick tutorial:

https://www.digitalocean.com/community/tutorials/how-to-install-java-on-ubuntu-with-apt-get

Once that's setup, copy your server java files, and compile them with ```javac UDPSocketsServerExample```. 
Then run the server with ```java UDPSocketsServerExample```.
Now on your local machine, run the client after you've changed the IP to be the IP
address of your server. You should see that your message printed out, but UDP packets
can be dropped, so you may have to send a few messages. 


#### 5. Quick Conclusion

Congrats if you made it this far! In this workshop, you built client and server sockets configured to use UDP, and then you had them communicate some data from your
commandline. C00L! With this knowledge, you know how to build a foundational
element of a massive amount of server infrastructure applications.

Now, what did you not do? You didn't use TCP sockets, which are very useful to know. The following blog does a good job of describing how to build a client and server with TCP sockets:

http://tutorials.jenkov.com/java-multithreaded-servers/singlethreaded-server.html

Likewise, we didn't see how to make this a multithreaded application, which is
an important aspect of writing a server that processes incoming data! (so that a
  single client can't hog the server). As a final caveat, this post compresses a TON of information, and so not all of it is 100% accurate. You'll have to do some extra research to start to build your own mental model of how the internet works. For example, here's the specifications for TCP and UDP:
  
https://tools.ietf.org/html/rfc793

https://www.ietf.org/rfc/rfc768.txt

Nevertheless, this should give you a good understanding of the type of things
flask is doing to respond to and listen for the HTTP requests you receive.

Also, if you find this stuff interesting, I encourage you to get involved with the systems research at VT.


