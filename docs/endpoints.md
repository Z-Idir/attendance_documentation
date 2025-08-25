# Format specification
All requests that have a body require a **content-type** of **multipart/form-data** , and the response of all requests (with or without a body) have content type **application/json** , ==if an endpoint doesn't follow this format then it will be specified==.
# Endpoints
## Employee Endpoints

<table>
    <tr>
        <th>endpoint</th>
        <th>method</th>
        <th>body</th>
        <th>description</th>
    </tr>
    <tr>
        <td>/employee</td>
        <td>GET</td>
        <td>/</td>
        <td>get a full list of employees</td>
    </tr>
    <tr>
        <td>/employee/{id}</td>
        <td>GET</td>
        <td>/</td>
        <td>get an employee by id</td>
    </tr>
    <tr>
        <td>/employee</td>
        <td>POST</td>
        <td>name : String<br>salary : Float</td>
        <td>create an employee</td>
    </tr>
    <tr>
        <td>/employee/{id}</td>
        <td>POST</td>
        <td>name : String<br>salary : Float</td>
        <td>update an employee</td>
    </tr>
    <tr>
        <td>/employee/{id}</td>
        <td>DELETE</td>
        <td>/</td>
        <td>delete an employee</td>
    </tr>
</table>

## Attendance Endpoints

<table>
    <thead>
        <tr>
            <th>endpoint</th>
            <th>method</th>
            <th>body</th>
            <th>description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>/attendance/{id}</td>
            <td>GET</td>
            <td>/</td>
            <td>get an attendance by id</td>
        </tr>
        <tr>
            <td>/attendance/{emp_id}/month/{yyyy}-{mm}</td>
            <td>GET</td>
            <td>/</td>
            <td>get the attendance report of an employee for a given month</td>
        </tr>
        <tr>
            <td>/attendance/{emp_id}/month/{yyyy}-{mm}-{dd}</td>
            <td>GET</td>
            <td>/</td>
            <td>get the attendance entry of an employee at a given day</td>
        </tr>
        <tr>
            <td>/attendance</td>
            <td>POST</td>
            <td>
                date : Date<br>
                employee_id : Int<br>
                check_in : Time
            </td>
            <td>create an attendance</td>
        </tr>
        <tr>
            <td>/attendance/{id}</td>
            <td>POST</td>
            <td>
                field : Enum<br>
                time : Time
            </td>
            <td>update an attendance entry</td>
        </tr>
    </tbody>
</table>

### Field details
- POST /attendance/{id}
   ``field : Enum('check_in','check_out','break_start','break_end')``
# Responses 
## 400 Bad Request
Validation error occurs when the request body contains invalid data either by sending fileds containing with incorrect data types or the data itself is incorrect, for example : trying to update an attendance field ``break_start`` to ``08:30:22`` while ``check_in`` is set to `09:01:22`, here ``break_start`` can't be set to be before ``check_in`` which results in a bad-request response .
```json
{

    "success": false,

    "message": "Validation failed",

    "errors": "Valeur invalide pour '${fieldName}'."

}
```
``${fieldName}`` is the name of the field on which validation failed 
## 500 Internal Server Error
This type of error occurs because of run-time exceptions, all exceptions must be caught and an appropriate user-friendly message be returned , as for lower layers like service layers exceptions must be thrown to be handled by the upper layers.
```json
{
	"success": false,
	"message": {error message}
}
```
