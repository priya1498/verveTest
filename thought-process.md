Overview
The task at hand involves building a Spring Boot application that provides a REST service to process a large number of incoming requests while ensuring high throughput and performance. The application is designed to handle an endpoint that accepts an integer id as a mandatory query parameter and an optional string endpoint query parameter.

The application is expected to:

Process at least 10,000 requests per second.
Track the number of unique requests based on the id parameter.
Periodically log the unique request counts.
Send the request count to a specified external endpoint (HTTP GET or POST).
Additionally, the following extensions were considered:

Switch the external request method from GET to POST.
Ensure that the service is able to handle duplicate id values across different instances behind a load balancer.
Instead of logging request counts locally, send them to a distributed streaming service.
Approach
1. Service Architecture
The application architecture follows a microservice-style design:

Controller Layer: Exposes a REST API that accepts GET requests. The controller takes the request parameters, processes them via the service layer, and returns the appropriate response.
Service Layer: Handles the core business logic. This includes tracking unique request ids, interacting with external endpoints, and logging data.
External Request Handling: The service makes outbound HTTP requests to the provided external URL, logging the response status code and body.
Logging: A simple logging mechanism tracks unique request counts every minute.
2. Request Processing & Performance
Unique Request Identification: Requests are tracked using the id query parameter. A HashSet is used to store unique request IDs, ensuring no duplicate processing.
High Throughput Handling: To handle 10K requests per second, performance optimizations such as minimal synchronization and the use of in-memory collections (e.g., HashSet) were considered. The application could be further optimized by introducing a caching mechanism (e.g., Redis) for tracking unique requests across multiple instances behind a load balancer.
3. External HTTP Requests
The service is designed to fire an HTTP GET or POST request to an external endpoint with the count of unique requests as a query parameter. For simplicity, the first version of the implementation uses RestTemplate for outbound HTTP requests.
GET Request: The RestTemplate.getForEntity() method is used to send a GET request to the external URL.
POST Request: As an extension, the HTTP method can be switched to POST by changing the request logic to use RestTemplate.postForEntity(). The body of the POST request will contain the unique request count.
4. Logging and Monitoring
Logging: The application logs the unique request count every minute to a local file using the Slf4j logger. This allows tracking the number of requests processed and enables debugging.
Distributed Logging/Streaming: As an extension, the service can be modified to push the unique request count to a distributed streaming service such as Apache Kafka or AWS Kinesis for real-time monitoring and analysis.
5. Dealing with Load Balancer Scenarios
Cross-instance Deduplication: In a real-world production scenario where the service is behind a load balancer, multiple instances of the application might receive the same request (with the same id). To ensure accurate deduplication across instances, a shared data store like Redis could be used to synchronize request counts and ensure uniqueness even if requests are routed to different instances.
In the current solution, request IDs are tracked in an in-memory HashSet. For distributed environments, Redis or a similar solution would be considered.
6. Scalability Considerations
The initial solution works well for a single instance, but in production, the system would need to scale horizontally to handle large numbers of concurrent requests. Using technologies like Redis for distributed caching, or even a more advanced message queue for high-throughput data processing, would be key in ensuring scalability.
Asynchronous Processing: For high-performance environments, tasks like sending HTTP requests and logging counts can be processed asynchronously to prevent blocking incoming requests.
Design Decisions
1. Spring Boot Framework
Spring Boot was chosen for its rapid development, ease of configuration, and extensive ecosystem. It is well-suited for building microservices and REST APIs, and the @RestController and @GetMapping annotations were leveraged to define the service endpoint.

2. Handling HTTP Requests
RestTemplate: This is the primary HTTP client used for making external requests. While Spring WebFlux’s WebClient could offer better performance for non-blocking I/O, RestTemplate was chosen for simplicity and familiarity. For high-concurrency scenarios, WebClient could be introduced.
3. Request Deduplication
In-memory HashSet: Initially, a simple HashSet was used to track unique id values for deduplication. This works well for a single instance but would need a distributed caching system for horizontal scalability.
4. Logging Strategy
Slf4j with Logback: Used for logging the unique count of requests each minute. A custom scheduled task logs the count to the console or a file.
External Streaming Service: As an extension, the service could be further enhanced to send this data to a distributed streaming service like Kafka for better data management and analysis.
5. Testing & Validation
The solution was tested using both unit tests and integration tests. The tests ensure the core functionalities, such as:

Correct processing of unique IDs.
Proper logging and counting.
Correct HTTP request handling, including making outbound GET/POST requests to external endpoints.
Future Improvements
1. Distributed Deduplication
In a multi-instance setup behind a load balancer, request IDs should be stored in a centralized data store such as Redis to ensure unique IDs are consistently tracked across instances.

2. Handling Rate-Limited External APIs
For better robustness, rate-limiting logic could be introduced to prevent overloading external APIs. For instance, integrating a backoff policy or retry mechanism would make the application more resilient.

3. High Throughput Optimization
Although the application can handle a large number of requests, further optimizations can be done using asynchronous processing for tasks like logging and external API calls.

4. Docker and Kubernetes
For scaling and deployment, a Docker container could be used, and Kubernetes could manage the deployment in production to ensure high availability and scaling based on the number of requests.

