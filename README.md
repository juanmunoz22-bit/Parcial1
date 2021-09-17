# Parcial1

BASE DE DATOS


BEGIN;


CREATE TABLE IF NOT EXISTS public.address
(
    address_id integer NOT NULL,
    line1 character varying(250) NOT NULL,
    line2 character varying(250) NOT NULL,
    city_id integer NOT NULL,
    PRIMARY KEY (address_id)
);

CREATE TABLE IF NOT EXISTS public.branchoffice
(
    branch_id integer NOT NULL,
    name character varying(250) NOT NULL,
    address_id integer NOT NULL,
    PRIMARY KEY (branch_id)
);

CREATE TABLE IF NOT EXISTS public.city
(
    city_id integer NOT NULL,
    name character varying(250) NOT NULL,
    country_id integer NOT NULL,
    PRIMARY KEY (city_id)
);

CREATE TABLE IF NOT EXISTS public.country
(
    country_id integer NOT NULL,
    name character varying(250) NOT NULL,
    PRIMARY KEY (country_id)
);

CREATE TABLE IF NOT EXISTS public.department
(
    department_id integer NOT NULL,
    name character varying(250) NOT NULL,
    PRIMARY KEY (department_id)
);

CREATE TABLE IF NOT EXISTS public.employee
(
    employee_id integer NOT NULL,
    full_name character varying(250) NOT NULL,
    branch_id integer NOT NULL,
    department_id integer NOT NULL,
    position_id integer NOT NULL,
    address_id integer NOT NULL,
    supervisor_id integer,
    PRIMARY KEY (employee_id)
);

CREATE TABLE IF NOT EXISTS public.employeeaudit
(
    employee_id integer NOT NULL,
    full_name character varying(250) NOT NULL,
    branch_office character varying(250) NOT NULL,
    department character varying(250) NOT NULL,
    supervisor character varying(250) NOT NULL,
    "position" character varying(250) NOT NULL,
    address character varying(250) NOT NULL,
    city character varying(250) NOT NULL,
    country character varying(250) NOT NULL,
    event character varying(250) NOT NULL,
    registered_at date NOT NULL
);

CREATE TABLE IF NOT EXISTS public."position"
(
    position_id integer NOT NULL,
    name character varying(250) NOT NULL,
    PRIMARY KEY (position_id)
);

ALTER TABLE public.address
    ADD FOREIGN KEY (city_id)
    REFERENCES public.city (city_id)
    NOT VALID;


ALTER TABLE public.branchoffice
    ADD FOREIGN KEY (address_id)
    REFERENCES public.address (address_id)
    NOT VALID;


ALTER TABLE public.city
    ADD FOREIGN KEY (country_id)
    REFERENCES public.country (country_id)
    NOT VALID;


ALTER TABLE public.employee
    ADD FOREIGN KEY (address_id)
    REFERENCES public.address (address_id)
    NOT VALID;


ALTER TABLE public.employee
    ADD FOREIGN KEY (branch_id)
    REFERENCES public.branchoffice (branch_id)
    NOT VALID;


ALTER TABLE public.employee
    ADD FOREIGN KEY (department_id)
    REFERENCES public.department (department_id)
    NOT VALID;


ALTER TABLE public.employee
    ADD FOREIGN KEY (position_id)
    REFERENCES public."position" (position_id)
    NOT VALID;


ALTER TABLE public.employee
    ADD FOREIGN KEY (supervisor_id)
    REFERENCES public.employee (employee_id)
    NOT VALID;

END;

Punto 1. CREATE OR REPLACE FUNCTION employee_change()
  RETURNS TRIGGER 
  AS $$
  
declare
e_id INTEGER;
nombre character varying;
branch character varying;
dept character varying;
supervisor character varying;
pos character varying;
address_line character varying;
city_name character varying;
country_name character varying;

BEGIN
	
	if (TG_OP = 'UPDATE') THEN
		select employee.employee_id, employee.full_name, branchoffice."name", department."name",
		jefe.full_name, "position"."name", address.line1, city."name", country."name"
		into e_id, nombre, branch, dept, supervisor, pos, address_line, city_name, country_name
		from employee inner join branchoffice ON branchoffice.branch_id = employee.branch_id
		left join employee as jefe on employee.supervisor_id = jefe.employee_id
		inner join department ON department.department_id = employee.department_id
		inner join "position" on "position".position_id = employee.position_id
		inner join address ON address.address_id = employee.address_id
		inner join city ON city.city_id = address.city_id
		inner join country ON country.country_id = city.country_id
		where employee.employee_id = new.employee_id;
		insert into employeeaudit values (e_id, nombre, branch, dept, supervisor, pos, address_line, city_name, country_name, TG_OP, now());
	return new;
	
	elsif (TG_OP = 'DELETE') then
		select employee.employee_id, employee.full_name, branchoffice."name", department."name",
		jefe.full_name, "position"."name", address.line1, city."name", country."name"
		into e_id, nombre, branch, dept, supervisor, pos, address_line, city_name, country_name
		from employee inner join branchoffice ON branchoffice.branch_id = employee.branch_id
		left join employee as jefe on employee.supervisor_id = jefe.employee_id
		inner join department ON department.department_id = employee.department_id
		inner join "position" on "position".position_id = employee.position_id
		inner join address ON address.address_id = employee.address_id
		inner join city ON city.city_id = address.city_id
		inner join country ON country.country_id = city.country_id
		where employee.employee_id = new.employee_id;
		insert into employeeaudit values (e_id, nombre, branch, dept, supervisor, pos, address_line, city_name, country_name, TG_OP, now());
	return old;
	
	elsif (TG_OP = 'INSERT') then
		select employee.employee_id, employee.full_name, branchoffice."name", department."name",
		jefe.full_name, "position"."name", address.line1, city."name", country."name"
		into e_id, nombre, branch, dept, supervisor, pos, address_line, city_name, country_name
		from employee inner join branchoffice ON branchoffice.branch_id = employee.branch_id
		left join employee as jefe on employee.supervisor_id = jefe.employee_id
		inner join department ON department.department_id = employee.department_id
		inner join "position" on "position".position_id = employee.position_id
		inner join address ON address.address_id = employee.address_id
		inner join city ON city.city_id = address.city_id
		inner join country ON country.country_id = city.country_id
		where employee.employee_id = new.employee_id;
		insert into employeeaudit values (e_id, nombre, branch, dept, supervisor, pos, address_line, city_name, country_name, TG_OP, now());
	return new;
	END IF;
	
END
$$
LANGUAGE PLPGSQL;

create trigger employee_change_trigger after insert or update or delete on employee
for each row 
execute procedure employee_change();


Punto 2.

create or replace view public."punto2" as
select employee.full_name Nombre, employeeaudit."position" Cargo, employeeaudit.department Departamento
from employeeaudit natural join employee natural join department
where employee.employee_id = 2 and department.name = 'Human Resources';
