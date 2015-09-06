# socketgd

Guaranteed delivery for socket.io. Works in node and in the browser.

## Huh?

socketgd is a wrapper around socket.io sockets that provides reliable message delivery. It tracks messages sent
over the socket, and waits for `ack`s to be received from the other end indicating that the messages
were handled successfully.

If a socket gets disconnected then the application can provide a new socket to
the socketgd wrapper.
In the client, all messages that have not received an ack from the server will be sent again over the new socket.
In the server, if messages arrive that it has already acked, then they will be silently discarded.

## Client Usage

```javascript
var SocketGD = require('socketgd').SocketGD;

// in the browser
// <script src="socketgd.js"></script>

// create the socket.io socket
var socket = io(url, config);

// wrap the socket with socketgd
var socketgd = new SocketGD(socket);

// use socketgd instead of the socket
socketgd.on('connect', function() {
  console.log("I'm connected!");
});

socketgd.send('I am sending you a message');
socketgd.emit('your-event', {hi: 'there'});

// ...the socket gets disconnected before receiving an ack for 'your-event'...

// the un-acked 'your-event' message will be silently sent again
socketgd.setSocket(newSocket);

```

## Server Usage

```javascript
var SocketGD = require('socketgd').SocketGD;
var socketgd = new SocketGD();

// socket.io server
io.on('connection', function(socket) {

  // set the underlying socket in socketgd.
  // if any already acked messages arrive they will be silently discarded
  socketgd.setSocket(socket);

  socketgd.on('message', function(message, ack, msgId) {
    console.log('Got message ' + message + ' with id ' + msgId);
    // finished handling the message, ack it
    ack();
  });

  socketgd.on('my-event', function(data, ack, msgId) {
    console.log('Got my-event ' + data + ' with id ' + msgId);
    // finished handling the event, ack it
    ack();
  });

});
```

## License

The MIT License (MIT)

Copyright (c) 2015 Capriza Inc.

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.

