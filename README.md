# Library Management System using SQL Project --P2

## Project Overview

**Project Title**: Library Management System  
**Database**: `library_mng_sys_p2`

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.

[![plot](C:\Users\DELL\Desktop\SQL_PROJECTS\Project2_Library_Management_System\library.jpg)](https://github.com/AMOL48/sql_library_management_system_p2/blob/main/library.jpg)

## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure

### 1. Database Setup

[![plot](C:\Users\DELL\Desktop\SQL_PROJECTS\Project2_Library_Management_System\schemas.pgerd.png)](https://github.com/AMOL48/sql_library_management_system_p2/blob/main/schemas.pgerd.png)

- **Database Creation**: Created a database named `library_mng_sys_p2`.
- **Table Creation**: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

```sql
CREATE DATABASE library_mng_sys_p2;

-- creating branch table

DROP TABLE IF EXISTS branch;
CREATE TABLE branch
	(
		branch_id VARCHAR(10) PRIMARY KEY,
		manager_id VARCHAR(15),
		branch_address VARCHAR(75),
		contact_no VARCHAR(20)
	);

-- creating employees table

DROP TABLE IF EXISTS employees;
CREATE TABLE employees
	(
		emp_id VARCHAR(15) PRIMARY KEY,
		emp_name VARCHAR(50),
		position VARCHAR(25),
		salary INT,
		branch_id VARCHAR(25) --FK
	 );


-- creating books table

DROP TABLE IF EXISTS books;
CREATE TABLE books
	(
		isbn VARCHAR(25) PRIMARY KEY,
		book_title VARCHAR(75),
		category VARCHAR(25),
		rental_price FLOAT,
		status VARCHAR(15),
		author VARCHAR(55),
		publisher VARCHAR(55)
    );


-- creating members table

DROP TABLE IF EXISTS members;
CREATE TABLE members
	(
		member_id VARCHAR(25) PRIMARY KEY,
		member_name VARCHAR(25),
		member_address VARCHAR(75),
		reg_date DATE
    );


-- creating issued_status table

DROP TABLE IF EXISTS issued_status;
CREATE TABLE issued_status
	(
		issued_id VARCHAR(15) PRIMARY KEY,
		issued_member_id VARCHAR(15), --FK
		issued_book_name VARCHAR(75),
		issued_date	DATE,
		issued_book_isbn VARCHAR(35), --FK
		issued_emp_id VARCHAR(15)     --FK
	);


-- creating return_status table

DROP TABLE IF EXISTS return_status;
CREATE TABLE return_status
	(
		return_id VARCHAR(15) PRIMARY KEY,
		issued_id VARCHAR(15),
		return_book_name VARCHAR(75),
		return_date DATE,
		return_book_isbn VARCHAR(55)
	);

```

### 2. CRUD Operations

- **Create**: Inserted sample records into the `books` table.
- **Read**: Retrieved and displayed data from various tables.
- **Update**: Updated records in the `employees` table.
- **Delete**: Removed records from the `members` table as needed.

**Task 1. Create a New Book Record**
-- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

```sql
INSERT INTO books (isbn, book_title, category, rental_price, status, author, publisher)
VALUES
	('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');

```

**Task 2: Update an Existing Member's Address**

```sql
UPDATE members
SET member_address = '125 Main St'
WHERE member_id = 'C101';
```

**Task 3: Delete a Record from the Issued Status Table**
-- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.

```sql
DELETE FROM issued_status
WHERE issued_id='IS121';
```

**Task 4: Retrieve All Books Issued by a Specific Employee**
-- Objective: Select all books issued by the employee with emp_id = 'E101'.

```sql
SELECT * FROM issued_status
WHERE issued_emp_id = 'E101'
```

**Task 5: List Members Who Have Issued More Than One Book**
-- Objective: Use GROUP BY to find members who have issued more than one book.

```sql
SELECT
 	m.member_name,
    ist.issued_member_id,
	COUNT(ist.issued_id) AS total_issued
FROM issued_status AS ist
INNER JOIN members AS m
ON m.member_id = ist.issued_member_id
GROUP BY ist.issued_member_id,m.member_name
HAVING COUNT(ist.issued_id) > 1;
```

### 3. CTAS (Create Table As Select)

- **Task 6: Create Summary Tables**: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt\*\*

```sql
CREATE TABLE book_issued_count
AS
SELECT b.isbn, b.book_title, COUNT(ist.issued_id) AS no_issued
FROM books AS b
JOIN
issued_status AS ist
ON ist.issued_book_isbn = b.isbn
GROUP BY b.isbn,b.book_title;
```

### 4. Data Analysis & Findings

The following SQL queries were used to address specific questions:

Task 7. **Retrieve All Books in a Specific Category**:

```sql
SELECT * FROM books
WHERE category = 'Classic';
```

8. **Task 8: Find Total Rental Income by Category**:

```sql
SELECT
	b.category,
	COUNT(ist.issued_id) AS no_issued,
	SUM(b.rental_price) AS total_rent
FROM books AS b
JOIN
issued_status AS ist
ON ist.issued_book_isbn = b.isbn
GROUP BY 1;
```

9. **List Members Who Registered in the Last 180 Days**:

```sql
SELECT * FROM members
WHERE reg_date >= CURRENT_DATE - INTERVAL '180 days';
```

10. **List Employees with Their Branch Manager's Name and their branch details**:

```sql
SSELECT
	e1.emp_id,
	e1.emp_name,
	e1.position,
	e1.salary,
	b.*,
	e2.emp_name AS manager
FROM employees AS e1
JOIN
branch AS b
ON e1.branch_id = b.branch_id
JOIN
employees as e2
ON e2.emp_id = b.manager_id
ORDER BY manager;
```

Task 11. **Create a Table of Books with Rental Price Above a Certain Threshold**:

```sql
CREATE TABLE rental_price_greater_than_7
AS
SELECT *
FROM books
WHERE rental_price > 7;
```

Task 12: **Retrieve the List of Books Not Yet Returned**

```sql
SELECT DISTINCT ist.issued_book_name AS book_not_returned
FROM issued_status AS ist
LEFT JOIN return_status AS rst
ON rst.issued_id = ist.issued_id
WHERE rst.return_id IS NULL;
```

## Advanced SQL Operations

**Task 13: Identify Members with Overdue Books**  
Write a query to identify members who have overdue books and fine ($0.75/overdue_day) (assume a 30-day return period).
Display the member's_id, member's name, book title, issue date, and days overdue and fine.

```sql
SELECT
	ist.issued_member_id,
	m.member_name,
	bk.book_title,
	ist.issued_date,
	rs.return_date,
	((CURRENT_DATE - ist.issued_date)-30) AS overdues_days,
	((CURRENT_DATE - ist.issued_date)-30)*0.75 AS fine
FROM issued_status AS ist
JOIN
members AS m
	ON m.member_id = ist.issued_member_id
JOIN
books AS bk
ON bk.isbn = ist.issued_book_isbn
LEFT JOIN
return_status AS rs
ON rs.issued_id = ist.issued_id
WHERE
		rs.return_date IS NULL
		AND
		((CURRENT_DATE - ist.issued_date) -30 ) >=1

ORDER BY 1;
```

**Task 14: Update Book Status on Return**  
Write a query to update the status of books in the books table to "Yes" when they are returned (based on entries in the return_status table).

```sql

CREATE OR REPLACE PROCEDURE add_returned_book(p_return_id VARCHAR(15),p_issued_id VARCHAR(15),p_book_quality VARCHAR(15))
LANGUAGE plpgsql
AS $$

DECLARE
	v_isbn VARCHAR(25);
	v_book_name VARCHAR(75);
BEGIN
	-- inserting into returns table based on user input
	INSERT INTO return_status (return_id, issued_id, return_date, book_quality)
	VALUES
	(p_return_id, p_issued_id, CURRENT_DATE, p_book_quality);

	SELECT
		issued_book_isbn,
		issued_book_name
		INTO
		v_isbn,
		v_book_name
	FROM issued_status
	WHERE issued_id = p_issued_id;

	UPDATE books
	SET status = 'yes'
	WHERE  isbn = v_isbn;

	RAISE NOTICE 'Thank you for returning the book: %', v_book_name;
END;
$$


-- Testing FUNCTION add_return_records

issued_id = IS135
ISBN = WHERE isbn = '978-0-307-58837-1'

SELECT * FROM books
WHERE isbn = '978-0-307-58837-1';

SELECT * FROM issued_status
WHERE issued_book_isbn = '978-0-307-58837-1';

SELECT * FROM return_status
WHERE issued_id = 'IS135';

-- calling function
CALL add_return_records('RS138', 'IS135', 'GOOD');

issued_id = IS140
isbn = '978-0-330-25864-8'

SELECT * FROM books
WHERE isbn = '978-0-330-25864-8'

SELECT * FROM issued_status
WHERE issued_book_isbn =  '978-0-330-25864-8'

SELECT * FROM return_status
WHERE issued_id = 'IS140'

-- Calling Function
CALL add_returned_book('RS148', 'IS140', 'GOOD');


```

**Task 15: Branch Performance Report**  
Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned,
and the total revenue generated from book rentals.

```sql
CREATE TABLE branch_reports
AS
	SELECT
		b.branch_id,
		b.manager_id,
		COUNT(ist.issued_id) AS number_book_issued,
		COUNT(rs.return_id) AS number_book_returned,
		SUM(bk.rental_price) AS total_revenue
	FROM issued_status AS ist

	JOIN employees e
	ON e.emp_id = ist.issued_emp_id

	JOIN branch AS b
	ON e.branch_id=b.branch_id

	LEFT JOIN return_status AS rs
	ON rs.issued_id = ist.issued_id

	JOIN books AS bk
	ON ist.issued_book_isbn = bk.isbn

	GROUP BY b.branch_id,
		     b.manager_id
	ORDER BY 1;

SELECT * FROM branch_reports;
```

**Task 16: CTAS: Create a Table of Active Members**  
Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the
last 6 months.

```sql

CREATE TABLE active_member
AS
SELECT * FROM members
WHERE member_id IN (SELECT
						DISTINCT issued_member_id
					FROM issued_status
					WHERE
	   					 issued_date >= CURRENT_DATE - INTERVAL '2 month'
					);

SELECT * FROM active_member;

```

**Task 17: Find Employees with the Most Book Issues Processed**  
Write a query to find the top 3 employees who have processed the most book issues.
Display the employee name, number of books processed, and their branch

```sql
SELECT
	  e.emp_name,
	  b.*,
	  COUNT (ist.issued_id) AS book_issue_frequency
FROM issued_status AS ist
	JOIN employees AS e
	ON e.emp_id = ist.issued_emp_id

	JOIN branch AS b
	ON b.branch_id = e.branch_id

GROUP BY 1,2
ORDER BY COUNT (ist.issued_id) DESC
LIMIT 3;

```

**Task 18: Identify Members Issuing High-Risk Books**  
Write a query to identify members who have issued books with the status "damaged" in the books table.
Display the member name, book title, and the number of times they've issued damaged books.

```sql
CREATE TABLE member_issued_damage_books
AS
	SELECT
    	m.member_id,
    	m.member_name,
    	bk.book_title,
        COUNT(rst.book_quality) AS times_issued_damaged
FROM
	     issued_status AS ist
	JOIN return_status AS rst
    ON rst.issued_id = ist.issued_id
	JOIN books AS bk
    ON bk.isbn = ist.issued_book_isbn
	JOIN members AS m
    ON m.member_id = ist.issued_member_id

WHERE rst.book_quality = 'Damaged'
GROUP BY m.member_id, m.member_name, bk.book_title
ORDER BY times_issued_damaged DESC;

```

**Task 19: Stored Procedure**
Objective:
Create a stored procedure to manage the status of books in a library system.
Description:
Write a stored procedure that updates the status of a book in the library based on its issuance.
The procedure should function as follows: The stored procedure should take the book_id as an input parameter.
The procedure should first check if the book is available (status = 'yes').
If the book is available, it should be issued, and the status in the books table should be updated to 'no'.
If the book is not available (status = 'no'),
the procedure should return an error message indicating that the book is currently not available.

```sql

CREATE OR REPLACE PROCEDURE issue_book(p_issued_id VARCHAR(10), p_issued_member_id VARCHAR(30), p_issued_book_isbn VARCHAR(30), p_issued_emp_id VARCHAR(10))
LANGUAGE plpgsql
AS $$

DECLARE
-- all the variabable
    v_status VARCHAR(10);

BEGIN
-- all the code
    -- checking if book is available 'yes'
    SELECT
        status
        INTO
        v_status
    FROM books
    WHERE isbn = p_issued_book_isbn;

    IF v_status = 'yes' THEN

        INSERT INTO issued_status(issued_id, issued_member_id, issued_date, issued_book_isbn, issued_emp_id)
        VALUES
        (p_issued_id, p_issued_member_id, CURRENT_DATE, p_issued_book_isbn, p_issued_emp_id);

        UPDATE books
            SET status = 'no'
        WHERE isbn = p_issued_book_isbn;



        RAISE NOTICE 'Book records added successfully for book isbn : %', p_issued_book_isbn;


    ELSE
        RAISE NOTICE 'Sorry to inform you the book you have requested is unavailable book_isbn: %', p_issued_book_isbn;
    END IF;
END;
$$


-- Testing The function
SELECT * FROM books;
-- "978-0-553-29698-2" -- yes
-- "978-0-375-41398-8" -- no
SELECT * FROM issued_status;

CALL issue_book('IS155', 'C108', '978-0-553-29698-2', 'E104');
CALL issue_book('IS156', 'C108', '978-0-375-41398-8', 'E104');

```

## Reports

- **Database Schema**: Detailed table structures and relationships.
- **Data Analysis**: Insights into book categories, employee salaries, member registration trends, and issued books.
- **Summary Reports**: Aggregated data on high-demand books and employee performance.

## Conclusion

This project demonstrates the application of my SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.

Thank you!
