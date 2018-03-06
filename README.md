[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.me/romshark)

# WebWire for JavaScript
[WebWire](https://github.com/qbeon/webwire-go) is a high-level asynchronous duplex messaging library built on top of [WebSockets](https://developer.mozilla.org/de/docs/WebSockets) and an open source binary message protocol with builtin sessions and support for UTF8 and UTF16 encoding.
The [webwire-js](https://github.com/qbeon/webwire-js) library provides a client implementation for JavaScript environments.

## WebWire Binary Protocol
WebWire is built for speed and portability implementing an open source binary protocol.
![Protocol Subset Diagram](https://github.com/qbeon/webwire-go/blob/master/docs/img/wwr_msgproto_diagram.svg)

More information about the protocol is available at [WebWire](https://github.com/qbeon/webwire-go).

## Examples
- **[Chat Room](https://github.com/qbeon/webwire-js/tree/master/examples/chatroom-client-vue)** - Demonstrates advanced use of the library. The corresponding [Golang Chat Room Server](https://github.com/qbeon/webwire-go/tree/master/examples/chatroom) implements the server-side part of the example.

## Features
### Request-Reply
Clients can initiate multiple simultaneous requests and receive replies asynchronously. Requests are multiplexed through the connection similar to HTTP2 pipelining.

```javascript
// Send a request to the server, will block the goroutine until replied
const {reply, err} = await client.request("", "sudo rm -rf /")
if (err != null) {
  // Oh oh, request failed for some reason!
}
reply // Here we go!
 ```

Timed requests will timeout and return an error if the server doesn't manage to reply within the specified time frame.

```javascript
// Send a request to the server, will fail if no reply is received within 200ms
const {reply, err} = await client.request("", "hurry up!", null, 200)
if (err != null) {
  // Probably timed out!
}
reply // Just in time!
```

### Client-side Signals
Individual clients can send signals to the server. Signals are one-way messages guaranteed to arrive not requiring any reply though.

```javascript
// Send signal to server
const err = await client.signal("eventA", "something")
```

### Server-side Signals
The server also can send signals to individual connected clients.

```javascript
const client = new WebWireClient(serverAddr, {
  onSignal: signal => {
    signal.payload // Handle server-side signal
  },
})
```

### Namespaces
Different kinds of requests and signals can be differentiated using the builtin namespacing feature.

```javascript
// Request authentication
const {
  reply: authReply,
  err: authReqErr
} = await client.request("auth", "user:pass")
if (authReqErr != null) {
  // Oh oh, authentication failed!
}

// Request data query
const {
  reply: queryReply,
  err: queryErr
} = await client.request("query", "sudo get sandwich")
if (queryErr != null) {
  // Oh oh, data query failed!
}
```

```javascript
const {err: aErr} = await client.signal("eventA", "something happend")
const {err: bErr} = await client.signal("eventB", "something else happened")
```

### Sessions
Individual connections can get sessions assigned to identify them. The state of the session is automagically synchronized between the client and the server. WebWire doesn't enforce any kind of authentication technique though, it just provides a way to authenticate a connection.

```javascript
const client = new WebWireClient(serverAddr, {
  onSessionCreated: newSession => {
    // The newly created session was just synchronized to the client
  },
})
```

### Automatic Session Restoration
WebWire clients persist their session to the local storage and try to restore it when connecting to the server repeatedly assuming the server didn't yet close this session.

```javascript
const client = new WebWireClient(serverAddr)
const err = await client.connect()
if (err != null) {
  // Oh, oh! Connection failed
}
client.session // Won't be null, if a previous session was restored
```

----

© 2018 Roman Sharkov <roman.sharkov@qbeon.com>
