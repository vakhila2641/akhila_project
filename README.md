# akhila_project
# Air Cargo SQL Analysis Project
This project performs an in-depth analysis of air cargo shipment data using **SQL**. It demonstrates:

- Schema creation with constraints
- Sample data loading from Excel/CSV
- Data analysis using advanced SQL queries, functions, procedures, views, and indexing

 Project Structure
 akhila_project/
├── data/
│ └── air_cargo_data.xlsx
├── sql/
│ ├── schema.sql
│ ├── insert-data.sql
│ └── queries.sql
└── README.md

## Key Features

### Schema & Table Creation
- Create schema: `AIR`
- Create `route_details` table with:
  - Unique and check constraints
  - Validations on flight numbers and distances

### Advanced SQL Queries
- Passenger analysis by route ID
- Business class revenue
- Customer full name concatenation
- Brand-specific customer filters
- Use of window functions (`ROW_NUMBER`, `MAX OVER`)
- Use of `ROLLUP` for aggregations

### Stored Procedures & Functions
- **Dynamic Procedures**:
  - `Flight_route_range`: route-based passenger lookup with error handling
  - `Distance_Travelled`, `DT_grouping`: analyze routes based on miles
- **Functions**:
  - `ID_Name`, `cs`: get customer name/services logic
- **View Creation**: only business-class customers
- **Cursor Usage**: fetch specific customer record (e.g., last name ending with "Scott")

### Performance
- Use of indexes and `EXPLAIN ANALYZE` to improve performance

## Technologies Used

- MySQL
- Excel (for data input)
- SQL Workbench 


## Future Enhancements

- Visual dashboards using Power BI or Tableau
- Integration with Python for automation
- Data validation with triggers

> Built as a part of my learning journey in SQL and data analysis. Feedback is welcome!


/* **Air Cargo Analysis Project** */
**Create a schema AIR**
create schema Air;

/* 2.	Write a query to create a route_details table using suitable data types for the fields,
 such as route_id, flight_num, origin_airport, destination_airport, aircraft_id, and distance_miles. 
 Implement the check constraint for the flight number and unique constraint for the route_id fields.
 Also, make sure that the distance miles field is greater than 0. */



Create table route_details(
route_id int unique, 
flight_num int check((substr(flight_num,1,2) = 11)), 
origin_airport text, 
destination_airport text, 
aircraft_id text, 
distance_miles int check( distance_miles >0)
);
 use air;
Select * from customer;
Select * from passengers_on_flights;
Select * from routes;
Select * from ticket_details;
Select * from Route_details;

/* 3.	Write a query to display all the passengers (customers) who have travelled in routes 01 to 25. 
Take data from the passengers_on_flights table. */

Select * from passengers_on_flights where route_id between 01 and 25;
Select customer_id from passengers_on_flights where route_id between 01 and 25;

/* 4.	Write a query to identify the number of passengers and total revenue in business class 
from the ticket_details table. */
Select * from ticket_details;
Select count(customer_id) as total_passengers, sum(Price_per_ticket) as Total_revenue 
from ticket_details where class_id = 'Bussiness';
-- Using window function
select count(customer_id) as Total_Cus,sum(Price_per_ticket) as Total_Revenue,
row_number() over () as RN
 from air.ticket_details where class_id= 'Bussiness';
 
/* 5.	Write a query to display the full name of the customer by extracting the first name 
and last name from the customer table.*/
Select * from air.customer;
Select concat(First_name,' ',Last_name) as Customer_name from air.customer;
/* 6.	Write a query to extract the customers who have registered and booked a ticket. 
Use data from the customer and ticket_details tables.*/
Select * from air.ticket_details;
Select c.Customer_id, concat(c.First_name,' ',c.Last_name) as Customer_name,
count(t.no_of_tickets) as Total_Tic
from air.customer as c 
inner join air.ticket_details as t 
on c.Customer_id = t.Customer_id group by c.Customer_id, customer_name;

