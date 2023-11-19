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

3. ...
4. ...
