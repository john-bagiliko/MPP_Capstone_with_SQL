# MPP_Capstone_with_SQL
In this repo, I used PostgreSQL to gain insights into the MPP capstone data

### Install Postgres on Ubuntu
https://tecadmin.net/install-postgresql-server-on-ubuntu/


### Login to Database 
```{bash}
sudo su - postgres
psql
```

### Create database 
```{sql}
CREATE DATABASE capstone;
```
### Create Table
```{sql}
CREATE TABLE train_values (
    row_id         INT,
    loan_type      INT,
    property_type  INT,
    loan_purpose   INT,
    occupancy      INT,
    loan_amount    float,
    preapproval    INT,
    msa_md         INT,
    state_code     INT,
    county_code            INT,
    applicant_ethnicity    INT,
    applicant_race         INT,
    applicant_sex          INT,
    applicant_income       float,
    population                float,
    minority_population_pct       float,
    ffiecmedian_family_income      float,
    tract_to_msa_md_income_pct     float,
    number_of_owner_occupied_units float,
    number_of_1_to_4_family_units  float,
    lender                         INT,
    co_applicant                   bool
);
```
### Import CSV into table
```{sql}
COPY train_values
FROM '/home/johnb/Desktop/DataHack/Capstone/sql/train_values.csv'
WITH (
  FORMAT CSV,
  HEADER true,
  NULL ''
);
```
### Queries with Calculations
1. Find minimum loan_amount
```{sql}
select min(loan_amount) from train_values;
```
2. Find maximum loan_amount
```{sql}
select max(loan_amount) from train_values;
```
3. Find median loan_amount
```{sql}
select percentile_disc(0.5) within group (order by loan_amount) as median from train_values;
```
4. Find average loan_amount
```{sql}
select AVG(loan_amount) from train_values;
```
5. Find the standard deviation of the loan amount
```{sql}
select stddev_pop(loan_amount) as std from train_values;
```
6. Calculate modal loan_amount
```{sql}
select mode() WITHIN GROUP (ORDER BY loan_amount) as modal from train_values;

```

### Create the train_labels table in the database
```{sql}
CREATE TABLE train_labels (
    row_id         INT,
    accepted       INT
);
```

### Import train_labels CSV into train_labels table
Import CSV into table
```{sql}
COPY train_labels
FROM '/home/johnb/Desktop/DataHack/Capstone/train_labels.csv'
WITH (
  FORMAT CSV,
  HEADER true,
  NULL ''
);
```
### Join two tables
Use Natural Join
```{sql}
select * from train_values natural join train_labels limit 5;
```
#### check whether number of rows after join will be the same
```{sql}
select count(DISTINCT subquery.row_id) from (select * from train_values natural join train_labels) as subquery;
```
#### Make normal queries on result.
```{sql}
select subquery.* from (select * from train_values natural join train_labels) as subquery;
```
#### Select row_id, loan_amount, state_code, applicant_income, accepted from the joined table
```{sql}
select subquery.row_id, subquery.loan_amount, subquery.state_code, subquery.applicant_income, subquery.accepted from (select * from train_values natural join train_labels) as subquery where subquery.state_code = 48 limit 5;
```

#### This is tiring! Let's just create a new table from join
```{sql}
CREATE TABLE train_vals_labs AS 
  select * from train_values natural join train_labels;
```

#### Select row_id, loan_amount, state_code, applicant_income, accepted from the joined table (this becomes simple but not very effective if you want to update original tables later)
```{sql}
select row_id, loan_amount, state_code, applicant_income, accepted from train_vals_labs where state_code = 48 limit 5;
```
