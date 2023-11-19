# Attendance-Project
Table of Contents
=================

* [Setting database up](#setting-database-up)
* [Setting API Webservice](#Setting-API-Webservice)
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
    | name         | VARCHAR(255)        |             | Yes      |     |
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
 | created_date  | DATETIME   |             |          |               

 ### Database diagram
 ```mermaid
classDiagram
  class Courses {
    +id: INT (PK, NN)
    +name: VARCHAR(255) (NN)
    +category_id: INT (NN, FK)
    +type: VARCHAR(50)
    +outside_university: BOOLEAN
    +description: TEXT (NN)
    +social_group: VARCHAR(255)
    +place: VARCHAR(255) (NN)
    +max_number: INT (NN)
    +exam_required: BOOLEAN
    +lectures: INT
  }

  class CourseAttendance {
    +id: INT (PK, NN)
    +course_id: INT (NN, FK)
    +number: INT (NN)
    +code: VARCHAR(30)
    +closed: BOOLEAN (NN)
    +created_date: DATETIME
  }

  class CourseSubs {
    +id: INT (PK, NN)
    +name: VARCHAR(255) (NN)
    +course_id: INT (NN, FK)
  }

  class CourseSubsAttendance {
    +id: INT (PK, NN)
    +course_subs_id: INT (NN, FK)
    +course_attendance_id: INT (NN, FK)
  }

CourseSubsAttendance -- CourseSubs: course_subs_id --> id
CourseSubsAttendance -- CourseAttendance: course_attendance_id --> id

```

## Setting API Webservice
In this step, we'll simplify the process using straightforward PHP statements. We'll create PHP files that handle requests from the mobile app, making it easy for us to understand the entire flow.
### 1. Adding a New Course
- Initiate the process by adding a new course.

### 2. Adding New Lecture Attendance for the Course
- Record attendance for each lecture associated with the course.

### 3. Adding Attendance in the m2m Table
- Update the many-to-many (m2m) table to track attendance, indicating that a particular `CourseSubs` has attended the course.

### 4. Retrieve the Subscribers for the Course
- Fetch the list of subscribers enrolled in the course.

### 5. Retrieve the Attendance List for Every Lecture
- Access the attendance records for each lecture, providing a comprehensive list.

### init.php
  add the database credincials
  ```php
    <?php

function connectDB() {
    $database_host = "your_database_host";
    $database_username = "your_database_username";
    $database_password = "your_database_password";
    $database_name = "your_database_name";

    $conn = new mysqli($database_host, $database_username, $database_password, $database_name);

    if ($conn->connect_error) {
        die("Connection failed: " . $conn->connect_error);
    }

    return $conn;
}

?>

```
### 1. add_course.php

```php
<?php

include('init.php');

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

### 2. add_lecture_attendance.php
```php
<?php

include('init.php');

// Function to add new lecture attendance for a course
function addLectureAttendance($course_id, $number, $code, $closed) {
    $conn = connectDB();

    // Use prepared statement to prevent SQL injection
    $sql = "INSERT INTO course_attendance (course_id, number, code, closed) VALUES (?, ?, ?, ?)";

    $stmt = $conn->prepare($sql);
    $stmt->bind_param("iiss", $course_id, $number, $code, $closed);

    if ($stmt->execute()) {
        echo "New lecture attendance added successfully!";
    } else {
        echo "Error: " . $stmt->error;
    }

    $stmt->close();
    $conn->close();
}

// Check if the request is a POST request
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    // Get data from the POST request
    $course_id = $_POST["course_id"];
    $number = $_POST["number"];
    $code = $_POST["code"];
    $closed = $_POST["closed"];

    // Call the function to add new lecture attendance
    addLectureAttendance($course_id, $number, $code, $closed);
} else {
    echo "Invalid request method!";
}

?>
```
### 3.add_attendance_m2m.php
```php
<?php

include('init.php');

// Function to add attendance in the m2m table
function addAttendanceM2M($course_subs_id, $course_attendance_id) {
    $conn = connectDB();

    // Use prepared statement to prevent SQL injection
    $sql_insert = "INSERT INTO course_subs_attendance (course_subs_id, course_attendance_id) VALUES (?, ?)";
    $stmt_insert = $conn->prepare($sql_insert);
    $stmt_insert->bind_param("ii", $course_subs_id, $course_attendance_id);

    if ($stmt_insert->execute()) {
        // Retrieve the course_subs name for display
        $sql_select_name = "SELECT name FROM course_subs WHERE id = ?";
        $stmt_select_name = $conn->prepare($sql_select_name);
        $stmt_select_name->bind_param("i", $course_subs_id);
        $stmt_select_name->execute();
        $stmt_select_name->bind_result($course_subs_name);
        $stmt_select_name->fetch();
        $stmt_select_name->close();

        echo "Attendance added in the m2m table for course_subs: $course_subs_name successfully!";
    } else {
        echo "Error: " . $stmt_insert->error;
    }

    $stmt_insert->close();
    $conn->close();
}

// Check if the request is a POST request
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    // Get data from the POST request
    $course_subs_id = $_POST["course_subs_id"];
    $course_attendance_id = $_POST["course_attendance_id"];

    // Call the function to add attendance in the m2m table
    addAttendanceM2M($course_subs_id, $course_attendance_id);
} else {
    echo "Invalid request method!";
}

