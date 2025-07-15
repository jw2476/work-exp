### Basic server
A server has to accept multiple connections, and handle incoming messages from each connection at the same time. To do this we need to use some form of concurrency, there's many ways of doing this such as multithreading, however the easiest approach for networking is asynchronous programming. Asynchronous programming uses the `async`/`await` keywords allows other code to run while waiting for data from connections.

```python
import asyncio

async def handle_connection(reader, writer):
    # .read() can be used to receive incoming data
    # from the other computer, this blocks the program
    # until some data is available, so await is used
    # to allow other code to run while waiting. It returns
    # a list of bytes.
	
	# The parameter sets the maximum amount of data to read,
	# if more data is available, it is not read into the list.
	# This data will be read on the next call to .read().
	
	# You will need to work out a way of making sure you read 
	# an entire message so no data is lost.
    data = await reader.read(100)
	
	# When reading in a string, .decode() can be used to
	# parse the bytes.
    message = data.decode()
	
	# .write() can be used to send data to the other computer,
	# this data has to be raw binary, so you can't send lists,
	# dictionaries, sets, etc without first serialising them to
	# binary.
    writer.write(data)
	
	# Calling .write() doesn't always actually send the data, it will
	# queue it to be sent, but to ensure the data is actually sent,
	# use .drain(). This will block the program until all the data is sent,
	# so await is used so other code can be run while waiting.
    await writer.drain()
	
	# The socket connection can be closed by either the client or the
	# server using .close(). You will need to handle either the client or
	# the server closing the connection at any time.
    print("Close the connection")
    writer.close()
    await writer.wait_closed()

async def main():
	# Bind the server to a specific port (in this case 8888), 
	# the address can be used to limit which computers can 
	# connect to the server, 0.0.0.0 allows any computer on 
	# the network to connect.
    server = await asyncio.start_server(
        handle_connection, '0.0.0.0', 8888)
	
	# A single server can listen on many physical connections 
	# at the same time, for example it could listen on a wired 
	# Ethernet connection and a WiFi connection at the same time.
	# Each one of these will have it's own IP address, this gets
	# all of those addresses, concatenates them and prints them. 
    addrs = ', '.join(str(sock.getsockname()) for sock in server.sockets)
    print(f'Serving on {addrs}')
	
	# Listen for incoming connections forever.
    async with server:
        await server.serve_forever()

asyncio.run(main())
```
### Basic client
Clients don't tend to be as complicated as servers, since they only have to handle a single connection, but they still mostly have to make sure they don't block the program. In something like a game, or a chat app, blocking on networking will cause the game/UI to freeze, and make the app feel unresponsive. 
```python
import socket 

# The address of the other computer, 
# 127.0.0.1 is a special address which
# refers to this computer. If you run the
# server and the client at the same time,
# connecting to 127.0.0.1 will allow you to
# test everything on one laptop.
ADDRESS = '127.0.0.1'     
PORT = 8888 # The port used by the server

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# Connect the socket to the server
s.connect((ADDRESS, PORT))

# Sends "Hello, world" to the server, 
# the b in front of the string tells
# Python to represent the string in binary,
# which allows it to be sent using the socket.

# unlike .write() for the asyncronous socket, 
# .sendall() does actually send the data instead 
# of buffering, so there is no .drain()
s.sendall(b'Hello, world')

# This would fail, since a string isn't binary data
# s.sendall('Hello, world')

# .recv() works like .read() on the server, the
# parameter is used as the maximum amount of data
# to read. Any further data will not be read until the
# next .recv().
data = s.recv(1024)
print('Received', repr(data))

# The socket connection can be closed by either the client or the
# server using .close(). You will need to handle either the client or
# the server closing the connection at any time.
s.close()
```
