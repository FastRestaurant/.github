<div align="center">

  <h1>🍽️ FastRestaurant</h1>
  <h3>Sistema de Gestión Inteligente para Restaurantes</h3>

  <p>
    <b>Microservicios • Tiempo Real (SignalR) • Orquestación de Cocina</b>
  </p>

  <p>
    <a href="#-arquitectura-del-sistema">
      <img src="https://img.shields.io/badge/VER_ARQUITECTURA-0f0706?style=for-the-badge&logo=mermaid&logoColor=white" alt="Architecture" />
    </a>
    <a href="https://github.com/FastRestaurant/Frontend">
      <img src="https://img.shields.io/badge/FRONTEND-512BD4?style=for-the-badge&logo=blazor&logoColor=white" alt="Frontend" />
    </a>
  </p>
</div>

---

## 💡 Sobre el Proyecto

**FastRestaurant** nace para resolver un problema muy concreto de los restaurantes medianos: la falta de sincronización entre el salón y la cocina durante las horas pico. Sin un sistema que ordene los tiempos de preparación, los mozos cargan comandas "a ciegas", los platos llegan fríos o desincronizados a la mesa, y cada noche de pico se convierte en una apuesta.

El desafío principal no fue solo digitalizar la toma de pedidos, sino diseñar un **motor de orquestación de tiempos** capaz de retrasar automáticamente los platos rápidos hasta que los lentos estén en la etapa correcta — y recalcular todo en tiempo real ante imprevistos (un plato arruinado, una cancelación) — sobre una arquitectura de **Microservicios en .NET 8** comunicados vía HTTP y eventos en tiempo real con **SignalR**.

### 🌟 Features Clave
* **Motor de Orquestación:** Sincroniza los tiempos de cocción para que toda la mesa reciba sus platos al mismo tiempo y a temperatura ideal.
* **Kitchen Display System (KDS):** Pantalla interactiva que agrupa y prioriza tareas de cocina por tiempo de preparación, no por orden de ingreso.
* **Recálculo en Tiempo Real:** Ante un plato arruinado o una cancelación, el sistema renotifica al mozo y recalcula la proyección de entrega al instante.
* **Roles y Permisos:** Accesos diferenciados para Gerente, Mozo y Cocinero, de extremo a extremo.
* **Estados en Tiempo Real:** Flujo `Recibido → En Preparación → Listo → Entregado` con alertas instantáneas vía WebSockets.

---

## 🏗️ Arquitectura del Sistema

Arquitectura de microservicios sobre **.NET 8** con **Clean Architecture** en cada servicio, cliente **Blazor WebAssembly** y comunicación en tiempo real vía **SignalR**.

```mermaid
graph TD
    %% --- ESTILOS DE ALTO CONTRASTE ---
    classDef frontend fill:#E1F5FE,stroke:#0277BD,stroke-width:2px,color:#000000;
    classDef gateway fill:#FFF8E1,stroke:#FF8F00,stroke-width:2px,color:#000000;
    classDef service fill:#FFFFFF,stroke:#2E7D32,stroke-width:2px,color:#000000,rx:5,ry:5;
    classDef db fill:#F3E5F5,stroke:#7B1FA2,stroke-width:2px,stroke-dasharray: 5 5,color:#000000;
    classDef realtime fill:#FFEBEE,stroke:#C62828,stroke-width:2px,color:#000000;

    %% --- NODOS PRINCIPALES ---
    Client["💻 Blazor WebAssembly (SPA)"]:::frontend
    Hub["📡 SignalR Hub\n(Eventos en Tiempo Real)"]:::realtime
    DB[("🗄️ SQL Server\n(1 por servicio)")]:::db

    %% --- CONTEXTO DE MICROSERVICIOS ---
    subgraph Services ["🏗️ Ecosistema de Microservicios"]
        direction TB
        UserS["👤 UserService\n(Auth, JWT y Roles)"]:::service
        MenuS["🍔 MenuService\n(Platos, Bebidas, Categorías)"]:::service
        StockS["📦 StockService\n(Control de Inventario)"]:::service
        OrderS["🧾 OrderService\n(Pedidos, Mesas y Estados)"]:::service
        KitchenS["👨‍🍳 KitchenService\n(Kitchen Display System)"]:::service
        MenuS ~~~ UserS ~~~ StockS
    end

    %% --- CONEXIONES (FLUJO) ---

    %% 1. Entrada
    Client ==>|"1. HTTPS + JWT"| UserS
    Client ==>|"Toma de pedidos"| OrderS
    Client ==>|"Gestión de catálogo"| MenuS

    %% 2. Dependencias entre servicios (HTTP)
    OrderS -->|"Valida usuario / mozo"| UserS
    OrderS -->|"Consulta catálogo"| MenuS
    OrderS -->|"Reserva / libera stock"| StockS
    OrderS -->|"Envía pedido a preparar"| KitchenS

    %% 3. Comunicación en Tiempo Real (SignalR)
    OrderS -.->|"Notifica cambio de estado"| Hub
    KitchenS -.->|"Plato listo / Recálculo"| Hub
    Hub -.->|"Push en vivo"| Client

    %% 4. Base de Datos
    UserS --- DB
    MenuS --- DB
    StockS --- DB
    OrderS --- DB
    KitchenS --- DB

    %% Truco visual para alinear
    OrderS ~~~ KitchenS
```

