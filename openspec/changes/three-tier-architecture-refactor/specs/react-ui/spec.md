## MODIFIED Requirements

### Requirement: React UI Integration with Agent
The React frontend SHALL communicate exclusively with the Agent layer via WebSocket protocol for all user interactions. The UI SHALL NOT make direct HTTP calls to the Java API.

#### Scenario: User sends message to Agent
- **WHEN** a user types a message in the chat interface
- **THEN** the React UI sends the message to Agent via WebSocket
- **AND** the message includes user_id, message_id, content, and session_context

#### Scenario: Agent streams response to UI
- **WHEN** Agent sends a streaming response via WebSocket
- **THEN** the React UI updates the interface progressively as chunks arrive
- **AND** the UI displays "thinking" state, step-by-step reasoning, and final results

#### Scenario: WebSocket connection established
- **WHEN** the React application initializes
- **THEN** it establishes a WebSocket connection to Agent at ws://localhost:8000
- **AND** the connection is automatically maintained with reconnection logic

#### Scenario: WebSocket connection failure
- **WHEN** the WebSocket connection fails or drops
- **THEN** the React UI attempts reconnection with exponential backoff
- **AND** the UI displays a connection status indicator to the user

#### Scenario: Agent returns structured data
- **WHEN** Agent sends structured data (e.g., flight search results) in the response
- **THEN** the React UI parses and displays the data using appropriate UI components
- **AND** the data format matches the agreed JSON schema between Agent and UI

### Requirement: React UI Response Handling
The React UI SHALL handle different response types from Agent (thinking, response, error, complete) and update the UI accordingly.

#### Scenario: Display Agent thinking state
- **WHEN** Agent sends a message with type="thinking"
- **THEN** the UI displays a loading or thinking indicator
- **AND** the UI shows Agent's current reasoning step if provided

#### Scenario: Display Agent response
- **WHEN** Agent sends a message with type="response"
- **THEN** the UI appends the content to the conversation display
- **AND** the UI scrolls to show the latest message

#### Scenario: Display Agent error
- **WHEN** Agent sends a message with type="error"
- **THEN** the UI displays an error message to the user
- **AND** the error message is user-friendly and actionable

#### Scenario: Agent marks completion
- **WHEN** Agent sends a message with type="complete"
- **THEN** the UI updates the state to indicate the current task is complete
- **AND** the UI displays appropriate completion indicators

### Requirement: React UI State Management
The React UI SHALL maintain conversation state and session context to support multi-step interactions with the Agent.

#### Scenario: Maintain conversation history
- **WHEN** multiple messages are exchanged between user and Agent
- **THEN** the UI maintains a complete conversation history
- **AND** the history is displayed to the user
- **AND** the history is sent to Agent on reconnection for context restoration

#### Scenario: Store user preferences
- **WHEN** the Agent collects user preferences (budget, dates, destinations)
- **THEN** the UI stores these preferences in session context
- **AND** the preferences are included in subsequent messages to Agent
- **AND** the UI can display collected preferences to the user

#### Scenario: Handle session expiration
- **WHEN** the WebSocket session expires after inactivity
- **THEN** the UI displays a session expired message
- **AND** the UI offers to start a new conversation
- **AND** the UI clears temporary state data

### Requirement: React UI Multimedia Support
The React UI SHALL support multiple media types in the conversation with the Agent.

#### Scenario: Send text message
- **WHEN** user types text in the input field
- **THEN** the UI sends the text message to Agent
- **AND** the UI displays the sent message in the conversation

#### Scenario: Display text response
- **WHEN** Agent responds with text content
- **THEN** the UI displays the text with appropriate formatting (markdown support)
- **AND** the UI renders links and special formatting correctly

#### Scenario: Display image in response
- **WHEN** Agent includes an image URL in the response
- **THEN** the UI displays the image inline in the conversation
- **AND** the image is loaded and displayed appropriately

#### Scenario: Display structured data (flights, hotels)
- **WHEN** Agent returns structured data like flight or hotel results
- **THEN** the UI displays the data in a card or table format
- **AND** the UI supports sorting and filtering of the results
- **AND** the UI handles empty result sets gracefully

## REMOVED Requirements

### Requirement: Direct Java API Communication
**Reason**: To enforce strict three-tier architecture, React UI must only communicate through Agent layer, not directly with Java API.
**Migration**: All existing direct Java API calls in React UI have been refactored to use WebSocket communication with Agent. The Agent handles all data retrieval and business logic via Java API.
