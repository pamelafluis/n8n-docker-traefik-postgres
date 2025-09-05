## Architecture

This setup demonstrates how to run **n8n** securely inside an **OCI Compute VM** using Docker and **Traefik** as the reverse proxy.

### 1\. Traffic Flow

* A user accesses [`n8n.example.com`](http://n8n.example.com), which resolves via DNS to the VM’s **public IP**.
    
* Requests on **ports 80/443** reach the **Traefik container** inside the VM.
    
* Traefik forwards HTTPS traffic through the `traefik-public` network to the **n8n container**.
    

### 2\. Security & Certificates

* Traefik manages SSL/TLS certificates automatically with **Let’s Encrypt CA**.
    
* Certificates are issued and renewed using the **ACME TLS challenge**.
    
* All certificates are stored securely in `./letsencrypt/acme.json`.
    

### 3\. Service Discovery

* Traefik integrates with the **Docker Socket** to dynamically discover running containers.
    
* This eliminates manual configuration whenever services are added or updated.
    

### 4\. Application Layer

* **n8n container** hosts the workflow automation platform.
    
* **Postgres container** provides persistent database storage, connected via the `n8n-network` (port 5432).
    

### 5\. Infrastructure

* All components (Traefik, n8n, Postgres) run as **Docker containers** inside a single **OCI Compute VM**.
    
* Networking is logically separated using Docker networks (`traefik-public` and `n8n-network`).





```mermaid
---
config:
  layout: elk
---
flowchart TB
 subgraph DockerHost["🐳 Docker Host"]
    direction TB
        Traefik["🚦 Traefik (container)"]
        N8N["⚡ n8n (container)"]
        DB["🗄️ Postgres (container)"]
        DockerSocket["🔌 Docker Socket"]
        ACMEFile["🔑 ./letsencrypt/acme.json"]
  end
 subgraph VM["🖥️ OCI Compute VM"]
        DockerHost
  end
    Browser["💻 User Browser"] -- DNS resolves --> DNS["🌐 n8n.example.com → Public IP"]
    DNS -- Ports 80/443 --> Traefik
    Traefik -- routes HTTPS traffic --> N8N
    N8N -- 5432 --> DB
    Traefik -- uses --- DockerSocket
    Traefik -- stores certificates in --- ACMEFile
    Traefik -- ACME TLS challenge --> LetsEncrypt@{ label: "🔐 Let's Encrypt CA" }
    LetsEncrypt@{ shape: rect}

    %% Classes
     Traefik:::traefik
     N8N:::n8n
     DB:::db
     DockerSocket:::socket
     ACMEFile:::acme
     Browser:::external
     DNS:::external
     LetsEncrypt:::ca
     VM:::infra

    %% Styles
    classDef external fill:#fef3c7,stroke:#f59e0b,stroke-width:2px,color:#b45309,font-weight:bold,font-size:14px
    classDef ca fill:#dcfce7,stroke:#22c55e,stroke-width:2px,color:#166534,font-weight:bold,font-size:14px
    classDef infra fill:#f0f9ff,stroke:#0ea5e9,stroke-width:2px,color:#075985,font-weight:bold,font-size:14px

    %% Different colors per container
    classDef traefik fill:#fee2e2,stroke:#ef4444,stroke-width:2px,color:#991b1b,font-weight:bold,font-size:14px
    classDef n8n fill:#ede9fe,stroke:#8b5cf6,stroke-width:2px,color:#4c1d95,font-weight:bold,font-size:14px
    classDef db fill:#dcfce7,stroke:#22c55e,stroke-width:2px,color:#166534,font-weight:bold,font-size:14px
    classDef socket fill:#cffafe,stroke:#06b6d4,stroke-width:2px,color:#0e7490,font-weight:bold,font-size:14px
    classDef acme fill:#fef9c3,stroke:#eab308,stroke-width:2px,color:#854d0e,font-weight:bold,font-size:14px

