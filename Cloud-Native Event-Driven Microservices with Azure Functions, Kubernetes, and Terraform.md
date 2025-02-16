### **Project Overview**

This project simulates a **real-time order processing system** with microservices that handle **orders, payments, and notifications**. It will be **event-driven**, meaning that services will communicate asynchronously using **Azure Event Grid**. The infrastructure will be provisioned using **Terraform**, and everything will be deployed on **Azure Kubernetes Service (AKS)**.

---

# **Step 1: Set Up Azure Environment**

### **1.1. Install Required Tools**

Ensure you have the following installed:

* **Azure CLI** → `az login`  
* **Terraform** → Install here  
* **Kubectl** → `az aks install-cli`  
* **Docker** → For containerizing microservices

### **1.2. Create an Azure Resource Group**

Run:

`az group create --name event-driven-rg --location eastus`

---

# **Step 2: Provision Infrastructure Using Terraform**

We will use Terraform to deploy:

* **Azure Kubernetes Service (AKS)**  
* **Azure Event Grid**  
* **Azure Cosmos DB**  
* **Storage Account**

### **2.1. Create a `main.tf` Terraform File**

Create a folder `azure-microservices` and inside it, create `main.tf`:

`provider "azurerm" {`  
  `features {}`  
`}`

`resource "azurerm_resource_group" "rg" {`  
  `name     = "event-driven-rg"`  
  `location = "East US"`  
`}`

`resource "azurerm_kubernetes_cluster" "aks" {`  
  `name                = "event-driven-aks"`  
  `location            = azurerm_resource_group.rg.location`  
  `resource_group_name = azurerm_resource_group.rg.name`  
  `dns_prefix          = "eventdrivenaks"`  
  `default_node_pool {`  
    `name       = "default"`  
    `node_count = 2`  
    `vm_size    = "Standard_B2s"`  
  `}`  
  `identity {`  
    `type = "SystemAssigned"`  
  `}`  
`}`

`resource "azurerm_cosmosdb_account" "cosmos" {`  
  `name                = "event-driven-db"`  
  `location            = azurerm_resource_group.rg.location`  
  `resource_group_name = azurerm_resource_group.rg.name`  
  `offer_type          = "Standard"`  
  `kind                = "MongoDB"`  
`}`

`resource "azurerm_eventgrid_topic" "eventgrid" {`  
  `name                = "order-events"`  
  `resource_group_name = azurerm_resource_group.rg.name`  
  `location            = azurerm_resource_group.rg.location`  
`}`

### **2.2. Deploy Terraform Infrastructure**

`terraform init`  
`terraform apply -auto-approve`

This will provision the **AKS cluster, CosmosDB, and Event Grid**.

---

# **Step 3: Develop Microservices (Order, Payment, Notification)**

We will create three **Flask microservices**, containerize them with **Docker**, and deploy them on AKS.

### **3.1. Order Service**

This service creates orders and sends an event to Event Grid.

Create a file `order_service.py`:

python

`from flask import Flask, request, jsonify`  
`import requests`

`app = Flask(__name__)`

`EVENT_GRID_URL = "https://order-events.eastus-1.eventgrid.azure.net/api/events"`  
`EVENT_GRID_KEY = "your-eventgrid-key"`

`@app.route('/order', methods=['POST'])`  
`def create_order():`  
    `order = request.json`  
    `event = [{`  
        `"id": "1",`  
        `"subject": "New Order",`  
        `"eventType": "OrderCreated",`  
        `"eventTime": "2024-02-17T00:00:00Z",`  
        `"data": order,`  
        `"dataVersion": "1.0"`  
    `}]`  
    `headers = {"aeg-sas-key": EVENT_GRID_KEY, "Content-Type": "application/json"}`  
    `requests.post(EVENT_GRID_URL, json=event, headers=headers)`  
    `return jsonify({"message": "Order Created!"})`

`if __name__ == '__main__':`  
    `app.run(host='0.0.0.0', port=5001)`

### **3.2. Dockerize Order Service**

Create a `Dockerfile`:

dockerfile

`FROM python:3.9`  
`WORKDIR /app`  
`COPY . .`  
`RUN pip install flask requests`  
`CMD ["python", "order_service.py"]`

### **3.3. Build and Push Docker Image**

`docker build -t mydockerhub/order-service .`  
`docker push mydockerhub/order-service`

---

# **Step 4: Deploy Microservices on AKS**

Now, let’s deploy the **Order Service** to AKS.

### **4.1. Create Kubernetes Deployment**

Create `order-deployment.yaml`:

yaml

`apiVersion: apps/v1`  
`kind: Deployment`  
`metadata:`  
  `name: order-service`  
`spec:`  
  `replicas: 2`  
  `selector:`  
    `matchLabels:`  
      `app: order-service`  
  `template:`  
    `metadata:`  
      `labels:`  
        `app: order-service`  
    `spec:`  
      `containers:`  
      `- name: order-service`  
        `image: mydockerhub/order-service`  
        `ports:`  
        `- containerPort: 5001`  
`---`  
`apiVersion: v1`  
`kind: Service`  
`metadata:`  
  `name: order-service`  
`spec:`  
  `selector:`  
    `app: order-service`  
  `ports:`  
    `- protocol: TCP`  
      `port: 80`  
      `targetPort: 5001`  
  `type: LoadBalancer`

### **4.2. Deploy to AKS**

`kubectl apply -f order-deployment.yaml`

---

# **Step 5: Connect Payment and Notification Services**

* **Payment Service**: Listens for **OrderCreated** event and processes payment.  
* **Notification Service**: Sends notifications to users.

These will be implemented as **Azure Functions**.

### **5.1. Create Payment Function**

python

`import json`  
`import azure.functions as func`

`def main(event: func.EventGridEvent):`  
    `order = event.get_json()`  
    `print(f"Processing payment for Order ID: {order['id']}")`

Deploy it with:

`func azure functionapp publish payment-function`

### **5.2. Create Notification Function**

python

`import json`  
`import azure.functions as func`

`def main(event: func.EventGridEvent):`  
    `order = event.get_json()`  
    `print(f"Sending notification for Order ID: {order['id']}")`

Deploy it with:

`func azure functionapp publish notification-function`

---

# **Step 6: Monitor & Scale System**

### **6.1. Enable Azure Monitor**

`az monitor log-analytics workspace create --resource-group event-driven-rg --workspace-name event-monitor`

### **6.2. Enable Autoscaling in AKS**

bash  
CopyEdit  
`kubectl autoscale deployment order-service --cpu-percent=50 --min=1 --max=5`

---

## **Final Results**

* **Order Service** creates an order → sends event to Event Grid  
* **Payment Service** listens to Event Grid → processes payment  
* **Notification Service** listens to Event Grid → sends notifications  
* All services run on **Azure Kubernetes Service (AKS)** and scale automatically

