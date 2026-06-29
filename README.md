                            ┌───────────────────────┐
                            │  Frontend (React)     │
                            │  Port 3000            │
                            └───────────┬───────────┘
                                        │
                            ┌───────────▼───────────┐
                            │   API Gateway         │
                            │   Port 4000           │
                            │ (JWT, rate limit,     │
                            │  circuit breaker)     │
                            └───┬───────┬───────┬───┘
              ┌─────────────────┼───────┼───────┼───────────────┐
              │                 │       │       │               │
   ┌──────────▼──────┐ ┌────────▼─┐ ┌───▼────┐ ┌▼──────────┐ ┌──▼──────────┐
   │ User Service    │ │ Search   │ │ Admin  │ │ Booking   │ │ Payment     │
   │ Port 4001       │ │ 4002     │ │ 4003   │ │ 4005      │ │ 4006        │
   └────────┬────────┘ └────┬─────┘ └───┬────┘ └─────┬─────┘ └─────┬───────┘
            │               │           │            │             │
            │          ┌────▼──────┐    │     ┌──────▼─────┐       │
            │          │ Inventory │    │     │ Notification│      │
            │          │ 4007      │    │     │ 4004 (Kafka)│      │
            │          └────┬──────┘    │     └──────┬──────┘      │
            │               │           │            │             │
            └───────────────┴───────────┴────────────┴─────────────┘
                                      │
                ┌─────────────────────┼─────────────────────┐
                │                     │                     │
        ┌───────▼──────┐    ┌─────────▼────────┐   ┌────────▼────────┐
        │  PostgreSQL  │    │  Redis Stack     │   │   Kafka         │
        │  Port 5432   │    │  Port 6379/8001  │   │   Port 9092/9093│
        └──────────────┘    └──────────────────┘   └─────────────────┘
                                                            │
                                                  ┌─────────▼─────────┐
                                                  │  Elasticsearch    │
                                                  │  Port 9200        │
                                                  └───────────────────┘
Highlights:

- API Gateway is the single entrypoint for the frontend; it proxies to the right service and enforces JWT, rate limits, and circuit breakers.
- Database-per-service — each service owns its own Postgres database, search-service uses Elasticsearch.
- Event-driven — Kafka decouples booking, payment, inventory, search and notification flows. Topics are centralized in shared/constants/kafka-topics.js.
- Notification service has no HTTP API — it is purely a Kafka consumer that sends emails via SendGrid.
