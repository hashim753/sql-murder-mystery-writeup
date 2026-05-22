# SQL Murder Mystery
[SQL Murder Mystery](https://mystery.knightlab.com/)
A crime has taken place and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a **​murder​** that occurred sometime on ​**Jan.15, 2018​** and that it took place in ​**SQL City​**. Start by retrieving the corresponding crime scene report from the police department’s database.

## **Run this query to find the names of the tables in this database.**
`SELECT name` 
`FROM sqlite_master`
` where type = 'table' `

| **name**               |
| ---------------------- |
| crime_scene_report     |
| drivers_license        |
| facebook_event_checkin |
| interview              |
| get_fit_now_member     |
| get_fit_now_check_in   |
| solution               |
| income                 |
| person                 |

## **Run this query to find the structure of the `crime_scene_report` table**

`SELECT sql` 
`FROM sqlite_master`
`where name = 'crime_scene_report'`

| **sql**                                                                                  |
| ---------------------------------------------------------------------------------------- |
| CREATE TABLE crime_scene_report ( date integer, type text, description text, city text ) |
## **Checking out the report**

`SELECT * FROM crime_scene_report` 
`WHERE type = 'murder'`
`AND city='SQL City';`

| date     | type   | description                                                                                                                                                                               | city     |
| -------- | ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| 20180215 | murder | REDACTED REDACTED REDACTED                                                                                                                                                                | SQL City |
| 20180215 | murder | Someone killed the guard! He took an arrow to the knee!                                                                                                                                   | SQL City |
| 20180115 | murder | Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave". | SQL City |

## **Structure of person table**

`SELECT sql` 
`FROM sqlite_master`
`where name = 'person';`

| **sql**                                                                                                                                                                                                                   |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CREATE TABLE person (id integer PRIMARY KEY, name text, license_id integer, address_number integer, address_street_name text, ssn CHAR REFERENCES income (ssn), FOREIGN KEY (license_id) REFERENCES drivers_license (id)) |


## **Looking into first witness**

`SELECT * from person`
`where LOWER(address_street_name) ='northwestern dr'`
`order by  address_number desc limit 1;`              

| **id** | **name**       | **license_id** | **address_number** | **address_street_name** | **ssn**   |
| ------ | -------------- | -------------- | ------------------ | ----------------------- | --------- |
| 14887  | Morty Schapiro | 118009         | 4919               | Northwestern Dr         | 111564949 |

## Looking into second witness

`select * from person`
`where lower(name) like '%annabel%'`
`and address_street_name = 'Franklin Ave';`

| **id** | **name**       | **license_id** | **address_number** | **address_street_name** | **ssn**   |
| ------ | -------------- | -------------- | ------------------ | ----------------------- | --------- |
| 16371  | Annabel Miller | 490173         | 103                | Franklin Ave            | 318771143 |

## Structure of interview table

`select sql from sqlite_master`
`where name = 'interview';`

| **sql**                                                                                                      |
| ------------------------------------------------------------------------------------------------------------ |
| CREATE TABLE interview ( person_id integer, transcript text, FOREIGN KEY (person_id) REFERENCES person(id) ) |

> [!Foreign key?] 
> Why is there no column for foreign key for person and interview?
> Eventhough its in structure i dont see it.

---
It seems the foreign key is something like a definition.The whole point of sql is to connect tables so one id should be same in all tables but can be different name so foreign key is used.
`FOREIGN KEY (person_id) REFERENCES person(id)` 
this means **person_id** is the same as **id** from **person**.
This is where JOIN comes in.

## Looking at the interview for both

`select name,transcript from interview join person on interview.person_id=person.id`
`where id = 16371 or id = 14887;`

| **name**       | **transcript**                                                                                                                                                                                                                  |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Morty Schapiro | I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W". |
| Annabel Miller | I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.                                                                                                           |

## Finding structure for get_fit_now_member ,get_fit_now_check_in  and  drivers_license

`select sql from sqlite_master where name = 'get_fit_now_member';`

| **sql**                                                                                                                                                                                     |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CREATE TABLE get_fit_now_member ( id text PRIMARY KEY, person_id integer, name text, membership_start_date integer, membership_status text, FOREIGN KEY (person_id) REFERENCES person(id) ) |
`select sql from sqlite_master where name = 'get_fit_now_check_in';`

| **sql**                                                                                                                                                                                       |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CREATE TABLE get_fit_now_check_in ( membership_id text, check_in_date integer, check_in_time integer, check_out_time integer, FOREIGN KEY (membership_id) REFERENCES get_fit_now_member(id) ) |
`select sql from sqlite_master where name = 'drivers_license';`

| **sql**                                                                                                                                                                              |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| CREATE TABLE drivers_license ( id integer PRIMARY KEY, age integer, height integer, eye_color text, hair_color text, gender text, plate_number text, car_make text, car_model text ) |
## Clues from driver_license
`select * from drivers_license` 
`where plate_number like '%H42W%'`
`and gender = 'male';`

|id|age|height|eye_color|hair_color|gender|plate_number|car_make|car_model|
|---|---|---|---|---|---|---|---|---|
|423327|30|70|brown|brown|male|0H42W2|Chevrolet|Spark LS|
|664760|21|71|black|black|male|4H42WR|Nissan|Altima|
## Clues from get_fit_now_check_in
`select * from get_fit_now_check_in` 
`where check_in_date = 20180109`
`and membership_id like '%48%';`

|membership_id|check_in_date|check_in_time|check_out_time|
|---|---|---|---|
|48Z7A|20180109|1600|1730|
|48Z55|20180109|1530|1700|
## Clues from get_fit_now_member
`SELECT * FROM get_fit_now_member`
`WHERE id LIKE '48Z%'`
`AND membership_status = 'gold';`

| id    | person_id | name          | membership_start_date | membership_status |
| ----- | --------- | ------------- | --------------------- | ----------------- |
| 48Z7A | 28819     | Joe Germuska  | 20160305              | gold              |
| 48Z55 | 67318     | Jeremy Bowers | 20160101              | gold              |

And we are down to 2 suspects...How do i proceed?
Ok so i did some googling and it seems i should join all these tables.
The problem is i needed a table that connects all these 3 tables.
After I looked at the diagram i realized i should use person to connect them all.
## Joining the tables
I used the [schema diagram](https://mystery.knightlab.com/schema.png) for this.
It is hard to see the connection otherwise.
`SELECT *` 
`FROM drivers_license as dl`
`INNER JOIN person as p ON dl.id=p.license_id`
`INNER JOIN get_fit_now_member as gf ON p.id=gf.person_id`
`INNER JOIN get_fit_now_check_in as ci ON gf.id=ci.membership_id`
`WHERE plate_number LIKE '%H42W%'`
`AND membership_status = 'gold'`
`AND check_in_date = 20180109;`

|id|age|height|eye_color|hair_color|gender|plate_number|car_make|car_model|id|name|license_id|address_number|address_street_name|ssn|id|person_id|name|membership_start_date|membership_status|membership_id|check_in_date|check_in_time|check_out_time|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|423327|30|70|brown|brown|male|0H42W2|Chevrolet|Spark LS|67318|Jeremy Bowers|423327|530|Washington Pl, Apt 3A|871539279|48Z55|67318|Jeremy Bowers|20160101|gold|48Z55|20180109|1530|1700|
Oh the table is too long but we got our killer!
Let me just shorten it 
`SELECT p.*` 
`FROM drivers_license as dl`
`INNER JOIN person as p ON dl.id=p.license_id`
`INNER JOIN get_fit_now_member as gf ON p.id=gf.person_id`
`INNER JOIN get_fit_now_check_in as ci ON gf.id=ci.membership_id`
`WHERE plate_number LIKE '%H42W%'`
`AND membership_status = 'gold'`
`AND check_in_date = 20180109;`
%% INNER JOIN AND JOIN ARE THE SAME %%

| id    | name          | license_id | address_number | address_street_name   | ssn       |
| ----- | ------------- | ---------- | -------------- | --------------------- | --------- |
| 67318 | Jeremy Bowers | 423327     | 530            | Washington Pl, Apt 3A | 871539279 |
Ok so our killer is Jeremy Bowers!
### Check your solution

Did you find the killer?
`INSERT INTO solution VALUES (1, 'Jeremy Bowers');`
`SELECT value FROM solution;`

|value|
|---|
|Congrats, you found the murderer! But wait, there's more... If you think you're up for a challenge, try querying the interview transcript of the murderer to find the real villain behind this crime. If you feel especially confident in your SQL skills, try to complete this final step with no more than 2 queries. Use this same INSERT statement with your new suspect to check your answer.|
What?There's more? 
Ok then Let's do this.
## Interview of the killer
`--67318 Jeremy Bowers`
`SELECT name,i.*`
`FROM interview as i`
`JOIN person as p ON p.id=i.person_id`
`WHERE id = 67318;`

|name|person_id|transcript|
|---|---|---|
|Jeremy Bowers|67318|I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.|
- Ok so money is in **income**
- height,gender,hair color,car is in **drivers_license**
- events is in **facebook_event_checkin**
- and no name
Ah shit! here we go again..
## Finding the mastermind
`SELECT p.*`
`FROM income as i`
`JOIN person as p ON i.ssn=p.ssn`
`JOIN drivers_license as dl ON p.license_id=dl.id`
`JOIN facebook_event_checkin as fb ON fb.person_id=p.id`
`WHERE height >= 65`
`AND height <= 67`
`AND gender = 'female'`
`AND hair_color = 'red'`
`AND car_make = 'Tesla'`
`AND car_model = 'Model S'`
`GROUP BY event_name`
`AND date LIKE '201712%'`
`ORDER BY annual_income DESC`
`LIMIT 5;`

|id|name|license_id|address_number|address_street_name|ssn|
|---|---|---|---|---|---|
|99716|Miranda Priestly|202298|1883|Golden Ave|987756388|
And Ladies and gentlemen, we have our villain.
Wow! I really did it with just two queries.
### Check your solution

Did you find the killer?
`INSERT INTO solution VALUES (1, 'Miranda Priestly');`
`SELECT value FROM solution;`

|value|
|---|
|Congrats, you found the brains behind the murder! Everyone in SQL City hails you as the greatest SQL detective of all time. Time to break out the champagne!|
Hell yeah!!!
Well then,see ya!
