## ADDED Requirements

### Requirement: JavaAPIClient Tool Library
The Agent system SHALL provide a JavaAPIClient tool library that handles all HTTP REST communication with the Java API layer.

#### Scenario: Initialize JavaAPIClient with configuration
- **WHEN** JavaAPIClient is instantiated with base_url and timeout parameters
- **THEN** the client is configured with the provided settings
- **AND** the client uses httpx for async HTTP requests
- **AND** the default timeout is 30 seconds if not specified

#### Scenario: Call Java API with retry on transient failure
- **WHEN** a Java API call fails with a 5xx error or timeout
- **THEN** JavaAPIClient retries the request with exponential backoff (1s, 2s, 4s)
- **AND** the client retries up to 3 times before failing
- **AND** the retry attempts are logged

#### Scenario: Call Java API without retry on client error
- **WHEN** a Java API call fails with a 4xx error (client error)
- **THEN** JavaAPIClient does not retry the request
- **AND** the error is immediately propagated to the calling code
- **AND** the error details are logged

#### Scenario: Log all Java API calls
- **WHEN** JavaAPIClient makes a request to the Java API
- **THEN** the client logs the request method, URL, and parameters
- **AND** the client logs the response status and duration
- **AND** the logs include request_id for correlation tracking

### Requirement: JavaAPIClient Flight Search Methods
JavaAPIClient SHALL provide methods for searching flights and hotels via the Java API.

#### Scenario: Search flights successfully
- **WHEN** search_flights(origin, destination, date, passengers) is called with valid parameters
- **THEN** JavaAPIClient makes a GET request to /api/flights/search with the parameters
- **AND** the method returns flight data from the Java API response
- **AND** the response includes flight details (times, prices, airlines)

#### Scenario: Search flights with invalid date format
- **WHEN** search_flights() is called with an invalid date format
- **THEN** JavaAPIClient returns a validation error
- **AND** the error message indicates the date format issue

#### Scenario: Search hotels successfully
- **WHEN** search_hotels(destination, check_in, check_out, guests) is called with valid parameters
- **THEN** JavaAPIClient makes a GET request to /api/hotels/search with the parameters
- **AND** the method returns hotel data from the Java API response
- **AND** the response includes hotel details (name, location, price, amenities)

#### Scenario: Search hotels with invalid date range
- **WHEN** search_hotels() is called with check_in after check_out
- **THEN** JavaAPIClient returns a validation error
- **AND** the error message indicates the date range issue

### Requirement: JavaAPIClient Destination and Attractions Methods
JavaAPIClient SHALL provide methods for retrieving destination information and attractions.

#### Scenario: Get attractions for destination
- **WHEN** get_attractions(destination) is called with a valid destination
- **THEN** JavaAPIClient makes a GET request to /api/attractions with the destination parameter
- **AND** the method returns a list of attractions with details (name, description, rating, images)

#### Scenario: Get destination information
- **WHEN** get_destination_info(destination_id) is called with a valid ID
- **THEN** JavaAPIClient makes a GET request to /api/destinations/{id}
- **AND** the method returns destination details (name, description, location, best time to visit)

#### Scenario: Get destination reviews
- **WHEN** get_reviews(destination_id) is called with a valid ID
- **THEN** JavaAPIClient makes a GET request to /api/reviews for the destination
- **AND** the method returns a list of reviews with details (author, rating, text, date)

### Requirement: JavaAPIClient Weather Methods
JavaAPIClient SHALL provide methods for retrieving weather forecasts.

#### Scenario: Get weather forecast successfully
- **WHEN** get_weather(destination, date) is called with valid parameters
- **THEN** JavaAPIClient makes a GET request to /api/weather with the parameters
- **AND** the method returns weather data (temperature, conditions, precipitation)

#### Scenario: Get weather for past date
- **WHEN** get_weather() is called with a date in the past
- **THEN** JavaAPIClient returns an error indicating historical weather is not available
- **AND** the error message is user-friendly

### Requirement: JavaAPIClient Booking Methods
JavaAPIClient SHALL provide methods for creating, confirming, and querying bookings.

#### Scenario: Create booking successfully
- **WHEN** create_booking(booking_data) is called with valid booking information
- **THEN** JavaAPIClient makes a POST request to /api/bookings with the booking data
- **AND** the method returns booking details including a booking_id
- **AND** the booking status is set to "pending"

