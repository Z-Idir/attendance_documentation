# Overview
The api is a centralized and contained system for managing the needs of a single client (business) , in the future we will be working towards making it into a full Saas where having a new user for the system requires that user to create an account rather than requiring the deployment of a new instance of the server to achieve it.
We have in this system two entities, [employees](#Employee%20management) and [attendances](#Attendance%20management) .

# Notes
- Notice that the update endpoints use `POST` method instead of `PATCH` or `PUT` and that is because of a 'flaw' in **php** as it is unable to automatically parse incoming request body when the method is other than `POST` , **Laravel** framework implemented a workaround this by using a `POST` request and using a *magic string* within the request body that specifies that it actually is a `PUT` request, click [here](https://bugs.php.net/bug.php?id=55815) for more info.
# Format specification
All requests that have a body require a **content-type** of **multipart/form-data** , and the response of all requests (with or without a body) have content type **application/json** , ==if an endpoint doesn't follow this format then it will be specified==.
For the responses that are the same for all endpoints such as **server errors** `500` will not be specified here but rather in the exhaustive endpoints page [here](./endpoints.md)

## Employee management
For the employees we have a list of endpoints [here](./endpoints#employees), and here we will provide the  workflow and details about the usage of these endpoints : 
### 1. **Create en employee**
The first functionality we will see is that of employee creation :
#### Endpoint : 
```r
POST /employee
```
#### Request body :
```
name : String
salary : Float
```
#### Responses : 
1. **Success** `200 OK`
```JSON 
{

	"success": true,

	"message": "Employee created successfully"

}
```
1. **Validation Error** `400 Bad Request`
2. Server Error `500 Internal Server Error`
### **2. Get an employee by ID**
#### Endpoint :
```r
GET /employee/{id}
```
#### Responses :

1. **Success** `200 OK`
```json
{

    "success": true,

    "data": {

        "id": 1,

        "name": "hydrion",

        "salary": 35000,

        "created_at": null,

        "updated_at": null

    }

}
```
    
2. **Employee not found** ``404 Not Found``
```json
{

    "success": false,

    "message": "Employee not found"

}
```

3. **Server Error** `500 Internal Server Error`
### **3. Get all employees**
#### Endpoint :
```r
GET /employee
```
#### Responses :

1. **Success** `200 OK`
`data` holds an *array* of employees
```json
{

    "success": true,

    "data": [

        {
            "id": 1,
            "name": "hydrion",
            "salary": 35000,
            "created_at": null,
            "updated_at": null
        },
        {
            "id": 2,
            "name": "sarpedon",
            "salary": 787878,
            "created_at": null,
            "updated_at": null
        } ...
			]
}
```
2. **Server Error** `500 Internal Server Error`
### **4. Update an employee**
#### Endpoint : 
```r
POST /employee/{id}
```
#### Request body : 
```
name : String
salary : Float
```
#### Responses : 
1. **Update Successful** `200 OK`
```json
{

    "success": true,

    "data": {

        "id": 1,

        "name": "hydrion",

        "salary": "35000",

        "created_at": null,

        "updated_at": null

    }

}
```
2. **Employee not found** `404  Not Found`
3. **Server Error** `500 Internal Server Error`
4. **Validation Error** `400 Bad Request`
### **5. Delete an employee**
#### Endpoint : 
```r
DELETE /employee/{id}
```
#### Responses : 
1. **Delete Successful** `200 OK`
```json
{
    "success": true,
    "message": "Employee deleted successfully"
}
```
2. **Employee not found** `404  Not Found`
3. **Server Error** `500 Internal Server Error`
## Attendance management
### **1. Create an attendance**
#### Endpoint : 
```r
POST /attendance
```
#### Request body : 
```
date : Date (yyyy-mm-dd)
employee_id : Int
check_in : Time (hh:mm:ss)
```
- date : is the date for which the attendance entry is created for
- employee_id : an integer that references the employee for which the entry is created
- check_in :  the time of clock-in of the employee
#### Responses : 
1. **Success** `200 OK` 
```json
{

    "success": true,

    "data": {

        "id": 31,

        "attendance": {

            "date": "2025-03-02",

            "employee_id": "3",

            "check_in": "08:50:52",

            "id": 31

        }

    }

}
```
the response contains a status field `success` and data which contains the created attendance entry and an `id` which is redundant and will be removed later
2. **Attendance already exists** `409 Conflict`
```json
{
    "success": false,
    "message": "Attendance already exists today",
    "data": {
        "id": 31,
        "attendance": {
            "id": 31,
            "employee_id": 3,
            "date": "2025-03-02",
            "check_in": "08:50:52",
            "check_out": null,
            "break_start": null,
            "break_end": null,
            "last_action": null,
            "created_at": null,
            "updated_at": null
        }
    }
}
```
this response occurs in the case where the creation of an attendance entry is requested while there already is an entry corresponding to that employee and date given in the request's body, we get the attendance that causes the conflict in the response under : `data.attendance`
3. **Validation Error** `400 Bad Request`
4. **Server Error** `500 Internal Server Error`
### **2. Modify an attendance**
#### Endpoint : 
```r
POST /attendance/{id}
```
#### Request body : 
```
field : Enum('check_in','check_out','break_start','break_end')
time : Time (hh:mm:ss)
```
- field : the field which we want to modify 
- time :  the time value to which we want the specified field to be set to
#### Responses : 
1. **Success** `200 OK` 
```json
{

    "success": true,

    "message": "Attendance updated successfully.",

    "data": {

        "id": 22,

        "employee_id": 3,

        "date": "2025-02-24",

        "check_in": "08:50:52",

        "check_out": "17:0:00",

        "break_start": null,

        "break_end": null,

        "last_action": "check_out",

        "created_at": null,

        "updated_at": null

    }

}
```
2. **Attendance not found** `404 Not Found`
3. **Validation Error** `400 Bad Request`
	this could occur due to these reasons :
	- simple validation error : a field contains an invalid type or the `field` value is not one of `['check_in','check_out','break_start','break_end']` 
	- the request tried to set the value of a column to a **chronologically** invalid value, for example we have ``check_in`` set to : `09:02:12` and we try to set `break_start` to : `08:30:12` and this will trigger a bad request response
4. **Server Error** `500 Internal Server Error`
### 3. **Get a month's report**
#### Endpoint : 
this endpoint returns the month's history, worked and office days, plus the payment 
```r
GET /attendance/{emp_id}/month/{yyyy}-{mm}
```
- `emp_id` : the id of the employee for which we want to generate the report for
- `{yyyy}-{mm}` : a formatted string that contains the year and month, ex : 2025-02
#### Responses : 
1. **Success** `200 OK`
```json
{
    "success": true,
    "data": {
        "workedDays": 4,
        "attendances": [
            {
                "id": 13,
                "employee_id": 1,
                "date": "2025-02-01",
                "check_in": "08:30:00",
                "check_out": null,
                "break_start": null,
                "break_end": null,
                "last_action": null,
                "created_at": null,
                "updated_at": null
            },
            {
                "id": 1,
                "employee_id": 1,
                ...
            }
            ...
        ],
        "officeDaysNumber": 20,
        "payement": 7000
    }
}
```
`data` : inside this field we have :
- `workedDays` : the number of days the employee has worked in the given month `{mm}` in the request params.
- `attendances` : a list of all attendance records for the month.
- `officeDaysNumber` : the number of office days for the month, they are calculated by subtracting the number of holidays and weekends from the total days of the month, the holidays' dates are obtained from google calendar
- `payement` : is the total salary of that employee, obtained from the ``workedDays`` and the employee's ``salary`` field
2. **Employee Not Found** `404 Not Found`
3. **Server Error** `500 Internal Server Error`
### 4. **Get the attendance of a certain day**
#### Endpoint : 
this endpoint returns the attendance entry specific to an employee for a specified date
```r
GET /attendance/{emp_id}/day/{yyyy}-{mm}-{dd}
```
- `emp_id` : the id of the employee for which we want to get the attendance
- `{yyyy}-{mm}`-{dd} : a formatted string that contains the date of the attendance, ex : 2025-02-01
#### Responses : 
1. **Success** `200 OK`
```json
{
    "success": true,
    "data": {
        "attendance": {
            "id": 23,
            "employee_id": 1,
            "date": "2025-03-03",
            "check_in": null,
            "check_out": "15:04:16",
            "break_start": "15:04:13",
            "break_end": "15:04:15",
            "last_action": null,
            "created_at": null,
            "updated_at": null
        }
    }
}
```
2. **Employee Not Found** `404 Not Found`
3. **Server Error** `500 Internal Server Error`
### 5. **Get an attendance by id**
#### Endpoint : 
this endpoint returns an attendance entry by its `id`
```r
GET /attendance/{att_id}
```
- `att_id` : the id of the attendance we want to fetch
#### Responses : 
1. **Success** `200 OK`
```json
{
    "success": true,
    "data": {
        "attendance": {
            "id": 21,
            "employee_id": 3,
            "date": "2025-02-28",
            "check_in": "11:01:10",
            "check_out": "11:55:12",
            "break_start": "11:54:47",
            "break_end": "11:55:02",
            "last_action": null,
            "created_at": null,
            "updated_at": null
        }
    }
}
```
2. **Attendance Not Found** `404 Not Found`
3. **Server Error** `500 Internal Server Error`
