# Attendance-Project
Table of Contents
=================

* [Setting database up](#setting-database-up)
* [Setting API Webservice](#api)
* [Mobile App](#web-app)

## Setting database up
I'll be utilizing a pure SQL connection without an ORM for a clearer understanding of the entire process. You can select only the necessary fields. For example, in the 'courses' table, you may create it with fields such as 'name,' 'description,' and 'max_number' for testing purposes.

1. **Courses Table**

  | Field Name         | Data Type        | Primary Key | Not Null | Foreign Key     |
  |---------------------|------------------|-------------|----------|-----------------|
  | id                  | INT              | Yes         | Yes      |                 |
  | name               | VARCHAR(255)    |             | Yes      |                 |
  | category_id   | INT              |             | Yes      | Yes (Categories) |
  | type               | VARCHAR(50)     |             | No       |                 |
  | outside_university | BOOLEAN          |             | No       |                 |
  | description    | TEXT             |             | Yes      |                 |
  | social_group | VARCHAR(255)   |             | No       |                 |
  | place             | VARCHAR(255)    |             | Yes      |                 |
  | max_number | INT               |             | Yes      |                 |
  | exam_required | BOOLEAN      |             | No       |                 |
  | lectures        | INT               |             | No       |                 |
  note: category_id not needed for the first test.

2. **Course_Subs and Course_Subs_Attendance Tables**: This table includes subscribers for the course. Perhaps in the future, you may need to associate it with students, users, or another table as a foreign key, and you can also ignore adding a profile table for the first test.

    a. **course_subs Table**
  
    | Field Name         | Data Type  | Primary Key | Not Null | Foreign Key       |
    |--------------------|------------|-------------|----------|-------------------|
    | id                 | INT        | Yes         | Yes      |                   |
    | profile_id         | INT        |             | Yes      | Yes (Profiles)    |
    | course_id          | INT        |             | Yes      | Yes (Courses)     |
  
    b. **course_subs_attendance Table**: associated with the course_subs table in a many-to-many (m2m) relationship. This table is designed to manage additional attendance records for subscribers, particularly in situations where a course comprises multiple lectures.
  
    | Field Name             | Data Type  | Primary Key | Not Null | Foreign Key            |
    |------------------------|------------|-------------|----------|------------------------|
    | id                     | INT        | Yes         | Yes      |                        |
    | course_subs_id         | INT        |             | Yes      | Yes (Course_Subs)      |
    | course_attendance_id   | INT        |             | No       | Yes (Course_Attendance)|

3. **course_attendance Table**

 | Field Name    | Data Type  | Primary Key | Not Null | Foreign Key   |
 |---------------|------------|-------------|----------|---------------|
 | id            | INT        | Yes         | Yes      |               |
 | course_id     | INT        |             | Yes      | Yes (courses) |
 | number        | INT        |             | Yes      |               |
 | code          | VARCHAR(30)|             |          |               |
 | closed        | BOOLEAN    |             | Yes      |               |
 | created_date  | DATETIME   |             |          |               |

## Setting API Webservice
In this step, we'll simplify the process using straightforward PHP statements. We'll create PHP files that handle requests from the mobile app, making it easy for us to understand the entire flow.
### 1. init.php
  add the database credincials
  ```php
    <?php
    
    $database_host = "your_database_host";
    $database_username = "your_database_username";
    $database_password = "your_database_password";
    $database_name = "your_database_name";
    
    ?>
```
### 2. init.php
### 1. add_course.php

```php
<?php
include('db_credentials.php');

// Function to establish a database connection
function connectDB() {
    global $database_host, $database_username, $database_password, $database_name;

    $conn = new mysqli($database_host, $database_username, $database_password, $database_name);

    if ($conn->connect_error) {
        die("Connection failed: " . $conn->connect_error);
    }

    return $conn;
}

// Function to add a new course
function addCourse($name, $category_id, $type, $outside_university, $description, $social_group, $place, $max_number, $exam_required, $lectures) {
    $conn = connectDB();

    $sql = "INSERT INTO courses (name, category_id, type, outside_university, description, social_group, place, max_number, exam_required, lectures) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)";

    $stmt = $conn->prepare($sql);
    $stmt->bind_param("sississsii", $name, $category_id, $type, $outside_university, $description, $social_group, $place, $max_number, $exam_required, $lectures);

    if ($stmt->execute()) {
        echo "New course added successfully!";
    } else {
        echo "Error: " . $stmt->error;
    }

    $stmt->close();
    $conn->close();
}

// Check if the request is a POST request
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    // Get data from the POST request
    $name = $_POST["name"];
    $category_id = $_POST["category_id"];
    $type = $_POST["type"];
    $outside_university = $_POST["outside_university"];
    $description = $_POST["description"];
    $social_group = $_POST["social_group"];
    $place = $_POST["place"];
    $max_number = $_POST["max_number"];
    $exam_required = $_POST["exam_required"];
    $lectures = $_POST["lectures"];

    // Call the function to add a new course
    addCourse($name, $category_id, $type, $outside_university, $description, $social_group, $place, $max_number, $exam_required, $lectures);
} else {
    echo "Invalid request method!";
}

?>
```

