# WebSockets & Real-time Communication

Purpose
- Guide on building and operating WebSocket or long-lived real-time connections.

Connection model
- Use a gateway or managed service (Azure Web PubSub) to handle scaling and reconnection logic.
- Keep connection state minimal on servers; prefer sticky sessions only when necessary and use external state stores for scale-out.

Authentication
- Authenticate on connection establishment (e.g., JWT token in a query param or during upgrade handshake) and renew tokens for long-lived connections.

Message patterns
- Use topic/room patterns for message routing and avoid broadcasting to all clients.
- Implement server-side rate limiting and backpressure for message producers.

Scaling
- Use a managed pub/sub gateway or a clustered broker for fan-out (Azure Web PubSub, Redis Streams + Pub/Sub, or Kafka for backend eventing).
- Offload heavy processing to background jobs and publish results to clients via events.

Operational
- Monitor connection counts, reconnection rates, and message error rates.
- Provide stable message schemas and versioning for client compatibility.
