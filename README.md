# E-Commerce Application Kubernetes Deployment

## Project Overview

This project aims to deploy an e-commerce application using Kubernetes. The application consists of multiple components including a frontend web application, backend API services, a MongoDB database for data persistence, and RabbitMQ for handling asynchronous message processing. Kubernetes manifests are provided to facilitate the deployment and management of these components in a Kubernetes cluster.

## Repository Structure

The repository is structured as follows:

- **backend**: Contains YAML files for deploying the backend microservices.
- **database**: Includes YAML files for deploying the MongoDB database.
- **frontend**: Consists of YAML files for deploying the frontend React application.
- **message-queue**: Contains YAML files for deploying RabbitMQ for message processing.

Each directory contains Kubernetes manifests specific to the corresponding component of the e-commerce application.

## Deployment Instructions

To deploy the e-commerce application, follow the deployment order below:

1. MongoDB
2. RabbitMQ
3. Backend
4. Frontend

### MongoDB

#### Assumptions

- Listens on port 27017.
- Implements a `/healthz` endpoint that returns a 200 OK status when the database is reachable.
- Health and readiness probes are configured to check the MongoDB server's availability and responsiveness.
- The MongoDB image contains an entrypoint script responsible for creating a database and user with appropriate configurations from environment variables supplied via Secrets

#### Deployment Architecture

We have chosen to deploy MongoDB using a StatefulSet for its ability to maintain stable, persistent storage and ordered, graceful deployment and scaling.

- **PersistentVolumes**: PersistentVolumes are used to provide persistent storage for MongoDB data. The StatefulSet's volumeClaimTemplates define the PersistentVolumeClaim specifications for dynamically provisioning persistent storage.

- **ConfigMap**: The `mongodb-config` ConfigMap stores the MongoDB configuration file (`mongod.conf`) which can be mounted into MongoDB containers to provide custom configurations.

- **Secret**: The `mongodb-secret` Secret contains sensitive information such as database name, username, and password required for MongoDB authentication and connection.

- **NetworkPolicy**: The `mongodb-network-policy` NetworkPolicy restricts incoming traffic to MongoDB pods, allowing access only from backend pods. This enhances security by limiting access to MongoDB to only authorized services within the cluster.

- **HorizontalPodAutoscaler (HPA)**: The `mongodb-hpa` HorizontalPodAutoscaler is configured to automatically scale MongoDB pods based on CPU utilization, ensuring optimal performance and resource utilization.

#### Deployment Command

```bash
kubectl apply -f database/
```

### RabbitMQ

#### Assumptions

- Listens on port 5672.
- Implements a `/healthz` endpoint that returns a 200 OK status when the message queue is operational.
- Health and readiness probes are configured to check the RabbitMQ server's availability and responsiveness.
- PersistentVolumes are configured to provide data persistence and resilience for RabbitMQ.
- The RabbitMQ image contains an entrypoint script responsible for configuring RabbitMQ with appropriate settings from environment variables supplied via Secrets.

#### Deployment Architecture

RabbitMQ is deployed using the following components and configurations:

- **StatefulSet**: RabbitMQ is deployed as a StatefulSet to ensure stable and predictable network identifiers, persistent storage, and ordered deployment and scaling.
- **PersistentVolumes**: PersistentVolumes are used to provide data persistence and resilience for RabbitMQ. This ensures that data is retained even if a RabbitMQ pod fails or is rescheduled to a different node.

- **ConfigMap**: A ConfigMap is utilized to load all required RabbitMQ configurations. This allows for easy management and modification of RabbitMQ settings without modifying the application code.

- **Secret**: Secrets are used to store sensitive information such as user credentials. In this case, the Secret contains the user credentials required for RabbitMQ authentication.

- **NetworkPolicy**: The NetworkPolicy allows only incoming traffic from backend pods, ensuring that RabbitMQ is accessible only to authorized services within the Kubernetes cluster.