/* 7.	Write a query to identify the customer’s first name and last name based on their customer ID 
and brand (Emirates) from the ticket_details table. */
Select * from air.ticket_details where brand = 'Emirates';

Select c.customer_id,c.first_name,c.Last_name 
from air.customer c inner join air.ticket_details t 
on c.customer_id = t.customer_id where brand = 'Emirates';

drop function ID_Name;

Delimiter $$
Create function ID_Name (P_c_id int)
returns varchar(100)
Deterministic
	Begin
		Declare F_name  varchar(100);
		Select C_name into F_name from (Select concat(c.First_name, ' ',c.last_name) C_Name,c.customer_id from air.Customer as C
		inner join air.ticket_details as t
		on c.customer_id = t.customer_id where t.brand= 'Emirates' ) as p where p.customer_id = p_c_id;
        return F_name; 
		
	end $$
    
    Drop function ID_Name;
    
  Select Id_name(27);
  use air;

Select * from customer;
Select * from Ticket_details;
use air ;

Delimiter $$
Create function Brand(Brand_name varchar(50))
Returns int
Deterministic
Begin 
	Declare c_id int;
		Select customer_id into c_id from air.ticket_details
        where brand = Brand_name
        limit 1;
        return c_id;
        end ; $$
 Drop   function  Brand;    
Select Brand('Jet Airways');  
Select * from ticket_details;  


DELIMITER $$

CREATE PROCEDURE GetCustomerIdsByBrand(IN Brand_name VARCHAR(50))
BEGIN
    SELECT customer_id
    FROM air.ticket_details
    WHERE brand = Brand_name;
END $$

DELIMITER ;
Select customer_id from air.ticket_details where brand = 'Emirates';
Call GetCustomerIdsByBrand('Emirates');

/*8.	Write a query to identify the customers who have travelled by Economy Plus class using Group By 
and Having clause on the passengers_on_flights table. */
Select * from passengers_on_flights where class_id = 'Economy Plus';
Select 
count(*) as C_count,
c.customer_id, concat(c.first_name,' ',c.last_name) as Full_name, p.class_id from 
air.customer c inner join air.passengers_on_flights p
on c.customer_id = p.customer_id 
group by c.customer_id,Full_name, p.class_id
having class_id = 'Economy Plus';

/* 9.	Write a query to identify whether the revenue 
has crossed 10000 using the IF clause on the ticket_details table.*/

Select * from ticket_details;
Select 
if(
sum(price_per_ticket) >10000, 'yes','no') as Rev_check
from ticket_details;

/*11.	Write a query to find the maximum ticket price for each class
 using window functions on the ticket_details table. */
 
 Select * from ticket_details;
 
 Select class_id, Brand, Price_per_ticket, 
 Max(Price_per_ticket) over(partition by class_id) as Max_tic_price 
 from ticket_details order by Max_tic_price,class_id,Brand;
 
/* 12.	Write a query to extract the passengers whose route ID is 4 by improving 
the speed and performance of the passengers_on_flights table.*/

Select * from passengers_on_flights;
create index The_route_id on passengers_on_flights(route_id);
Select * from air.passengers_on_flights where route_id = 4;

/* For the route ID 4, write a query to view the execution plan of the passengers_on_flights table.*/

Explain analyze select * FROM air.passengers_on_flights WHERE route_id = '4';
 
 /* 15.	Write a query to create a view with only business class customers along with the brand of airlines. */
 
 Select * from ticket_details;
 create view BC as
 Select * from ticket_details where class_id = 'Bussiness';
 Select * from Bc;
 
 /* 16.	Write a query to create a stored procedure to get the details of all passengers flying between a range of routes 
 defined in run time. Also, return an error message if the table doesn't exist.*/

