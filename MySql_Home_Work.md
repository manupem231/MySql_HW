

```python
use sakila;

#1a. Display the first and last names of all actors from the table actor.

select first_name, last_name from actor;

#1b. Display the first and last name of each actor in a single column in upper case letters. Name the column Actor Name.

select concat(first_name, " ", last_name) as Actor_Name from actor;

#2a. You need to find the ID number, first name, and last name of an actor, of whom you know only the first name, "Joe." What is one query would you use to obtain this information?

select actor_id, first_name, last_name from actor
where first_name = 'Joe';

#2b. Find all actors whose last name contain the letters GEN:

select * from actor
where last_name like '%GEN%';

#2c. Find all actors whose last names contain the letters LI. This time, order the rows by last name and first name, in that order:

select * from actor
where last_name like '%LI%'
order by last_name, first_name;

#2d. Using IN, display the country_id and country columns of the following countries: Afghanistan, Bangladesh, and China:

select country_id, country from country
where country in ('Afghanistan', 'Bangladesh', 'China');

#3a. Add a middle_name column to the table actor. Position it between first_name and last_name. Hint: you will need to specify the data type.

alter table actor
add middle_name varchar(50) after first_name;

#3b. You realize that some of these actors have tremendously long last names. Change the data type of the middle_name column to blobs.

alter table actor modify middle_name blob;

#3c. Now delete the middle_name column.

alter table actor drop column middle_name;

#4a. List the last names of actors, as well as how many actors have that last name.

select last_name, count(last_name) as list_of_actors from actor
group by last_name;

#4b. List last names of actors and the number of actors who have that last name, but only for names that are shared by at least two actors

select last_name, count(last_name) from actor
group by last_name
having count(last_name) >= 2;

#4c. Oh, no! The actor HARPO WILLIAMS was accidentally entered in the actor table as GROUCHO WILLIAMS, the name of Harpo's second cousin's husband's yoga teacher. Write a query to fix the record.

update actor
set first_name = 'HARPO'
where first_name = 'GROUCHO' and last_name = 'WILLIAMS';

#4d. Perhaps we were too hasty in changing GROUCHO to HARPO. It turns out that GROUCHO was the correct name after all! In a single query, if the first name of the actor is currently HARPO, change it to GROUCHO. Otherwise, change the first name to MUCHO GROUCHO, as that is exactly what the actor will be with the grievous error. BE CAREFUL NOT TO CHANGE THE FIRST NAME OF EVERY ACTOR TO MUCHO GROUCHO, HOWEVER! (Hint: update the record using a unique identifier.)

update actor
set first_name = case
	when (first_name = 'HARPO' and last_name = 'WILLIAMS') then 'GROUCHO'
    when (first_name != 'HARPO' and last_name = 'WILLIAMS') then 'MUCHO GROUCHO'
end
where (first_name = 'HARPO' and last_name = 'WILLIAMS') or (first_name != 'HARPO' and last_name = 'WILLIAMS');

#5a. You cannot locate the schema of the address table. Which query would you use to re-create it? HINT: CHECK THIS OUT

show create table address;

#6a. Use JOIN to display the first and last names, as well as the address, of each staff member. Use the tables staff and address:

select s.first_name, s.last_name, a.address
from staff s
join address a on
s.address_id = a.address_id;

#6b. Use JOIN to display the total amount rung up by each staff member in August of 2005. Use tables staff and payment.

select s.first_name, s.last_name, concat('$', format(sum(p.amount), 2)) as amount_in_aug_2005
from staff s
join payment p on s.staff_id = p.staff_id
where p.payment_date like '2005-08-%'
group by p.staff_id;

#6c. List each film and the number of actors who are listed for that film. Use tables film_actor and film. Use inner join.

select f.title, count(fa.actor_id) as no_of_actors
from film f
join film_actor fa on
f.film_id = fa.film_id
group by f.title;

#6d. How many copies of the film Hunchback Impossible exist in the inventory system?

select count(inventory_id) as no_of_copies from inventory
where film_id in (select film_id from film where title = 'Hunchback Impossible');

#6e. Using the tables payment and customer and the JOIN command, list the total paid by each customer. List the customers alphabetically by last name:
#  ![Total amount paid](Images/total_payment.png)

select c.first_name, c.last_name, concat('$', format(sum(p.amount), 2)) as total_paid
from customer c
join payment p on
c.customer_id = p.customer_id
group by p.customer_id
order by c.last_name;

#7a. The music of Queen and Kris Kristofferson have seen an unlikely resurgence. As an unintended consequence, films starting with the letters K and Q have also soared in popularity. Use subqueries to display the titles of movies starting with the letters K and Q whose language is English.

select title from film
where title like 'Q%' or title like 'K%' and language_id in(
select language_id from language where name = 'English');

#7b. Use subqueries to display all actors who appear in the film Alone Trip.

select first_name, last_name from actor where actor_id in(
select actor_id from film_actor where film_id in(
select film_id from film where title = 'Alone Trip'));

#7c. You want to run an email marketing campaign in Canada, for which you will need the names and email addresses of all Canadian customers. Use joins to retrieve this information.

select first_name, last_name, email from customer where address_id in(
select address_id from address where city_id in(
select city_id from city where country_id in(
select country_id from country where country = 'canada')));

#7d. Sales have been lagging among young families, and you wish to target all family movies for a promotion. Identify all movies categorized as famiy films.

select title from film where film_id in(
select film_id from film_category where category_id in(
select category_id from category where name = 'family')); 


#7e. Display the most frequently rented movies in descending order.

select f.title, count(r.inventory_id) as frequency_of_rented_movies
from film f, rental r, payment p, inventory i
where
f.film_id = i.film_id and
i.inventory_id = r.inventory_id and
r.rental_id = p.rental_id
group by f.title
order by frequency_of_rented_movies desc;

#7f. Write a query to display how much business, in dollars, each store brought in.

select s.store_id, concat('$', format(sum(p.amount), 2)) as total_business  
from staff s, payment p, store st
where
s.store_id = st.store_id and
p.staff_id = s.staff_id
group by s.staff_id;

#7g. Write a query to display for each store its store ID, city, and country.

select s.store_id, c.city, cc.country
from store s, city c, country cc, address a
where
s.address_id = a.address_id and
a.city_id = c.city_id and
c.country_id = cc.country_id;

#7h. List the top five genres in gross revenue in descending order. (Hint: you may need to use the following tables: category, film_category, inventory, payment, and rental.)

select c.name as categoty, sum(p.amount) as gross_revenue from
category c, film_category fc, inventory i, rental r, payment p
where
c.category_id = fc.category_id and
fc.film_id = i.film_id and
i.inventory_id = r.inventory_id and
r.customer_id = p.customer_id 
group by c.name
order by gross_revenue desc limit 5;

#8a. In your new role as an executive, you would like to have an easy way of viewing the Top five genres by gross revenue. Use the solution from the problem above to create a view. If you haven't solved 7h, you can substitute another query to create a view.

create view category_view as
select c.name as categoty, sum(p.amount) as gross_revenue from
category c, film_category fc, inventory i, rental r, payment p
where
c.category_id = fc.category_id and
fc.film_id = i.film_id and
i.inventory_id = r.inventory_id and
r.customer_id = p.customer_id 
group by c.name
order by gross_revenue desc limit 5;

#8b. How would you display the view that you created in 8a?

select * from category_view;

#8c. You find that you no longer need the view top_five_genres. Write a query to delete it.

drop view category_view;
```
