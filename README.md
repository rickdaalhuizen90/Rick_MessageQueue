# Magento 2 RabbitMQ Example

This module showcases asynchronous event handling in Magento 2 with RabbitMQ, providing a practical example to improve store performance and handle increased traffic.

## Key Concepts

- **Producer:** Sends messages to the waiting line (RabbitMQ queue)
- **Queue:** The waiting line itself, holding messages until they're processed.
- **Topic:** Topics are used to group related messages together based on a common theme or subject.
- **Consumer:** Listens to the queue, grabs messages, and takes action. (We have two in this example!)

## Real-world Uses:

- **Order Processing:** Don't freeze your store while handling orders. The queue takes care of it in the background.
- **Product Updates:** Update product info (like stock levels) without slowing down shoppers.
- **Emails:** Send emails (like order confirmations) in batches, improving performance.
- **Indexing:** Prioritize important tasks and optimize resource usage by using the queue.

## Prerequisites

- A working Magento 2 store.
- RabbitMQ server installed and configured (refer to [official documentation](https://experienceleague.adobe.com/en/docs/commerce-operations/installation-guide/prerequisites/rabbitmq) for details)

## Installation

1. Enable the module:
    ```bash
    bin/magento module:enable Rick_RabbitMQExample
    ```

2. Upgrade Magento:
    ```bash
    bin/magento setup:upgrade
    ```

## Configuration

### 1. RabbitMQ Configuration (app/etc/env.php):

Update the queue section with your RabbitMQ details (don't use defaults for real stores!).

```php
'queue' => [
    'consumers_wait_for_messages' => 0, // Processes messages immediately, minimizing latency.
    'only_spawn_when_message_available' => 1, // Spawns consumer processes only when messages are available, optimizing resource utilization.
    'amqp' => [
        'host' => 'rabbitmq', // Assumes RabbitMQ server runs in a "rabbitmq" Docker container.
        'port' => 5672,
        'user' => 'guest',
        'password' => 'guest',
        'virtualhost' => '/',
        'ssl' => false // Change to `true` for secure connections (recommended for production)
    ]
],
```

### 2. Check Queues (after setting up RabbitMQ):

```bash
bin/magento queue:consumers:list
```

### 3. Start Queue Listeners:

Run these commands to start listening for messages:

```bash
bin/magento queue:consumers:start test_consumer_one
```

```bash
bin/magento queue:consumers:start test_consumer_two
```

In production, you'll want to use cron to automatically start queue listeners, which runs every minute (* * * * *).

Add consumers to cron_consumers_runner in app/etc/env.php:
```php
'cron_consumers_runner' => [
    'cron_run' => true,
    'max_messages' => 20000,
    'consumers' => [
        'test_consumer_one',
        'test_consumer_two',
    ],
],
```

## Module Implementation (Custom Logic)

1. **Define Events:** Identify the events in your module that should trigger message queue messages.
2. **Create Message Producer:** Use the `Magento\Framework\RabbitMQExample\PublisherInterface` to publish messages to the RabbitMQ queue.
3. **Implement Message Consumer:** Create a queue consumer class that extends `Vendor\Module\Api\ConsumerInterface` and processes messages received from the queue.
4. **Configure Queue and Consumer:** Register your queue and consumer in the `etc` folder using XML configuration files (`communication.xml`, `queue_topology.xml`, `queue_consumer.xml`, and `queue_publisher.xml`). These files define message routing logic and consumer association with the queue.

For more detaults on implementing custom message queues, refer to the Magento documentation:

- [Magento Message Queue Framework (MQF)](https://developer.adobe.com/commerce/php/development/components/message-queues/)

### Additional Resources

- [Asynchronous configuration](https://developer.adobe.com/commerce/php/development/components/message-queues/async-configuration/)
- [Configure message queues](https://developer.adobe.com/commerce/php/development/components/message-queues/configuration/)
- [Topics in asynchronous API](https://developer.adobe.com/commerce/php/development/components/message-queues/async-topics/)
- [RabbitMQ Documentation](https://www.rabbitmq.com/docs)
- [Asynchronous Magento](https://www.phparch.com/2020/08/asynchronous-magento/)
