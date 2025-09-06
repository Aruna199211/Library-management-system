# Library Management System using SQL Project --P2

## Project Overview

**Project Title**: Library Management System  
**Level**: Intermediate  
**Database**: `library_project`

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.

## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure

### 1. Database Setup
library ERD.png

- **Database Creation**: Created a database named `library_project`.
- **Table Creation**: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

```sql
-- creating branch table

drop table if exists branch;
create table branch 
(branch_id varchar(10) primary key,
manager_id varchar(10),
branch_address varchar(20),
contact_no varchar(10) );

alter table branch
modify column contact_no varchar(20);

-- creating employee table
drop table if exists employee;
create table employee
(emp_id varchar(10) primary key,
emp_name varchar(20),
position varchar(10),
salary int,
branch_id varchar(10) ); -- fk

-- creating table books
drop table if exists books;
create table books
( isbn varchar(20) primary key,
book_title varchar(60),
category varchar(20),
rental_price float,
status varchar(5),
author varchar(60),
publisher varchar(60) );

-- ( i accidently put wrong data in a wrong table to undo that i did this)
-- Step 1: Disable foreign key checks temporarily (only if needed)
SET FOREIGN_KEY_CHECKS = 0;
-- now u can drop table
-- Step 2: Re-enable foreign key checks

set foreign_key_checks=1;
-- creating table members
drop table if exists members;
create table members
(member_id varchar(10) primary key,
member_name varchar(20),
member_address varchar(15),
reg_date date );

-- creating table issued_id
drop table if exists issued_id;
create table issued_id
( issued_id varchar(10) primary key, 
issued_member_id varchar(10), -- fk
issued_book_name varchar(60),
issued_date date,
issued_book_isbn varchar(25), -- fk
issued_emp_id varchar(10) ); -- fk

-- creating table return_status
drop table if exists return_status;
create table return_status
(return_id varchar(10) primary key,
issued_id varchar(10), -- fk
return_book_name varchar(75),
return_date	date,
return_book_isbn varchar(20) );

-- adding foreign key
alter table issued_id
add constraint fk_members
foreign key (issued_member_id)
references members(member_id);

alter table issued_id
add constraint fk_book
foreign key (issued_book_isbn)
references books(isbn);

alter table issued_id
add constraint fk_employees
foreign key (issued_emp_id)
references employee(emp_id);

alter table employee
add constraint fk_branch
foreign key (branch_id)
references branch(branch_id);

alter table return_status
add constraint fk_issued_id
foreign key (issued_id)
references issued_id(issued_id);

```

### 2. CRUD Operations

- **Create**: Inserted sample records into the `books` table.
- **Read**: Retrieved and displayed data from various tables.
- **Update**: Updated records in the `employees` table.
- **Delete**: Removed records from the `members` table as needed.

**Task 1. Create a New Book Record**
-- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

```sql
insert into books (isbn,book_title,category,rental_price,status,author,publisher)
values("978-1-60129-456-2",'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
```
**Task 2: Update an Existing Member's Address**

```sql
select * from members;
update members set member_address='125 Oak St'
where member_id='C103';
```

**Task 3: Delete a Record from the Issued Status Table**
-- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.

```sql
select * from issued_status;
delete from issued_status
where issued_id='IS121';
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
select issued_emp_id,
count(*)
from issued_status
group by issued_emp_id
having count(*)>1;
```

### 3. CTAS (Create Table As Select)

- **Task 6: Create Summary Tables**: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt**

```sql
create table book_count
as
select b.isbn,
b.book_title, 
count(i.issued_id) as no_issued
from books as b
join
issued_status as i
on i.issued_book_isbn=b.isbn
group by b.isbn,b.book_title;

select * from book_count;
```


### 4. Data Analysis & Findings

The following SQL queries were used to address specific questions:

Task 7. **Retrieve All Books in a Specific Category**:

```sql
SELECT * FROM books
WHERE category = 'Children';
```

8. **Task 8: Find Total Rental Income by Category**:

```sql
select b.category,
sum(b.rental_price),
count(*)
from books as b
join issued_status as i
on i.issued_book_isbn=b.isbn
group by b.category;
```

9. **List Members Who Registered in the Last 180 Days**:
```sql
select * from members;
insert into members(member_id,member_name,member_address,reg_date) values
('C120','Sam','145 main st','2025-06-11'),
('C121','john','133 main st','2025-05-09');

select * from members
where reg_date>= curdate()-interval 180 day;
```

