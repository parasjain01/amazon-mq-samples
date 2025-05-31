# RabbitMQ Compatibility Guide for Amazon MQ

This document provides detailed compatibility information for migrating from self-managed RabbitMQ to Amazon MQ for RabbitMQ.

## Supported Versions

Amazon MQ for RabbitMQ currently supports:
- RabbitMQ 3.8.x
- RabbitMQ 3.9.x
- RabbitMQ 3.10.x

## Protocol Support

Amazon MQ for RabbitMQ supports the following protocols:

| Protocol | Support | Port (TLS) | Notes |
|----------|---------|------------|-------|
| AMQP 0-9-1 | Full | 5671 | Primary protocol |
| AMQP 1.0 | Full | 5671 | Via plugin |
| MQTT | Full | 8883 | Via plugin |
| STOMP | Full | 61614 | Via plugin |

## Feature Compatibility

### Fully Compatible Features

- Exchanges (direct, fanout, topic, headers)
- Queues (standard, quorum)
- Bindings
- Message TTL
- Queue TTL
- Queue Length Limits
- Dead Letter Exchanges
- Alternate Exchanges
- Priority Queues
- Consumer Prefetch
- Publisher Confirms
- Consumer Acknowledgements
- Exclusive Queues
- Auto-delete Queues
- Durable Queues and Exchanges
- Message Persistence
- Mandatory Messages
- Immediate Messages
- Basic.Return
- Basic.Reject
- Basic.Nack
- Federation Plugin
- Shovel Plugin
- Consistent Hash Exchange
- Exchange to Exchange Bindings

### Partially Compatible Features

| Feature | Compatibility | Notes |
|---------|---------------|-------|
| Management Plugin | Partial | Web UI available, some operations restricted |
| Clustering | Partial | Managed by AWS, limited configuration options |
| Policies | Partial | Most policy features supported, some limitations |
| User Management | Partial | Through AWS console/API, not RabbitMQ API |
| Virtual Hosts | Partial | Can create and manage, with some limitations |

### Unsupported Features

- Custom Plugins (beyond those pre-installed)
- Direct access to the RabbitMQ nodes
- File-based configuration
- Custom Erlang configuration
- LDAP Authentication
- OAuth 2.0 Authentication
- Direct access to Mnesia database
- Custom log configuration
- Tracing via Firehose
- Core OS access
- Direct metrics collection (use CloudWatch instead)

## Configuration Differences

### Broker Configuration

Amazon MQ for RabbitMQ is configured primarily through the AWS Management Console, AWS CLI, or AWS SDKs, rather than through configuration files:

1. **Cluster Setup**: Managed by AWS, you choose single-instance or cluster deployment
2. **Plugin Management**: Pre-installed plugins only, cannot add custom plugins
3. **Resource Limits**: Tied to the instance type
4. **Security**: Managed through AWS security mechanisms

### Policies and Parameters

Policies and parameters can be configured through the RabbitMQ Management UI or API:

**Self-managed RabbitMQ:**
```bash
rabbitmqctl set_policy ha-all ".*" '{"ha-mode":"all"}' --apply-to queues
```

**Amazon MQ for RabbitMQ:**
Use the RabbitMQ Management UI provided by Amazon MQ, or use the RabbitMQ HTTP API with appropriate authentication.

## Client Compatibility

### Java Clients

Java clients using the RabbitMQ Java client library are fully compatible with Amazon MQ for RabbitMQ. Update the connection parameters to point to your Amazon MQ broker:

```java
// Before: Self-managed RabbitMQ
ConnectionFactory factory = new ConnectionFactory();
factory.setHost("localhost");
factory.setPort(5672);
factory.setUsername("guest");
factory.setPassword("guest");

// After: Amazon MQ for RabbitMQ
ConnectionFactory factory = new ConnectionFactory();
factory.setHost("b-1234a5b6-78cd-901e-2fgh-3i45j6k178l9.mq.us-east-1.amazonaws.com");
factory.setPort(5671);
factory.setUsername("myuser");
factory.setPassword("mypassword");
factory.useSslProtocol();
```

### Other Language Clients

Clients in other languages (Python, Node.js, Go, etc.) are compatible with Amazon MQ for RabbitMQ. Ensure TLS is configured properly and update connection parameters.

## Security Considerations

### Authentication

Amazon MQ for RabbitMQ supports username/password authentication. Configure users through the AWS Management Console or AWS CLI.

### Authorization

RabbitMQ's built-in authorization mechanisms are supported. Configure user permissions through the AWS Management Console, AWS CLI, or RabbitMQ Management UI.

### Encryption

Amazon MQ for RabbitMQ requires TLS for all connections. Ensure your clients are configured to use TLS when connecting to Amazon MQ.

## Performance Considerations

### Connection Management

- Use connection pooling where possible
- Reuse channels instead of creating new ones for each operation
- Implement proper connection recovery mechanisms

```java
// Example connection recovery in Java client
factory.setAutomaticRecoveryEnabled(true);
factory.setNetworkRecoveryInterval(10000); // 10 seconds
```

### Publisher Confirms

Use publisher confirms to ensure messages are safely received by the broker:

```java
Channel channel = connection.createChannel();
channel.confirmSelect();
channel.addConfirmListener(new ConfirmListener() {
    public void handleAck(long deliveryTag, boolean multiple) {
        // Handle successful message confirmation
    }
    public void handleNack(long deliveryTag, boolean multiple) {
        // Handle failed message confirmation
    }
});
```

### Consumer Prefetch

Configure appropriate prefetch values based on your consumer patterns:

```java
// Set prefetch count to 100 messages
channel.basicQos(100);
```

## Federation and Shovel

Amazon MQ for RabbitMQ supports the Federation and Shovel plugins for connecting brokers:

### Federation Example

```java
// Configure federation through the Management UI or API
// Example HTTP API call:
// PUT /api/parameters/federation-upstream/%2f/my-upstream
// {"value":{"uri":"amqps://username:password@source-broker.example.com:5671","expires":3600000}}
```

### Shovel Example

```java
// Configure shovel through the Management UI or API
// Example HTTP API call:
// PUT /api/parameters/shovel/%2f/my-shovel
// {"value":{"src-uri":"amqp://","src-queue":"source","dest-uri":"amqps://username:password@dest-broker.example.com:5671","dest-queue":"destination"}}
```

## Migration Checklist

1. **Inventory**:
   - List all exchanges and queues
   - Document bindings and routing patterns
   - Identify client applications and their connection methods
   - Document plugins in use

2. **Compatibility Check**:
   - Verify protocol compatibility
   - Check plugin compatibility
   - Review custom configurations

3. **Configuration Migration**:
   - Create broker in Amazon MQ
   - Set up users and permissions
   - Configure policies
   - Set up federation or shovel if needed

4. **Client Updates**:
   - Update connection URLs
   - Configure TLS
   - Update credentials handling
   - Implement connection recovery

5. **Testing**:
   - Verify connectivity
   - Test message production and consumption
   - Validate message routing
   - Test cluster behavior (if using cluster deployment)

6. **Monitoring Setup**:
   - Configure CloudWatch alarms
   - Set up logging
   - Create dashboards for key metrics

## Conclusion

Migrating from self-managed RabbitMQ to Amazon MQ for RabbitMQ is generally straightforward for standard deployments. The main considerations are around plugin compatibility, configuration management, and ensuring clients are properly configured for TLS. By understanding the differences and limitations, you can plan and execute a successful migration with minimal changes to your applications.