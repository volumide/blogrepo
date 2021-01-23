## Developing API Using PHP

We will be  using object oriented programming programming as much as possible in this tutorial. And after you can use for all your projects that are not that complex and for learning purpose also 

#### Step 1 
in This tutorial we are using mysql as our database

Create a connection file like that connects to your database ex. **connection.php** and 
create a class that connects to your mysql database. 

```
  <?php
    class Database{
        private static $host = 'localhost';
        private static $db_name = '***';
        private static $username = '***';
        private static $password = '***';
        private static $conn;

        public static  function connect(){
            self::$conn = new mysqli(self::$host, self::$username, self::$password, self::$db_name);
            if ((self::$conn)->connect_errno) {
                die('Connection error:'.  (self::$conn)->connect_errno. "'") ;
            }
            return self::$conn;
        }
    }
?>
``` 

Now we can now write our CRUD function to read and write to our database and reusable as an api for our applications.

create another php file and give it a name ex **controler.php**.

This **controller.php** file will house our functions that helps us with out crud api.

#### Step 2
 since we are using oop we are going tobe declaring some private class that will be useful as we write our code


```
<?php
    class Controller{
        private string $db_table;
        private object $conn;
        private array $fillables;
        private array $values = [];
        private $response;
``` 
Now we will create a constructor function. 

#### Step 3 
Read more about php contructor here  [understand php constructor](https://www.w3schools.com/php/php_oop_constructor.asp) 


```
public function __construct(string $db, object $conn, array $fillables = []){
   $this->db_table = $db;
   $this->conn = $conn;
   $this->fillables =  $fillables;
}
        
``` 

#### Step 4
Now with our constructor set up we can continue to write our crud functions 

####  Create function 
This function will help to send data to the database

```
  public function create():  string {
       error_reporting(0);
       $input = json_decode(file_get_contents("php://input"));

        foreach ($this->fillables as $fillable) {
                array_push($this->values, mysqli_real_escape_string($this->conn, $input->$fillable));
            }

        $values = '"' . join('", "', $this->values). '"';
        $fillable = join(',',  $this->fillables);

        $query = "INSERT INTO $this->db_table ($fillable)  VALUES ($values)";

         if ($this->conn->query($query)) {
                $this->response = array(
                    "status" => "success",
                    "message" => "message sent successfully",
                    "data" => $input
                );
                return json_encode($this->response);
          }

        return json_encode(["message" => $this->conn->error], JSON_PRETTY_PRINT);
        }
``` 

####  Read function 

This function reads/get all data from the database

```
    public function read(): string {
            $query = "SELECT * FROM $this->db_table ORDER BY id";
            $result = $this->conn->query($query);
            if(mysqli_num_rows($result) < 1 ){
                return json_encode(array(
                    "status" => "success",
                    "message" => "No content",
                ));
            }

            while($row = mysqli_fetch_assoc($result)){
                array_push($this->values, $row);
            }

            $this->response = array(
                "status" => "success",
                "data" => $this->values,
            );
            return json_encode($this->response, 0 , 200);
        }
``` 

####  Read function 

This function reads/get data from the database by id

```
 public function readSingle(): string {
     if (!isset($_GET['id'])) {
             return $this->getMessage();
     }

     $id = $_GET['id'];
     $query = "SELECT * FROM $this->db_table WHERE id = $id LIMIT 1";
     $result = $this->conn->query($query);
     if (mysqli_num_rows($result) < 1) {
            return json_encode(array(
               "status" => "fail",
               "message" => "not found",
            ));
     }
    while($row = mysqli_fetch_assoc($result)){
          $this->values = $row;
    }

    $this->response = array(
           "status" => "success",
           "data" => $this->values,
        );
   return json_encode($this->response, 0 , 200);
 }
``` 

####  Update function 

This function Updates data by id

```
 public function update(): string {
    error_reporting(0);
    $seperator = '';
    $query = "UPDATE $this->db_table SET ";
    $input = json_decode(file_get_contents("php://input"));
            
    foreach ($this->fillables as $fillable) {
         $query .= $seperator. $fillable . "='" . mysqli_real_escape_string($this->conn, $input->$fillable). "'";
         $seperator = ',';
     }
          if (isset($_GET['update'])) {
                $id = $_GET['update'];
                $query .= "WHERE id = $id";
                $result = $this->conn->query($query)  ? json_encode([
                    "status"=> "success",
                    "message" => "updated successfully"
                ]) : json_encode(["mesaage" => $this->conn->error]);

                return $result ;
            }
        }
``` 

####  Delete function 

This function Delete data by id
 
```
 public function delete(){
    if (isset($_GET['delete'])) {
           $id = $_GET['delete'];
           $query =  "DELETE FROM $this->db_table WHERE id = $id";
           $result = $this->conn->query($query) ? json_encode([
                "status" => "success",
                "message" => "successfully deleted"
            ])  : json_encode([
                "status" => "fail",
                "message" => $this->conn->error
             ]);
             return $result;
          }

    }
``` 

Now that our functions are set we are create an api that can be used in any of our applications.

Create a php file name **api.php**
this file will serve as our endpoint 

#### Step 5

incude your connection.php and controller.php folder in your api.php folder like this 

```
<?php
    header('Access-Control-Allow-Origin: *');
   
    include '../config/connection.php';
    include '../models/controller.php';

    $db_table = Database::connect(); // the name of our class in connection.php is Database
``` 

With this we can use connect to our databse that i called from connection.php and use  any of the crud funcitons in our controller class.

Perform a create operation


```
   // the fillables represent all the fields that we want to send to out database depending on and corresponding to our table name
   $fillables = [
        "name",
        "email",
        "phone"
    ];

    echo (new Controller('contacts', $db_table , $fillables))->create();
``` 

To peform real all operation

```
     echo (new Controller('contacts', $db_table))->read();
``` 

To get by id
```
      if (isset($_GET['id'])) {
         echo (new Controller('contacts', $db_table))->readSingle();
      }
``` 

To update by id
``` 
       // the update corespond to the get method in our onnection class for update
      if (isset($_GET['update'])) {
        echo (new Controller('contacts', $db_table , $fillables))->update();
        return;
     }
``` 


To delete by id
``` 
      if (isset($_GET['delete'])) {
        echo (new Controller('contacts', $db_table))->delete();
        return;
    }
``` 

This is the end of our crude api.

full code in the   [github repo](https://github.com/volumide/phpapi_oop) 




















