# Diagrama de Arquitectura - Fullstack3

![Arquitectura Fullstack3](architecture.png)

```mermaid
graph TB
    subgraph Cliente["Cliente Web"]
        F["Frontend Vue.js<br/>puerto 5173"]
    end

    subgraph Gateway["API Gateway (Spring Cloud Gateway)"]
        GW["API Gateway<br/>puerto 8080"]
    end

    subgraph Discovery["Service Discovery"]
        EU["Eureka Server<br/>puerto 8761"]
    end

    subgraph BFF["Backend for Frontend"]
        BFF_SERVICE["BFF Service<br/>puerto 8084"]
    end

    subgraph Microservicios["Microservicios"]
        US["User Service<br/>puerto 8082"]
        RS["Report Service<br/>puerto 8081"]
        AS["Alert Service<br/>puerto 8083"]
    end

    subgraph DB["Base de Datos"]
        MYSQL["MySQL<br/>puerto 3306"]
    end

    F -->|"HTTP /api/*"| GW
    GW -->|"lb://bff-service"| BFF_SERVICE
    GW -->|"lb://user-service"| US
    GW -->|"lb://report-service"| RS
    GW -->|"lb://alert-service"| AS
    BFF_SERVICE -->|"Feign Client /api/usuario/**"| US
    BFF_SERVICE -->|"Feign Client /api/reporte-incendio/**"| RS
    BFF_SERVICE -->|"Feign Client /api/alerta-emergencia/**"| AS
    AS -->|"RestTemplate @LoadBalanced"| RS
    US -->|"JPA / JDBC"| MYSQL
    RS -->|"JPA / JDBC"| MYSQL
    AS -->|"JPA / JDBC"| MYSQL
    GW -.->|"Registra en Eureka"| EU
    BFF_SERVICE -.->|"Registra en Eureka"| EU
    US -.->|"Registra en Eureka"| EU
    RS -.->|"Registra en Eureka"| EU
    AS -.->|"Registra en Eureka"| EU

    style F fill:#42b883,color:#fff
    style GW fill:#42a5f5,color:#fff
    style EU fill:#ff7043,color:#fff
    style BFF_SERVICE fill:#ab47bc,color:#fff
    style US fill:#66bb6a,color:#fff
    style RS fill:#66bb6a,color:#fff
    style AS fill:#66bb6a,color:#fff
    style MYSQL fill:#78909c,color:#fff
```

## Flujo de una solicitud

```
Frontend (5173) → GET /api/bff/dashboard/estadisticas
    ↓
API Gateway (8080) → reconoce ruta /api/bff/**
    ↓ lb://bff-service
BFF Service (8084) → llama a ReportClient.getAllReportes()
    ↓ Feign Client (Eureka descubre report-service)
Report Service (8081) → consulta MySQL, devuelve datos
    ↓
BFF → procesa estadísticas (cuenta por estado)
    ↓
API Gateway → devuelve respuesta al Frontend
```

## Cobertura de Tests

| Servicio | Tests | Cobertura línea |
|---|---|---|
| User Service | 25 | ~95% |
| Report Service | 17 | ~97% |
| Alert Service | 22 | ~83% |
| BFF | 9 | ~98% |
| API Gateway | 1 | N/A |
| Eureka Server | 1 | N/A |
| Frontend (Vitest) | 20 | N/A |
