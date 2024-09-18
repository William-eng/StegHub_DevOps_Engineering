
## What is a RESTful API?
A RESTful API (Representational State Transfer Application Programming Interface) is a set of rules and conventions for building 
and interacting with web services. It leverages standard HTTP methods and status codes to enable communication between a client 
and a server over the web. RESTful APIs are designed to be stateless and scalable, making them a popular choice for web development.
REST technology is generally preferred to the more robust Simple Object Access Protocol (SOAP) technology because REST uses less bandwidth, 
simple and flexible making it more suitable for internet usage. 
It’s used to fetch or give some information from a web service. All communication done via REST API uses only HTTP request.

## Working Process: 
A request is sent from client to server in the form of a web URL as HTTP GET or POST or PUT or DELETE request.
After that, a response comes back from the server in the form of a resource which can be anything like HTML, XML, Image, 
or JSON. But now JSON is the most popular format being used in Web Services.

## Key Concepts of RESTful APIs
### Resources: 
In REST, everything is considered a resource, which can be anything from a user profile to a list of products. 
Each resource is identified by a unique URL.
### Example URL: https://api.example.com/users/123

## HTTP Methods:
### A. RESTful APIs use standard HTTP methods to perform operations on resources. The most common methods are:

1. GET: Retrieve data from the server.
2. POST: Create a new resource on the server.
3. PUT: Update an existing resource on the server.
4. DELETE: Remove a resource from the server.
5. Statelessness: Each request from the client to the server must contain all the information needed to understand and process the request. The server does not store any state information between requests.

### B. JSON or XML: RESTful APIs typically use JSON (JavaScript Object Notation) or XML (eXtensible Markup Language) to format the data exchanged between client and server. JSON is more commonly used due to its simplicity and ease of use.

### C. Endpoints: The URL structure of a RESTful API is called an endpoint. Each endpoint corresponds to a specific resource or collection of resources.

Example Endpoint: https://api.example.com/products
### D. HTTP Status Codes: RESTful APIs use HTTP status codes to indicate the result of an API request.

- 200 OK: The request was successful.
- 201 Created: A new resource was successfully created.
- 204 No Content: The request was successful, but there is no content to return.
- 400 Bad Request: The request could not be understood or was missing required parameters.
- 404 Not Found: The requested resource could not be found.
- 500 Internal Server Error: An error occurred on the server side.

## Example of a RESTful API Interaction
### GET Request: Retrieve a list of users.

Request: GET https://api.example.com/users

      [
      {"id": 1, "name": "Alice"},
      {"id": 2, "name": "Bob"}
      ]
### POST Request: Create a new user.

Request: POST https://api.example.com/users

      {"name": "Charlie"}
Response:

    {"id": 3, "name": "Charlie"}
### PUT Request: Update a user's information.

Request: PUT https://api.example.com/users/1


    {"name": "Alice Smith"}
Response:

    {"id": 1, "name": "Alice Smith"}


