# Notification-Service
# Real-Time Notifications with Next.js and Spring Boot

This project demonstrates how to implement real-time notifications in a Next.js application using WebSockets with a Spring Boot backend. It provides a comprehensive guide on setting up WebSocket communication between a Next.js frontend and a Spring Boot backend.

## Table of Contents

1. [Spring Boot Backend](#spring-boot-backend)
2. [Next.js Frontend](#nextjs-frontend)
3. [Running the Application](#running-the-application)
4. [Testing](#testing)
5. [Summary](#summary)

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
