# 🏪 Bookstore Microservices API

Un sistema de microservicios simple para gestionar una librería, construido con **NestJS**, **Prisma** y **arquitectura Hexagonal + Clean + DDD**. Incluye dos servicios independientes:

- 📚 **Books Service** → Gestión de libros.
- 🛒 **Orders Service** → Gestión de órdenes, validando libros a través del servicio de Books vía HTTP.

---

## 📑 Tabla de Contenidos
- [📌 Descripción General](#-descripción-general)
- [🏗️ Arquitectura del Sistema](#️-arquitectura-del-sistema)
  - [🌐 Diagrama general](#-diagrama-general)
- [🧰 Stack Tecnológico](#-stack-tecnológico)
- [⚙️ Requisitos Previos](#-requisitos-previos)
- [🚀 Instalación & Ejecución](#-instalación--ejecución)
  - [🐳 Docker Compose](#-docker-compose)
  - [💻 Ejecución local](#-ejecución-local)
- [📝 Endpoints Principales](#-endpoints-principales)
- [🔧 Variables de Entorno](#-variables-de-entorno)
- [🛠️ Decisiones de Arquitectura](#-decisiones-de-arquitectura)
- [📚 Documentación Detallada](#-documentación-detallada)
- [👨‍💻 Autor](#-autor)
- [📄 Licencia](#-licencia)

---

## 📌 Descripción General

Este sistema implementa dos microservicios independientes que se comunican vía **HTTP** siguiendo principios de **arquitectura de microservicios**:

- **Books Service** → permite crear, leer y buscar libros.
- **Orders Service** → permite crear y listar órdenes. Antes de crear una orden, valida la existencia del libro llamando al Books Service.

Cada servicio tiene **su propio schema en una base de datos compartida** y no acceden a los schemas de otros servicios.

### ✨ Características Principales

- **Arquitectura desacoplada**: Cada servicio maneja su propio dominio de negocio
- **Comunicación asíncrona**: HTTP/REST entre servicios sin dependencias directas de base de datos
- **Tolerancia a fallos**: Manejo de errores de red y timeouts entre servicios
- **Escalabilidad**: Cada servicio puede desplegarse y escalarse independientemente
- **Documentación automática**: Swagger UI integrado en cada servicio

---

## 🏗️ Arquitectura del Sistema

### 📂 Estructura de directorios

```
.
├── books-service/        # Microservicio de libros
├── orders-service/       # Microservicio de órdenes
├── docker-compose.yml    # Orquestación de servicios
└── README.md             # Documentación raíz
```

### 🌐 Diagrama general

```mermaid
flowchart TB
    subgraph External["🌐 Clientes"]
        UI[Web UI / Postman]
        SWAGGER[Swagger UI]
        CURL[cURL / API Clients]
    end

    subgraph BooksService["📚 Books Microservice (Port 3000)"]
        subgraph BooksInfra["🌐 Interfaces Layer"]
            BookController["BookController<br/>@Controller<br/>• GET /books<br/>• GET /books/:id<br/>• POST /books"]
            BookDTO["DTOs<br/>• CreateBookDto<br/>• GetBookDto"]
        end

        subgraph BooksApp["⚙️ Application Layer"]
            BookUseCases["Use Cases<br/>• CreateBookUseCase<br/>• GetBookByIdUseCase<br/>• ListBooksUseCase"]
        end

        subgraph BooksDomain["🛠️ Domain Layer"]
            BookEntity["Book Entity<br/>• id, title, author<br/>• price, currency, stock<br/>• Factory methods"]
            BookRepoPort["BookRepository<br/>Interface (Port)"]
        end

        subgraph BooksInfraImpl["🔧 Infrastructure Layer"]
            PrismaBookRepo["PrismaBookRepository<br/>Adapter"]
            PrismaBooks["PrismaService"]
        end
    end

    subgraph OrdersService["🛒 Orders Microservice (Port 3001)"]
        subgraph OrdersInfra["🌐 Interfaces Layer"]
            OrderController["OrderController<br/>@Controller<br/>• GET /orders<br/>• POST /orders"]
            OrderDTO["DTOs<br/>• CreateOrderDto<br/>• ListOrdersDto"]
        end

        subgraph OrdersApp["⚙️ Application Layer"]
            OrderUseCases["Use Cases<br/>• CreateOrderUseCase<br/>• ListOrdersUseCase"]
        end

        subgraph OrdersDomain["🛠️ Domain Layer"]
            OrderEntity["Order Entity<br/>• id, bookId, quantity<br/>• totalAmount, status<br/>• Factory methods"]
            OrderRepoPort["OrderRepository<br/>Interface (Port)"]
        end

        subgraph OrdersInfraImpl["🔧 Infrastructure Layer"]
            PrismaOrderRepo["PrismaOrderRepository<br/>Adapter"]
            PrismaOrders["PrismaService"]
            BooksHttpClient["BooksHttpClient<br/>Axios Adapter"]
        end
    end

    subgraph Database["🗄️ PostgreSQL"]
        BooksSchema[("books schema<br/>• book table")]
        OrdersSchema[("orders schema<br/>• order table")]
    end

    %% Client connections
    External --> BookController
    External --> OrderController

    %% Books Service internal flow
    BookController --> BookDTO
    BookController --> BookUseCases
    BookUseCases --> BookRepoPort
    BookUseCases --> BookEntity
    BookRepoPort -.implements.-> PrismaBookRepo
    PrismaBookRepo --> PrismaBooks
    PrismaBooks --> BooksSchema

    %% Orders Service internal flow
    OrderController --> OrderDTO
    OrderController --> OrderUseCases
    OrderUseCases --> OrderRepoPort
    OrderUseCases --> OrderEntity
    OrderUseCases --> BooksHttpClient
    OrderRepoPort -.implements.-> PrismaOrderRepo
    PrismaOrderRepo --> PrismaOrders
    PrismaOrders --> OrdersSchema

    %% Inter-service communication
    BooksHttpClient -.->|HTTP GET /books/:id| BookController

    %% Styling
    classDef microservice fill:#e3f2fd,stroke:#1976d2,stroke-width:3px
    classDef interfaces fill:#fff3e0,stroke:#f57c00,stroke-width:2px
    classDef application fill:#e8f5e8,stroke:#388e3c,stroke-width:2px
    classDef domain fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    classDef infrastructure fill:#fce4ec,stroke:#c2185b,stroke-width:2px
    classDef external fill:#f1f8e9,stroke:#689f38,stroke-width:2px
    classDef database fill:#e0f2f1,stroke:#00796b,stroke-width:2px

    class BooksService,OrdersService microservice
    class BooksInfra,OrdersInfra interfaces
    class BooksApp,OrdersApp application
    class BooksDomain,OrdersDomain domain
    class BooksInfraImpl,OrdersInfraImpl infrastructure
    class External external
    class Database database
```

---

## 🧰 Stack Tecnológico

| Componente | Tecnología | Versión |
|------------|------------|---------|
| Runtime | Node.js | v22 |
| Framework | NestJS | Latest |
| Lenguaje | TypeScript | Latest |
| ORM | Prisma | Latest |
| Base de datos | PostgreSQL | 17 |
| HTTP Client | Axios | Latest |
| Documentación | Swagger/OpenAPI | Latest |
| Containerización | Docker & Docker Compose | Latest |

---

## ⚙️ Requisitos Previos

- Node.js v22
- Docker & Docker Compose
- PostgreSQL (opcional si no usas Docker)

---

## 🚀 Instalación & Ejecución

Clona el repositorio junto a sus submódulos y entra al directorio:

```bash
git clone --recurse-submodules git@github.com:SebastianBC09/Bookstore-microservice-system.git
cd Bookstore-microservice-system
```

### 🐳 Docker Compose

**Opción recomendada** - Levanta todo el sistema con un comando:

```bash
docker-compose up --build
```

Esto iniciará:
- Books Service en `http://localhost:3000`
- Orders Service en `http://localhost:3001`
- PostgreSQL con ambos schemas
- Swagger UI disponible en cada servicio

### 💻 Ejecución local (sin Docker)

1. Configurar variables de entorno en cada servicio
2. Instalar dependencias: `npm install` en cada carpeta
3. Ejecutar migraciones de Prisma en cada servicio
4. Levantar cada servicio: `npm run start:dev`

---

## 📝 Endpoints Principales

### 📚 Books Service (Puerto 3000)
- `GET /books` - Lista todos los libros
- `GET /books/:id` - Obtiene un libro por ID
- `POST /books` - Crea un nuevo libro
- `GET /api/docs` - Documentación Swagger

### 🛒 Orders Service (Puerto 3001)
- `GET /orders` - Lista todas las órdenes
- `POST /orders` - Crea una nueva orden
- `GET /api/docs` - Documentación Swagger

## 🔧 Variables de Entorno

Cada servicio requiere su propio archivo `.env`:

### Books Service
```env
DATABASE_URL="postgresql://user:password@localhost:5432/bookstore?schema=books"
PORT=3000
```

### Orders Service
```env
DATABASE_URL="postgresql://user:password@localhost:5432/bookstore?schema=orders"
BOOKS_SERVICE_URL="http://localhost:3000"
PORT=3001
```

---

## 🛠️ Decisiones de Arquitectura

### 🏛️ Patrones Implementados
- **Arquitectura Hexagonal**: Separación clara entre `domain`, `application`, `infrastructure` e `interfaces`
- **Clean Architecture**: Dependencias apuntan hacia el dominio
- **Domain Driven Design**: Entidades y agregados bien definidos
- **Repository Pattern**: Abstracción de acceso a datos
- **Factory Pattern**: Creación controlada de entidades

### 🔗 Comunicación entre Servicios
- **HTTP/REST**: Comunicación síncrona entre microservicios
- **No shared database**: Cada servicio maneja su propio schema
- **Error handling**: Manejo robusto de fallos de red y timeouts

### 🎯 Principios Aplicados
- **Single Responsibility**: Cada servicio tiene una responsabilidad clara
- **Dependency Inversion**: Uso de interfaces y puertos/adaptadores
- **Open/Closed**: Extensible sin modificar código existente

---

## 📚 Documentación Detallada

Para información específica de implementación, consulta los READMEs de cada servicio:

- [📚 Books Service README](./books-service/README.md)
- [🛒 Orders Service README](./orders-service/README.md)

---

## 👨‍💻 Autor

**Sebastian Ballen C** - _FullStack Developer_

- LinkedIn: [Sebastian B.](https://www.linkedin.com/in/sebastianballencastaneda-softwaredeveloper)
- Email: sebastian.ballenc@gmail.com

---

## 📄 Licencia

Este proyecto está licenciado bajo la Licencia MIT - vea el archivo [LICENSE](LICENSE) para más detalles.

---

⭐️ **Si te resulta útil este proyecto, ¡no olvides darle una estrella en GitHub!** ⭐️
