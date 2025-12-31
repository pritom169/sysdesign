## Load Balancing

### Introduction to Load Balancing

Load balancing is a crucial component of System Design, as it helps distribute incoming requests and traffic evenly across multiple servers. The main goal of load balancing is to ensure high availability, reliability, and performance by avoiding overloading a single server and avoiding downtime.

Typically a load balancer sits between the client and the server accepting incoming network and application traffic and distributing the traffic across multiple backend servers using various algorithms. By balancing application requests across multiple servers, a load balancer reduces the load on individual servers and prevents any one server from becoming a single point of failure, thus improving overall application availability and responsiveness.

```mermaid
flowchart LR
    %% Define the "Client" group with three user nodes
    subgraph Client_Group [Client]
        C1[User 1]
        C2[User 2]
        C3[User 3]
    end

    %% Define the Internet node
    I([Internet])

    %% Define the Load Balancer node
    LB[Load Balancer]

    %% Define the "Servers" group with four server nodes
    subgraph Server_Group [Servers]
        S1[Server 1]
        S2[Server 2]
        S3[Server 3]
        S4[Server 4]
    end

    %% Define the connections (flow)
    C1 --> I
    C2 --> I
    C3 --> I
    I --> LB
    LB --> S1
    LB --> S2
    LB --> S3
    LB --> S4

    %% Optional: Styling to match the image's colors somewhat
    style I fill:#87CEEB,stroke:#333,stroke-width:2px,color:white
    style LB fill:#367FA9,stroke:#333,stroke-width:2px,color:white
    style Client_Group fill:none,stroke:none
    style Server_Group fill:none,stroke:none
```

To utilize full scalability and redundancy, we can try to balance the load at each layer of the system. We can add LBs at three places:

- Between the user and the web server
- Between web servers and an internal platform layer, like application servers or cache servers
- Between internal platform layer and database.

```mermaid
flowchart LR
    %% Define the Client node
    Client([Client])

    %% Define the first Load Balancer
    LB1{Load Balancer}

    %% Define the Web Server group
    subgraph Web_Tier [Web Servers]
        WS1[Web Server 1]
        WS2[Web Server 2]
    end

    %% Define the second Load Balancer
    LB2{Load Balancer}

    %% Define the Application Server group
    subgraph App_Tier [Application Servers]
        AS1[Application Server 1]
        AS2[Application Server 2]
    end

    %% Define the third Load Balancer
    LB3{Load Balancer}

    %% Define the Database Server group
    subgraph DB_Tier [Database Servers]
        DS1[(Database Server 1)]
        DS2[(Database Server 2)]
    end

    %% Define the connections (flow)
    Client --> LB1
    LB1 --> WS1
    LB1 --> WS2
    WS1 --> LB2
    WS2 --> LB2
    LB2 --> AS1
    LB2 --> AS2
    AS1 --> LB3
    AS2 --> LB3
    LB3 --> DS1
    LB3 --> DS2

    %% Optional: Styling to match the image's colors and shapes
    style Client fill:#87CEEB,stroke:#333,stroke-width:2px,color:white,shape:cloud
    style LB1 fill:#F5B041,stroke:#333,stroke-width:2px
    style LB2 fill:#82E0AA,stroke:#333,stroke-width:2px
    style LB3 fill:#85C1E9,stroke:#333,stroke-width:2px
    style WS1 fill:#C39BD3,stroke:#333,stroke-width:2px
    style WS2 fill:#C39BD3,stroke:#333,stroke-width:2px
    style AS1 fill:#8E44AD,stroke:#333,stroke-width:2px,color:white
    style AS2 fill:#8E44AD,stroke:#333,stroke-width:2px,color:white
    style DS1 fill:#7F8C8D,stroke:#333,stroke-width:2px,color:white
    style DS2 fill:#7F8C8D,stroke:#333,stroke-width:2px,color:white
    style Web_Tier fill:none,stroke:none
    style App_Tier fill:none,stroke:none
    style DB_Tier fill:none,stroke:none
```

#### Key terminology and concepts

**Load Balancer:** A device or software that distributes network traffic across multiple servers based on predefined rules or algorithms.

**Backend Servers:** The servers that receive and process requests forwarded by the load balancer. Also referred to as the server pool or server farm.

**Load Balancing Algorithm:** The method used by the load balancer to determine how to distribute incoming traffic among the backend servers.

**Health Checks:** Periodic tests performed by the load balancer to determine the availability and performance of backend servers. Unhealthy servers are removed from the server pool until they recover.

**Session Persistence:** A technique used to ensure that subsequent requests from the same client are directed to the same backend server, maintaining session state and providing a consistent user experience.

**SSL/TLS Termination:** The process of decrypting SSL/TLS-encrypted traffic at the load balancer level, offloading the decryption burden from backend servers and allowing for centralized SSL/TLS management.
