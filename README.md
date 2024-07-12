# SQL-data-cleaning
_This is an educational project on data cleaning and preparation using SQL. The original database in CSV format is located in the file club_member_infor.csv. Here, we will explore the steps that need to be applied to obtain a cleansed version of the dataset._

_Let's inspect the initial rows to analyze the data in its original format._

```sql
    SELECT *
    FROM club_member_info
    ORDER BY id
    LIMIT 10;
```

**The result:**
|full_name|age|martial_status|email|phone|full_address|job_title|membership_date|
|---------|---|--------------|-----|-----|------------|---------|---------------|
|addie lush
|40|married|alush0@shutterfly.com|254-389-8708|3226 Eastlawn Pass,Temple,Texas|Assistant Professor|7/31/2013|
|      ROCK CRADICK|46|married|rcradick1@newsvine.com|910-566-2007|4 Harbort Avenue,Fayetteville,North Carolina|Programmer III|5/27/2018|
|Sydel Sharvell|46|divorced|ssharvell2@amazon.co.jp|702-187-8715|4 School Place,Las Vegas,Nevada|Budget/Accounting Analyst I|10/6/2017|
|Constantin de la cruz|35||co3@bloglines.com|402-688-7162|6 Monument Crossing,Omaha,Nebraska|Desktop Support Technician|10/20/2015|
|  Gaylor Redhole|38|married|gredhole4@japanpost.jp|917-394-6001|88 Cherokee Pass,New York City,New York|Legal Assistant|5/29/2019|
|Wanda del mar       |44|single|wkunzel5@slideshare.net|937-467-6942|10864 Buhler Plaza,Hamilton,Ohio|Human Resources Assistant IV|3/24/2015|
|Joann Kenealy|41|married|jkenealy6@bloomberg.com|513-726-9885|733 Hagan Parkway,Cincinnati,Ohio|Accountant IV|4/17/2013|
|   Joete Cudiff|51|divorced|jcudiff7@ycombinator.com|616-617-0965|975 Dwight Plaza,Grand Rapids,Michigan|Research Nurse|11/16/2014|
|mendie alexandrescu|46|single|malexandrescu8@state.gov|504-918-4753|34 Delladonna Terrace,New Orleans,Louisiana|Systems Administrator III|3/12/1921|
| fey kloss|52|married|fkloss9@godaddy.com|808-177-0318|8976 Jackson Park,Honolulu,Hawaii|Chemical Engineer|11/5/2014|

# Create a new table for cleaning

Let's generate a new table where we can manipulate and restructure the data without modifying the original dataset.

```sql
    CREATE TABLE club_member_info_cleaned (
	    full_name VARCHAR(50),
	    age INTEGER,
	    martial_status VARCHAR(50),
	    email VARCHAR(50),
	    phone VARCHAR(50),
	    full_address VARCHAR(50),
	    job_title VARCHAR(50),
	    membership_date VARCHAR(50)
    );
```

# Copy all values from original table

```sql
    INSERT INTO club_member_info_cleaned
    SELECT * FROM club_member_info
```


# Full name column cleaning

Even from the beginning we can find out that full_name column contains a lot of inconsistencies. I plan to:

- Trim whitespaces
- Covert all names to uppercase
- Replace empty names with `NULL`

  ## Trim whitespaces
  ```sql
  UPDATE club_member_info_cleaned SET full_name = TRIM(full_name)
  ```

  ## To uppercase
  ```sql
  UPDATE club_member_info_cleaned SET
         full_name = CASE WHEN full_name = '' THEN NULL ELSE full_name END;
  ```

  ## Age cleaning

  Possible issues with age:

  - It could be empty.
  - It could fall outside the realistic range (18 - 90).
 
  Let's check if there are such values in our dataset:
  ```sql
  SELECT COUNT(*) FROM club_member_infor_cleaned
         WHERE age < 18 OR age >  90 OR age is NULL;
  ```
  **Result:**
