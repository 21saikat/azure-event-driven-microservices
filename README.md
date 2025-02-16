# Azure Event-Driven Microservices

This project demonstrates the creation of a **cloud-native, event-driven microservices** architecture using **Azure Functions**, **Azure Kubernetes Service (AKS)**, **Event Grid**, and **Terraform**. The application is a **real-time order processing system** composed of multiple microservices, each handling different tasks such as order creation, payment processing, and user notifications.

## Key Features
- **Event-Driven Architecture**: Uses **Azure Event Grid** to enable asynchronous communication between microservices.
- **Microservices**: 
  - **Order Service**: Creates orders and sends events to Event Grid.
  - **Payment Service**: Processes payments based on the events received.
  - **Notification Service**: Sends notifications when the order is processed.
- **Infrastructure as Code**: Deployed using **Terraform** to automate the provisioning of resources such as AKS, Event Grid, and Cosmos DB.
- **Kubernetes**: Microservices deployed and managed on **Azure Kubernetes Service (AKS)**, ensuring scalability and reliability.
- **Serverless Functions**: **Azure Functions** handle payment processing and notification tasks.
- **Monitoring & Scaling**: Implemented auto-scaling for Kubernetes and integrated **Azure Monitor** for observability.

## Technologies Used
- **Azure Functions** (for serverless event-driven processing)
- **Azure Kubernetes Service (AKS)** (for microservices deployment)
- **Azure Event Grid** (for event-based communication)
- **Terraform** (for Infrastructure as Code)
- **Azure Cosmos DB** (for data storage)
- **Docker** (for containerizing microservices)
- **Helm** (for managing Kubernetes applications)

## Prerequisites
- **Azure CLI** for managing Azure resources
- **Terraform** for provisioning infrastructure
- **Kubectl** for managing Kubernetes
- **Docker** for containerizing services

## How to Run
1. Clone this repository.
2. Use **Terraform** to provision Azure resources.
3. Build and deploy microservices to **AKS**.
4. Test the event-driven flow by creating orders through the **Order Service**.

## License
This project is licensed under the MIT License.
