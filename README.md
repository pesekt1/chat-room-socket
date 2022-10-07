# Socket communication between client and server

## Dependencies:
- socket.io - for the serer
- socket.io-client - for the client

## Install the dependencies:
```
npm install 
```

## Start server and client:
```
npm run start:client
npm run start:server
```

## Server
We create a socket and define events for the server. The server listens to the events and sends back a response:
- connection
- send
- disconnect
- username

emit() function sends data to the channel. The data is received by all the clients connected to the channel.

```js
import http from "http";
import socket from "socket.io";

const server = http.createServer();

const io = socket(server, { transports: ["websocket", "polling"] });

const users = [];
io.on("connection", (client) => {
  client.on("username", (username) => {
    const user = {
      name: username,
      id: client.id,
    };
    users[client.id] = user;
    io.emit("connected", user);
    io.emit("users", Object.values(users));
  });

  client.on("send", (message) => {
    io.emit("message", {
      text: message,
      date: new Date().toISOString(),
      user: users[client.id],
    });
  });

  client.on("disconnect", () => {
    delete users[client.id];
    io.emit("disconnected", client.id);
  });
});
server.listen(3000);
```

## Client
Client connects to the server. 
It listens to thee events.
It also emmits data to the server:
- username - after a new user connects, the server adds this new user to an array and emmits the array to all the clients.
- message - when a new message is sent, the server emmits the message to all the clients.

```js
import React, { useEffect, useState } from "react";
import ReactDOM from "react-dom";
import io from "socket.io-client";

const username = prompt("what is your username");

const socket = io("http://localhost:3000", {
  transports: ["websocket", "polling"],
});

const App = () => {
  const [users, setUsers] = useState([]);
  const [message, setMessage] = useState("");
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    socket.on("connect", () => {
      socket.emit("username", username);
    });

    socket.on("users", (users) => {
      setUsers(users);
    });

    socket.on("message", (message) => {
      setMessages((messages) => [...messages, message]);
    });

    socket.on("connected", (user) => {
      setUsers((users) => [...users, user]);
    });

    socket.on("disconnected", (id) => {
      setUsers((users) => {
        return users.filter((user) => user.id !== id);
      });
    });
  }, []);

  const submit = (event) => {
    event.preventDefault();
    socket.emit("send", message);
    setMessage("");
  };

  //HTML markup
  return (...);
};

ReactDOM.render(<App />, document.getElementById("root"));
```