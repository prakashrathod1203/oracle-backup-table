#### 1. CREATE USER AND GRANT PERMISSION (IF REQURD THEN)

``` sql
CREATE USER <username> IDENTIFIED BY <username>;
GRANT CREATE TABLE TO <username>;
GRANT RESOURCE TO <username>;
GRANT CONNECT TO <username>;
GRANT CREATE VIEW TO <username>;
GRANT CREATE SESSION TO <username>;
GRANT UNLIMITED TABLESPACE TO <username>;
CONNECT <username>;
```
#### 2. CREATE TABLE

``` sql
-- Actual table 
CREATE TABLE customers
( customer_id number(10) NOT NULL,  
  customer_name varchar2(50) NOT NULL,  
  city varchar2(50),
  createdate date,
  CONSTRAINT customers_pk PRIMARY KEY (customer_id)  
);
  
-- Backup table 
CREATE TABLE customers_bkp
( customer_id number(10) NOT NULL,  
  customer_name varchar2(50) NOT NULL,  
  city varchar2(50),
  createdate date,	  
  CONSTRAINT customers_bkp_pk PRIMARY KEY (customer_id)  
);
```

#### 3. INSERT RECORD IN ACTUAL TABLE
``` sql
INSERT INTO customers VALUES (1, 'Prakash', 'Ahmedabad', sysdate);
INSERT INTO customers VALUES (2, 'Deepak', 'Bangalore', TO_DATE('01-APR-21'));
INSERT INTO customers VALUES (3, 'Akshay', 'Mumbai', TO_DATE('01-JAN-21'));
```

#### 4. SELECT DATA
``` sql
SELECT * FROM customers;
SELECT * FROM customers_bkp;
```

#### 5. SELECT LAST THREE MONTH RECORD AND INSERT IN BACKUP TABLE
- It's one time job, after that it will take care by trigger
``` sql
INSERT INTO customers_bkp SELECT * FROM customers WHERE createdate >= add_months(trunc(sysdate,'MM'),-2);
```

#### 6. CROSS VERIFY ONCE IN BACKUP TABLE LAST THREE MONTH RECORD PROPERLY INSERTED OR NOT
``` sql
SELECT * FROM customers_bkp;
```

#### 7. CREATE TRIGGER FOR INSERT, UPDATE AND DELETE RECORDS, HERE SCHEMA NAME IS OPTIONAL, IF YOUR BACKUP TABLE AND ORIGINAL TABLE ARE IN DIFFERENT SCHEMA THEN YOU NEED
``` sql
-- Syntax: Insert or Update trigger 
CREATE OR REPLACE TRIGGER  <TRIGGER_NAME>  
  AFTER  
  INSERT OR UPDATE ON <SCHEMA_NAME>.<ORIGINAL_TABLENAME>  REFERENCING NEW AS <TABLE_ALIAS_NAME>
  FOR EACH ROW  
BEGIN  
  DELETE FROM <SCHEMA_NAME>.<BKP_TABLENAME> WHERE <PK_FIELD_NAME> = :<TABLE_ALIAS_NAME>.<PK_FIELD_NAME>;
  INSERT INTO <SCHEMA_NAME>.<BKP_TABLENAME> VALUES(:<TABLE_ALIAS_NAME>.<PK_FIELD_NAME>, :<TABLE_ALIAS_NAME>.<FIELD_NAME>, :<TABLE_ALIAS_NAME>.<FIELD_NAME>, :<TABLE_ALIAS_NAME>.<FIELD_NAME>);  
END; 

-- Example: Insert or Update trigger
CREATE OR REPLACE TRIGGER  in_up_customers_trigger  
  AFTER  
  INSERT OR UPDATE ON customers  REFERENCING NEW AS cust
  FOR EACH ROW  
BEGIN  
  DELETE FROM customers_bkp WHERE customer_id=:cust.customer_id;
  INSERT INTO customers_bkp VALUES(:cust.customer_id, :cust.customer_name, :cust.city, :cust.createdate);  
END; 

-- Syntax: Delete trigger 
CREATE OR REPLACE TRIGGER  <TRIGGER_NAME>  
  BEFORE  
  DELETE ON <SCHEMA_NAME>.<ORIGINAL_TABLENAME>
  FOR EACH ROW  
BEGIN  
  DELETE FROM <SCHEMA_NAME>.<BKP_TABLENAME> WHERE <PK_FIELD_NAME> = :old.<PK_FIELD_NAME>;
END;

-- Example: Delete trigger
CREATE OR REPLACE TRIGGER  del_customers_trigger  
  BEFORE  
  DELETE ON customers
  FOR EACH ROW  
BEGIN  
  DELETE FROM customers_bkp WHERE customer_id=:old.customer_id;
END;
```

#### 8. INSERT RECORD IN ACTUAL TABLE, THAT RECORD WILL AUTO POPULATED IN BACKUP TABLE
``` sql
INSERT INTO customers VALUES (4, 'Aman', 'Baroda', sysdate);
SELECT * FROM customers;
SELECT * FROM customers_bkp;
```

#### 9. UPDATE RECORD IN ACTUAL TABLE, THAT RECORD WILL AUTO UPDATE IN BACKUP TABLE
``` sql
UPDATE customers SET city='Bhavnager' WHERE customer_id = 4;
SELECT * FROM customers;
SELECT * FROM customers_bkp;
```

#### 10. DELETE RECORD IN ACTUAL TABLE, THIS RECORD WILL AUTO DELETED IN BACKUP TABLE
``` sql
DELETE FROM customers WHERE customer_id=4;
SELECT * FROM customers;
SELECT * FROM customers_bkp;
```

#### OTHER
##### Recompiling a Trigger
``` sql
ALTER TRIGGER <trigger_name> COMPILE;
DROP TRIGGER <trigger_name>;

```
