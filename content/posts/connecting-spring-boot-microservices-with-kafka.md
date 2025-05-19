+++
date = '2023-07-10T23:27:03+07:00'
draft = false
title = 'Connecting Spring Boot Microservices With Kafka'
author = 'ahmad'
tags = ['kafka', 'spring-boot', 'java', 'microservices']
featuredImage = '/images/connecting-spring-boot-microservices-with-kafka/featured.png'
+++
There are many available ways to connect microservices. But, one of the most reliable ways is using message queue protocol. Apache Kafka is one of many message queue systems that you can use freely and it offers so many features that we can customize based on our need.

In this blog post, I will share with you how to connect two Spring Boot microservices using Apache Kafka.

# Project Architecture


![Image description](/images/connecting-spring-boot-microservices-with-kafka/spring-boot-kafka-1.webp)

## What this Project does

We will follow the architecture as shown in the image above. Things that happening in this project are :

- We create a project that grabs news (from a public news API) and then stores the data in our Redis database.

- We search the news by the published date. (yyyy-MM-dd).

- If the data already exists in our Redis database, it will grab directly into the database. Otherwise, it will send a request to the news API and then stores it in the Redis database.

# Microservices Compositions

Our project will consist of 2 microservices named :

- `user-api`

    - provides REST API that is accessible by clients' applications.

    - works as a topics publisher to Kafka.

    - it will check whether the data exists. Otherwise, it will send a topic to Kafka that later will be consumed by worker.

- `worker`

    - consumes topics that are published by the publisher (user-api) via Kafka.

    - stores data from News API to Redis.

## Other Requirements

To do this project, we need several things installed in our system :

- Docker (To run containers)

- Zookeeper (containered)

- Apache Kafka (containered)

- Redis (containered)

# Setup Kafka Message Broker + Redis

To setup a Kafka message broker (plus Redis), we gonna choose the easiest way by using Docker. To install the containers into Docker, you can use this `.yml` script.

```yaml
version: '3'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - 22181:2181
  
  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - 29092:29092
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_CREATE_TOPICS: "news:1:3"
      KAFKA_OFFSETS_RETENTION_MINUTES: 60
  
  redis:
    image: redis:latest
    restart: always
    ports:
      - 6379:6379
    environment:
      REDIS_PASSWORD: <change to your password>
    volumes: 
      - redis-data:/data
volumes:
  redis-data:
```

Important explanations :

- we set Kafka topic by utilizing `KAFKA_CREATE_TOPICS` environment variable. In this case, we name the topic with news.

- we can set the offset (retention period) of Kafka message by using `KAFKA_OFFSETS_RETENTION_MINUTES`. We set it to 60 minutes in this project.

- Redis password is optional. If you don't want to password in your Redis server you can remove the `REDIS_PASSWORD`.

Save this file with the name `docker-compose.yml`. Then, open your terminal in the same directory in which you put `docker-compose.yml` file. Run this command to install :

```
docker-compose up -d
```

After the command is finished, you can check the installation by using this command :

```
docker ps
```

If the installation is successful, you will see 3 new containers in your container list :

```
CONTAINER ID   IMAGE                              COMMAND                  CREATED         STATUS        PORTS                                         NAMES
e5e5245248ed   confluentinc/cp-kafka:latest       "/etc/confluent/dock…"   47 hours ago    Up 11 hours   9092/tcp, 0.0.0.0:29092->29092/tcp            monorepo-kafka-1
bf3a4892effa   confluentinc/cp-zookeeper:latest   "/etc/confluent/dock…"   47 hours ago    Up 11 hours   2888/tcp, 3888/tcp, 0.0.0.0:22181->2181/tcp   monorepo-zookeeper-1
3b0f8606ad00   redis:latest                       "docker-entrypoint.s…"   47 hours ago    Up 12 hours   0.0.0.0:6379->6379/tcp                        monorepo-redis-1
```

# Setup News API