- **HorizontalPodAutoscaler**: The HorizontalPodAutoscaler automatically adjusts the number of RabbitMQ replicas based on CPU utilization, ensuring optimal performance and resource utilization.

#### Deployment Command

```bash
kubectl apply -f message-queue/
```

### Backend

#### Assumptions

- Listens on port 3000.
- Implements a /healthz endpoint that returns a 200 OK status when the service is healthy and ready to serve traffic.
- Health and readiness probes are configured to check the /healthz endpoint to determine the service's health and readiness status.
- Prometheus might scrape metrics from the backend service on port 9091.

#### Deployment Architecture

The backend service is deployed using the following components and configurations:

- **Deployment**: The backend service is deployed as a Deployment, allowing for easy scaling, rolling updates, and management of the application's replicas.

- **HorizontalPodAutoscaler**: The HorizontalPodAutoscaler automatically adjusts the number of backend replicas based on CPU utilization, ensuring optimal performance and resource utilization.

- **NetworkPolicy**: The NetworkPolicy defines rules for ingress and egress traffic to and from the backend pods. In this case, egress traffic is allowed to MongoDB and RabbitMQ pods, enabling communication between the backend service and the database and message queue.

- **Secret**: Secrets are used to securely store sensitive information such as database credentials and connection endpoints. The backend service accesses MongoDB and RabbitMQ credentials stored in the Secret to establish connections with the respective services.

#### Deployment Command

```bash
kubectl apply -f backend/
```

### Frontend

#### Assumptions

- Listens on port 443.
- Health and readiness probes are configured to check the /healthz endpoint to determine the application's health and readiness status.
- Prometheus might scrape metrics from the frontend service on port 9090.

#### Deployment Architecture

The frontend service is deployed using the following components and configurations:

- **Deployment**: The frontend service is deployed as a Deployment, which allows for easy scaling, rolling updates, and management of the application's replicas. The deployment strategy is set to RollingUpdate, ensuring minimal downtime during updates.

- **HorizontalPodAutoscaler**: The HorizontalPodAutoscaler automatically adjusts the number of frontend replicas based on CPU utilization, ensuring optimal performance and resource utilization.

- **Ingress**: Ingress is configured to route incoming traffic to the frontend service. It defines rules for accessing the frontend application from the outside world, including host and path configurations.

- **NetworkPolicy**: The NetworkPolicy defines rules for ingress and egress traffic to and from the frontend pods. It ensures that only authorized traffic is allowed to access the frontend service and restricts egress traffic to specific ports.

#### Deployment Command

```bash
kubectl apply -f frontend/
```

## Trobleshotting Guide

### Frontend Accessibility

**Issue**: The frontend service is not accessible externally post-deployment.

**Steps to Troubleshoot**:

1. **Check Ingress Configuration**:

   - Verify the Ingress resource configuration, including host and path settings.
   - Ensure the correct host and path are defined in the Ingress rules.
   - Check if the Ingress controller is running and properly configured.

2. **Verify Network Policies**:

   - Ensure that the NetworkPolicy allows incoming traffic to the frontend pods on port 80.
   - Check for any misconfigured network policies that might block external access to the frontend service.

3. **Inspect Service Configuration**:

   - Verify that the frontend service is correctly exposed on correct port.
   - Check the selector labels to ensure the service is targeting the correct pods.

4. **Review Pod Logs**:

   - Check the logs of the frontend pods for any errors or issues related to the application startup.
   - Look for any connection errors or failures that might indicate networking issues.

5. **Check External Connectivity**:

   - Ensure that there are no firewall rules or network restrictions blocking incoming traffic to the frontend service.
   - Test external connectivity by accessing the frontend URL using tools like cURL or web browsers.

6. **Validate DNS Resolution**:

   - Verify that the DNS records for the frontend domain are correctly configured and resolving to the correct IP address.