|COUNT(*)|
|--------|
|18|

Okay, let's fix it replacing all wrong values. Since age of an individual is not as crucial in data analysis, let's replace all values not in the valid range with the mode.

```sql
UPDATE club_member_info_cleaned
        SET age = (SELECT MODE(age) FROM club_member_info_cleaned)
        WHERE age < 18 OR age > 90 OR age is NULL;
```

## Marital status cleaning

There are only 3 possible values for marital status:
- single
- married
- divorced

  I plan to replace all values not in this list with NULL. But before doing this we need at least trim it and standardise a letter case not to loose right values with typos.

  ```sql
  UPDATE club_member_info_cleaned
         SET marital_status = TRIM(LOWER(marital_status));
  ```

  Now we are ready to cleann all wrong values:

  ```sql
  UPDATE club_member_info_cleaned
         SET marital_status = NULL
         WHERE NOT marital_status IN ('single', 'married', 'divorced');
  ```

  ## Other text columns

  It seems we can trim and lowercase all other text columns in our table as a base action. Let's do it:

```sql
UPDATE club_member_info_cleaned SET
       phone = TRIM(LOWER(phone)),
       full_address = TRIM(LOWER(full_address)),
       job_title = TRIM(LOWER(job_title)),
       membership_date = TRIM(LOWER(membership_date));
```

Also we can replace all empty values with `NUL`:

```sql
UPDATE club_member_info_cleaned SET
       phone = NULL WHERE phone = '';
UPDATE club_member_info_cleaned SET
       full_address - NULL WHERE full_address = '';
UPDATE club_member_info_cleaned SET
       job_title = NULL WHERE job_title = '';
UPDATE club_member_info_cleaned SET
       membership_date = NULL WHERE membership_date = '';
```

## Take a look at cleaned data
Let's take a look at our cleaned table data.

```sql
SELECT * FROM club_member_info_cleaned LIMIT 10;
```

**Result:**
|full_name|age|martial_status|email|phone|full_address|job_title|membership_date|
|---------|---|--------------|-----|-----|------------|---------|---------------|
|addie lush
|40|married|alush0@shutterfly.com|254-389-8708|3226 Eastlawn Pass,Temple,Texas|Assistant Professor|7/31/2013|
|      ROCK CRADICK|46|married|rcradick1@newsvine.com|910-566-2007|4 Harbort Avenue,Fayetteville,North Carolina|Programmer III|5/27/2018|
|Sydel Sharvell|46|divorced|ssharvell2@amazon.co.jp|702-187-8715|4 School Place,Las Vegas,Nevada|Budget/Accounting Analyst I|10/6/2017|
|Constantin de la cruz|35||co3@bloglines.com|402-688-7162|6 Monument Crossing,Omaha,Nebraska|Desktop Support Technician|10/20/2015|
|  Gaylor Redhole|38|married|gredhole4@japanpost.jp|917-394-6001|88 Cherokee Pass,New York City,New York|Legal Assistant|5/29/2019|
|Wanda del mar       |44|single|wkunzel5@slideshare.net|937-467-6942|10864 Buhler Plaza,Hamilton,Ohio|Human Resources Assistant IV|3/24/2015|
|Joann Kenealy|41|married|jkenealy6@bloomberg.com|513-726-9885|733 Hagan Parkway,Cincinnati,Ohio|Accountant IV|4/17/2013|
|   Joete Cudiff|51|divorced|jcudiff7@ycombinator.com|616-617-0965|975 Dwight Plaza,Grand Rapids,Michigan|Research Nurse|11/16/2014|
|mendie alexandrescu|46|single|malexandrescu8@state.gov|504-918-4753|34 Delladonna Terrace,New Orleans,Louisiana|Systems Administrator III|3/12/1921|
| fey kloss|52|married|fkloss9@godaddy.com|808-177-0318|8976 Jackson Park,Honolulu,Hawaii|Chemical Engineer|11/5/2014|

  

  