10. **List Employees with Their Branch Manager's Name and their branch details**:

```sql
select e1.*,
b.manager_id,
e2.emp_name as manager
from employee as e1
join branch as b
on b.branch_id=e1.branch_id
join employee as e2
on b.manager_id=e2.emp_id;
```

Task 11. **Create a Table of Books with Rental Price Above a Certain Threshold**:
```sql
create table book_greater_7 as
select * from books where rental_price>7;
select * from book_greater_7;;
```

Task 12: **Retrieve the List of Books Not Yet Returned**
```sql
select i.issued_book_name
from issued_status as i 
left join return_status as r
on r.issued_id=i.issued_id
where r.issued_id is null;
```

## Advanced SQL Operations

-- to perform advance Analysis I added some data

-- INSERT INTO book_issued in last 30 days
-- SELECT * from employees;
-- SELECT * from books;
-- SELECT * from members;
-- SELECT * from issued_status;


INSERT INTO issued_status(issued_id, issued_member_id, issued_book_name, issued_date, issued_book_isbn, issued_emp_id)
VALUES
('IS151', 'C118', 'The Catcher in the Rye', CURRENT_DATE - INTERVAL 24 day,  '978-0-553-29698-2', 'E108'),
('IS152', 'C119', 'The Catcher in the Rye', CURRENT_DATE - INTERVAL 13 day,  '978-0-553-29698-2', 'E109'),
('IS153', 'C106', 'Pride and Prejudice', CURRENT_DATE - INTERVAL 7 day,  '978-0-14-143951-8', 'E107'),
('IS154', 'C105', 'The Road', CURRENT_DATE - INTERVAL 32 day,  '978-0-375-50167-0', 'E101');

-- Adding new column in return_status

ALTER TABLE return_status
ADD Column book_quality VARCHAR(15) DEFAULT('Good');

UPDATE return_status
SET book_quality = 'Damaged'
WHERE issued_id 
    IN ('IS112', 'IS117', 'IS118');
SELECT * FROM return_status;

**Task 13: Identify Members with Overdue Books**  
Write a query to identify members who have overdue books (assume a 30-day return period). Display the member's_id, member's name, book title, issue date, and days overdue.

```sql
select i.issued_member_id, 
 m.member_name, 
 b.book_title,
 i.issued_date,
 datediff(curdate(),i.issued_date) as over_dues_days
 from issued_status as i 
 join members as m
 on m.member_id=i.issued_member_id
 join books as b
 on b.isbn=i.issued_book_isbn
 left join return_status as r
 on r.issued_id=i.issued_id
 where r.return_date is null
 and datediff(curdate(),i.issued_date)>30
 order by i.issued_member_id;
```


**Task 14: Update Book Status on Return**  
Write a query to update the status of books in the books table to "Yes" when they are returned (based on entries in the return_status table).


```sql

select * from issued_status;
-- store procedure
drop procedure if exists add_return_records;
delimiter $$
create procedure add_return_records
(in p_return_id varchar(10),
in p_issued_id varchar(10),
in p_book_quality varchar(15))


begin
declare v_isbn varchar(50);
declare v_book_name varchar(60);

-- all your logic and code
-- inserting into returns based on user input
insert into return_status(return_id,issued_id,return_date,book_quality)
values
(p_return_id,p_issued_id,curdate(),p_book_quality);

select issued_book_isbn,
issued_book_name
into v_isbn, v_book_name
 from issued_status
where issued_id=p_issued_id;

update books
set status='Yes'
where isbn=v_isbn;

select concat('thankyou for returning the book: %', v_book_name) as message;



end;
$$
delimiter ;

call add_return_records()

-- testing function add_return_records
issued_id=IS135
ISBN=where isbn='978-0-307-58837-1'
select * from books
where isbn='978-0-375-41398-8';

select * from issued_status
where issued_book_isbn='978-0-375-41398-8';

select * from return_status
where issued_id='IS134';

-- calling function or adding records
call add_return_records('RS138','IS135','Good');
call add_return_records('RS140','IS134','Good');

```




**Task 15: Branch Performance Report**  
Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.

```sql
create table branch_reports as
select b.branch_id,
b.manager_id,
count(i.issued_id) as no_of_books_issued,
count(r.return_id) as no_book_return,
sum(bk.rental_price) as total_revenue
from issued_status as i
join employee as e
on e.emp_id=i.issued_emp_id
join branch as b
on e.branch_id=b.branch_id
left join return_status as r
on r.issued_id=i.issued_id
join books as bk
on i.issued_book_isbn=bk.isbn
group by b.branch_id,b.manager_id;
select * from branch_reports;
```