#### Scenario: Process payment successfully
- **WHEN** process_payment(payment_data) is called with valid payment information
- **THEN** JavaAPIClient makes a POST request to /api/payments with the payment data
- **AND** the method returns payment confirmation details
- **AND** the payment status is set to "paid"

#### Scenario: Confirm booking after payment
- **WHEN** confirm_booking(booking_id) is called for a paid booking
- **THEN** JavaAPIClient makes a PUT request to /api/bookings/{id}/confirm
- **AND** the method returns the confirmed booking details
- **AND** the booking status is updated to "confirmed"

#### Scenario: Get booking status
- **WHEN** get_booking_status(booking_id) is called with a valid booking ID
- **THEN** JavaAPIClient makes a GET request to /api/bookings/{id}
- **AND** the method returns the current booking status and details

### Requirement: SearchAgent Refactor
SearchAgent SHALL use JavaAPIClient for all flight and hotel search operations instead of direct implementation.

#### Scenario: SearchAgent searches flights
- **WHEN** SearchAgent receives a flight search request from the user
- **THEN** the Agent uses JavaAPIClient.search_flights() to query the Java API
- **AND** the Agent formats and presents the results to the user
- **AND** the Agent handles any errors from JavaAPIClient gracefully

#### Scenario: SearchAgent searches hotels
- **WHEN** SearchAgent receives a hotel search request from the user
- **THEN** the Agent uses JavaAPIClient.search_hotels() to query the Java API
- **AND** the Agent formats and presents the results to the user
- **AND** the Agent handles any errors from JavaAPIClient gracefully

#### Scenario: SearchAgent compares results
- **WHEN** SearchAgent is asked to compare flight or hotel results
- **THEN** the Agent compares the data returned by JavaAPIClient
- **AND** the Agent presents a comparison summary to the user
- **AND** the comparison highlights key differences (price, duration, amenities)

#### Scenario: SearchAgent filters by budget
- **WHEN** SearchAgent is asked to filter results by budget
- **THEN** the Agent filters the data returned by JavaAPIClient based on the budget
- **AND** the Agent presents only results within the budget
- **AND** the Agent informs the user how many results were filtered

### Requirement: RecommendationAgent Refactor
RecommendationAgent SHALL use JavaAPIClient for all destination, attraction, and weather queries.

#### Scenario: RecommendationAgent gets destination info
- **WHEN** RecommendationAgent needs information about a destination
- **THEN** the Agent uses JavaAPIClient.get_destination_info() to query the Java API
- **AND** the Agent incorporates the destination details into recommendations

#### Scenario: RecommendationAgent gets attractions
- **WHEN** RecommendationAgent is providing recommendations for activities
- **THEN** the Agent uses JavaAPIClient.get_attractions() to get attraction data
- **AND** the Agent presents top-rated attractions to the user
- **AND** the Agent considers user preferences when ranking attractions

#### Scenario: RecommendationAgent gets weather forecast
- **WHEN** RecommendationAgent is planning a trip and needs weather information
- **THEN** the Agent uses JavaAPIClient.get_weather() to get the forecast
- **AND** the Agent incorporates weather considerations into recommendations

#### Scenario: RecommendationAgent gets reviews
- **WHEN** RecommendationAgent needs user feedback about a destination
- **THEN** the Agent uses JavaAPIClient.get_reviews() to get review data
- **AND** the Agent summarizes key points from reviews
- **AND** the Agent highlights both positive and negative aspects mentioned

### Requirement: BookingAgent Refactor
BookingAgent SHALL use JavaAPIClient for all booking and payment operations.

#### Scenario: BookingAgent creates booking
- **WHEN** BookingAgent receives a booking request from the user
- **THEN** the Agent uses JavaAPIClient.create_booking() to create the booking in Java API
- **AND** the Agent confirms the booking details with the user before proceeding

#### Scenario: BookingAgent processes payment
- **WHEN** BookingAgent needs to process payment for a booking
- **THEN** the Agent uses JavaAPIClient.process_payment() to handle the payment
- **AND** the Agent informs the user of the payment status

#### Scenario: BookingAgent confirms booking
- **WHEN** BookingAgent receives confirmation that payment is successful
- **THEN** the Agent uses JavaAPIClient.confirm_booking() to confirm the booking
- **AND** the Agent presents the final booking confirmation to the user