?>
```
4. get_course_subscribers.php
```php
<?php

include('init.php');

// Function to retrieve subscribers for a course
function getCourseSubscribers($course_id) {
    $conn = connectDB();

    // Use prepared statement to prevent SQL injection
    $sql = "SELECT id, name FROM course_subs WHERE course_id = ?";

    $stmt = $conn->prepare($sql);
    $stmt->bind_param("i", $course_id);
    $stmt->execute();
    $result = $stmt->get_result();
    $subscribers = [];

    while ($row = $result->fetch_assoc()) {
        $subscribers[] = [
            'id' => $row['id'],
            'name' => $row['name'],
        ];
    }

    $stmt->close();
    $conn->close();

    return $subscribers;
}

// Check if the request is a POST request
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    // Get data from the POST request
    $course_id = $_POST["course_id"];

    // Call the function to retrieve subscribers for the course
    $subscribers = getCourseSubscribers($course_id);

    // Output the result as JSON
    header('Content-Type: application/json');
    echo json_encode($subscribers);
} else {
    echo "Invalid request method!";
}

?>
```
5. get course lecture list.
  you can do it yourself based on the information provided

### Mobile Application

## Add New Course Activity

1. **Create New Activity:**
   - In your Android Studio project, create a new activity named "AddNewCourseActivity" to handle the addition of a new course.

2. **Design UI Layout:**
   - Design the UI layout for the "AddNewCourseActivity" in the associated XML file (`activity_add_new_course.xml`). Include necessary input fields such as EditText for course name, description, etc., and a Button to submit the new course.

3. **Add Volley Dependency:**
   - Add the Volley library dependency to your app's `build.gradle` file:
     ```gradle
     implementation 'com.android.volley:volley:1.2.0'
     ```

4. **Handle Button Click:**
   - In the Java code for `AddNewCourseActivity`, handle the button click event to initiate the data submission using Volley.

5. **Send POST Request:**
   - Use Volley to send a POST request to the `add_course.php` file on the localhost server.
     ```java
     // Instantiate the RequestQueue.
     RequestQueue queue = Volley.newRequestQueue(this);
     String url = "http://localhost/your_project_folder/add_course.php";

     // Create a HashMap with parameters for the new course.
     Map<String, String> params = new HashMap<>();
     params.put("course_name", courseName);
     params.put("category_id", categoryId);
     // Add other parameters as needed.

     // Create a StringRequest to make a POST request.
     StringRequest stringRequest = new StringRequest(Request.Method.POST, url,
             response -> {
                 // Show a Toast with the response message.
                 Toast.makeText(AddNewCourseActivity.this, response, Toast.LENGTH_SHORT).show();

                 // Additional handling if needed.
             },
             error -> {
                 // Handle errors that occur during the request.
                 Toast.makeText(AddNewCourseActivity.this, "Error occurred", Toast.LENGTH_SHORT).show();
             }) {
         @Override
         protected Map<String, String> getParams() {
             return params;
         }
     };

     // Add the request to the RequestQueue.
     queue.add(stringRequest);
     ```
6. **Handle PHP Script Response:**
   - In the `add_course.php` script on the server, handle the received POST data, validate it, and perform the necessary database operations. Respond with appropriate success or error messages.

## Adding New Lecture Attendance for the Course
1. Previous steps.

2. **Send POST Request:**
   - Use Volley to send a POST request to the `add_lecture_attendance.php` file on the localhost server.
     ```java
     // Instantiate the RequestQueue.
     RequestQueue queue = Volley.newRequestQueue(this);
     String url = "http://localhost/your_project_folder/add_lecture_attendance.php";

     // Create a HashMap with parameters for the new lecture attendance.
     Map<String, String> params = new HashMap<>();
     params.put("course_id", courseId);
     params.put("lecture_number", lectureNumber);
     params.put("attendance_code", attendanceCode);
     // Add other parameters as needed.

     // Create a StringRequest to make a POST request.
     StringRequest stringRequest = new StringRequest(Request.Method.POST, url,
             response -> {
                 // Show a Toast with the response message.
                 Toast.makeText(AddLectureAttendanceActivity.this, response, Toast.LENGTH_SHORT).show();

                 // Additional handling if needed.
             },
             error -> {
                 // Handle errors that occur during the request.
                 Toast.makeText(AddLectureAttendanceActivity.this, "Error occurred", Toast.LENGTH_SHORT).show();
             }) {
         @Override
         protected Map<String, String> getParams() {
             return params;
         }
     };

     // Add the request to the RequestQueue.
     queue.add(stringRequest);
     ```

