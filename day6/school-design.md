
# School Database Design

## Tables

### 1. Students
The students table stores information about each student. It contains
student_id as the primary key, name, and email. The email is required
and must be unique so that two students cannot use the same email
address.

### 2. Courses
The courses table stores the courses offered by the school. It contains
course_id as the primary key and course_name, which is required.
Each course has a unique ID and course name.

### 3. Enrolments
The enrolments table records which student takes which course. It
contains enrolment_id as the primary key, student_id and course_id
as foreign keys, and grade. The grade can be NULL if it has not yet
been assigned.

## Relationships

Students and enrolments have a one-to-many relationship because one
student can have multiple enrolment records, while each enrolment
belongs to one student.

Courses and enrolments also have a one-to-many relationship because
one course can have multiple enrolment records, while each enrolment
refers to one course.

Students and courses therefore have a many-to-many relationship.
A student can take several courses, and a course can have several
students. The enrolments table is needed to connect these tables and
store information specific to each enrolment, such as the grade.
The UNIQUE constraint on student_id and course_id prevents a student
from enrolling in the same course twice.

## Index

I added an index on enrolments(course_id). It helps the database find
enrolments for a particular course more efficiently, especially when
listing students on a course or counting students per course. The
UNIQUE constraint on student_id and course_id also creates an index
in SQLite, helping enforce the rule against duplicate enrolments.

## SQL or NoSQL?

I would choose SQL for this school system because students, courses,
and enrolments have clear relationships and require reliable data
integrity. Primary keys, foreign keys, NOT NULL constraints, and
UNIQUE constraints help prevent invalid or duplicate records.
SQL also supports JOIN and GROUP BY queries, which make it easier
to list courses, find students, and calculate enrolment totals.
A relational database such as SQLite is suitable for this small
system. If the application later needed many simultaneous users
across multiple servers, a database such as PostgreSQL could be a
better choice.