---

## 📦 Repositorios

| Servicio | Descripción |
|---|---|
| [**Frontend**](https://github.com/FastRestaurant/Frontend) | Cliente Blazor WebAssembly (SPA) — interfaz para mozos, cocina y gerentes. |
| [**MicroservicioAuth**](https://github.com/FastRestaurant/MicroservicioAuth) | Autenticación, JWT y gestión de roles (Gerente, Mozo, Cocinero). |
| [**MicroservicioMenu**](https://github.com/FastRestaurant/MicroservicioMenu) | Catálogo de platos, bebidas y categorías. |
| [**MicroservicioStock**](https://github.com/FastRestaurant/MicroservicioStock) | Control de inventario e integración con el flujo de pedidos. |
| [**MicroservicioOrder**](https://github.com/FastRestaurant/MicroservicioOrder) | Gestión de pedidos, mesas y motor de orquestación de estados. |
| [**MicroservicioKitchen**](https://github.com/FastRestaurant/MicroservicioKitchen) | Kitchen Display System (KDS) — visualización y priorización de comandas. |

---

## 🛠️ Stack Tecnológico
<div align="left"> 
  <img src="https://img.shields.io/badge/C%23-239120?style=for-the-badge&logo=c-sharp&logoColor=white"/> 
  <img src="https://img.shields.io/badge/.NET_8-512BD4?style=for-the-badge&logo=dotnet&logoColor=white"/> 
  <img src="https://img.shields.io/badge/Blazor_WASM-512BD4?style=for-the-badge&logo=blazor&logoColor=white"/> 
  <img src="https://img.shields.io/badge/SignalR-512BD4?style=for-the-badge&logo=dotnet&logoColor=white"/> 
  <img src="https://img.shields.io/badge/Entity_Framework_Core-512BD4?style=for-the-badge&logo=dotnet&logoColor=white"/> 
  <img src="https://img.shields.io/badge/SQL_Server-CC2927?style=for-the-badge&logo=microsoftsqlserver&logoColor=white"/> 
  <img src="https://img.shields.io/badge/Swagger-85EA2D?style=for-the-badge&logo=swagger&logoColor=black"/> 
  <img src="https://img.shields.io/badge/Git_%2F_GitHub-181717?style=for-the-badge&logo=github&logoColor=white"/> 
</div>

---

## 👥 El Equipo — "SincroComanda"

Proyecto desarrollado por un equipo de **11 integrantes** bajo metodología **Scrum**, en **3 Sprints** de 2 semanas, con entregas demo desde el Sprint 1.

| Rol | Responsabilidad |
|---|---|
| Scrum Master | Facilita ceremonias y remueve obstáculos del equipo |
| Product Owner | Prioriza el backlog y valida criterios de aceptación |
| QA | Pruebas funcionales y control de calidad continuo |
| Developers (8) | Implementación distribuida por microservicio |
