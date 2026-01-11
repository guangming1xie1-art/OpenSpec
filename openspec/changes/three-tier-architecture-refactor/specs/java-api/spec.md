## ADDED Requirements

### Requirement: Standardized API Response Format
All Java API endpoints SHALL return responses in a standardized JSON format with success status, data, and optional error details.

#### Scenario: Successful response
- **WHEN** an API endpoint successfully processes a request
- **THEN** the response includes "success": true
- **AND** the response includes a "data" object with the result
- **AND** the response does not include an "error" field

#### Scenario: Error response
- **WHEN** an API endpoint encounters an error
- **THEN** the response includes "success": false
- **AND** the response includes an "error" object with code and message
- **AND** the response does not include a "data" field
- **AND** the error code is a standard code (VALIDATION_ERROR, NOT_FOUND, SERVICE_UNAVAILABLE)

#### Scenario: Validation error response
- **WHEN** an API endpoint receives invalid input
- **THEN** the HTTP status is 422 (Unprocessable Entity)
- **AND** the response success is false
- **AND** the error code is "VALIDATION_ERROR"
- **AND** the error message describes which fields are invalid

#### Scenario: Not found response
- **WHEN** an API endpoint cannot find a requested resource
- **THEN** the HTTP status is 404 (Not Found)
- **AND** the response success is false
- **AND** the error code is "NOT_FOUND"
- **AND** the error message describes what resource was not found

### Requirement: Flight Search Endpoint
The Java API SHALL provide a RESTful endpoint for searching flights.

#### Scenario: Search flights with valid parameters
- **WHEN** a GET request is made to /api/flights/search with origin, destination, date, and passengers
- **THEN** the API returns HTTP 200 status
- **AND** the response includes success: true
- **AND** the data includes a list of flights with details (flight number, airline, departure time, arrival time, price, duration)
- **AND** flights are sorted by price ascending by default

#### Scenario: Search flights with invalid date format
- **WHEN** a GET request is made to /api/flights/search with an invalid date format
- **THEN** the API returns HTTP 422 status
- **AND** the response includes success: false
- **AND** the error code is "VALIDATION_ERROR"
- **AND** the error message indicates the date format issue

#### Scenario: Search flights returns no results
- **WHEN** a GET request is made to /api/flights/search for a route with no available flights
- **THEN** the API returns HTTP 200 status
- **AND** the response includes success: true
- **AND** the data includes an empty flights array
- **AND** the response includes a message indicating no flights were found

### Requirement: Hotel Search Endpoint
The Java API SHALL provide a RESTful endpoint for searching hotels.

#### Scenario: Search hotels with valid parameters
- **WHEN** a GET request is made to /api/hotels/search with destination, check_in, check_out, and guests
- **THEN** the API returns HTTP 200 status
- **AND** the response includes success: true
- **AND** the data includes a list of hotels with details (name, address, rating, price per night, amenities, images)
- **AND** hotels are sorted by rating descending by default

#### Scenario: Search hotels with invalid date range
- **WHEN** a GET request is made to /api/hotels/search with check_in date after check_out date
- **THEN** the API returns HTTP 422 status
- **AND** the response includes success: false
- **AND** the error code is "VALIDATION_ERROR"
- **AND** the error message indicates the date range is invalid

#### Scenario: Search hotels filters by guest count
- **WHEN** a GET request is made to /api/hotels/search with guests parameter
- **THEN** the API returns only hotels that can accommodate the specified number of guests
- **AND** each hotel result includes the maximum capacity

### Requirement: Attractions Endpoint
The Java API SHALL provide a RESTful endpoint for retrieving attractions by destination.

#### Scenario: Get attractions for destination
- **WHEN** a GET request is made to /api/attractions?destination={destination_name}
- **THEN** the API returns HTTP 200 status
- **AND** the response includes success: true
- **AND** the data includes a list of attractions with details (name, description, rating, category, images, address)

#### Scenario: Get attractions for unknown destination
- **WHEN** a GET request is made to /api/attractions?destination={unknown_destination}
- **THEN** the API returns HTTP 200 status
- **AND** the response includes success: true
- **AND** the data includes an empty attractions array

