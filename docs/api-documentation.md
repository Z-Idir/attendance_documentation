# Overview
The api is a centralized and contained system for managing the needs of a single client (business) , in the future we will be working towards making it into a full Saas where having a new user for the system does't require the deployment of a new instance of the server to achieve it rather 
We have in this system two entities, [employees](#Employee%20management) and [attendances](#Attendance%20management) .

# Format specification
All requests that have a body require a **content-type** of **multipart/form-data** , and the response of all request (with or without a body) has content type **application/json** , ==if an endpoint doesn't follow this it will be specified==.

# Employee management
For the employees we have a list of endpoints [here](./endpoints#employees), and here we will provide a certain workflow and details about the usage of these endpoints : 
## Employee creation :
The first functionality we will see is that of employee creation :
- endpoint : `POST /attendance` 
- request body :
```
name : String
salary : Float
```
- response : 


# Attendance management