Select * from passengers_on_flights;
DELIMITER $$
CREATE PROCEDURE Flight_route_range(in flight_route_id1 INT, in flight_route_id2 INT)
BEGIN

    DECLARE passengers_table_exists INT;

    SELECT COUNT(*) INTO passengers_table_exists
    FROM information_schema.tables
    WHERE table_schema = database() AND table_name = 'passengers_on_flights';

   
    IF passengers_table_exists = 0 THEN
        SELECT 'Error: Table does not exist. ' AS Message;

    ELSE

        SET @num_rows = (
            SELECT COUNT(*)
            FROM passengers_on_flights AS p
            WHERE p.route_id >= flight_route_id1 AND p.route_id <= flight_route_id2 
        );

        IF @num_rows = 0 THEN
            SELECT 'Error: No data found for the specified flight route range.' AS Message;
        ELSE

            SELECT p.route_id,
                   p.depart,
                   p.arrival,
                   p.seat_num,
                   c.*
            FROM passengers_on_flights AS p
            INNER JOIN customer AS c ON p.customer_id = c.customer_id
            WHERE p.route_id >= flight_route_id1 AND p.route_id <= flight_route_id2 
            ORDER BY p.route_id;
        END IF;
    END IF;
END $$
DELIMITER ;

CALL Flight_route_range(1,5);

/* 17.	Write a query to create a stored procedure that extracts 
all the details from the routes table where the travelled distance is more than 2000 miles.*/

Select * from routes;
Delimiter $$
create Procedure Distance_Travelled()
Deterministic
Begin
	Declare d_mile int;
	Select * from routes where distance_miles = D_mile or distance_miles >2000;
End $$    
Drop procedure Distance_travelled;
Call  Distance_Travelled();

-- another way of code
Delimiter $$
Create procedure DT()
deterministic
Begin
Select * from routes where distance_miles > 2000;
End $$
Call DT();  

/* 18.	Write a query to create a stored procedure that groups the distance travelled by each flight 
into three categories. The categories are,  short distance travel (SDT) for >=0 AND <= 2000 miles, 
intermediate distance travel (IDT) for >2000 AND <=6500, and long-distance travel (LDT) for >6500.*/

Delimiter $$
Create Procedure DT_grouping()
Deterministic
Begin
	Select *,
    case when distance_miles between 0 and 2000 then 'Short-DT'
		 when distance_miles between 2000 AND 6500 then 'Intermediate-DT'
         when distance_miles >= 6500 then 'Long-DT'
     else 'none'
     end as DTS
       from routes;
 end $$      
Call DT_grouping();

/*19.	Write a query to extract ticket purchase date, customer ID, class ID and specify if the 
complimentary services are provided for the specific class using a stored function in stored 
procedure on the ticket_details table. 
Condition: If the class is Business and Economy Plus, then complimentary services are given as Yes, else it is No */

Select * from ticket_details order by p_date ;

Delimiter ^^
Create function cs(p_customer_id int, p_p_date text)
returns varchar(50)
deterministic
begin
	declare v_services varchar(50);
	select 	
		case when class_id = 'Bussiness' then 'Yes'
			 when class_id = 'Economy PLus' then 'No'
             else 'No'
         end as Services into v_services from ticket_details
         where customer_id = p_customer_id and p_date = p_p_date;
      return v_services;   
 end ^^
 
 Select cs(customer_id, p_date) from ticket_details;

Delimiter &&
create procedure Complimentary_services()
begin 
	Select p_date, customer_id, class_id, cs(customer_id, p_date)  from ticket_details;
end ;

call Complimentary_services();
use air;

/* 20.	Write a query to extract the first record of the customer whose last name ends with 
Scott using a cursor from the customer table. */

Select * from customer where last_name like  'scott%' limit 1;
delimiter //
create procedure Name_by_cursor()
Begin	
	Declare v_First_Name varchar(50);
    Declare v_Last_Name varchar(50);
    Declare M_cursor Cursor for Select first_name,last_name from customer
			where last_name = 'Scott';
	open M_cursor;
    Repeat fetch M_cursor into v_First_Name,v_Last_Name;
    until v_first_Name = 0 end repeat;
    Select v_First_Name as first_name, v_Last_Name as last_name;
    close m_cursor; end ;
    //
call  Name_by_cursor();    