### Requirement: Destination Information Endpoint
The Java API SHALL provide a RESTful endpoint for retrieving detailed information about a destination.

#### Scenario: Get destination information by ID
- **WHEN** a GET request is made to /api/destinations/{id} with a valid destination ID
- **THEN** the API returns HTTP 200 status
- **AND** the response includes success: true
- **AND** the data includes destination details (name, description, location, best time to visit, average temperature, currency, language)

#### Scenario: Get destination information for non-existent ID
- **WHEN** a GET request is made to /api/destinations/{id} with a non-existent ID
- **THEN** the API returns HTTP 404 status
- **AND** the response includes success: false
- **AND** the error code is "NOT_FOUND"

### Requirement: Weather Forecast Endpoint
The Java API SHALL provide a RESTful endpoint for retrieving weather forecasts.

#### Scenario: Get weather forecast for destination and date
- **WHEN** a GET request is made to /api/weather?destination={destination}&date={date}
- **THEN** the API returns HTTP 200 status
- **AND** the response includes success: true
- **AND** the data includes weather details (temperature, conditions, humidity, precipitation chance, wind speed)

#### Scenario: Get weather for past date
- **WHEN** a GET request is made to /api/weather with a date in the past
- **THEN** the API returns HTTP 400 status
- **AND** the response includes success: false
- **AND** the error code is "VALIDATION_ERROR"
- **AND** the error message indicates historical weather is not available

### Requirement: Reviews Endpoint
The Java API SHALL provide a RESTful endpoint for retrieving destination reviews.

#### Scenario: Get reviews for destination
- **WHEN** a GET request is made to /api/reviews?destination_id={id}
- **THEN** the API returns HTTP 200 status
- **AND** the response includes success: true
- **AND** the data includes a list of reviews with details (author, rating, text, date, helpful votes)
- **AND** reviews are sorted by most recent first

#### Scenario: Get reviews with rating filter
- **WHEN** a GET request is made to /api/reviews?destination_id={id}&min_rating={rating}
- **THEN** the API returns only reviews with rating >= min_rating
- **AND** the response includes the filtered review list

### Requirement: Booking Creation Endpoint
The Java API SHALL provide a RESTful endpoint for creating bookings.

#### Scenario: Create booking with valid data
- **WHEN** a POST request is made to /api/bookings with valid booking data
- **THEN** the API returns HTTP 201 status
- **AND** the response includes success: true
- **AND** the data includes booking details (booking_id, status: "pending", created_at, booking_data)

#### Scenario: Create booking with invalid data
- **WHEN** a POST request is made to /api/bookings with missing required fields
- **THEN** the API returns HTTP 422 status
- **AND** the response includes success: false
- **AND** the error code is "VALIDATION_ERROR"
- **AND** the error message indicates which fields are missing

### Requirement: Payment Processing Endpoint
The Java API SHALL provide a RESTful endpoint for processing payments.

#### Scenario: Process payment with valid data
- **WHEN** a POST request is made to /api/payments with valid payment data
- **THEN** the API returns HTTP 200 status
- **AND** the response includes success: true
- **AND** the data includes payment details (payment_id, status: "paid", amount, processed_at)

#### Scenario: Process payment with invalid card
- **WHEN** a POST request is made to /api/payments with invalid payment details
- **THEN** the API returns HTTP 422 status
- **AND** the response includes success: false
- **AND** the error code is "VALIDATION_ERROR"
- **AND** the error message indicates the payment details are invalid

#### Scenario: Process payment with insufficient funds
- **WHEN** a POST request is made to /api/payments that fails due to insufficient funds
- **THEN** the API returns HTTP 402 status (Payment Required)
- **AND** the response includes success: false
- **AND** the error code is "PAYMENT_FAILED"
- **AND** the error message indicates insufficient funds

### Requirement: Booking Confirmation Endpoint
The Java API SHALL provide a RESTful endpoint for confirming bookings.

#### Scenario: Confirm booking after successful payment
- **WHEN** a PUT request is made to /api/bookings/{id}/confirm for a booking with status="paid"
- **THEN** the API returns HTTP 200 status
- **AND** the response includes success: true
- **AND** the data includes booking details with status="confirmed"
- **AND** the data includes confirmed_at timestamp

