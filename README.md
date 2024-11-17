
## Architecture Overview

In this architecture:

- The **Exchange** (`topic_logs`) broadcasts the event to multiple queues based on the routing key.
- Each **Queue** is bound to specific routing keys. The **Accounting System** listens for `user.created.accounting.*`, and the **Email System** listens for `user.created.email.*`.

---

## Usage

### 1. EmitUserCreatedEvent (Producer)

The **EmitUserCreatedEvent** class is used to publish a user creation event to RabbitMQ. It requires a routing key and a message (username).

#### Command Line Usage

```bash
java org.example.EmitUserCreatedEvent [routing_key] [message]


• [routing_key]: The routing key for the event (e.g., user.created.accounting or user.created.email).
• [message]: The message that describes the new user (e.g., "New user created: John Doe").


Example:
bash
Copy code


java org.example.EmitUserCreatedEvent user.created.accounting "New user created: John Doe"


This will send an event to the RabbitMQ exchange with the routing key user.created.accounting and the message "New user created: John Doe".

Default Values:
◇ Routing Key: If no routing key is provided, it defaults to user.created.
◇ Message: If no message is provided, it defaults to "New user created".


2. ReceiveUserEvents (Consumer)
The ReceiveUserEvents class is used to receive user creation events and simulate the actions for the accounting and email systems.

Command Line Usage
bash
Copy code


java org.example.ReceiveUserEvents [binding_key]...


◇ [binding_key]: You can bind the consumer to different queues by specifying the routing keys. For example, binding to both user.created.accounting.* and user.created.email.*.


Example:
bash
Copy code


java org.example.ReceiveUserEvents user.created.accounting.* user.created.email.*


This will bind the receiver to both the accounting and email system queues and process the messages accordingly.

3. Expected Output
When the producer sends a message, the consumer will print the following output depending on the routing key.

Producer Example:
bash
Copy code


java org.example.EmitUserCreatedEvent user.created.accounting "New user created: John Doe"



Consumer Output:
plaintext
Copy code


[*] Waiting for user created events. To exit press CTRL+C
[x] Received 'user.created.accounting.*':'New user created: John Doe'
[Accounting System] Creating user in accounting: New user created: John Doe
[x] Received 'user.created.email.*':'New user created: John Doe'
[Email System] Sending welcome email to: New user created: John Doe


If the message has the routing key user.created.accounting.*, the consumer will process the event for the accounting system. If the routing key is user.created.email.*, the consumer will process the event for the email system.

Code Explanation

EmitUserCreatedEvent.java (Producer)
◇ Purpose: Publishes user creation events to the topic_logs exchange.
◇ Routing Key: The routing key is either passed as the first argument (argv[0]) or defaults to "user.created".
◇ Message: The message is either passed as the second argument (argv[1]) or defaults to "New user created".
◇ Key Methods:▪ getRouting(): Determines the routing key.
▪ getMessage(): Joins the remaining arguments to form the message.
▪ joinStrings(): Combines multiple arguments into a single string.




ReceiveUserEvents.java (Consumer)
◇ Purpose: Listens for user creation events from RabbitMQ and processes them based on the routing key.
◇ Routing Keys: The consumer can be bound to multiple routing keys (e.g., user.created.accounting.*, user.created.email.*).
◇ Event Processing: The event is processed based on the routing key:▪ user.created.accounting.*: Simulates creating the user in the accounting system.
▪ user.created.email.*: Simulates sending a welcome email.




Test Cases

Test Case 1: Send and Receive Accounting Event
1. Start the receiver and bind to user.created.accounting.*:
bash
Copy code


java org.example.ReceiveUserEvents user.created.accounting.*


1. Send a message with the routing key user.created.accounting:
bash
Copy code


java org.example.EmitUserCreatedEvent user.created.accounting "New user created: John Doe"



Expected Output on the receiver side:
plaintext
Copy code


[*] Waiting for user created events. To exit press CTRL+C
[x] Received 'user.created.accounting.*':'New user created: John Doe'
[Accounting System] Creating user in accounting: New user created: John Doe



Test Case 2: Send and Receive Email Event
1. Start the receiver and bind to user.created.email.*:
bash
Copy code


java org.example.ReceiveUserEvents user.created.email.*


1. Send a message with the routing key user.created.email:
bash
Copy code


java org.example.EmitUserCreatedEvent user.created.email "New user created: John Doe"



Expected Output on the receiver side:
plaintext
Copy code


[*] Waiting for user created events. To exit press CTRL+C
[x] Received 'user.created.email.*':'New user created: John Doe'
[Email System] Sending welcome email to: New user created: John Doe



Conclusion
This project simulates a topic-based messaging system using RabbitMQ for handling events in an e-commerce system. With EmitUserCreatedEvent as the producer, we send events about new users being created, and with ReceiveUserEvents, we have multiple consumers (accounting and email systems) that process these events based on their routing keys.
You can modify and extend this project by adding more subscribers, events, and routing keys as needed.
