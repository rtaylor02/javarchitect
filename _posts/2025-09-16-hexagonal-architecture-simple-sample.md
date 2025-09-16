---
title: "Hexagonal Architecture - Simple Implementation in Java"
classes: wide
---



# Hexagonal Architecture in Java: A Simple Implementation with Multiple Ports

Hexagonal Architecture, also known as **Ports and Adapters**, is a software architectural style introduced by Alistair Cockburn. Its goal is to isolate the **core business logic** from external systems — such as databases, message brokers, or user interfaces — by putting them behind well-defined **ports** and implementing them through **adapters**.

This decoupling brings versatility: the same domain logic can interact with different clients and persistence mechanisms without modification.

---

## Why Hexagonal Architecture?

In a traditional layered architecture, the domain model often leaks dependencies on persistence or UI layers. For example, switching from a REST API to a CLI or from a relational database to an in-memory store can require changes to the business code.

Hexagonal architecture solves this by:

* **Defining input ports**: how external actors (users, APIs, scheduled jobs) communicate with the application.
* **Defining output ports**: how the application communicates with external systems (databases, queues, external services).
* **Keeping the core domain pure**: the business logic depends only on ports, not on specific technologies.

---

## Example Scenario

Let’s implement a simple **Task Management** system with:

* **2 Input Ports**:

  1. A REST API (adapter).
  2. A Command-Line Interface (adapter).

* **2 Output Ports**:

  1. A persistence store (adapter for in-memory or database).
  2. A notification service (adapter for console logging or email).

The core domain logic only knows about ports, not about HTTP, JDBC, or logging.

---

## Step 1: Define the Domain Model

```java
public class Task {
    private final String id;
    private final String description;
    private boolean completed;

    public Task(String id, String description) {
        this.id = id;
        this.description = description;
        this.completed = false;
    }

    public void markCompleted() {
        this.completed = true;
    }

    public boolean isCompleted() {
        return completed;
    }

    public String getId() {
        return id;
    }

    public String getDescription() {
        return description;
    }
}
```

---

## Step 2: Define Ports

### Input Ports (driving the application)

```java
public interface TaskUseCase {
    Task createTask(String description);
    void completeTask(String taskId);
}
```

### Output Ports (driven by the application)

```java
public interface TaskRepository {
    Task save(Task task);
    Task findById(String id);
}

public interface NotificationService {
    void notify(String message);
}
```

---

## Step 3: Core Application Logic

```java
import java.util.UUID;

public class TaskService implements TaskUseCase {

    private final TaskRepository taskRepository;
    private final NotificationService notificationService;

    public TaskService(TaskRepository taskRepository, NotificationService notificationService) {
        this.taskRepository = taskRepository;
        this.notificationService = notificationService;
    }

    @Override
    public Task createTask(String description) {
        Task task = new Task(UUID.randomUUID().toString(), description);
        taskRepository.save(task);
        notificationService.notify("Task created: " + task.getDescription());
        return task;
    }

    @Override
    public void completeTask(String taskId) {
        Task task = taskRepository.findById(taskId);
        task.markCompleted();
        taskRepository.save(task);
        notificationService.notify("Task completed: " + task.getDescription());
    }
}
```

Notice that `TaskService` doesn’t know if persistence is a database or in-memory, or if notifications go to email, SMS, or console.

---

## Step 4: Implement Adapters

### Output Adapters

In-memory persistence:

```java
import java.util.HashMap;
import java.util.Map;

public class InMemoryTaskRepository implements TaskRepository {
    private final Map<String, Task> store = new HashMap<>();

    @Override
    public Task save(Task task) {
        store.put(task.getId(), task);
        return task;
    }

    @Override
    public Task findById(String id) {
        return store.get(id);
    }
}
```

Console notification:

```java
public class ConsoleNotificationService implements NotificationService {
    @Override
    public void notify(String message) {
        System.out.println("NOTIFICATION: " + message);
    }
}
```

### Input Adapters

Command-line interface:

```java
public class CliAdapter {
    private final TaskUseCase taskUseCase;

    public CliAdapter(TaskUseCase taskUseCase) {
        this.taskUseCase = taskUseCase;
    }

    public void run() {
        Task task = taskUseCase.createTask("Write hexagonal architecture article");
        taskUseCase.completeTask(task.getId());
    }
}
```

REST controller (example using Spring Web):

```java
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/tasks")
public class RestAdapter {

    private final TaskUseCase taskUseCase;

    public RestAdapter(TaskUseCase taskUseCase) {
        this.taskUseCase = taskUseCase;
    }

    @PostMapping
    public Task create(@RequestParam String description) {
        return taskUseCase.createTask(description);
    }

    @PostMapping("/{id}/complete")
    public void complete(@PathVariable String id) {
        taskUseCase.completeTask(id);
    }
}
```

---

## Step 5: Wiring It Together

```java
public class Application {
    public static void main(String[] args) {
        TaskRepository repository = new InMemoryTaskRepository();
        NotificationService notification = new ConsoleNotificationService();
        TaskUseCase taskService = new TaskService(repository, notification);

        // CLI example
        CliAdapter cli = new CliAdapter(taskService);
        cli.run();

        // For REST, Spring Boot would autowire `taskService` into `RestAdapter`
    }
}
```

---

## Key Benefits

1. **Versatility**: The same `TaskService` can serve a CLI, REST API, or even a message consumer without modification.
2. **Replaceability**: Swap the `InMemoryTaskRepository` with a `JdbcTaskRepository` without touching the core logic.
3. **Testability**: The core can be tested with mock ports, no need for real databases or frameworks.
4. **Future-proofing**: Adding new adapters (e.g., GraphQL, Kafka, Email) requires no change in business logic.

---

## Conclusion

This simple implementation illustrates the power of Hexagonal Architecture in Java. Even with minimal domain logic, you can see how **ports and adapters provide flexibility and clarity**. The business core is free from infrastructure concerns, making the application more maintainable, extensible, and adaptable to future changes.