#### Scenario: Confirm booking for non-existent booking
- **WHEN** a PUT request is made to /api/bookings/{id}/confirm for a non-existent booking ID
- **THEN** the API returns HTTP 404 status
- **AND** the response includes success: false
- **AND** the error code is "NOT_FOUND"

#### Scenario: Confirm booking not yet paid
- **WHEN** a PUT request is made to /api/bookings/{id}/confirm for a booking with status="pending"
- **THEN** the API returns HTTP 400 status
- **AND** the response includes success: false
- **AND** the error code is "VALIDATION_ERROR"
- **AND** the error message indicates the booking must be paid first

### Requirement: Booking Status Endpoint
The Java API SHALL provide a RESTful endpoint for retrieving booking status.

#### Scenario: Get booking status by ID
- **WHEN** a GET request is made to /api/bookings/{id} with a valid booking ID
- **THEN** the API returns HTTP 200 status
- **AND** the response includes success: true
- **AND** the data includes booking details (booking_id, status, created_at, updated_at, booking_data, payment_data)

#### Scenario: Get booking status for non-existent ID
- **WHEN** a GET request is made to /api/bookings/{id} with a non-existent booking ID
- **THEN** the API returns HTTP 404 status
- **AND** the response includes success: false
- **AND** the error code is "NOT_FOUND"

### Requirement: OpenAPI Documentation
All Java API endpoints SHALL be documented using OpenAPI/Swagger annotations.

#### Scenario: Endpoint has OpenAPI annotations
- **WHEN** an API endpoint is defined
- **THEN** the endpoint has OpenAPI annotations describing the operation
- **AND** the annotations include summary, description, and tags
- **AND** the annotations include request parameter definitions
- **AND** the annotations include response schemas

#### Scenario: Endpoint documented with examples
- **WHEN** an API endpoint has OpenAPI annotations
- **THEN** the annotations include example requests and responses
- **AND** the examples demonstrate common use cases
- **AND** the examples include error response examples

#### Scenario: Generate OpenAPI specification
- **WHEN** the application is running
- **THEN** the OpenAPI specification is available at /api-docs
- **AND** the specification includes all documented endpoints
- **AND** the specification is valid OpenAPI 3.0 format

### Requirement: Swagger UI
The Java API SHALL provide an interactive Swagger UI for API exploration and testing.

#### Scenario: Access Swagger UI
- **WHEN** a GET request is made to /swagger-ui.html
- **THEN** the API returns the Swagger UI interface
- **AND** the UI displays all documented endpoints
- **AND** the UI allows interactive testing of endpoints

#### Scenario: Test endpoint via Swagger UI
- **WHEN** a user uses the Swagger UI to test an endpoint
- **THEN** the UI sends the request to the API
- **AND** the UI displays the response
- **AND** the UI handles errors gracefully

### Requirement: Global Error Handler
The Java API SHALL implement a global error handler to ensure consistent error responses across all endpoints.

#### Scenario: Handle unexpected exception
- **WHEN** an unexpected exception occurs during request processing
- **THEN** the global error handler catches the exception
- **AND** the API returns HTTP 500 status
- **AND** the response includes success: false
- **AND** the error code is "INTERNAL_ERROR"
- **AND** the exception is logged with full details

#### Scenario: Handle validation exception
- **WHEN** a validation exception occurs (e.g., @Valid annotation fails)
- **THEN** the global error handler catches the exception
- **AND** the API returns HTTP 422 status
- **AND** the response includes success: false
- **AND** the error code is "VALIDATION_ERROR"
- **AND** the error message includes field-level validation details

#### Scenario: Handle resource not found exception
- **WHEN** a resource is not found (e.g., EntityNotFoundException)
- **THEN** the global error handler catches the exception
- **AND** the API returns HTTP 404 status
- **AND** the response includes success: false
- **AND** the error code is "NOT_FOUND"
- **AND** the error message describes what resource was not found

