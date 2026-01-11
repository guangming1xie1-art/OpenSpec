## Context
The lvyou travel assistant system consists of three separate projects:
- **React UI** (`travel-assistant-front`): Currently handles user interface and makes direct API calls to the Java backend
- **Python Agent** (`travel-assistant-agent`): Implements intelligent travel planning with LangGraph, but skills are self-contained and don't integrate with the Java API
- **Java API** (`travel-assistant`): Provides data services and business logic via Spring Cloud

Current limitations:
- No clear separation between UI, business logic, and data services
- Agent capabilities are not utilized by the UI
- Tight coupling makes independent scaling difficult
- Cloud deployment is complex due to lack of clear boundaries

Stakeholders:
- Development team needs clear separation of concerns
- Operations team needs containerized, deployable services
- Users need responsive, intelligent travel planning

## Goals / Non-Goals
**Goals:**
- Establish clear three-tier architecture: React UI → Agent → Java API
- Enable Agent to serve as intelligent orchestrator for all user interactions
- Make Java API a pure data and service layer
- Support independent scaling and cloud deployment
- Improve maintainability through clear boundaries

**Non-Goals:**
- Complete rewrite of existing components (reuse and refactor)
- Implementation of authentication/authorization between layers (may be added later)
- Real-time features beyond basic WebSocket communication
- Mobile app support (React Web focus only)

## Decisions

### Architecture Decision 1: Strict Three-Tier Separation
**Decision:** Enforce strict layer boundaries where React UI only talks to Agent, and Agent only talks to Java API.

**Rationale:**
- Clear separation of concerns
- Enables independent scaling (UI for interaction, Agent for intelligence, API for data)
- Simplifies testing and maintenance
- Facilitates cloud deployment with separate services

**Alternatives considered:**
- Option A: UI can talk to both Agent and Java API (rejected - adds complexity)
- Option B: Agent sits in front of Java API as middleware proxy (rejected - Agent should orchestrate, not proxy)
- Option C: All three components merge into monolith (rejected - defeats purpose of refactor)

### Architecture Decision 2: Agent as Intelligent Orchestrator
**Decision:** Agent becomes the central decision engine, handling all business logic, workflow coordination, and user conversation state.

**Rationale:**
- Leverages LangGraph's natural language and reasoning capabilities
- Reduces UI complexity (just sends user messages)
- Keeps Java API focused on data and services
- Enables sophisticated multi-step workflows (search → compare → recommend → book)

**Alternatives considered:**
- Option A: UI handles workflow orchestration (rejected - UI should be thin)
- Option B: Java API handles business logic (rejected - should be data-centric)

### Architecture Decision 3: HTTP REST for Agent ↔ Java API Communication
**Decision:** Use standard HTTP REST API with JSON for all Agent to Java API calls.

**Rationale:**
- Language-agnostic standard
- Easy to debug and monitor
- Well-supported tooling (OpenAPI, curl, Postman)
- Fits existing Java API architecture

**Alternatives considered:**
- Option A: gRPC (rejected - adds complexity, not needed for current scale)
- Option B: GraphQL (rejected - Java API already REST-based)
- Option C: Direct Python-Java bridge (rejected - tight coupling)

### Architecture Decision 4: WebSocket for React ↔ Agent Communication
**Decision:** Use WebSocket for real-time conversational interface between React UI and Agent.

