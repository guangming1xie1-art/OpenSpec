# Three-Tier Architecture Refactor

This OpenSpec change document outlines the complete refactoring of the lvyou travel assistant system from its current architecture to a clean three-tier separation: **React UI → Agent (Python) → Java API**.

## Overview

### Current State
- Three separate projects with unclear boundaries
- React UI makes direct calls to Java API
- Agent skills are self-contained without Java API integration
- Tightly coupled architecture, difficult to scale independently

### Target State
```
React UI (Gradio) → Agent (LangGraph) → Java API (Spring Cloud)
       ↓                      ↓                    ↓
  User Interface      Intelligent Orchestrator   Data Services
```

## What This Change Includes

### 1. Proposal (`proposal.md`)
- **Why**: Problems with current architecture and need for separation of concerns
- **What Changes**: High-level description of the refactor
- **Impact**: Affected specs and code areas

### 2. Design (`design.md`)
Comprehensive technical design document covering:
- Architecture decisions and rationale
- Communication protocols (WebSocket for React↔Agent, HTTP REST for Agent↔Java)
- Data flow examples with complete scenarios
- JavaAPIClient tool library design
- Error handling and retry mechanisms
- Migration plan (5 phases, 8 weeks)
- Risks, trade-offs, and mitigation strategies

### 3. Tasks (`tasks.md`)
Detailed implementation checklist with 48 main tasks organized by phase:
- **Phase 1** (Weeks 1-2): Foundation - JavaAPIClient, flight search refactor, WebSocket integration
- **Phase 2** (Weeks 3-4): Agent Skill Migration - All SearchAgent, RecommendationAgent, BookingAgent skills
- **Phase 3** (Week 5): API Standardization - OpenAPI documentation, error handling
- **Phase 4** (Weeks 6-7): Containerization - Docker files, docker-compose, infrastructure as code
- **Phase 5** (Week 8): Testing & Optimization - E2E tests, performance optimization, security review

### 4. Specification Deltas (`specs/`)

#### React UI (`specs/react-ui/spec.md`)
- MODIFIED: React UI integration with Agent via WebSocket
- MODIFIED: Response handling for thinking/response/error/complete message types
- MODIFIED: State management for conversation history and user preferences
- ADDED: Multimedia support (text, images, structured data)
- REMOVED: Direct Java API communication

#### Agent Layer (`specs/agent-layer/spec.md`)
- ADDED: JavaAPIClient tool library with HTTP client and retry logic
- ADDED: Flight and hotel search methods
- ADDED: Destination, attractions, and weather retrieval methods
- ADDED: Booking and payment processing methods
- ADDED: Complete refactor of SearchAgent, RecommendationAgent, BookingAgent
- ADDED: Error handling and logging across all Agent skills
- MODIFIED: WebSocket server for React UI communication

#### Java API (`specs/java-api/spec.md`)
- ADDED: Standardized JSON response format (success, data, error)
- ADDED: Flight search endpoint
- ADDED: Hotel search endpoint
- ADDED: Attractions and destination information endpoints
- ADDED: Weather forecast endpoint
- ADDED: Reviews endpoint
- ADDED: Booking creation, confirmation, and status endpoints
- ADDED: Payment processing endpoint
- ADDED: OpenAPI documentation for all endpoints
- ADDED: Swagger UI for interactive API testing
- ADDED: Global error handler
- ADDED: Input validation
- ADDED: RESTful API design principles
- MODIFIED: API Security and Authentication (optional)

## Key Technical Decisions

1. **Strict Layer Separation**: React UI only talks to Agent, Agent only talks to Java API
2. **Agent as Orchestrator**: Agent handles all business logic, workflow coordination, and conversation state
3. **HTTP REST for Agent↔Java API**: Language-agnostic, well-supported, easy to debug
4. **WebSocket for React↔Agent**: Supports streaming responses and bidirectional communication
5. **Error Handling with Retry**: Exponential backoff (1s, 2s, 4s) for transient failures
6. **Standardized Response Format**: JSON with success, data, and error structure

## Implementation Timeline

| Phase | Duration | Key Deliverables |
|-------|----------|------------------|
| 1 | 2 weeks | End-to-end flight search flow |
| 2 | 2 weeks | All Agent skills migrated to Java API |
| 3 | 1 week | Complete API documentation and standardization |
| 4 | 2 weeks | Docker containerization and cloud deployment prep |
| 5 | 1 week | Testing, optimization, and production readiness |

**Total Estimated Effort**: 8 weeks

## Validation

This change has been validated using OpenSpec's strict validation:

```bash
npx openspec validate three-tier-architecture-refactor --strict
# ✅ Change is valid
```

## Getting Started

1. **Review the Proposal**: Read `proposal.md` to understand the high-level changes
2. **Review the Design**: Read `design.md` to understand technical decisions
3. **Review the Tasks**: Use `tasks.md` as your implementation checklist
4. **Reference Spec Deltas**: Use the spec files in `specs/` to understand requirements for each layer
5. **Start Phase 1**: Begin with Phase 1 tasks to establish the foundation

## Success Criteria

- ✅ React UI only communicates with Agent via WebSocket
- ✅ Agent only communicates with Java API via HTTP REST
- ✅ All Agent skills delegate to Java API endpoints
- ✅ Complete travel planning flow works (preferences → search → recommend → book)
- ✅ All Java API endpoints documented with OpenAPI
- ✅ Services containerized with Docker
- ✅ Ready for cloud deployment

## Related Documentation

- OpenSpec AGENTS.md: `/openspec/AGENTS.md` - General OpenSpec workflow guidelines
- Project.md: `/openspec/project.md` - Project conventions and technology stack

## Questions?

Refer to the Open Questions section in `design.md` for areas that may need further clarification during implementation.