7. **Monitor Ingress Controller Logs**:

   - Check the logs of the Ingress controller for any errors or warnings related to routing or connectivity issues.
   - Look for any indications of misconfigurations or failed requests.

8. **Test Ingress Connectivity**:
   - Use `kubectl port-forward` to forward traffic directly to the frontend pods and bypass the Ingress controller.
   - This can help isolate whether the issue lies with the Ingress configuration or the frontend service itself.

### Intermittent Backend-Database Connectivity

**Issue**: The backend services occasionally lose connection to the MongoDB cluster, causing request failures.

**Steps to Troubleshoot**:

1. **Check Network Policies**:

   - Review the network policies applied to both the backend and MongoDB pods.
   - Ensure that the network policies allow incoming traffic from backend pods to MongoDB pods on the appropriate port (default 27017).
   - Look for any rules that might inadvertently block or restrict communication between the backend and MongoDB.

2. **Inspect Service Discovery**:

   - Verify that the backend services are correctly configured to discover and connect to the MongoDB cluster.
   - Check the MongoDB connection string or endpoint used by the backend services to ensure it points to the correct MongoDB instances.
   - Validate DNS resolution for the MongoDB service endpoint to ensure it resolves to the correct IP address.

3. **Review MongoDB Configuration**:

   - Check the MongoDB configuration for any settings related to connection pooling, timeouts, or maximum connections.
   - Ensure that the MongoDB server is configured to handle the expected number of incoming connections from the backend services.
   - Monitor MongoDB logs for any errors or warnings related to connection issues or resource constraints.

4. **Monitor Backend Logs**:

   - Examine the logs of the backend services for any error messages or warnings indicating connectivity issues with the MongoDB cluster.
   - Look for connection timeout errors, connection refused errors, or other indications of failed database connections.
   - Identify patterns or specific circumstances under which the connectivity issues occur, such as high load periods or specific database operations.

5. **Check Resource Utilization**:

   - Monitor resource utilization on both the backend and MongoDB pods, including CPU, memory, and network metrics.
   - Identify any resource constraints or performance bottlenecks that might impact database connectivity.
   - Consider scaling MongoDB or backend pods if resource constraints are detected during peak usage periods.

6. **Test Connection Stability**:
   - Perform connection stability tests between the backend pods and MongoDB instances using tools like `telnet` or `nc` (netcat) to verify network connectivity.
   - Monitor connection establishment times and latency to identify potential network issues or bottlenecks.

### Troubleshooting Guide

#### 3. Order Processing Delays

- **Issue**: Users report delays in order processing, suspecting issues with the RabbitMQ message queue.
- **Task**: Analyze and optimize the message queue setup, ensuring efficient message processing and minimal latency.

**Steps to Troubleshoot**:

1. **Monitor RabbitMQ Metrics**:

   - Use RabbitMQ management UI or command-line tools to monitor queue depths, message rates, and consumer activity.
   - Identify queues with high message backlogs or unusually long processing times.
   - Look for patterns or spikes in message traffic that might coincide with order processing delays.

2. **Review RabbitMQ Configuration**:

   - Check RabbitMQ configuration settings such as queue limits, message TTL (time-to-live), and exchange configurations.
   - Ensure that queues are appropriately sized and configured to handle expected message volumes without accumulating excessive backlog.
   - Review queue policies, dead-letter exchanges, and other settings that might impact message routing and delivery.

3. **Check Network and Resource Utilization**:

   - Monitor network traffic and resource utilization on RabbitMQ nodes to ensure they are not overloaded or experiencing network congestion.
   - Review CPU, memory, and disk usage to identify resource constraints or performance bottlenecks.
   - Consider scaling RabbitMQ nodes or optimizing resource allocation based on observed utilization patterns.

4. **Monitor System Health and Performance**:
   - Implement comprehensive monitoring and alerting for RabbitMQ infrastructure and message processing pipelines.
   - Set up alerts for queue backlogs, consumer lag, and other key metrics indicating potential performance issues.
