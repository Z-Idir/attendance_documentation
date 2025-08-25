This page provides a thorough guide for developers to set up , run and maintain this project,
the github repo for it is private so you will need to get the appropriate permissions from the admin.
# Prerequisites
In order to run the project you will need the following installed on your system : 
- Docker and Docker Compose
# Installation and Setup
1. Clone the repository : 
```shell
git clone https://github.com/Z-Idir/pointeuse
cd pointeuse/backend
```
We are only interested in the backend folder as it contains the API, the frontend is not within the scope of this documentation
2. Start the docker containers :
```shell
docker compose up -d
```
this will spin up the following containers : 
- Apache HTTP server : handles client requests
- MySQL Server : manages the sql database
# Accessing the API
The API will be accessible at :
```
http://localhost:8080/
```
This is the base URL on which the URLs specified in the [endpoints](./endpoints.md#endpoints) are to be mounted on .
# Container Access and Development
All development should be performed inside of the Docker container to insure consistency.
## Accessing the Apache server : 
Run the following command : 
```shell
docker compose exec web bash
```
it will run the shell of the Apache server inside of the terminal you used to run the command.
### Installing dependencies
once inside the server's terminal run the following command to install all dependencies using composer ( if composer is not available, first check out [this](./issues#composer) , and if that didn't help raise an issue report)
```shell
composer install
```
## Running DB initialization and migration
When starting development the database will be clean, without even schema definitions, so in order to create the tables and so run the following command : 
```shell
php Src/db/migrate.php migrate
```
## Fetching Holidays 
The script `calendarSetup.php` that is found inside of `backend/` is responsible for fetching holidays and cleaning the data and putting it into a json file under the name `calendar.json` , in order to run it just run this command : 
```
php calendarSetup.php getCalendar
```
**Note :**  If any holiday besides the french ones any date can be added inside of the ``calendar.json`` file to accommodate for any additional holidays, the calendar file looks like this : 
```json
[ 
    "2025-01-01",
    "2025-04-21",
    "2025-05-01",
    "2025-05-08",
    "2025-05-29",
    "2025-06-09",
    "2025-07-14",
    ...
]
```
so just append a date anywhere in the array in the format : `yyyy-mm-dd` and it will be used by the api.
## Namespace definition
Namespaces are a way to map file paths to namespaces and make classes accessible that way, we can group relevant classes under the same namespace (this corresponds to modules), if we want to include a class in a module we have to specify the namespace at the top of the file where that class is defined : 
```php
namespace App;
class foo {...}
```
the definition of namespaces is found inside of the `composer.json` file under `autoload.psr-4`,
for example lets say we have a folder called **Src/models** and we want to associate it with a namespace in order to be able to access the classes defined within it by referencing `namespace.className`, let's call it ``Models`` we do the following : 
1. add a namespace definition to `autoload.psr-4` :
```json
"autoload":{"psr-4": {
            "DB\\": "Src/db/",
			"Models\\": "Src/models",
			...
    }},
```
 2. next we have to generate the autoload files which is a crucial step , inside the server's execution shell we want to execute `composer dump-autoload` : 
 ```shell
 docker compose exec web bash
```
once inside the server's terminal run : 
```shell
composer dump-autoload -o
```
after this classes can be accessed through the namespaces associated with them 
## Connecting to the database
The database connection is delegated to the **Eloquent ORM** by creating what is called a capsule and attaching it to the global scope, the script responsible for connecting to the db is  `Src/db/Bootstrap.php` , so in order to be able to perform database operations we need to first boot it like this : 
```php
use DB\Bootstrap;
Bootstrap::boot();
```
and after that the capsule is set to global scope and we can execute db operations.
**NOTE :** notice the difference between the file structure `Src/db` and the namespace `DB` for an explanation see [namespaces](#namespace-definition)
# Project Structure

<pre>  
backend/ 
├── src/ 
│ ├── Controllers/ 
│ ├── dao/ 
│ ├── db/ 
│ ├── Schemas/ 
│ ├── Services/ 
│ └── Utils/ 
├── vendor/ 
└── ...  </pre>
# Making changes

## Feature addition
Follow these steps in order to add a feature, the order is of not matter actually just make sure to do all of these
1. Create a new DAO (model) inside ``Src/dao/``
2. Create a new Service inside ``Src/Services/``
3. Create a new Controller inside of ``Src/Controllers/`` folder
4. Add the corresponding routes inside  of ``Router.php``
5. Test the added routes using postman or curl.
if validation is needed then : 
- Create the validation schema inside of `Src/Schemas`
- Call the `Validator::validate` function with the data and schema passed as parameters , the following is dummy code and incomplete so it won't run : 
```php
use Schemas\AttendanceSchema;
use Utils\Validator;

$validation = Validator::validate($_POST, AttendanceSchema::creationSchema());
```
- `$_POST` is the data that needs to be validated
- ``AttendanceSchema::creationSchema()`` is a function that returns the schema for the attendance creation data
## Database changes
The database schema is defined inside of `Src/db/SchemaDefinition.php` , so if changes are made before container build (clean database where migrations are not performed yet) then changes to the database are to be added directly inside `SchemaDefinition.php` in the *MIGRATION SECTION*, if the database is already created then an update must be performed which should be added to the *UPDATE SECTION* and be added to the *MIGRATION SECTION* as well .
### Examples
1. The migration section is denoted by a comment and is inside of an if-statement that checks the command provided through the command line like so : 
```php
$command = $argv[1] ?? null;

// MIGRATIONS SECTION

if($command === 'migrate'){
    SchemaDefinition::createEmployeesTable();
    ...
```
2. In order to add to the schema definitions they should be added to the `SchemaDefinition` class and grouped as static methods :
```php
use Illuminate\Database\Capsule\Manager as Capsule;
use Illuminate\Database\Schema\Blueprint;

class SchemaDefinition {
    public static function createEmployeesTable(){
        Capsule::schema()->create('employees',function(Blueprint $table){
            $table->increments('id');
            $table->string('name');
            $table->float('salary');
            $table->timestamps();
        });
    }
```
and then they must be called inside the *MIGRATION SECTION* of `migrate.php`
3. If it is an update to the schema that we wish to do then a modification function should be created inside the `SchemaDefinition` class, here is an example of how this function should look like : 
```php
public static function modifyAttendanceTable() {
        Capsule::schema()->table('attendance', function (Blueprint $table) {
            $table->string('last_action')->nullable()->after('break_end');
        });
    }
```
instead of calling `create` method on `Capsule::schema()` we call : `table` which selects an already existing table to modify its schema, ==remember that every schema modification will be required to be added in the migration section in order to be included in future builds of the image without having to resort to manually updating it== 