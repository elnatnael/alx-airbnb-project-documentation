1. User Authentication
API Endpoints:

POST /auth/register: User registration
	input:
		{
  "first_name": "string",
  "last_name": "string",
  "email": "string (unique)",
  "password": "string",
  "phone_number": "string (optional)",
  "role": "enum('guest', 'host', 'admin')"
}
	Output:
		{
  "message": "User registered successfully",
  "user_id": "UUID",
  "role": "guest"
}

Validation:
Email: Must be a valid email and unique.
Password: Should be at least 8 characters long.
Role: Must be one of 'guest', 'host', or 'admin'.

POST /auth/login: User login
Input:
	{
  "email": "string",
  "password": "string"
}
Output:
	{
  "message": "Login successful",
  "token": "JWT token"
}

Validation:
Email: Must exist in the system.
Password: Must match the stored password hash.
POST /auth/logout: User logout (invalidate token)
Input: No input required
Output:
	{
  "message": "Logout successful"
}
Validation: Token in the header must be valid.


Performance Criteria:

The registration process should take less than 2 seconds for a new user.
The login process should take 1 second for validating credentials and generating a JWT token.
JWT tokens should have a lifetime of 30 minutes and be refreshed via a refresh token.


2. Property Management
API Endpoints:

POST /properties: Create a new property listing
Input:
	{
  "host_id": "UUID (referencing User table)",
  "name": "string",
  "description": "string",
  "location": "string",
  "price_per_night": "decimal",
  "availability_dates": ["start_date", "end_date"],
  "amenities": ["Wi-Fi", "Pool", "Pet-friendly"]
}

Output:
	{
  "message": "Property created successfully",
  "property_id": "UUID"
}

Validation:
Host_id: Must be a valid UUID that references a user in the User table with the role host.
Price_per_night: Must be a positive decimal value.
Availability Dates: Ensure no overlap with existing bookings for the property.

PUT /properties/{property_id}: Update property listing
Input:
	{
  "name": "string",
  "description": "string",
  "location": "string",
  "price_per_night": "decimal",
  "availability_dates": ["start_date", "end_date"],
  "amenities": ["Wi-Fi", "Pool", "Pet-friendly"]
}
Output:
	{
  "message": "Property updated successfully"
}

Validation: Same as POST /properties, with additional check to ensure that only the host of the property can update it.
DELETE /properties/{property_id}: Delete a property listing
Input: No input required
Output:
	{
  "message": "Property deleted successfully"
}

Validation: Property must belong to the authenticated host, and there should be no active bookings for the property.
Performance Criteria:

Property creation: The system should handle property creation in under 2 seconds.
Property update: Property updates should be completed within 1 second.
Property deletion: Property deletion should occur in under 2 seconds, ensuring no active bookings are linked.

3. Booking System
API Endpoints:

POST /bookings: Create a new booking
Input:
	{
  "property_id": "UUID (referencing Property table)",
  "user_id": "UUID (referencing User table)",
  "start_date": "date",
  "end_date": "date",
  "total_price": "decimal"
}

Output:
	{
  "message": "Booking created successfully",
  "booking_id": "UUID"
}
Validation:
Property availability: Ensure that the property is available for the given date range.
Guest capacity: Ensure the booking does not exceed the property’s guest capacity.
Price: Ensure total_price is calculated correctly based on price per night and duration.
PUT /bookings/{booking_id}/cancel: Cancel a booking
Input: No input required
Output:
	{
  "message": "Booking canceled successfully"
}

Validation: Ensure cancellation is allowed based on the property's cancellation policy.
GET /bookings/{user_id}: List all bookings for a user (both guests and hosts)
Input: No input required
Output:
	[
  {
    "booking_id": "UUID",
    "property_name": "string",
    "start_date": "date",
    "end_date": "date",
    "status": "enum('pending', 'confirmed', 'canceled')",
    "total_price": "decimal"
  }
]

Validation: Ensure that the user has permission to view the bookings (guest or host).
Performance Criteria:

Booking creation: The system should complete the booking creation process within 2 seconds.
Booking cancellation: The booking cancellation process should take less than 2 seconds.
Booking retrieval: Retrieving a user’s bookings should be completed within 1 second.

=========================================================

General Validation Rules:
Ensure JWT is valid for every request that requires authentication.
Use rate-limiting to avoid excessive requests (e.g., limit login attempts).
Input data should be sanitized to prevent SQL injection and XSS attacks.


Conclusion
The above backend specifications include:

API Endpoints: To handle user registration, property management, booking creation, and cancellations.
Input/Output Specifications: Define the structure of data exchanged between the client and the backend.
Validation Rules: Ensure proper input data validation for security and correctness.
Performance Criteria: Ensure the system performs efficiently and scales well with increasing traffic.