#### Scenario: BookingAgent checks booking status
- **WHEN** BookingAgent is asked about the status of a booking
- **THEN** the Agent uses JavaAPIClient.get_booking_status() to retrieve the status
- **AND** the Agent provides a clear status update to the user

### Requirement: Agent Error Handling
All Agent skills SHALL handle errors from JavaAPIClient gracefully and provide user-friendly error messages.

#### Scenario: Handle Java API timeout
- **WHEN** JavaAPIClient times out calling the Java API
- **THEN** the Agent logs the timeout error
- **AND** the Agent informs the user that the service is temporarily unavailable
- **AND** the Agent offers to retry the operation

#### Scenario: Handle Java API service unavailable
- **WHEN** JavaAPIClient receives a 503 Service Unavailable error
- **THEN** the Agent logs the service error
- **AND** the Agent informs the user that the backend service is down
- **AND** the Agent suggests trying again later

#### Scenario: Handle validation error from Java API
- **WHEN** JavaAPIClient receives a 422 validation error from Java API
- **THEN** the Agent parses the error details
- **AND** the Agent provides specific feedback about what was invalid
- **AND** the Agent asks the user to correct the input

#### Scenario: Handle not found error from Java API
- **WHEN** JavaAPIClient receives a 404 not found error
- **THEN** the Agent informs the user that the requested resource was not found
- **AND** the Agent suggests alternative options if available

### Requirement: Agent Logging
All Agent operations SHALL be logged for debugging and monitoring purposes.

#### Scenario: Log Agent decision
- **WHEN** an Agent makes a routing decision (e.g., route to SearchAgent)
- **THEN** the decision is logged with context (user input, chosen skill, reasoning)
- **AND** the log includes timestamp and correlation ID

#### Scenario: Log JavaAPIClient call
- **WHEN** an Agent skill calls a JavaAPIClient method
- **THEN** the method name and parameters are logged
- **AND** the response status and duration are logged
- **AND** any errors are logged with full details

#### Scenario: Log conversation turn
- **WHEN** a complete conversation turn completes (user message + Agent response)
- **THEN** the full exchange is logged
- **AND** the log includes user message content (sanitized) and Agent response
- **AND** the log includes session ID for tracking

### Requirement: Agent State Management
The Agent SHALL maintain conversation state and context across multiple user interactions.

#### Scenario: Store user preferences
- **WHEN** the Agent collects user preferences (budget, dates, destinations, etc.)
- **THEN** the preferences are stored in the session context
- **AND** the preferences are available to all Agent skills for the session
- **AND** the preferences are included in subsequent requests if needed

#### Scenario: Track conversation flow
- **WHEN** the conversation progresses through multiple stages (preferences → search → recommend → book)
- **THEN** the Agent tracks the current stage
- **AND** the Agent maintains state of completed steps
- **AND** the Agent can resume from any stage if interrupted

#### Scenario: Clear session on new conversation
- **WHEN** a user starts a new conversation
- **THEN** the Agent clears the previous session context
- **AND** the Agent starts with a clean state
- **AND** the Agent acknowledges the new conversation start

## MODIFIED Requirements

### Requirement: WebSocket Server for React UI Communication
The Agent SHALL provide a WebSocket server that accepts connections from the React UI and streams responses back.

#### Scenario: Accept WebSocket connection from React UI
- **WHEN** React UI establishes a WebSocket connection to Agent
- **THEN** the Agent accepts the connection
- **AND** the Agent assigns a unique session_id to the connection
- **AND** the Agent acknowledges the connection

#### Scenario: Receive message from React UI
- **WHEN** React UI sends a message via WebSocket
- **THEN** the Agent parses the message (message_id, user_id, content, session_context)
- **AND** the Agent routes the message to the appropriate skill
- **AND** the Agent logs the incoming message

#### Scenario: Stream response to React UI
- **WHEN** an Agent skill generates a response
- **THEN** the Agent streams the response back to React UI via WebSocket
- **AND** the response includes type (thinking/response/error/complete), content, and optional data
- **AND** the Agent sends multiple chunks for long responses

#### Scenario: Handle WebSocket disconnection
- **WHEN** React UI disconnects from the WebSocket
- **THEN** the Agent cleans up the session state
- **AND** the Agent logs the disconnection
- **AND** the Agent is ready to accept new connections
