# Notification-Service
# Real-Time Notifications with Next.js and Spring Boot

This project demonstrates how to implement real-time notifications in a Next.js application using WebSockets with a Spring Boot backend. It provides a comprehensive guide on setting up WebSocket communication between a Next.js frontend and a Spring Boot backend.

## Table of Contents

1. [Spring Boot Backend](#spring-boot-backend)
2. [Next.js Frontend](#nextjs-frontend)

## Spring Boot Backend

### WebSocket Configuration

**`src/main/java/com/example/config/WebSocketConfig.java`**

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;
import org.springframework.web.socket.server.support.HttpSessionHandshakeInterceptor;

@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(driverNotificationHandler(), "/ws/notifications")
                .setAllowedOrigins("*")
                .addInterceptors(new HttpSessionHandshakeInterceptor());
    }

    @Bean
    public DriverNotificationHandler driverNotificationHandler() {
        return new DriverNotificationHandler();
    }
}
```
## WebSocket Handler
**`src/main/java/com/example/config/DriverNotificationHandler.java`**

```java
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.handler.TextWebSocketHandler;
import org.springframework.web.socket.TextMessage;

import java.io.IOException;
import java.util.HashSet;
import java.util.Set;

public class DriverNotificationHandler extends TextWebSocketHandler {

    private Set<WebSocketSession> sessions = new HashSet<>();

    @Override
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {
        sessions.add(session);
    }

    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {
        sessions.remove(session);
    }

    public void sendNotification(String driverId, String message) {
        for (WebSocketSession session : sessions) {
            try {
                if (session.isOpen()) {
                    session.sendMessage(new TextMessage(driverId + ": " + message));
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## Notification Controller
**`src/main/java/com/example/config/NotificationController.java`**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class NotificationController {

    @Autowired
    private DriverNotificationHandler driverNotificationHandler;

    @PostMapping("/api/notify")
    public String notifyDriver(@RequestParam String driverId, @RequestParam String message) {
        driverNotificationHandler.sendNotification(driverId, message);
        return "Notification sent";
    }
}
```

## nextjs frontend

## Create a New Next.js Project

```bash
npx create-next-app@latest next-websocket-app
cd next-websocket-app

```
## Create a WebSocket Hook
**`hooks/useWebSocket.js`**

```js
import { useEffect, useState } from 'react';

const useWebSocket = (url) => {
  const [messages, setMessages] = useState([]);
  const [ws, setWs] = useState(null);

  useEffect(() => {
    const socket = new WebSocket(url);

    socket.onmessage = (event) => {
      setMessages((prevMessages) => [...prevMessages, event.data]);
    };

    socket.onopen = () => {
      console.log('WebSocket connection opened');
    };

    socket.onclose = () => {
      console.log('WebSocket connection closed');
    };

    socket.onerror = (error) => {
      console.error('WebSocket error:', error);
    };

    setWs(socket);

    return () => {
      socket.close();
    };
  }, [url]);

  return messages;
};

export default useWebSocket;
```
## Create a WebSocket Hook
**`hooks/useWebSocket.js`**
```js
import React from 'react';
import useWebSocket from '../hooks/useWebSocket';

const DriverNotifications = () => {
  const messages = useWebSocket('ws://localhost:8080/ws/notifications');

  return (
    <div className="notification-container">
      <h2>Driver Notifications</h2>
      <div>
        {messages.map((message, index) => (
          <div key={index} className="notification">
            {message}
          </div>
        ))}
      </div>
      <style jsx>{`
        .notification-container {
          padding: 20px;
          background-color: #f4f4f4;
          border-radius: 5px;
          border: 1px solid #ddd;
          max-width: 600px;
          margin: 0 auto;
        }

        .notification {
          padding: 10px;
          margin-bottom: 10px;
          border: 1px solid #ddd;
          border-radius: 4px;
          background-color: #fff;
          box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
        }
      `}</style>
    </div>
  );
};

export default DriverNotifications;
```
## Create a WebSocket Hook
**`hooks/useWebSocket.js`**
```js
import Head from 'next/head';
import DriverNotifications from '../components/DriverNotifications';

const Home = () => {
  return (
    <div>
      <Head>
        <title>Driver Notifications</title>
        <meta name="description" content="Real-time driver notifications" />
        <link rel="icon" href="/favicon.ico" />
      </Head>

      <main>
        <h1>Welcome to the Driver Notifications App</h1>
        <DriverNotifications />
      </main>

      <style jsx>{`
        main {
          padding: 20px;
          text-align: center;
        }

        h1 {
          font-size: 2rem;
        }
      `}</style>
    </div>
  );
};

export default Home;
```


