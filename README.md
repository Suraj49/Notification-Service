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
