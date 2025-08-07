# Architecture of `filespin-asset-post-process`

This document outlines the architecture of the `filespin-asset-post-process` service.

## Overview

The service is a decoupled, event-driven microservice built on a task queue architecture. Its primary responsibility is to orchestrate post-processing tasks on assets by reacting to events and delegating the actual work to other specialized "addon" services.

## Architecture Diagram

```mermaid
graph TD
    subgraph FileSpin Ecosystem
        A[External Event <br> e.g., file-saved, file-deleted] --> B{FileSpin Backend};
        B --> |1. Pushes message| C[<img src='[https://symbols.getvecta.com/stencil_25/37_redis-icon.d46f22622f.svg](https://symbols.getvecta.com/stencil_25/37_redis-icon.d46f22622f.svg)' width='20'/> Redis <br> Message Queue & Cache];
    end

    subgraph " "
        subgraph "asset-post-process Service (Docker Container)"
            style "asset-post-process Service (Docker Container)" fill:#f9f9f9,stroke:#333,stroke-width:2px

            D1[<img src='[https://www.celeryproject.org/img/logo.png](https://www.celeryproject.org/img/logo.png)' width='20'/> Celery Worker <br> (process_asset)]
            D2[<img src='[https://www.celeryproject.org/img/logo.png](https://www.celeryproject.org/img/logo.png)' width='20'/> Celery Worker <br> (trigger_addon)]
            E[<img src='[https://fastapi.tiangolo.com/img/logo-margin/logo-teal.png](https://fastapi.tiangolo.com/img/logo-margin/logo-teal.png)' width='20'/> FastAPI / Gunicorn <br> (Healthcheck API)]

            C --> |2. Consumes task| D1;
            D1 --> |4. Pushes addon task| C;
            C --> |5. Consumes addon task| D2;

            D1 <-->|3. Gets asset/user details| B;
            D2 --> |6. Triggers addon via API call| F((<img src='[https://cdn.worldvectorlogo.com/logos/microservices.svg](https://cdn.worldvectorlogo.com/logos/microservices.svg)' width='20'/><br>Addon Services <br> Emotion, Scene, FR, etc.));

        end
    end

    subgraph Deployment
        G[<img src='[https://cdn.worldvectorlogo.com/logos/docker.svg](https://cdn.worldvectorlogo.com/logos/docker.svg)' width='20'/> Docker Swarm] -->|"Deploys & Manages"| H["asset-post-process Service (Docker Container)"];
        I[<img src='[https://upload.wikimedia.org/wikipedia/commons/thumb/1/1b/Supervisor_logo.svg/2560px-Supervisor_logo.svg.png](https://upload.wikimedia.org/wikipedia/commons/thumb/1/1b/Supervisor_logo.svg/2560px-Supervisor_logo.svg.png)' width='20'/> Supervisor] -->|"Runs inside container"| D1;
        I -->|"Runs inside container"| D2;
        I -->|"Runs inside container"| E;
    end


    linkStyle 0 stroke-width:2px,fill:none,stroke:orange;
    linkStyle 1 stroke-width:2px,fill:none,stroke:green;
    linkStyle 2 stroke-width:2px,fill:none,stroke:blue;
    linkStyle 3 stroke-width:2px,fill:none,stroke:red;
    linkStyle 4 stroke-width:2px,fill:none,stroke:blue;
    linkStyle 5 stroke-width:2px,fill:none,stroke:purple;
    linkStyle 6 stroke-width:2px,fill:none,stroke:purple;
    linkStyle 7 stroke-width:2px,fill:none,stroke:grey;
    linkStyle 8 stroke-width:2px,fill:none,stroke:grey;
    linkStyle 9 stroke-width:2px,fill:none,stroke:grey;
