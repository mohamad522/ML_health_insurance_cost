User Event System with RabbitMQ (Topic-based Messaging)
=======================================================

This project demonstrates how to use RabbitMQ for simulating user creation events in an e-commerce system. We utilize **Topic-based messaging** to trigger multiple actions upon a new user registration. The system sends an event when a new user is created, and this event is received by multiple subscribers. In our case, the **accounting system** creates a corresponding user and the **email system** sends a welcome email.

Project Structure
-----------------

1.  **EmitUserCreatedEvent.java** - A producer that publishes a user creation event to the RabbitMQ exchange.
2.  **ReceiveUserEvents.java** - A consumer that listens for user events and processes them (either for accounting or email systems).

* * * * *

## Running the Project

### Prerequisites

Before running the project, ensure that you have the necessary tools and dependencies set up:

-   **Docker** to run RabbitMQ locally.
-   **IntelliJ IDEA** for creating and running the Maven project.

### 1\. Installing Docker and Running RabbitMQ

First, install Docker if it's not already installed:

bash

Copy code

`sudo apt install docker.io`

Then, use Docker to run RabbitMQ with the management plugin (this will allow you to access RabbitMQ's web management interface):

bash

Copy code

`docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:4.0-management`

-   **5672** is the default RabbitMQ port for messaging.
-   **15672** is the port for RabbitMQ's web management UI.

Once RabbitMQ is running, you can access its management interface by navigating to `http://localhost:15672` in your web browser. The default login credentials are:

-   **Username**: `guest`
-   **Password**: `guest`

### 2\. Creating a Maven Project in IntelliJ IDEA

1.  Open **IntelliJ IDEA** and click on **Create New Project**.
2.  Choose **Maven** as the project type and click **Next**.
3.  Set the **GroupId** and **ArtifactId** for your project. For example:
    -   **GroupId**: `org.example`
    -   **ArtifactId**: `rabbitmq-topics`
4.  Click **Finish** to create the project.

### 3\. Adding Dependencies

Ensure you have the following dependency in your `pom.xml` to connect to RabbitMQ using Java:

xml

Copy code

`<dependency>     <groupId>com.rabbitmq</groupId>     <artifactId>amqp-client</artifactId>     <version>5.14.0</version> </dependency>`

This dependency is required to work with RabbitMQ from your Java code.

### 4\. Running the Project with Parameters in IntelliJ IDEA

To run the project with the appropriate parameters in IntelliJ IDEA, follow these steps:

1.  Open the **Run/Debug Configurations** by clicking on the dropdown in the top-right corner and selecting **Edit Configurations**.
2.  Click the **+** icon and choose **Application**.
3.  In the **Name** field, give it a name like "Run User Events".
4.  In the **Main class** field, set it to the class you want to run, e.g., `org.example.ReceiveUserEvents`.
5.  In the **Program Arguments** field, provide the binding keys as parameters, for example:

    bash

    Copy code

    `user.created.accounting.* user.created.email.*`

    These parameters will bind the consumer to both the accounting and email queues.

6.  Click **OK** to save the configuration.
7.  Now, click the **Run** button (the green triangle) in IntelliJ to start the application.

This will start the consumer program with the given routing key patterns. You can now send messages to RabbitMQ using the producer (`EmitUserCreatedEvent`) with routing keys like `user.created.accounting.new` or `user.created.email.welcome`.

### 5\. Running the Producer in IntelliJ IDEA

To run the producer, follow similar steps to create a new **Run Configuration** for the producer (`EmitUserCreatedEvent`). For example, if you want to send a message to the accounting system:

1.  Open **Run/Debug Configurations**.
2.  Add a new configuration for **Application**.
3.  Set the **Main class** to `org.example.EmitUserCreatedEvent`.
4.  In the **Program Arguments** field, provide the routing key and message, for example:

    bash

    Copy code

    `user.created.accounting "New user created: John Doe"`

5.  Click **OK** and run the producer.

This will publish a message to RabbitMQ, which will then be consumed by the receiver (based on the routing key binding).

* * * * *

RabbitMQ Architecture
---------------------

### Components:

-   **Exchange**: `topic_logs` (Topic-based exchange)
-   **Queues**:
    -   For accounting: `user.created.accounting`
    -   For email: `user.created.email`

### Routing Keys:

-   **Routing Key for Accounting System**: `user.created.accounting.*`
-   **Routing Key for Email System**: `user.created.email.*`

### Schema Diagram:

plaintext

Copy code

 `+-------------------+
                             |     Exchange      |
                             |    topic_logs     |
                             +-------------------+
                                    |
                                    | Topic Binding
                                    |
                  +-----------------+-------------------+
                  |                                     |
   +-----------------------+              +-----------------------+
   |   user.created.email.* |              |  user.created.accounting.* |
   +-----------------------+              +-----------------------+
                  |                                     |
        +-------------------+                  +-------------------+
        |    Email Queue     |                  | Accounting Queue  |
        +-------------------+                  +-------------------+`

In this architecture:

-   The Exchange (`topic_logs`) broadcasts the event to multiple queues based on the routing key.
-   Each Queue is bound to specific routing keys. The Accounting System listens for `user.created.accounting.*`, and the Email System listens for `user.created.email.*`.

* * * * *

Usage
-----

### 1\. EmitUserCreatedEvent (Producer)

The **EmitUserCreatedEvent** class is used to publish a user creation event to RabbitMQ. It requires a routing key and a message (username).

#### Command Line Usage

bash

Copy code

`java org.example.EmitUserCreatedEvent [routing_key] [message]`

-   **[routing_key]**: The routing key for the event (e.g., `user.created.accounting` or `user.created.email`).
-   **[message]**: The message that describes the new user (e.g., `"New user created: John Doe"`).

#### Example:

bash

Copy code

`java org.example.EmitUserCreatedEvent user.created.accounting "New user created: John Doe"`

This will send an event to the RabbitMQ exchange with the routing key `user.created.accounting` and the message `"New user created: John Doe"`.

#### Default Values:

-   **Routing Key**: If no routing key is provided, it defaults to `user.created`.
-   **Message**: If no message is provided, it defaults to `"New user created"`.

* * * * *

### 2\. ReceiveUserEvents (Consumer)

The **ReceiveUserEvents** class is used to receive user creation events and simulate the actions for the accounting and email systems.

#### Command Line Usage

bash

Copy code

`java org.example.ReceiveUserEvents [binding_key]...`

-   **[binding_key]**: You can bind the consumer to different queues by specifying the routing keys. For example, binding to both `user.created.accounting.*` and `user.created.email.*`.

#### Example:

bash

Copy code

`java org.example.ReceiveUserEvents user.created.accounting.* user.created.email.*`

This will bind the receiver to both the accounting and email system queues and process the messages accordingly.

* * * * *

### 3\. Expected Output

When the producer sends a message, the consumer will print the following output depending on the routing key.

#### Producer Example:

bash

Copy code

`java org.example.EmitUserCreatedEvent user.created.accounting "New user created: John Doe"`

#### Consumer Output:

plaintext

Copy code

`[*] Waiting for user created events. To exit press CTRL+C
[x] Received 'user.created.accounting.*':'New user created: John Doe'
[Accounting System] Creating user in accounting: New user created: John Doe
[x] Received 'user.created.email.*':'New user created: John Doe'
[Email System] Sending welcome email to: New user created: John Doe`

If the message has the routing key `user.created.accounting.*`, the consumer will process the event for the accounting system. If the routing key is `user.created.email.*`, the consumer will process the event for the email system.

* * * * *

Code Explanation
----------------

### EmitUserCreatedEvent.java (Producer)

-   **Purpose**: Publishes user creation events to the `topic_logs` exchange.
-   **Routing Key**: The routing key is either passed as the first argument (`argv[0]`) or defaults to `"user.created"`.
-   **Message**: The message is either passed as the second argument (`argv[1]`) or defaults to `"New user created"`.
-   **Key Methods**:
    -   `getRouting()`: Determines the routing key.
    -   `getMessage()`: Joins the remaining arguments to form the message.
    -   `joinStrings()`: Combines multiple arguments into a single string.

### ReceiveUserEvents.java (Consumer)

-   **Purpose**: Listens for user creation events from RabbitMQ and processes them based on the routing key.
-   **Routing Keys**: The consumer can be bound to multiple routing keys (e.g., `user.created.accounting.*`, `user.created.email.*`).
-   **Event Processing**: The event is processed based on the routing key:
    -   `user.created.accounting.*`: Simulates creating the user in the accounting system.
    -   `user.created.email.*`: Simulates sending a welcome email.

* * * * *

Test Cases
----------

### Test Case 1: Send and Receive Accounting Event

1.  Start the receiver and bind to `user.created.accounting.*`:

bash

Copy code

`java org.example.ReceiveUserEvents user.created.accounting.*`

1.  Send a message with the routing key `user.created.accounting`:

bash

Copy code

`java org.example.EmitUserCreatedEvent user.created.accounting "New user created: John Doe"`

#### Expected Output on the receiver side:

plaintext

Copy code

`[*] Waiting for user created events. To exit press CTRL+C
[x] Received 'user.created.accounting.*':'New user created: John Doe'
[Accounting System] Creating user in accounting: New user created: John Doe`

* * * * *

### Test Case 2: Send and Receive Email Event

1.  Start the receiver and bind to `user.created.email.*`:

bash

Copy code

`java org.example.ReceiveUserEvents user.created.email.*`

1.  Send a message with the routing key `user.created.email`:

bash

Copy code

`java org.example.EmitUserCreatedEvent user.created.email "New user created: John Doe"`

#### Expected Output on the receiver side:

plaintext

Copy code

`[*] Waiting for user created events. To exit press CTRL+C
[x] Received 'user.created.email.*':'New user created: John Doe'
[Email System] Sending welcome email to: New user created: John Doe`

* * * * *

Conclusion
----------

This project simulates a topic-based messaging system using RabbitMQ for handling events in an e-commerce system. With **EmitUserCreatedEvent** as the producer, we send events about new users being created, and with **ReceiveUserEvents**, we have multiple consumers (accounting and email systems) that process these events based on their routing keys.

You can modify and extend this project by adding more subscribers, events, and routing keys as needed.