**Rationale:**
- Supports streaming responses (important for Agent's step-by-step reasoning)
- Natural for chat-like interfaces
- Enables bidirectional communication
- Lower latency than HTTP polling

**Alternatives considered:**
- Option A: HTTP POST for each message (rejected - no streaming support)
- Option B: Server-Sent Events (rejected - Agent needs bidirectional for clarifications)

### Architecture Decision 5: Error Handling with Retry Mechanism
**Decision:** Implement exponential backoff retry for Agent → Java API calls with clear error messages to user.

**Rationale:**
- Network reliability issues are common
- Improves user experience (transient failures don't block)
- Standard pattern for distributed systems

**Alternatives considered:**
- Option A: Fail fast on first error (rejected - poor UX)
- Option B: Queue-based async processing (rejected - adds complexity not needed initially)

## Communication Protocols

### React ↔ Agent (Conversational Layer)
**Protocol:** WebSocket over HTTP
**Port:** 8000 (Agent server)
**Message Format:**
```json
{
  "message_id": "uuid",
  "user_id": "uuid",
  "content": "string",
  "session_context": {}
}
```

**Response Format (streaming):**
```json
{
  "message_id": "uuid",
  "type": "thinking|response|error|complete",
  "content": "string",
  "data": {}  // optional structured data
}
```

### Agent ↔ Java API (Data Layer)
**Protocol:** HTTP REST over JSON
**Base URL:** http://java-api:8080/api
**Standard Response Format:**
```json
{
  "success": true|false,
  "data": {},
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable message"
  }
}
```

**Standard Error Codes:**
- `VALIDATION_ERROR`: Invalid input parameters
- `NOT_FOUND`: Resource not found
- `SERVICE_UNAVAILABLE`: Java API down or timeout
- `AUTHENTICATION_FAILED`: (future) Auth token invalid

## Data Flow Examples

### Example 1: Flight Search Flow
1. User types "Search flights from Beijing to Shanghai on 2024-06-01" in React UI
2. React sends WebSocket message to Agent
3. Agent parses intent, routes to SearchAgent
4. SearchAgent uses JavaAPIClient.search_flights()
5. JavaAPIClient makes HTTP GET /api/flights/search?origin=PEK&destination=SHA&date=2024-06-01
6. Java API returns JSON with flight results
7. SearchAgent formats results, sends streaming response to React
8. React displays flight options to user

### Example 2: Booking Flow
1. User selects flights and clicks "Book" in React UI
2. React sends booking request to Agent via WebSocket
4. Agent routes to BookingAgent
5. BookingAgent calls JavaAPIClient.create_booking()
6. JavaAPIClient makes HTTP POST /api/bookings with booking data
7. Java API creates booking record, returns booking_id
8. BookingAgent calls JavaAPIClient.process_payment()
9. JavaAPIClient makes HTTP POST /api/payments
10. Java API processes payment, returns success
11. BookingAgent calls JavaAPIClient.confirm_booking(booking_id)
12. JavaAPIClient makes HTTP PUT /api/bookings/{id}/confirm
13. Java API confirms booking, returns final status
14. Agent streams success message to React
15. React displays confirmation to user

## JavaAPIClient Tool Library

### Interface Design
```python
class JavaAPIClient:
    def __init__(self, base_url: str, timeout: int = 30):
        self.base_url = base_url
        self.timeout = timeout
        self.session = httpx.AsyncClient()

    async def search_flights(self, origin: str, destination: str, date: str, passengers: int) -> dict:
        """Search flights via Java API"""

    async def search_hotels(self, destination: str, check_in: str, check_out: str, guests: int) -> dict:
        """Search hotels via Java API"""

    async def get_attractions(self, destination: str) -> dict:
        """Get attractions for destination"""

    async def get_weather(self, destination: str, date: str) -> dict:
        """Get weather forecast"""

    async def create_booking(self, booking_data: dict) -> dict:
        """Create booking record"""

    async def process_payment(self, payment_data: dict) -> dict:
        """Process payment"""

    async def confirm_booking(self, booking_id: str) -> dict:
        """Confirm booking"""

    async def get_booking_status(self, booking_id: str) -> dict:
        """Get booking status"""

    async def get_destination_info(self, destination_id: str) -> dict:
        """Get destination details"""

    async def get_reviews(self, destination_id: str) -> dict:
        """Get destination reviews"""

    # Private retry logic
    async def _call_with_retry(self, method: str, endpoint: str, **kwargs) -> dict:
        """Call Java API with exponential backoff retry"""
```

### Error Handling
- Retry on transient errors (5xx, network timeout): max 3 attempts with exponential backoff (1s, 2s, 4s)
- Fail immediately on client errors (4xx) with clear error message
- Log all API calls (request, response, duration) for debugging
- Translate Java API errors to user-friendly messages

## Risks / Trade-offs

| Risk | Impact | Mitigation |
|------|--------|-----------|
| Agent ↔ Java API communication failures | High (blocked user experience) | Implement retry mechanism, add timeout controls, provide clear error messages |
| Java API performance bottlenecks | Medium (slow responses) | Add caching at Agent layer, async processing, optimize Java API queries |
| Data format inconsistencies | Medium (integration bugs) | Define strict JSON schemas, validate responses, comprehensive API documentation |
| Increased latency due to additional layer | Low (acceptable for travel domain) | Optimize Agent logic, use connection pooling, cache where appropriate |
| State management complexity | Medium (conversation context) | Implement session management in Agent, clear session lifecycle |

### Trade-off: Additional Network Hop
Adding Agent as a middle layer adds one network hop between UI and Java API.

**Justification:**
- Agent provides significant value (intelligent orchestration, natural language understanding)
- Latency acceptable for travel planning domain (users expect some wait time for complex queries)
- Enables sophisticated workflows impossible with direct UI → API communication
- Benefits (better UX, cleaner architecture) outweigh minor latency cost

### Trade-off: Complexity vs. Flexibility
Three-tier architecture adds complexity compared to current simpler structure.

**Justification:**
- Complexity is necessary to achieve desired separation of concerns
- Each layer has clear, single responsibility
- Benefits (maintainability, scalability, deployability) significantly outweigh added complexity
- Complexity is well-contained within clear boundaries

## Migration Plan

### Phase 1: Foundation (Week 1-2)
**Goal:** End-to-end flow for one complete use case (flight search)

**Steps:**
1. Set up JavaAPIClient library structure in Agent project
2. Implement search_flights() method with retry logic
3. Refactor SearchAgent.search_flights skill to use JavaAPIClient
4. Add OpenAPI documentation to Java API /flights/search endpoint
5. Modify React UI to send messages to Agent instead of calling Java API directly
6. Implement WebSocket communication between React and Agent
7. Test complete flow: User → React → Agent → Java API → Agent → React

**Rollback:** Keep old React → Java API code in separate branch, can revert quickly

**Success Criteria:**
- User can search for flights through Agent
- Flight results display correctly in UI
- Error handling works (API failures, timeouts)

### Phase 2: Agent Skill Migration (Week 3-4)
**Goal:** All Agent skills delegate to Java API

**Steps:**
1. Implement remaining JavaAPIClient methods (hotels, attractions, weather, bookings, payments)
2. Refactor SearchAgent: search_hotels, compare_results, filter_by_budget
3. Refactor RecommendationAgent: get_destination_info, get_attractions, get_weather_forecast, get_destination_reviews
4. Refactor BookingAgent: create_booking, process_payment, confirm_booking, get_booking_status
5. Review/optimise InfoCollectionAgent (may keep or refactor)
6. Add comprehensive error handling and logging across all skills
7. Unit tests for JavaAPIClient methods
8. Integration tests for each Agent skill

**Rollback:** Keep original skill implementations in Git history

**Success Criteria:**
- All Agent skills use JavaAPIClient
- Complete travel planning flow works (collect preferences → search → recommend → book)
- Errors handled gracefully with user-friendly messages

### Phase 3: API Standardization & Documentation (Week 5)
**Goal:** Java API fully documented and standardized

**Steps:**
1. Audit all Java API endpoints for RESTful compliance
2. Standardize response format across all endpoints
3. Add OpenAPI/Swagger annotations to all endpoints
4. Generate OpenAPI specification document
5. Set up Swagger UI for interactive API exploration
6. Implement consistent error response format
7. Add API authentication mechanism (optional, based on security requirements)

**Rollback:** Keep existing endpoints in place, documentation is additive

**Success Criteria:**
- All endpoints documented in OpenAPI spec
- Swagger UI accessible and accurate
- Response format consistent across all endpoints

### Phase 4: Containerization & Cloud Prep (Week 6-7)
**Goal:** Services containerized and ready for cloud deployment

**Steps:**
1. Create Dockerfile for Agent service (Python runtime, dependencies)
2. Create Dockerfile for Java API service (Java runtime, Spring Boot executable)
3. Create docker-compose.yml for local development
4. Extract configuration to environment variables
5. Create configuration documentation (env vars, config files)
6. Design cloud deployment architecture (services, networking, storage)
7. Create infrastructure-as-code (CloudFormation/Terraform templates)
8. Write deployment documentation

**Rollback:** Docker files are additive, don't affect existing development workflow

**Success Criteria:**
- Services run successfully in containers
- docker-compose works for local development
- Infrastructure code ready for cloud deployment

### Phase 5: Testing & Optimization (Week 8)
**Goal:** System tested, optimized, and production-ready

**Steps:**
1. End-to-end testing of complete travel planning workflows
2. Performance testing and optimization
3. Security review (input validation, API security)
4. Load testing for Agent and Java API
5. Optimize slow operations (add caching, connection pooling)
6. Fix bugs and edge cases discovered during testing
7. Update documentation based on lessons learned

**Rollback:** N/A - this is the final polish phase

**Success Criteria:**
- All tests pass
- Performance meets requirements (response times < 3s for 95% of requests)
- Security review complete with no critical issues
- System ready for production deployment

## Open Questions

1. **Authentication:** Should we implement authentication between layers? (e.g., Agent authenticates to Java API with API key)
   - Status: Deferred to Phase 3 or later if needed
   - Rationale: Not required for initial functionality, can add later

2. **Session Management:** How should Agent maintain conversation state across multiple WebSocket messages?
   - Status: Use in-memory session storage initially, consider Redis later for scaling
   - Rationale: Simple approach sufficient for initial deployment

3. **Caching Strategy:** What should be cached at Agent layer to reduce Java API load?
   - Status: Cache flight search results (short TTL, 5-10 min), destination info (longer TTL, 1 hour)
   - Rationale: Reduce API calls for frequently accessed data

4. **WebSocket vs HTTP Long-Polling:** WebSocket is preferred, but do we need fallback for environments that don't support WebSocket?
   - Status: WebSocket only initially, add HTTP fallback if issues arise
   - Rationale: WebSocket is widely supported, unnecessary complexity to add fallback prematurely

5. **Streaming Response Format:** Should Agent send individual thought steps or just final result?
   - Status: Send key steps (reasoning, tool calls, final result) for better UX
   - Rationale: Users appreciate seeing Agent's thought process

6. **Monitoring & Observability:** What metrics should be collected?
   - Status: Track response times, error rates, API call counts per endpoint, conversation length
   - Rationale: Essential for performance optimization and issue detection

7. **Rollback Strategy:** If migration fails, what's the rollback plan?
   - Status: Maintain feature flag or separate deployment to quickly revert to old architecture
   - Rationale: Migration is significant, need safe rollback path

## Implementation Notes

### Agent Development Guidelines
- Keep Agent skills thin - delegate business logic to Java API
- Focus on conversation management, intent parsing, workflow orchestration
- Use LangGraph's state management for conversation context
- Log all decisions and tool calls for debugging
- Provide clear, conversational responses to users

### Java API Development Guidelines
- Keep endpoints focused on data CRUD and simple business rules
- Document all endpoints with OpenAPI annotations
- Validate all inputs (schema validation)
- Return consistent error format
- Version API if breaking changes needed (e.g., /api/v1/...)

### React UI Development Guidelines
- UI should be a thin client - send user messages to Agent, display responses
- Handle WebSocket connection and reconnection
- Display streaming responses progressively
- Handle structured data (e.g., flight results) with appropriate UI components
- Don't implement business logic in frontend

### Testing Strategy
- Unit tests: JavaAPIClient methods, individual Agent skills
- Integration tests: Agent ↔ Java API communication, React ↔ Agent WebSocket
- End-to-end tests: Complete user workflows (search, recommend, book)
- Performance tests: Response times, concurrent users
- Security tests: Input validation, API security

## Success Metrics

### Architecture Metrics
- Number of cross-layer dependencies: Minimal (only defined communication paths)
- API calls from Agent to Java API: All business data operations
- Direct API calls from React to Java API: Zero (after refactor complete)

### Quality Metrics
- Unit test coverage: > 80% for JavaAPIClient and Agent skills
- Integration test coverage: All critical paths tested
- API documentation completeness: 100% of endpoints documented
- Code quality: Pass linting, type checking

### Performance Metrics
- Average response time: < 3 seconds for 95% of requests
- P99 response time: < 10 seconds
- Agent ↔ Java API error rate: < 1%
- WebSocket message delivery: > 99.9% success

### User Experience Metrics
- Task completion rate: > 90% for travel planning flows
- User satisfaction: Qualitative assessment post-launch
- System uptime: > 99%

## Post-Deployment Considerations

### Monitoring
- Set up monitoring for Agent service (response times, error rates, conversation metrics)
- Set up monitoring for Java API (endpoint latency, error rates, request volume)
- Set up alerts for high error rates or slow response times
- Monitor WebSocket connections and message throughput

### Maintenance
- Regular review of Agent logs for improving conversation flows
- Periodic review of Java API performance, optimize slow endpoints
- Update API documentation when endpoints change
- Keep dependencies up to date (Python packages, Java libraries)

### Future Enhancements
- Add authentication/authorization between layers
- Implement session persistence (Redis) for scaling
- Add analytics for tracking user behavior and improving Agent responses
- Implement rate limiting for Java API
- Add caching layer (Redis) for frequently accessed data
- Consider message queue for async operations (if needed)
