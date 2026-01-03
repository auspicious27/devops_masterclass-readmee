# 📊 Zomato Microservices - Complete Architecture Diagrams

## 🎯 Overview

This document contains all Mermaid diagrams for the Zomato-style microservices architecture. These diagrams provide visual representations of system architecture, request flows, deployment processes, and troubleshooting procedures.

> **💡 Tip**: These diagrams are interactive in GitHub! Hover over elements to see details.

---

## 📋 Table of Contents

1. [Complete System Architecture](#1-complete-system-architecture)
2. [Request Flow Diagram](#2-request-flow-diagram)
3. [Deployment Flow Diagram](#3-deployment-flow-diagram)
4. [Registry Flow Diagram](#4-registry-flow-diagram)
5. [Service Interaction Diagram](#5-service-interaction-diagram)
6. [Auto-Healing Flow Diagram](#6-auto-healing-flow-diagram)
7. [Network Topology Diagram](#7-network-topology-diagram)
8. [Complete Request Lifecycle](#8-complete-request-lifecycle)
9. [Project Structure Diagram](#9-project-structure-diagram)
10. [Component Relationship Diagram](#10-component-relationship-diagram)
11. [Setup Timeline Diagram](#11-setup-timeline-diagram)
12. [Port Mapping Diagram](#12-port-mapping-diagram)
13. [Test Flow Diagram](#13-test-flow-diagram)
14. [Auto-Healing Process Visualization](#14-auto-healing-process-visualization)
15. [Pod Lifecycle Timeline](#15-pod-lifecycle-timeline)
16. [Auto-Healing Scenarios](#16-auto-healing-scenarios)
17. [Troubleshooting Flow Diagram](#17-troubleshooting-flow-diagram)
18. [Registry Naming Convention](#18-registry-naming-convention)
19. [Cleanup Flow Diagram](#19-cleanup-flow-diagram)
20. [Architecture Summary Mind Map](#20-architecture-summary-mind-map)
21. [Technology Stack Diagram](#21-technology-stack-diagram)

---

## 1. Complete System Architecture

This diagram shows the complete architecture of the Zomato microservices system, including all layers from client to infrastructure.

```mermaid
graph TB
    subgraph "Client Layer"
        Browser[🌐 Browser/Client<br/>localhost:8080]
    end
    
    subgraph "Ingress Layer"
        Ingress[🚦 Traefik Ingress Controller<br/>Port: 8080 → 80]
    end
    
    subgraph "Kubernetes Cluster (k3d)"
        subgraph "User Service"
            US1[👤 User Pod 1<br/>Node.js:3001]
            US2[👤 User Pod 2<br/>Node.js:3001]
            US_SVC[📡 User Service<br/>ClusterIP:3001]
        end
        
        subgraph "Restaurant Service"
            RS1[🍽️ Restaurant Pod 1<br/>Python:3002]
            RS2[🍽️ Restaurant Pod 2<br/>Python:3002]
            RS_SVC[📡 Restaurant Service<br/>ClusterIP:3002]
        end
        
        subgraph "Order Service"
            OS1[🛒 Order Pod 1<br/>Go:3003]
            OS2[🛒 Order Pod 2<br/>Go:3003]
            OS_SVC[📡 Order Service<br/>ClusterIP:3003]
        end
    end
    
    subgraph "Infrastructure"
        Registry[📦 k3d Registry<br/>localhost:5000<br/>k3d-zomato-registry:5000]
        K3D[☸️ k3d Cluster<br/>1 Server + 2 Agents]
    end
    
    Browser -->|GET /users| Ingress
    Browser -->|GET /restaurants| Ingress
    Browser -->|GET /orders| Ingress
    
    Ingress -->|Route /users| US_SVC
    Ingress -->|Route /restaurants| RS_SVC
    Ingress -->|Route /orders| OS_SVC
    
    US_SVC -->|Load Balance| US1
    US_SVC -->|Load Balance| US2
    RS_SVC -->|Load Balance| RS1
    RS_SVC -->|Load Balance| RS2
    OS_SVC -->|Load Balance| OS1
    OS_SVC -->|Load Balance| OS2
    
    K3D --> Registry
    
    style Browser fill:#e1f5ff
    style Ingress fill:#fff4e1
    style US1 fill:#e8f5e9
    style US2 fill:#e8f5e9
    style RS1 fill:#fce4ec
    style RS2 fill:#fce4ec
    style OS1 fill:#f3e5f5
    style OS2 fill:#f3e5f5
    style Registry fill:#fff9c4
    style K3D fill:#e0f2f1
```

### 📝 Explanation

- **Client Layer**: Browser accessing the application via `localhost:8080`
- **Ingress Layer**: Traefik Ingress Controller routes requests based on URL paths
- **Kubernetes Cluster**: Contains all microservices with 2 replicas each for high availability
- **Infrastructure**: k3d cluster and local Docker registry for image storage

---

## 2. Request Flow Diagram

This sequence diagram shows how a request flows through the system from browser to pod and back.

```mermaid
sequenceDiagram
    participant Browser
    participant Ingress as Traefik Ingress
    participant Service as Kubernetes Service
    participant Pod1 as Pod Replica 1
    participant Pod2 as Pod Replica 2
    participant Registry as k3d Registry
    
    Browser->>Ingress: GET http://localhost:8080/users
    Ingress->>Ingress: Route based on path (/users)
    Ingress->>Service: Forward to user-service:3001
    Service->>Service: Load Balance (Round Robin)
    alt Pod 1 Selected
        Service->>Pod1: Forward Request
        Pod1->>Pod1: Process Request
        Pod1-->>Service: Response: ["Rahul","Aisha","Zoya"]
    else Pod 2 Selected
        Service->>Pod2: Forward Request
        Pod2->>Pod2: Process Request
        Pod2-->>Service: Response: ["Rahul","Aisha","Zoya"]
    end
    Service-->>Ingress: Response
    Ingress-->>Browser: JSON Response
    
    Note over Pod1,Pod2: Both pods serve same content<br/>Load balancing ensures high availability
```

### 📝 Explanation

1. Browser sends GET request to Ingress
2. Ingress matches path and routes to appropriate service
3. Service load balances between pod replicas
4. Selected pod processes request and returns response
5. Response flows back through service and ingress to browser

---

## 3. Deployment Flow Diagram

This flowchart shows the complete process of deploying a microservice from code to production.

```mermaid
flowchart TD
    Start([Start: Developer]) --> Code[Write Service Code<br/>Node.js/Python/Go]
    Code --> Dockerfile[Create Dockerfile]
    Dockerfile --> Build[Build Docker Image<br/>docker build]
    Build --> Test{Test Locally?}
    Test -->|Yes| LocalTest[Run Container Locally<br/>docker run]
    LocalTest --> TestResult{Works?}
    TestResult -->|No| Code
    TestResult -->|Yes| Tag[Tag Image for Registry]
    Test -->|No| Tag
    Tag --> Push[Push to Registry<br/>docker push localhost:5000]
    Push --> K8sYAML[Create Kubernetes YAML<br/>Deployment + Service]
    K8sYAML --> Apply[Apply to Cluster<br/>kubectl apply]
    Apply --> K8sPull[Kubernetes Pulls Image<br/>from k3d-zomato-registry:5000]
    K8sPull --> Deploy[Deployment Creates Pods]
    Deploy --> Running{Pods Running?}
    Running -->|No| Debug[Debug: kubectl logs/describe]
    Debug --> Code
    Running -->|Yes| Ingress[Configure Ingress]
    Ingress --> Done([✅ Service Live])
    
    style Start fill:#e1f5ff
    style Done fill:#c8e6c9
    style Test fill:#fff9c4
    style Running fill:#fff9c4
```

### 📝 Explanation

The deployment process includes:
- Code development
- Docker image creation
- Local testing
- Image push to registry
- Kubernetes deployment
- Ingress configuration
- Final verification

---

## 4. Registry Flow Diagram

This diagram illustrates how Docker images flow between host machine, registry, and Kubernetes cluster.

```mermaid
graph LR
    subgraph "Host Machine (RHEL 9)"
        Dev[👨‍💻 Developer]
        DockerHost[Docker Engine]
        Localhost[localhost:5000]
    end
    
    subgraph "k3d Registry Container"
        Registry[k3d-zomato-registry:5000<br/>Docker Registry]
        Storage[(Image Storage)]
    end
    
    subgraph "Kubernetes Cluster"
        K8sNode[Kubernetes Node]
        Pod[Application Pod]
    end
    
    Dev -->|1. docker build| DockerHost
    DockerHost -->|2. docker tag| Localhost
    Localhost -->|3. docker push| Registry
    Registry -->|4. Store| Storage
    
    K8sNode -->|5. Pull Image| Registry
    Registry -->|6. Serve Image| K8sNode
    K8sNode -->|7. Create| Pod
    
    style Dev fill:#e1f5ff
    style Registry fill:#fff9c4
    style Pod fill:#c8e6c9
    style Storage fill:#f3e5f5
```

### 📝 Explanation

**Push Flow (Host → Registry)**:
1. Developer builds image
2. Image tagged for registry
3. Image pushed to registry
4. Registry stores image

**Pull Flow (Registry → Kubernetes)**:
5. Kubernetes node pulls image
6. Registry serves image
7. Pod created with image

---

## 5. Service Interaction Diagram

This diagram shows how services interact with each other and how load balancing works.

```mermaid
graph TB
    subgraph "External Access"
        Client[Client/Browser]
    end
    
    subgraph "Ingress Layer"
        Traefik[Traefik Ingress<br/>Port 8080]
    end
    
    subgraph "Service Layer"
        US[User Service<br/>ClusterIP:3001]
        RS[Restaurant Service<br/>ClusterIP:3002]
        OS[Order Service<br/>ClusterIP:3003]
    end
    
    subgraph "Pod Layer - User Service"
        UP1[User Pod 1<br/>Replica 1]
        UP2[User Pod 2<br/>Replica 2]
    end
    
    subgraph "Pod Layer - Restaurant Service"
        RP1[Restaurant Pod 1<br/>Replica 1]
        RP2[Restaurant Pod 2<br/>Replica 2]
    end
    
    subgraph "Pod Layer - Order Service"
        OP1[Order Pod 1<br/>Replica 1]
        OP2[Order Pod 2<br/>Replica 2]
    end
    
    Client -->|HTTP Request| Traefik
    Traefik -->|/users →| US
    Traefik -->|/restaurants →| RS
    Traefik -->|/orders →| OS
    
    US -.->|Load Balance| UP1
    US -.->|Load Balance| UP2
    RS -.->|Load Balance| RP1
    RS -.->|Load Balance| RP2
    OS -.->|Load Balance| OP1
    OS -.->|Load Balance| OP2
    
    style Client fill:#e1f5ff
    style Traefik fill:#fff4e1
    style US fill:#e8f5e9
    style RS fill:#fce4ec
    style OS fill:#f3e5f5
```

### 📝 Explanation

- **External Access**: Client makes HTTP requests
- **Ingress Layer**: Routes requests based on paths
- **Service Layer**: Kubernetes Services provide load balancing
- **Pod Layer**: Multiple replicas handle requests

---

## 6. Auto-Healing Flow Diagram

This sequence diagram shows how Kubernetes automatically recovers from pod failures.

```mermaid
sequenceDiagram
    participant K8s as Kubernetes Controller
    participant Pod as Pod (Running)
    participant NewPod as New Pod
    participant Service as Service
    
    Note over Pod: Pod Running Normally<br/>Serving Traffic
    
    K8s->>Pod: Delete Pod (Manual/Accident)
    Pod->>Pod: Pod Terminated
    
    K8s->>K8s: Detect Pod Count < Replicas (2)
    K8s->>K8s: Desired State: 2 Pods
    
    K8s->>NewPod: Create New Pod
    NewPod->>NewPod: Pull Image from Registry
    NewPod->>NewPod: Container Starting
    NewPod->>NewPod: Health Check Pass
    NewPod->>NewPod: Pod Ready
    
    K8s->>Service: Update Endpoints
    Service->>Service: Add New Pod to Load Balancer
    
    Note over K8s,Service: ✅ Auto-Healing Complete<br/>Service Uninterrupted
    
    Service->>NewPod: Route Traffic
```

### 📝 Explanation

1. Pod fails or is deleted
2. Kubernetes controller detects mismatch
3. New pod is created automatically
4. Image pulled and container started
5. Health check passes
6. Service updated with new pod
7. Traffic routed to new pod

---

## 7. Network Topology Diagram

This diagram shows the physical and logical network structure of the k3d cluster.

```mermaid
graph TB
    subgraph "RHEL 9 Host"
        subgraph "Docker Network: k3d"
            subgraph "k3d Cluster Network"
                subgraph "Control Plane"
                    CP[k3d-server<br/>Kubernetes Master]
                end
                
                subgraph "Worker Nodes"
                    WN1[k3d-agent-0<br/>Worker Node 1]
                    WN2[k3d-agent-1<br/>Worker Node 2]
                end
                
                subgraph "System Pods"
                    TraefikPod[Traefik Ingress Pod]
                    CoreDNS[CoreDNS Pod]
                    Metrics[Metrics Server Pod]
                end
            end
            
            subgraph "Application Pods"
                UP[User Pods<br/>Port 3001]
                RP[Restaurant Pods<br/>Port 3002]
                OP[Order Pods<br/>Port 3003]
            end
            
            RegistryContainer[k3d-zomato-registry<br/>Port 5000]
        end
        
        HostPort[Host Port Mapping<br/>8080:80]
    end
    
    Internet[Internet/Client] -->|localhost:8080| HostPort
    HostPort --> TraefikPod
    TraefikPod --> UP
    TraefikPod --> RP
    TraefikPod --> OP
    
    CP --> WN1
    CP --> WN2
    WN1 --> UP
    WN1 --> RP
    WN2 --> OP
    
    UP -.->|Pull Images| RegistryContainer
    RP -.->|Pull Images| RegistryContainer
    OP -.->|Pull Images| RegistryContainer
    
    style Internet fill:#e1f5ff
    style HostPort fill:#fff4e1
    style RegistryContainer fill:#fff9c4
    style CP fill:#e0f2f1
    style TraefikPod fill:#fff4e1
```

### 📝 Explanation

- **Control Plane**: Kubernetes master node
- **Worker Nodes**: Two agent nodes for running pods
- **System Pods**: Core Kubernetes components
- **Application Pods**: User services
- **Registry**: Local Docker registry for images

---

## 8. Complete Request Lifecycle

This flowchart shows all states a request goes through from initiation to completion.

```mermaid
flowchart LR
    Start([User opens browser]) --> ClientRequest[Client Request]
    ClientRequest --> IngressRouting[Ingress Routing<br/>GET localhost:8080/users]
    IngressRouting --> PathMatching[Path Matching<br/>Traefik receives request]
    PathMatching --> ServiceDiscovery[Service Discovery<br/>Match /users path]
    ServiceDiscovery --> LoadBalancing[Load Balancing<br/>Route to user-service:3001]
    LoadBalancing --> PodSelection[Pod Selection<br/>Select pod Round Robin]
    PodSelection --> PodProcessing[Pod Processing<br/>Forward to selected pod]
    PodProcessing --> DataRetrieval[Data Retrieval<br/>Process request]
    DataRetrieval --> ResponseGeneration[Response Generation<br/>Generate JSON response]
    ResponseGeneration --> ServiceResponse[Service Response<br/>Return to service]
    ServiceResponse --> IngressResponse[Ingress Response<br/>Return to Ingress]
    IngressResponse --> ClientResponse[Client Response<br/>Display data]
    ClientResponse --> End([Request Complete])
    
    style Start fill:#e1f5ff
    style End fill:#c8e6c9
    style LoadBalancing fill:#fff9c4
```

### 📝 Explanation

Request goes through these steps:
1. **Client Request**: User opens browser
2. **Ingress Routing**: GET request to localhost:8080/users
3. **Path Matching**: Traefik receives and matches path
4. **Service Discovery**: Match /users path to user-service
5. **Load Balancing**: Route to user-service:3001
6. **Pod Selection**: Select pod using Round Robin
7. **Pod Processing**: Forward to selected pod
8. **Data Retrieval**: Process request
9. **Response Generation**: Generate JSON response
10. **Response Delivery**: Return to browser and display

---

## 9. Project Structure Diagram

This diagram shows the file and directory structure of the project.

```mermaid
graph TD
    Root[zomato/] --> UserSvc[user-service/]
    Root --> RestSvc[restaurant-service/]
    Root --> OrderSvc[order-service/]
    Root --> K8s[k8s/]
    
    UserSvc --> USCode[server.js<br/>Node.js Express]
    UserSvc --> USDocker[Dockerfile<br/>node:18-alpine]
    
    RestSvc --> RSCode[app.py<br/>Python Flask]
    RestSvc --> RSDocker[Dockerfile<br/>python:3.11-slim]
    
    OrderSvc --> OSCode[main.go<br/>Go HTTP Server]
    OrderSvc --> OSDocker[Dockerfile<br/>golang:1.21-alpine]
    
    K8s --> UserYAML[user.yaml<br/>Deployment + Service]
    K8s --> RestYAML[restaurant.yaml<br/>Deployment + Service]
    K8s --> OrderYAML[order.yaml<br/>Deployment + Service]
    K8s --> IngressYAML[ingress.yaml<br/>Ingress Rules]
    
    style Root fill:#e1f5ff
    style UserSvc fill:#e8f5e9
    style RestSvc fill:#fce4ec
    style OrderSvc fill:#f3e5f5
    style K8s fill:#fff9c4
```

### 📝 Explanation

- **Root**: Main project directory
- **Service Directories**: Each microservice has its own directory
- **k8s Directory**: Contains all Kubernetes manifests

---

## 10. Component Relationship Diagram

This ER diagram shows the relationships between Kubernetes objects.

```mermaid
erDiagram
    INGRESS ||--o{ SERVICE : routes
    SERVICE ||--|{ POD : manages
    DEPLOYMENT ||--|{ POD : creates
    POD ||--|| CONTAINER : contains
    CONTAINER ||--|| IMAGE : uses
    IMAGE ||--|| REGISTRY : stored_in
    NODE ||--o{ POD : hosts
    CLUSTER ||--|{ NODE : contains
    
    INGRESS {
        string name "zomato-ingress"
        int port_8080 "8080"
        string controller "Traefik"
    }
    
    SERVICE {
        string name "user-service"
        int port_3001 "3001"
        string type "ClusterIP"
    }
    
    POD {
        string name "user-pod"
        string status "Running"
        int replicas_2 "2"
    }
    
    DEPLOYMENT {
        string name "user-service"
        int replicas_2 "2"
        string strategy "RollingUpdate"
    }
    
    CONTAINER {
        string image "user-service:v1"
        int port_3001 "3001"
    }
    
    IMAGE {
        string name "user-service"
        string tag "v1"
        string registry "k3d-zomato-registry:5000"
    }
    
    REGISTRY {
        string name "k3d-zomato-registry"
        int port_5000 "5000"
    }
    
    NODE {
        string name "k3d-agent-0"
        string role "worker"
    }
    
    CLUSTER {
        string name "zomato-cluster"
        int servers_1 "1"
        int agents_2 "2"
    }
```

### 📝 Explanation

Shows relationships:
- Ingress routes to Services
- Services manage Pods
- Deployments create Pods
- Pods contain Containers
- Containers use Images
- Images stored in Registry
- Nodes host Pods
- Cluster contains Nodes

---

## 11. Setup Timeline Diagram

This Gantt chart shows the timeline for setting up the entire system.

```mermaid
gantt
    title Zomato Microservices Setup Timeline
    dateFormat YYYY-MM-DD
    section Infrastructure
    Install Docker           :2024-01-01, 5d
    Install kubectl          :2024-01-06, 2d
    Install k3d             :2024-01-08, 2d
    Create Registry         :2024-01-10, 1d
    Create Cluster          :2024-01-11, 3d
    Install Ingress         :2024-01-14, 2d
    
    section Services
    User Service            :2024-01-16, 10d
    Restaurant Service      :2024-01-26, 10d
    Order Service           :2024-02-05, 10d
    
    section Kubernetes
    Deploy Services         :2024-02-15, 5d
    Configure Ingress       :2024-02-20, 2d
    Testing                 :2024-02-22, 3d
    
    section Verification
    Final Testing           :2024-02-25, 5d
```

### 📝 Explanation

Total setup time: ~60 minutes
- Infrastructure: 15 minutes
- Services: 30 minutes
- Kubernetes: 10 minutes
- Verification: 5 minutes

---

## 12. Port Mapping Diagram

This diagram shows how ports are mapped from external access to internal services.

```mermaid
graph LR
    subgraph "External Access"
        Browser[Browser<br/>localhost:8080]
    end
    
    subgraph "Ingress Controller"
        Ingress[Traefik<br/>Host: 8080<br/>Container: 80]
    end
    
    subgraph "Kubernetes Services"
        US[user-service<br/>Port: 3001]
        RS[restaurant-service<br/>Port: 3002]
        OS[order-service<br/>Port: 3003]
    end
    
    subgraph "Pods"
        UP[User Pod<br/>Container Port: 3001]
        RP[Restaurant Pod<br/>Container Port: 3002]
        OP[Order Pod<br/>Container Port: 3003]
    end
    
    Browser -->|:8080| Ingress
    Ingress -->|:3001| US
    Ingress -->|:3002| RS
    Ingress -->|:3003| OS
    US -->|:3001| UP
    RS -->|:3002| RP
    OS -->|:3003| OP
    
    style Browser fill:#e1f5ff
    style Ingress fill:#fff4e1
    style US fill:#e8f5e9
    style RS fill:#fce4ec
    style OS fill:#f3e5f5
```

### 📝 Explanation

- External: Port 8080
- Ingress: Port 80 internally
- Services: Ports 3001, 3002, 3003
- Pods: Same ports as services

---

## 13. Test Flow Diagram

This sequence diagram shows the testing process for verifying services.

```mermaid
sequenceDiagram
    participant Tester
    participant Browser
    participant Ingress
    participant Service
    participant Pod
    
    Tester->>Browser: Open http://localhost:8080/users
    Browser->>Ingress: HTTP GET /users
    Ingress->>Ingress: Match path /users
    Ingress->>Service: Forward to user-service:3001
    Service->>Pod: Load balance to pod
    Pod->>Pod: Process request
    Pod->>Service: Return JSON response
    Service->>Ingress: Forward response
    Ingress->>Browser: Return JSON
    Browser->>Tester: Display ["Rahul","Aisha","Zoya"]
```

### 📝 Explanation

Testing flow:
1. Tester opens browser
2. Request sent to Ingress
3. Ingress routes to service
4. Service forwards to pod
5. Pod processes and responds
6. Response displayed in browser

---

## 14. Auto-Healing Process Visualization

This flowchart shows the complete auto-healing process.

```mermaid
flowchart TD
    Start([System Running]) --> Normal[Normal Operation]
    Normal --> PodFailure[Pod Failure Detected]
    PodFailure --> ControllerDetect[Controller Detects Mismatch]
    ControllerDetect --> CheckDesired{Current < Desired?}
    CheckDesired -->|Yes| CreatePod[Create New Pod]
    CheckDesired -->|No| Normal
    CreatePod --> PullImage[Pull Image from Registry]
    PullImage --> StartContainer[Start Container]
    StartContainer --> HealthCheck{Health Check}
    HealthCheck -->|Pass| Ready[Pod Ready]
    HealthCheck -->|Fail| Restart[Restart Container]
    Restart --> StartContainer
    Ready --> Normal
    Normal --> End([System Restored])
    
    style Start fill:#e1f5ff
    style End fill:#c8e6c9
    style PodFailure fill:#ffcdd2
    style Ready fill:#c8e6c9
    style HealthCheck fill:#fff9c4
```

### 📝 Explanation

Auto-healing process:
1. **Normal Operation**: System running normally
2. **Pod Failure**: Pod crashes, deleted, or node fails
3. **Controller Detection**: Kubernetes detects replica mismatch
4. **Check Desired State**: Compare current vs desired replica count
5. **Create Pod**: Create new pod if needed
6. **Pull Image**: Pull container image from registry
7. **Start Container**: Start the container
8. **Health Check**: Verify pod is healthy
9. **Pod Ready**: Pod becomes ready and serves traffic
10. **System Restored**: Normal operation resumed

---

## 15. Pod Lifecycle Timeline

This Gantt chart shows the timeline for pod auto-healing.

```mermaid
gantt
    title Pod Auto-Healing Timeline
    dateFormat YYYY-MM-DD
    section Pod Lifecycle
    Pod Running (Normal)     :2024-01-01, 1d
    Pod Deleted              :2024-01-02, 1d
    Controller Detection     :2024-01-03, 1d
    Image Pull               :2024-01-04, 2d
    Container Start          :2024-01-06, 1d
    Health Check             :2024-01-07, 1d
    Pod Ready (Restored)     :2024-01-08, 1d
```

### 📝 Explanation

Total recovery time: ~43 seconds
- Detection: 2 seconds
- Image pull: 5 seconds
- Container start: 3 seconds
- Health check: 2 seconds

---

## 16. Auto-Healing Scenarios

This diagram shows different scenarios that trigger auto-healing.

```mermaid
graph TB
    Failure[Pod Failure] --> Scenario1[Scenario 1: Manual Deletion]
    Failure --> Scenario2[Scenario 2: Application Crash]
    Failure --> Scenario3[Scenario 3: Node Failure]
    Failure --> Scenario4[Scenario 4: Resource Exhaustion]
    
    Scenario1 --> Action1[Controller detects<br/>replica count mismatch]
    Scenario2 --> Action2[Controller detects<br/>pod status != Running]
    Scenario3 --> Action3[Controller detects<br/>node unavailable]
    Scenario4 --> Action4[Controller detects<br/>OOMKilled status]
    
    Action1 --> Heal[Auto-Healing Process]
    Action2 --> Heal
    Action3 --> Heal
    Action4 --> Heal
    
    Heal --> NewPod[Create New Pod]
    NewPod --> Running[Pod Running]
    
    style Failure fill:#ffcdd2
    style Heal fill:#fff9c4
    style Running fill:#c8e6c9
```

### 📝 Explanation

Four common failure scenarios:
1. Manual deletion
2. Application crash
3. Node failure
4. Resource exhaustion

All lead to automatic pod recreation.

---

## 17. Troubleshooting Flow Diagram

This flowchart shows the troubleshooting process for common issues.

```mermaid
flowchart TD
    Issue[Issue Detected] --> CheckType{What's the Issue?}
    
    CheckType -->|Service Not Working| CheckIngress[Check Ingress]
    CheckType -->|Pod Not Starting| CheckPod[Check Pod Status]
    CheckType -->|Image Pull Error| CheckRegistry[Check Registry]
    CheckType -->|404 Error| CheckRoutes[Check Routes]
    
    CheckIngress --> IngressStatus{kubectl get ingress}
    IngressStatus -->|Not Found| CreateIngress[Create/Apply Ingress]
    IngressStatus -->|Found| CheckTraefik[Check Traefik Pod]
    
    CheckPod --> PodStatus{kubectl get pods}
    PodStatus -->|CrashLoopBackOff| CheckLogs[Check Pod Logs]
    PodStatus -->|Pending| CheckResources[Check Resources]
    PodStatus -->|ImagePullBackOff| CheckImage[Check Image in Registry]
    
    CheckRegistry --> RegistryStatus{curl localhost:5000/v2/_catalog}
    RegistryStatus -->|Empty| PushImage[Push Image to Registry]
    RegistryStatus -->|Has Images| CheckTag[Check Image Tag]
    
    CheckRoutes --> RouteMatch{App Route = Ingress Path?}
    RouteMatch -->|No| FixRoute[Fix Application Route]
    RouteMatch -->|Yes| CheckService[Check Service]
    
    CheckLogs --> FixCode[Fix Code/Config]
    CheckResources --> ScaleDown[Reduce Replicas or Increase Resources]
    CheckImage --> TagCorrectly[Tag and Push Correctly]
    
    CreateIngress --> Test[Test Again]
    CheckTraefik --> RestartTraefik[Restart Traefik if needed]
    PushImage --> Test
    FixRoute --> Rebuild[Rebuild and Redeploy]
    FixCode --> Rebuild
    ScaleDown --> Test
    TagCorrectly --> Test
    Rebuild --> Test
    RestartTraefik --> Test
    
    Test --> Resolved{Issue Resolved?}
    Resolved -->|Yes| Success[✅ Success]
    Resolved -->|No| Issue
    
    style Issue fill:#ffcdd2
    style Success fill:#c8e6c9
    style Test fill:#fff9c4
```

### 📝 Explanation

Troubleshooting steps:
1. Identify issue type
2. Check relevant component
3. Apply fix
4. Test solution
5. Verify resolution

---

## 18. Registry Naming Convention

This diagram explains why we use different registry names from host vs Kubernetes.

```mermaid
graph TB
    subgraph "Host Machine Perspective"
        Dev[Developer]
        DockerHost[Docker Engine]
        Localhost[localhost:5000<br/>Host can reach]
    end
    
    subgraph "k3d Registry Container"
        RegistryContainer[k3d-zomato-registry<br/>Docker Container]
        RegistryPort[Port 5000<br/>Exposed to Host]
    end
    
    subgraph "Kubernetes Cluster Perspective"
        K8sCluster[Kubernetes Cluster]
        K8sRegistry[k3d-zomato-registry:5000<br/>Internal DNS Name]
    end
    
    subgraph "Same Physical Storage"
        Storage[(Image Storage<br/>/var/lib/registry)]
    end
    
    Dev -->|1. docker push| Localhost
    Localhost -->|2. Connect| RegistryPort
    RegistryPort -->|3. Store| Storage
    
    K8sCluster -->|4. Pull Image| K8sRegistry
    K8sRegistry -->|5. Resolve DNS| RegistryContainer
    RegistryContainer -->|6. Read| Storage
    
    Note1[Note: Both names point to<br/>the same registry container]
    
    style Localhost fill:#e1f5ff
    style K8sRegistry fill:#fff9c4
    style Storage fill:#c8e6c9
    style Note1 fill:#fff4e1
```

### 📝 Explanation

**Key Points:**
- `localhost:5000` - Used by Docker on host machine
- `k3d-zomato-registry:5000` - Used by Kubernetes pods (internal DNS)
- Both resolve to the same Docker container
- Images stored once, accessible from both contexts

---

## 19. Cleanup Flow Diagram

This flowchart shows the complete cleanup process.

```mermaid
flowchart TD
    Start([Start Cleanup]) --> Step1[1. Delete k3d Clusters<br/>k3d cluster delete --all]
    Step1 --> Step2[2. Delete k3d Registries<br/>k3d registry delete --all]
    Step2 --> Step3[3. Stop All Containers<br/>docker rm -f $ docker ps -aq]
    Step3 --> Step4[4. Remove All Images<br/>docker rmi -f $ docker images -aq]
    Step4 --> Step5[5. Remove Volumes<br/>docker volume prune -f]
    Step5 --> Step6[6. Remove Networks<br/>docker network prune -f]
    Step6 --> Step7[7. Deep Cleanup<br/>docker system prune -a -f --volumes]
    Step7 --> Verify[8. Verify Cleanup<br/>Check all resources]
    Verify --> Clean{Everything Clean?}
    Clean -->|Yes| Done([✅ Cleanup Complete])
    Clean -->|No| Step1
    
    style Start fill:#e1f5ff
    style Done fill:#c8e6c9
    style Clean fill:#fff9c4
```

### 📝 Explanation

Cleanup steps:
1. Delete k3d clusters
2. Delete k3d registries
3. Stop all containers
4. Remove all images
5. Remove volumes
6. Remove networks
7. Deep cleanup
8. Verify cleanup

---

## 20. Architecture Summary Mind Map

This mind map provides a high-level overview of the entire architecture.

```mermaid
mindmap
  root((Zomato<br/>Microservices))
    Infrastructure
      Docker
        Container Runtime
        Image Building
      k3d
        Lightweight K8s
        Local Development
      Registry
        Image Storage
        Local Registry
    Services
      User Service
        Node.js
        Port 3001
        2 Replicas
      Restaurant Service
        Python Flask
        Port 3002
        2 Replicas
      Order Service
        Go
        Port 3003
        2 Replicas
    Kubernetes
      Deployments
        Pod Management
        Replica Control
      Services
        Load Balancing
        Service Discovery
      Ingress
        External Access
        Path Routing
    Features
      Auto-Healing
      Load Balancing
      High Availability
      Service Discovery
```

### 📝 Explanation

Main categories:
- Infrastructure: Docker, k3d, Registry
- Services: Three microservices
- Kubernetes: Deployments, Services, Ingress
- Features: Auto-healing, Load balancing, etc.

---

## 21. Technology Stack Diagram

This diagram shows the technology stack from application to infrastructure.

```mermaid
graph TB
    subgraph "Application Layer"
        NodeJS[Node.js 18<br/>Express]
        Python[Python 3.11<br/>Flask]
        Go[Go 1.21<br/>net/http]
    end
    
    subgraph "Container Layer"
        Docker[Docker<br/>Containerization]
    end
    
    subgraph "Orchestration Layer"
        K8s[Kubernetes<br/>k3d]
        K8sDeploy[Deployments]
        K8sSvc[Services]
        K8sIngress[Ingress]
    end
    
    subgraph "Infrastructure Layer"
        Registry[Docker Registry<br/>k3d Registry]
        Network[Docker Network<br/>k3d Network]
    end
    
    NodeJS --> Docker
    Python --> Docker
    Go --> Docker
    Docker --> Registry
    Registry --> K8s
    K8s --> K8sDeploy
    K8s --> K8sSvc
    K8s --> K8sIngress
    K8s --> Network
    
    style NodeJS fill:#e8f5e9
    style Python fill:#fce4ec
    style Go fill:#f3e5f5
    style Docker fill:#e1f5ff
    style K8s fill:#fff9c4
```

### 📝 Explanation

Technology layers:
1. **Application Layer**: Node.js, Python, Go
2. **Container Layer**: Docker
3. **Orchestration Layer**: Kubernetes (k3d)
4. **Infrastructure Layer**: Registry, Network

---

## 📚 How to Use These Diagrams

1. **For Learning**: Study each diagram to understand system components
2. **For Troubleshooting**: Use troubleshooting flow diagram
3. **For Deployment**: Follow deployment flow diagram
4. **For Architecture**: Refer to system architecture diagrams

---

## 🔗 Related Documentation

- [README.md](./README.md) - Complete setup guide and instructions
- [GitHub Repository](https://github.com/auspicious27/devops_masterclass-readmee) - Source code and documentation

---

**🎉 Happy Learning! 🚀**