**Task 16: CTAS: Create a Table of Active Members**  
Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the last 2 months.

```sql
create table active_members as
select * from members
where member_id in (
select distinct issued_member_id
from issued_status
where issued_date>=curdate()-interval 6 month);
select * from active_members;
```


**Task 17: Find Employees with the Most Book Issues Processed**  
Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch.

```sql
select e.emp_name,
b.*,
count(i.issued_id) as no_of_book
from employee as e
join issued_status as i
on i.issued_emp_id=e.emp_id
join branch as b
on e.branch_id=b.branch_id
group by e.emp_name,
b.branch_id
order by no_of_book desc
limit 3;
```

**Task 18: Identify Members Issuing High-Risk Books**  
Write a query to identify members who have issued books more than twice with the status "damaged" in the books table. Display the member name, book title, and the number of times they've issued damaged books.    

```sql

select m.member_name,
b.book_title,
count(r.book_quality) as damaged_book
from issued_status as i
join books as b
on b.isbn=i.issued_book_isbn
join members as m
on i.issued_member_id=m.member_id
join return_status as r
on i.issued_id=r.issued_id
where r.book_quality='Damaged'
group by m.member_name,
b.book_title;
```

**Task 19: Stored Procedure**
Objective:
Create a stored procedure to manage the status of books in a library system.
Description:
Write a stored procedure that updates the status of a book in the library based on its issuance. The procedure should function as follows:
The stored procedure should take the book_id as an input parameter.
The procedure should first check if the book is available (status = 'yes').
If the book is available, it should be issued, and the status in the books table should be updated to 'no'.
If the book is not available (status = 'no'), the procedure should return an error message indicating that the book is currently not available.

```sql

select * from books;
drop procedure if exists issue_book;
delimiter $$
create procedure issue_book(in p_issued_id varchar(10),
in p_issued_member_id varchar(10),
in p_issued_book_isbn varchar(25),
in p_issued_emp_id varchar(10))
begin
declare v_status varchar(5);  -- all the variable

-- all the code and checking if book is available 'yes'
select status into v_status
from books
where isbn=p_issued_book_isbn;
if v_status='yes' then
insert into issued_status(issued_id,issued_member_id,issued_date,issued_book_isbn,issued_emp_id)
values(p_issued_id,p_issued_member_id,current_date(),p_issued_book_isbn,p_issued_emp_id);
update books set status='no' where isbn=P_issued_book_isbn;
select concat('Book records added successfully for book isbn : %',p_issued_book_isbn) as message;

else
select concat('Sorry to inform you the book you have requested is unavailable book_isbn:%',p_issued_book_isbn) as message;

end if;
end $$

select * from books;
-- '978-0-553-29698-2' -- yes
-- '978-0-375-41398-8'  --no
select * from issued_status;
select * from books where isbn= '978-0-375-41398-8';
call issue_book('IS155','C108','978-0-553-29698-2','E104');
call issue_book('IS156','C108','978-0-375-41398-8','E104');

```



**Task 20: Create Table As Select (CTAS)**
Objective: Create a CTAS (Create Table As Select) query to identify overdue books and calculate fines.

Description: Write a CTAS query to create a new table that lists each member and the books they have issued but not returned within 30 days. The table should include:
    The number of overdue books.
    The total fines, with each day's fine calculated at $0.50.
    The number of books issued by each member.
    The resulting table should show:
    Member ID
    Number of overdue books
    Total fines

```sql

create table overdue_books_and_cal_fines as 
select m.member_id,
count
(case
when
datediff(r.return_date,i.issued_date)>30 then 1 end) as overdue_duration_of_books,
sum
(case
when
datediff(r.return_date,i.issued_date)>30
then
(datediff(r.return_date,i.issued_date)-30)*0.50 else 0 end) as total_fine,
count(i.issued_id)as total_books_issued
from issued_status as i
join members as m
on i.issued_member_id=m.member_id
join return_status as r
on i.issued_id=r.issued_id
group by m.member_id;

select * from overdue_books_and_cal_fines;

## Reports

- **Database Schema**: Detailed table structures and relationships.
- **Data Analysis**: Insights into book categories, employee salaries, member registration trends, and issued books.
- **Summary Reports**: Aggregated data on high-demand books and employee performance.

## Conclusion

This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.


## Author - ARUNA SAINI

This project showcases SQL skills essential for database management and analysis. 
