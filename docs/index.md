# Welcome to the Attendance and Employee Management API
This API serves as the backbone for managing employee attendance and information within our system. Built entirely using pure PHP and powered by an SQL database, this system follows a client-server architecture. The primary focus of this documentation is on the API aspect of the project.

# Architecture and Structure
The project is organized into several distinct layers, each handling specific responsibilities:

- Service Layer: Utilizes the Eloquent ORM to interact with the database, providing clean and efficient data management.
- Model Layer: Represents database entities as PHP objects, leveraging the Eloquent ORM for database operations.
- Controller Layer: Handles incoming HTTP requests and dispatches them to the appropriate services. This layer is responsible for parsing the request and generating appropriate responses.
- Router Layer: Manages routing and URL parameter extraction, calling authorization middleware when necessary to secure endpoints.
# Server and Hosting
The API runs on an Apache HTTP server, which acts as the main request handler. The server leverages Apache’s request call method to efficiently process incoming requests. The project is containerized using Docker, ensuring consistent and isolated environments. The Docker Compose setup includes two key containers:

- Apache HTTP Server: Hosts and serves the API, processing client requests.
- SQL Server: Manages the database that stores employee and attendance data.
# Key Features
Clean and Modular Design: Clearly separated layers for routing, request handling, service logic, and database interaction.
Containerized Environment: Simplifies deployment and maintenance through Docker.
Efficient Data Management: Uses Eloquent ORM to streamline data access and manipulation.
Extensible Architecture: Easily add new features or extend existing functionality.