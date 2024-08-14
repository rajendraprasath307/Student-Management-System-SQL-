# Student-Management-System-SQL-
## Creating the database and tables. Inserting sample data. Performing various queries to extract useful information from the database.

-- Create the database
CREATE DATABASE StudentManagement;

-- Use the database
USE StudentManagement;

-- Create Students table
CREATE TABLE Students (
    StudentID INT AUTO_INCREMENT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    DateOfBirth DATE,
    EnrollmentDate DATE
);

-- Create Courses table
CREATE TABLE Courses (
    CourseID INT AUTO_INCREMENT PRIMARY KEY,
    CourseName VARCHAR(100),
    Credits INT
);

-- Create Enrollments table
CREATE TABLE Enrollments (
    EnrollmentID INT AUTO_INCREMENT PRIMARY KEY,
    StudentID INT,
    CourseID INT,
    EnrollmentDate DATE,
    FOREIGN KEY (StudentID) REFERENCES Students(StudentID),
    FOREIGN KEY (CourseID) REFERENCES Courses(CourseID)
);

-- Create Grades table
CREATE TABLE Grades (
    GradeID INT AUTO_INCREMENT PRIMARY KEY,
    EnrollmentID INT,
    Grade CHAR(1),
    FOREIGN KEY (EnrollmentID) REFERENCES Enrollments(EnrollmentID)
);

-- Create Attendance table
CREATE TABLE Attendance (
    AttendanceID INT AUTO_INCREMENT PRIMARY KEY,
    EnrollmentID INT,
    Date DATE,
    Status ENUM('Present', 'Absent'),
    FOREIGN KEY (EnrollmentID) REFERENCES Enrollments(EnrollmentID)
);

-- Insert data into Students
INSERT INTO Students (FirstName, LastName, DateOfBirth, EnrollmentDate)
VALUES 
    ('John', 'Doe', '2004-05-15', '2023-09-01'),
    ('Jane', 'Smith', '2003-11-22', '2023-09-01');

-- Insert data into Courses
INSERT INTO Courses (CourseName, Credits)
VALUES 
    ('Mathematics', 3),
    ('History', 4);

-- Insert data into Enrollments
INSERT INTO Enrollments (StudentID, CourseID, EnrollmentDate)
VALUES 
    (1, 1, '2023-09-01'),
    (1, 2, '2023-09-01'),
    (2, 1, '2023-09-01');

-- Insert data into Grades
INSERT INTO Grades (EnrollmentID, Grade)
VALUES 
    (1, 'A'),
    (2, 'B'),
    (3, 'A');

-- Insert data into Attendance
INSERT INTO Attendance (EnrollmentID, Date, Status)
VALUES 
    (1, '2024-08-10', 'Present'),
    (1, '2024-08-11', 'Absent'),
    (2, '2024-08-10', 'Present');

-- Queries

-- Retrieve all students with enrollment and course information
SELECT s.StudentID, s.FirstName, s.LastName, c.CourseName, e.EnrollmentDate
FROM Students s
JOIN Enrollments e ON s.StudentID = e.StudentID
JOIN Courses c ON e.CourseID = c.CourseID
ORDER BY s.StudentID, c.CourseName;

-- Retrieve students with their grades
SELECT s.FirstName, s.LastName, c.CourseName, g.Grade
FROM Students s
JOIN Enrollments e ON s.StudentID = e.StudentID
JOIN Courses c ON e.CourseID = c.CourseID
JOIN Grades g ON e.EnrollmentID = g.EnrollmentID
ORDER BY s.LastName, c.CourseName;

-- Get all enrollments and their attendance
SELECT s.FirstName, s.LastName, c.CourseName, a.Date, a.Status
FROM Students s
JOIN Enrollments e ON s.StudentID = e.StudentID
JOIN Courses c ON e.CourseID = c.CourseID
JOIN Attendance a ON e.EnrollmentID = a.EnrollmentID
ORDER BY s.LastName, a.Date;

-- Find students who are absent more than a specific number of times
SELECT s.FirstName, s.LastName, COUNT(a.AttendanceID) AS AbsentCount
FROM Students s
JOIN Enrollments e ON s.StudentID = e.StudentID
JOIN Attendance a ON e.EnrollmentID = a.EnrollmentID
WHERE a.Status = 'Absent'
GROUP BY s.StudentID
HAVING COUNT(a.AttendanceID) > 2
ORDER BY AbsentCount DESC;

-- Retrieve average grade for each course
SELECT c.CourseName, AVG(CASE g.Grade
                            WHEN 'A' THEN 4.0
                            WHEN 'B' THEN 3.0
                            WHEN 'C' THEN 2.0
                            WHEN 'D' THEN 1.0
                            WHEN 'F' THEN 0.0
                          END) AS AverageGrade
FROM Courses c
JOIN Enrollments e ON c.CourseID = e.CourseID
JOIN Grades g ON e.EnrollmentID = g.EnrollmentID
GROUP BY c.CourseName;

-- List all courses with their number of enrolled students
SELECT c.CourseName, COUNT(e.StudentID) AS NumberOfStudents
FROM Courses c
LEFT JOIN Enrollments e ON c.CourseID = e.CourseID
GROUP BY c.CourseName
ORDER BY NumberOfStudents DESC;

-- Retrieve attendance summary for a specific course
SELECT c.CourseName, a.Date, COUNT(a.AttendanceID) AS AttendanceCount
FROM Courses c
JOIN Enrollments e ON c.CourseID = e.CourseID
JOIN Attendance a ON e.EnrollmentID = a.EnrollmentID
GROUP BY c.CourseName, a.Date
ORDER BY c.CourseName, a.Date;

-- Find students who are enrolled in all courses
SELECT s.FirstName, s.LastName
FROM Students s
JOIN Enrollments e1 ON s.StudentID = e1.StudentID
JOIN Enrollments e2 ON s.StudentID = e2.StudentID
GROUP BY s.StudentID
HAVING COUNT(DISTINCT e1.CourseID) = (SELECT COUNT(*) FROM Courses);

-- List all students and their total credits based on enrolled courses
SELECT s.FirstName, s.LastName, SUM(c.Credits) AS TotalCredits
FROM Students s
JOIN Enrollments e ON s.StudentID = e.StudentID
JOIN Courses c ON e.CourseID = c.CourseID
GROUP BY s.StudentID
ORDER BY TotalCredits DESC;

-- Retrieve students with their enrollment dates and grades
SELECT s.FirstName, s.LastName, c.CourseName, e.EnrollmentDate, g.Grade
FROM Students s
JOIN Enrollments e ON s.StudentID = e.StudentID
JOIN Courses c ON e.CourseID = c.CourseID
JOIN Grades g ON e.EnrollmentID = g.EnrollmentID
ORDER BY s.LastName, c.CourseName;
