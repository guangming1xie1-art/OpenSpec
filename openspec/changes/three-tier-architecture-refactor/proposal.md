## Why
Current architecture lacks clear separation of concerns across the travel assistant system (React UI, Python Agent, Java API). Integrating Agent capabilities directly into the UI and tightly coupling business logic to data services makes maintenance difficult, inhibits independent scaling, and complicates cloud deployment. A three-tier architecture will enable the Agent to serve as an intelligent orchestrator, the UI to focus on user interaction, and the API to provide clean data services.

## What Changes
- Refactor to a strict three-tier architecture: React UI → Agent (Python) → Java API
- React UI: Replace direct Java API calls with Agent communication via HTTP/WebSocket for conversational interactions
- Agent Layer: Add JavaAPIClient tool library for HTTP REST calls to Java API; refactor all Agent skills (SearchAgent, RecommendationAgent, BookingAgent) to delegate to Java API endpoints instead of direct implementation
- Java API: Standardize all endpoints to follow RESTful principles with JSON responses; add OpenAPI/Swagger documentation
- Implement error handling and retry mechanisms for Agent ↔ Java API communication
- Add Docker containerization and configuration management for cloud deployment readiness

## Impact
- Affected specs: `react-ui`, `agent-layer`, `java-api`
- Affected code:
  - React frontend: `travel-assistant-front` (API endpoints, UI integration)
  - Python Agent: `travel-assistant-agent` (JavaAPIClient tool library, all Agent skill implementations)
  - Java API: `travel-assistant` (API standardization, OpenAPI documentation)