### Requirement: Input Validation
The Java API SHALL validate all input parameters and request bodies according to defined schemas.

#### Scenario: Validate required fields
- **WHEN** a request is missing a required field
- **THEN** the API returns a validation error
- **AND** the error message lists the missing fields
- **AND** the HTTP status is 422

#### Scenario: Validate field formats
- **WHEN** a request includes a field with invalid format (e.g., invalid date)
- **THEN** the API returns a validation error
- **AND** the error message describes the format requirement
- **AND** the HTTP status is 422

#### Scenario: Validate field constraints
- **WHEN** a request includes a field that violates constraints (e.g., negative value for positive-only field)
- **THEN** the API returns a validation error
- **AND** the error message describes the constraint violation
- **AND** the HTTP status is 422

### Requirement: RESTful API Design
All Java API endpoints SHALL follow RESTful design principles.

#### Scenario: Use appropriate HTTP methods
- **WHEN** defining API endpoints
- **THEN** GET is used for retrieving resources
- **AND** POST is used for creating resources
- **AND** PUT is used for updating resources
- **AND** DELETE is used for deleting resources

#### Scenario: Use resource-based URLs
- **WHEN** defining API endpoint paths
- **THEN** URLs use nouns (resources) not verbs
- **AND** URLs are hierarchical (e.g., /api/bookings/{id}/confirm)
- **AND** URLs are consistent in naming convention (kebab-case)

#### Scenario: Use appropriate HTTP status codes
- **WHEN** returning API responses
- **THEN** 200/201 is used for success
- **AND** 400/422 is used for client errors
- **AND** 404 is used for not found
- **AND** 500 is used for server errors

### Requirement: API Versioning (Future)
The Java API SHALL support versioning to enable breaking changes without breaking existing clients.

#### Scenario: Current version in URL
- **WHEN** endpoints are defined
- **THEN** the current version is included in the URL path (e.g., /api/v1/flights/search)
- **AND** a redirect from /api/ to /api/v1/ may be provided for convenience

#### Scenario: Future version support
- **WHEN** breaking changes are needed
- **THEN** a new version path is created (e.g., /api/v2/flights/search)
- **AND** the old version (/api/v1/) continues to work
- **AND** clients can upgrade to the new version at their own pace

## MODIFIED Requirements

### Requirement: API Security and Authentication
The Java API SHALL support optional authentication and authorization to secure API access from the Agent layer when configured.

#### Scenario: Agent authenticates with API key when enabled
- **WHEN** the Agent makes a request to the Java API and authentication is enabled
- **THEN** the Agent includes an API key in the request header (e.g., X-API-Key)
- **AND** the Java API validates the API key
- **AND** the Java API returns 401 Unauthorized if the API key is invalid or missing

#### Scenario: Rate limiting per API key when enabled
- **WHEN** requests are made with an API key and rate limiting is enabled
- **THEN** the Java API enforces rate limits per API key
- **AND** the Java API returns 429 Too Many Requests if limits are exceeded
- **AND** the response includes rate limit headers (X-RateLimit-Limit, X-RateLimit-Remaining)

#### Scenario: Requests without authentication when disabled
- **WHEN** authentication is not configured
- **THEN** the API accepts requests without authentication
- **AND** the API documentation indicates the authentication configuration status
- **AND** the API is designed to easily enable authentication later

### Requirement: API Monitoring and Metrics
The Java API SHALL collect metrics and provide monitoring endpoints for operational visibility.

#### Scenario: Collect request metrics
- **WHEN** requests are made to the API
- **THEN** the API collects metrics for request count, response time, and error rate
- **AND** metrics are aggregated per endpoint
- **AND** metrics are available for monitoring systems

#### Scenario: Health check endpoint
- **WHEN** a GET request is made to /actuator/health
- **THEN** the API returns a health status
- **AND** the response includes status (UP or DOWN)
- **AND** the response includes database connectivity status

#### Scenario: Metrics endpoint
- **WHEN** a GET request is made to /actuator/metrics
- **THEN** the API returns available metrics
- **AND** the response lists all metric names
- **AND** individual metrics can be queried (e.g., /actuator/metrics/http.server.requests)
