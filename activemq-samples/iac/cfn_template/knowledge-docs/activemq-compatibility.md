# ActiveMQ Compatibility Guide for Amazon MQ

This document provides detailed compatibility information for migrating from Apache ActiveMQ to Amazon MQ for ActiveMQ.

## Supported Versions

Amazon MQ for ActiveMQ currently supports the following versions:
- ActiveMQ 5.15.x
- ActiveMQ 5.16.x
- ActiveMQ 5.17.x

## Protocol Support

Amazon MQ for ActiveMQ supports the following protocols:

| Protocol | Support | Port (TLS) | Notes |
|----------|---------|------------|-------|
| OpenWire | Full | 61617 | Primary protocol for Java clients |
| AMQP | Full | 5671 | Good for cross-platform messaging |
| STOMP | Full | 61614 | Simple text-based protocol |
| MQTT | Full | 8883 | Lightweight IoT protocol |
| WSS (WebSocket) | Full | 61619 | Web browser support |

## Feature Compatibility

### Fully Compatible Features

- JMS 1.1 and 2.0 API support
- Queues and Topics
- Virtual Destinations
- Message Groups
- Message Selectors
- Delayed and Scheduled Message Delivery
- Dead Letter Queues
- Composite Destinations
- Mirrored Queues
- Exclusive Consumers
- Priority Consumers
- Retroactive Consumers
- Last Image Subscriptions
- Individual Acknowledge Mode
- Persistent and Non-persistent Messages
- Durable and Non-durable Subscriptions
- Shared Subscriptions (JMS 2.0)
- Message Expiration
- Message Priority
- Message Redelivery Policy
- Broker Statistics
- Advisory Messages

### Partially Compatible Features

| Feature | Compatibility | Notes |
|---------|---------------|-------|
| Network of Brokers | Partial | Supported with limitations on topology |
| Master/Slave | Partial | Replaced by Amazon MQ active/standby deployment |
| Broker Plugins | Partial | Only specific plugins are supported |
| Authentication | Partial | Simple authentication supported, custom plugins not supported |
| Authorization | Partial | Simple authorization supported, custom plugins not supported |

### Unsupported Features

- Custom Broker Plugins
- Embedded Broker
- KahaDB Modifications
- File System Access
- Custom Protocol Handlers
- Custom Transport Connectors
- Custom Network Bridges
- Custom Security Providers
- JMX Remote Management
- Custom Logging Configuration

## Configuration Differences

### Broker Configuration

Amazon MQ uses a configuration system similar to the `activemq.xml` file in Apache ActiveMQ, with some differences:

1. **XML Structure**: The basic structure is the same, but some elements may have different attributes or nested elements
2. **Persistence Adapter**: Amazon MQ uses KahaDB but with pre-configured settings
3. **Transport Connectors**: Pre-configured and cannot be modified extensively
4. **System Usage**: Memory and storage limits are tied to the instance type
5. **Security Settings**: Authentication and authorization follow a specific format

### Example Configuration Mapping

**Apache ActiveMQ:**
```xml
<broker xmlns="http://activemq.apache.org/schema/core" brokerName="localhost">
  <persistenceAdapter>
    <kahaDB directory="${activemq.data}/kahadb"/>
  </persistenceAdapter>
  <transportConnectors>
    <transportConnector name="openwire" uri="tcp://0.0.0.0:61616"/>
    <transportConnector name="amqp" uri="amqp://0.0.0.0:5672"/>
  </transportConnectors>
</broker>
```

**Amazon MQ for ActiveMQ:**
```xml
<broker xmlns="http://activemq.apache.org/schema/core" brokerName="MyBroker">
  <!-- persistenceAdapter is managed by Amazon MQ -->
  <!-- transportConnectors are managed by Amazon MQ -->
  <!-- Focus on application-specific configuration -->
  <destinationPolicy>
    <policyMap>
      <policyEntries>
        <policyEntry topic=">" producerFlowControl="true">
          <pendingMessageLimitStrategy>
            <constantPendingMessageLimitStrategy limit="1000"/>
          </pendingMessageLimitStrategy>
        </policyEntry>
      </policyEntries>
    </policyMap>
  </destinationPolicy>
</broker>
```

## Client Compatibility

### Java Clients

Java clients using the ActiveMQ client library or JMS API are fully compatible with Amazon MQ for ActiveMQ. Update the connection URL to point to your Amazon MQ broker:

```java
// Before: Apache ActiveMQ
ConnectionFactory factory = new ActiveMQConnectionFactory("tcp://localhost:61616");

// After: Amazon MQ for ActiveMQ
ConnectionFactory factory = new ActiveMQConnectionFactory("ssl://b-1234a5b6-78cd-901e-2fgh-3i45j6k178l9-1.mq.us-east-1.amazonaws.com:61617");
```

### Non-Java Clients

Non-Java clients using AMQP, STOMP, or MQTT protocols are compatible with Amazon MQ for ActiveMQ. Update the connection URL and ensure TLS is configured properly.

## Security Considerations

### Authentication

Amazon MQ for ActiveMQ supports simple username/password authentication. LDAP or custom authentication providers are not supported.

### Authorization

Amazon MQ for ActiveMQ supports role-based authorization similar to Apache ActiveMQ. Configure authorization in the broker configuration:

```xml
<authorizationPlugin>
  <map>
    <authorizationMap>
      <authorizationEntries>
        <authorizationEntry queue=">" read="admins,users" write="admins,producers" admin="admins"/>
        <authorizationEntry topic=">" read="admins,users" write="admins,producers" admin="admins"/>
      </authorizationEntries>
    </authorizationMap>
  </map>
</authorizationPlugin>
```

### Encryption

Amazon MQ for ActiveMQ requires TLS for all connections. Ensure your clients are configured to use TLS when connecting to Amazon MQ.

## Performance Considerations

### Connection Pooling

Use connection pooling to improve performance and reduce resource usage:

```java
// Create a pooled connection factory
PooledConnectionFactory pooledFactory = new PooledConnectionFactory();
pooledFactory.setConnectionFactory(factory);
pooledFactory.setMaxConnections(10);
```

### Prefetch Limits

Configure appropriate prefetch limits based on your consumer patterns:

```java
ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(brokerUrl);
factory.getPrefetchPolicy().setQueuePrefetch(100);
factory.getPrefetchPolicy().setTopicPrefetch(1000);
```

### Producer Flow Control

Configure producer flow control to prevent overwhelming the broker:

```xml
<policyEntry queue=">" producerFlowControl="true" memoryLimit="1mb">
  <pendingQueuePolicy>
    <vmQueueCursor/>
  </pendingQueuePolicy>
</policyEntry>
```

## Migration Checklist

1. **Inventory**:
   - List all queues and topics
   - Document client applications and their connection methods
   - Identify message patterns and volumes

2. **Compatibility Check**:
   - Verify protocol compatibility
   - Check feature usage against supported features
   - Review custom configurations

3. **Configuration Migration**:
   - Create equivalent broker configuration
   - Set up security settings
   - Configure destination policies

4. **Client Updates**:
   - Update connection URLs
   - Configure TLS
   - Implement connection pooling
   - Update credentials handling

5. **Testing**:
   - Verify connectivity
   - Test message production and consumption
   - Validate message persistence
   - Test failover (if using active/standby)

6. **Monitoring Setup**:
   - Configure CloudWatch alarms
   - Set up logging
   - Create dashboards for key metrics

## Conclusion

Migrating from Apache ActiveMQ to Amazon MQ for ActiveMQ is generally straightforward due to the high compatibility between the two. By understanding the differences and limitations, you can plan and execute a successful migration with minimal changes to your applications.