We will utilize Mediastack for this project. they offer a free tier that can be used for this project. To set up your Mediastack account you can follow their [documentation page](https://mediastack.com/documentation). You need an access key to use their APIs. You can generate it after successfully registered.

For this project, we will only use one API provided by Mediastack that is :

```
http://api.mediastack.com/v1/news?access_key=<your_access_key>&countries=us&date=2023-05-11&limit=25
```

Here is the example data :

```json
{
    "pagination": {
        "limit": 25,
        "offset": 0,
        "count": 25,
        "total": 10000
    },
    "data": [
        {
            "author": "Central Oregon Daily News Sources",
            "title": "Bend-La Pine school bus driver job fairs begin Thursday",
            "description": "People interested in becoming a school bus driver ...",
            "url": "https://centraloregondaily.com/bend-la-pine-school-bus-driver-job-fair/",
            "source": "kohd",
            "image": null,
            "category": "general",
            "language": "en",
            "country": "us",
            "published_at": "2023-05-11T00:02:02+00:00"
        },
        {
            "author": "Norni Mahadi",
            "title": "Miri City Council embarks on ‘international links’ to lure tourists to the Resort City",
            "description": "MIRI (May 11): The Miri City Council (MCC) is ready to do more ...",
            "url": "https://www.theborneopost.com/2023/05/11/miri-city-council-embarks-on-international-links-to-lure-tourists-to-the-resort-city/",
            "source": "theborneopost",
            "image": "https://www.theborneopost.com/newsimages/2023/05/myy-100523-nm-MiriCityDay-p1.jpeg",
            "category": "general",
            "language": "en",
            "country": "us",
            "published_at": "2023-05-11T00:00:19+00:00"
        }
    ]
}
```

We will save the data straightforward to Redis database as our focus is only to demonstrate how Kafka is working with microservices.

# Publisher Service (user-api)

## Project Detail

You can generate your project template via [Spring Initializr](https://start.spring.io/), with some notes to follow this project :

* Use **Gradle - Groovy** for the project manager (you can also use Maven if you feel more confident with it, just adjust accordingly).

* Use **Java** as the programming language.


List of Dependency :

* Lombok

* Kafka

* Redis Reactive

* Webflux

## Project Structure

You can use this project structure as a reference to make it easier to follow this tutorial.

```

├───.gradle
│   └─── ...
├───gradle
│   └───wrapper
└───src
	├───main
	│   ├───java
	│   │   └───com
	│   │       └───justahmed99
	│   │           └───userapi
	│   │               ├───UserApiApplication.java
	│   │               │   
	│   │               ├───config
    │	│				│   ├───KafkaProducerConfig.java
	│   │               │   ├───KafkaTopicConfig.java
	│   │               │   ├───KafkaTopicConfig.java
	│   │               │   └───RedisConfig.java
	│   │               │
	│   │               ├───controller
	│   │               │    └───MessageController.java
	│   │               ├───repository
	│   │               │
	│	│				├───NewsRepository.java
	│   │               │   └───impl
	│   │               │        └───NewsRepositoryImpl.java
	│   │               │
	│   │               └───service
	│   │                   │   MessageService.java
	│   │                   │
	│   │                   └───impl
	│   │                           MessageServiceImpl.java
	│   │
	│   └───resources
	│        └───application.yml
	│
	└───test
		└─── ...
```
## Codes

### UserApiApplication.java

```java
package com.justahmed99.userapi;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class UserApiApplication {
	public static void main(String[] args) {
		SpringApplication.run(UserApiApplication.class, args);
	}
}
```

### config/KafkaProducerConfig.java

```java
package com.justahmed99.userapi.config;

import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.core.ProducerFactory;

import java.util.HashMap;
import java.util.Map;

@Configuration
public class KafkaProducerConfig {
    @Value("${spring.kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Bean
    public Map<String, Object> producerConfig() {
        HashMap<String, Object> props = new HashMap<>();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);

        return props;
    }

    @Bean
    public ProducerFactory<String, String> producerFactory() {
        return new DefaultKafkaProducerFactory<>(producerConfig());
    }

    @Bean
    public KafkaTemplate<String, String> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
}
```

### config/KafkaTopicConfig.java

```java
package com.justahmed99.userapi.config;

import org.apache.kafka.clients.admin.NewTopic;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.config.TopicBuilder;

@Configuration
public class KafkaTopicConfig {
    @Bean
    public NewTopic newsTopic() {
        return TopicBuilder.name("news").build();
    }
}
```

### config/RedisConfig.java

```java
package com.justahmed99.userapi.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.ReactiveRedisConnectionFactory;
import org.springframework.data.redis.connection.RedisPassword;
import org.springframework.data.redis.connection.RedisStandaloneConfiguration;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.core.ReactiveRedisOperations;
import org.springframework.data.redis.core.ReactiveRedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.util.Objects;

@Configuration
public class RedisConfig {
    @Value("${spring.data.redis.host}")
    private String host;

    @Value("${spring.data.redis.port}")
    private String port;

    @Value("${spring.data.redis.password}")
    private String password;

    @Bean
    public ReactiveRedisConnectionFactory reactiveRedisConnectionFactory() {
        RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration();
        redisStandaloneConfiguration.setHostName(Objects.requireNonNull(host));
        redisStandaloneConfiguration.setPort(Integer.parseInt(Objects.requireNonNull(port)));
        redisStandaloneConfiguration.setPassword(RedisPassword.of(Objects.requireNonNull(password)));
        return new LettuceConnectionFactory(redisStandaloneConfiguration);
    }

    @Bean
    public ReactiveRedisOperations<String, Object> redisOperations(
            ReactiveRedisConnectionFactory reactiveRedisConnectionFactory
    ) {
        Jackson2JsonRedisSerializer<Object> serializer = new Jackson2JsonRedisSerializer<>(Object.class);

        RedisSerializationContext.RedisSerializationContextBuilder<String, Object> builder =
                RedisSerializationContext.newSerializationContext(new StringRedisSerializer());

        RedisSerializationContext<String, Object> context = builder.value(serializer).hashValue(serializer)
                .hashKey(serializer).build();

        return new ReactiveRedisTemplate<>(reactiveRedisConnectionFactory, context);
    }
}
```

### dto/request/MessageRequest.java

```java
package com.justahmed99.userapi.dto.request;

import lombok.Data;

@Data
public class MessageRequest {
    private String message;
}
```

### dto/response/DataResponse.java

```java
package com.justahmed99.userapi.dto.response;

import com.fasterxml.jackson.annotation.JsonInclude;
import lombok.AllArgsConstructor;
import lombok.Data;

@Data
@AllArgsConstructor
public class DataResponse<T> {
    private String message;
    private Boolean status;
    @JsonInclude(JsonInclude.Include.NON_NULL)
    private T data;
}
```

### repository/NewsRepository.java

```java
package com.justahmed99.userapi.repository;

import reactor.core.publisher.Mono;

public interface NewsRepository {
    Mono<Object> getNews(String date);
}
```

### repository/impl/NewsRepositoryImpl.java

```java
package com.justahmed99.userapi.repository.impl;

import com.justahmed99.userapi.repository.NewsRepository;
import org.springframework.data.redis.core.ReactiveRedisOperations;
import org.springframework.stereotype.Repository;
import reactor.core.publisher.Mono;

@Repository
public class NewsRepositoryImpl implements NewsRepository {

    private final ReactiveRedisOperations<String, Object> redisOperations;

    public NewsRepositoryImpl(
            ReactiveRedisOperations<String, Object> redisOperations
    ) {
        this.redisOperations = redisOperations;
    }

    @Override
    public Mono<Object> getNews(String date) {
        return redisOperations.opsForValue().get(date);
    }
}
```

### service/MessageService.java

```java
package com.justahmed99.userapi.service;

import reactor.core.publisher.Mono;

public interface MessageService {
    Mono<Void> publishToMessageBroker(String date);
    Mono<Object> getNews(String date);
}
```

### service/impl/MessageServiceImpl.java

```java
package com.justahmed99.userapi.service.impl;

import com.justahmed99.userapi.repository.NewsRepository;
import com.justahmed99.userapi.service.MessageService;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;
import reactor.core.publisher.Mono;

@Service
public class MessageServiceImpl implements MessageService {
    private final KafkaTemplate<String, String> kafkaTemplate;
    private final NewsRepository newsRepository;

    public MessageServiceImpl(
            KafkaTemplate<String, String> kafkaTemplate,
            NewsRepository newsRepository
    ) {
        this.kafkaTemplate = kafkaTemplate;
        this.newsRepository = newsRepository;
    }

    @Override
    public Mono<Void> publishToMessageBroker(String date) {
        ProducerRecord<String, String> record = new ProducerRecord<>("news", null, date);
        return Mono.fromFuture(kafkaTemplate.send(record))
                .then();
    }

    @Override
    public Mono<Object> getNews(String date) {
        return newsRepository.getNews(date)
                .flatMap(Mono::just)
                .switchIfEmpty(Mono.defer(() -> publishToMessageBroker(date)));
    }
}
```

### controller/MessageController.java

```java
package com.justahmed99.userapi.controller;

import com.justahmed99.userapi.dto.request.MessageRequest;
import com.justahmed99.userapi.dto.response.DataResponse;
import com.justahmed99.userapi.service.MessageService;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import reactor.core.publisher.Mono;

@RestController
@RequestMapping(value = "/message")
public class MessageController {
    private final MessageService service;

    public MessageController(
            MessageService service
    ) {
        this.service = service;
    }

    @GetMapping("")
    public Mono<ResponseEntity<DataResponse<Object>>> getNews(@RequestParam(name = "date") String date) {
        return service.getNews(date)
                .flatMap(data -> Mono.just(
                        ResponseEntity.status(HttpStatus.OK)
                                .body(new DataResponse<>
                                        ("data found", true, data))))
                .switchIfEmpty(Mono.defer(() -> Mono.just(
                        ResponseEntity.status(HttpStatus.NOT_FOUND).
                                body(new DataResponse<>
                                        ("data not found, sending request to broker", false, null)))));
    }
}
```

### application.yml

```yaml
server:
  port: 8080

spring:
  kafka:
    bootstrap-servers: localhost:29092
  data:
    redis:
      host: localhost
      port: 6379
      password: <your_redis_password>
```

Change `your_redis_password` with your Redis server's password. Otherwise, just get rid of it.

# Consumer Service (worker)

## Project Detail

For initializing this project, do the same with `user-api` project.

## Project Structure

You can use this project structure as a reference to make it easier to follow this tutorial.

```plaintext
├───.gradle
│   └─── ...
├───gradle
│   └───wrapper
└───src
	├───main
	│   ├───java
	│   │   └───com
	│   │       └───justahmed99
	│   │           └───worker
	│   │               ├───WorkerApplication.java
	│   │               │   
	│   │               ├───config
    │	│               ├───KafkaProducerConfig.java
	│   │               │   ├───KafkaTopicConfig.java
	│   │               │   ├───KafkaTopicConfig.java
	│   │               │   └───RedisConfig.java
	│   │               │   └───WebClientConfig.java
	│   │               │
	│   │               ├───listener
	│   │               │    └───KafkaListener.java
	│   │               │
	│   │               ├───repository
	│   │               │
	│	│				├───NewsRepository.java
	│   │               │   └───impl
	│   │               │        └───NewsRepositoryImpl.java
	│   │               │
	│   │               └───service
	│   │                   │   WebClientService.java
	│   │                   │
	│   │                   └───impl
	│   │                           WebClientServiceImpl.java
	│   │
	│   └───resources
	│        └───application.yml
	│
	└───test
		└─── ...
```

## Codes

### config/KafkaConfigConsumer.java

```java
package com.justahmed99.worker.config;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.config.KafkaListenerContainerFactory;
import org.springframework.kafka.core.ConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.listener.ConcurrentMessageListenerContainer;

import java.util.HashMap;
import java.util.Map;

@Configuration
public class KafkaConsumerConfig {
    @Value("${spring.kafka.bootstrap-servers}")
    private String bootstrapServers;

    public Map<String, Object> consumerConfig() {
        HashMap<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "message-group");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);

        return props;
    }

    @Bean
    public ConsumerFactory<String, String> consumerFactory() {
        return new DefaultKafkaConsumerFactory<>(consumerConfig());
    }

    @Bean
    public KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<String, String>> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        return factory;
    }
}
```

### config/KafkaTopicConfig.java

```java
package com.justahmed99.worker.config;

import org.apache.kafka.clients.admin.NewTopic;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.config.TopicBuilder;

@Configuration
public class KafkaTopicConfig {
    @Bean
    public NewTopic newsTopic() {
        return TopicBuilder.name("news").build();
    }
}
```

### config/RedisConfig.java

```java
package com.justahmed99.worker.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.ReactiveRedisConnectionFactory;
import org.springframework.data.redis.connection.RedisPassword;
import org.springframework.data.redis.connection.RedisStandaloneConfiguration;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.core.ReactiveRedisOperations;
import org.springframework.data.redis.core.ReactiveRedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.util.Objects;

@Configuration
public class RedisConfig {
    @Value("${spring.data.redis.host}")
    private String host;

    @Value("${spring.data.redis.port}")
    private String port;

    @Value("${spring.data.redis.password}")
    private String password;

    @Bean
    public ReactiveRedisConnectionFactory reactiveRedisConnectionFactory() {
        RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration();
        redisStandaloneConfiguration.setHostName(Objects.requireNonNull(host));
        redisStandaloneConfiguration.setPort(Integer.parseInt(Objects.requireNonNull(port)));
        redisStandaloneConfiguration.setPassword(RedisPassword.of(Objects.requireNonNull(password)));
        return new LettuceConnectionFactory(redisStandaloneConfiguration);
    }

    @Bean
    public ReactiveRedisOperations<String, Object> redisOperations(
            ReactiveRedisConnectionFactory reactiveRedisConnectionFactory
    ) {
        Jackson2JsonRedisSerializer<Object> serializer = new Jackson2JsonRedisSerializer<>(Object.class);

        RedisSerializationContext.RedisSerializationContextBuilder<String, Object> builder =
                RedisSerializationContext.newSerializationContext(new StringRedisSerializer());

        RedisSerializationContext<String, Object> context = builder.value(serializer).hashValue(serializer)
                .hashKey(serializer).build();

        return new ReactiveRedisTemplate<>(reactiveRedisConnectionFactory, context);
    }
}
```

### config/WebClientConfig.java

```java
package com.justahmed99.worker.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.reactive.function.client.WebClient;

@Configuration
public class WebClientConfig {

    @Value("${mediastack.uri}")
    private String apiUri;

    @Bean
    public WebClient webClient() {
        return WebClient.create(apiUri);
    }
}
```

### listener/KafkaListeners.java

```java
package com.justahmed99.worker.listener;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.justahmed99.worker.repository.NewsRepository;
import com.justahmed99.worker.service.WebClientService;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;
import reactor.core.publisher.Mono;


@Component
public class KafkaListeners {

    private final WebClientService webClientService;
    private final NewsRepository newsRepository;

    public KafkaListeners (
            WebClientService webClientService,
            NewsRepository newsRepository
    ) {
        this.webClientService = webClientService;
        this.newsRepository = newsRepository;
    }

    @KafkaListener(topics = "news", groupId = "message-group")
    void listener(String date) {
        System.out.printf("Listener received: %s%n", date);

        Mono<ResponseEntity<String>> responseEntity = webClientService.sendRequest(date);

        responseEntity.subscribe(response -> {
            HttpStatus status = (HttpStatus) response.getStatusCode();
            if (status.equals(HttpStatus.OK)) {
                try {
                    newsRepository.saveNews(date, response.getBody())
                            .subscribe(isSaved -> {
                                if (isSaved)
                                    System.out.println("Data successfully saved in cache");
                                else
                                    System.out.println("Data save failed");
                            });
                } catch (JsonProcessingException e) {
                    throw new RuntimeException(e);
                }
            }
            System.out.println(status.value());
        });
    }
}
```

### repository/NewsRepository.java

```java
package com.justahmed99.worker.repository;

import com.fasterxml.jackson.core.JsonProcessingException;
import org.springframework.stereotype.Repository;
import reactor.core.publisher.Mono;

@Repository
public interface NewsRepository {
    Mono<Boolean> saveNews(String date, Object newsObject) throws JsonProcessingException;
}
```

### repository/impl/NewsRepositoryImpl.java

```java
package com.justahmed99.worker.repository.impl;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.justahmed99.worker.repository.NewsRepository;
import org.springframework.data.redis.core.ReactiveRedisOperations;
import org.springframework.stereotype.Repository;
import reactor.core.publisher.Mono;

import java.time.Duration;

@Repository
public class NewsRepositoryImpl implements NewsRepository {
    private final ReactiveRedisOperations<String, Object> redisOperations;

    public NewsRepositoryImpl(
            ReactiveRedisOperations<String, Object> redisOperations
    ) {
        this.redisOperations = redisOperations;
    }

    @Override
    public Mono<Boolean> saveNews(String date, Object newsObject) throws JsonProcessingException {
        Duration ttl = Duration.ofHours(1);
        ObjectMapper objectMapper = new ObjectMapper();
        return redisOperations.opsForValue().set(date, objectMapper.readTree(newsObject.toString()), ttl);
    }
}
```

### service/WebClientService.java

```java
package com.justahmed99.worker.service;

import org.springframework.http.ResponseEntity;
import reactor.core.publisher.Mono;

public interface WebClientService {
    Mono<ResponseEntity<String>> sendRequest(String date);
}
```

### service/impl/WebClientServiceImpl.java

```java
package com.justahmed99.worker.service.impl;

import com.justahmed99.worker.service.WebClientService;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Service;
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;

@Service
public class WebClientServiceImpl implements WebClientService {

    @Value("${mediastack.api-key}")
    private String apiKey;

    @Value("${mediastack.countries}")
    private String countries;

    @Value("${mediastack.limit}")
    private String limit;

    private final WebClient webClient;

    public WebClientServiceImpl(WebClient webClient) {
        this.webClient = webClient;
    }

    @Override
    public Mono<ResponseEntity<String>> sendRequest(String date) {
        return webClient.get()
                .uri(uriBuilder -> uriBuilder
                        .queryParam("access_key", apiKey)
                        .queryParam("countries", countries)
                        .queryParam("limit", limit)
                        .queryParam("date", date)
                        .build())
                .retrieve()
                .toEntity(String.class);
    }
}
```

### application.yml

```yaml
server:
  port: 9090

spring:
  kafka:
    bootstrap-servers: localhost:29092
  data:
    redis:
      host: localhost
      port: 6379
      password: myredis

mediastack:
  uri: http://api.mediastack.com/v1/news
  api-key: <your-api-key>
  countries: us
  limit: 25
```

**NOTES:**

* Change `your_redis_password` with your Redis server's password. Otherwise, just get rid of it.

* Change `your-api-key` with API key you got from Mediastack.


# End Point

To try to make a request first you need to :

* Make sure zookeeper container is running

* Make sure Kafka container is running

* Make sure Redis container is running

* Run `user-api` service

* Run `worker` service


To make a request you can hit :

```plaintext
http://localhost:8080/date=<yyyy-MM-dd>
```

# Conclusion

There's how you can connect two Spring Boot microservices with Kafka. Hope this helps. 😊

Here is the [GitHub repo](https://github.com/justahmed99/spring-boot-kafka-monorepo).