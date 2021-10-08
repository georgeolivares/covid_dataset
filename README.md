# COVID-19 General Analysis by Country
Hey everybody! As my first project, I decided to do an analysis on the current situation on Covid worldwide.
If you are curious of where this data came from you can find it in the following website: <https://github.com/owid/covid-19-data/tree/master/public/data>

As a newcomer to this data analysis industry I asume we should have some questions or objectives before starting to upload or transform any data.
Covid is a really hot topic right now, as we all now how our lifes have changed thanks to the arrival of this 'creature'.

# Objectives for this project

I believe, according to the information in the dataset used, we can answer some of the following questions:

--> Top 5 countries with the most confirmed cases per million

--> Top 5 countries with the deaths per million

--> Are population density and confirmed cases related?

--> Is median age related to the death rate?

--> Is GDP per capita related to the death rate?

--> Are hospital beds per thousand related to the death rate?

# First bumps in the way

I thought everything was ok. I thought everything was gonna be fine until I tried importing my 180,000 rows file into Microsoft SQL Server and MySQL Workbench.

I KNOW THIS IS A SMALL FILE hahaha.
I know this is a small file in comparison to others used in the industry, nonetheless, it was my first time trying to import a relatively big file to a database.

I tried using the import wizard in MySQL Workbench, big mistake. I can't believe the time it takes for this kind of files to be imported to this tools.
I looked for a way around, youtube videos, blogs in Stackoverflow and nothing worked.
I tried importing the flat file using the MySQL Command Line window and, after a lot of tries, I gave up.
Seems like I am not 'worthy' of using this path yet hahaha.

Finally, by a friend's recommendation, I imported the file using DBeaver. I'll be using this platform to do this projects. We'll see how it goes!

# Apparently, Seychelles had the most cases per million!

I don't think comparing the total amount of cases of one country with the total amount cases of another is fair. Countries differ in a lot of aspects and I believe if we want to obtain any relevant insights we should level the play for all of them.

The following gr


```sql
with group1
	as(select location, max(total_cases_per_million) as cases_per_million, count(distinct(date))
	as unique_dates from owid_covid_data_csv ocdc group by location order by cases_per_million DESC),
group2 as(select avg(unique_dates) as average from
	(select count(distinct(date)) as unique_dates
	from owid_covid_data_csv ocdc group by location) mytable)
select * from group1 join group2 where unique_dates > average and cases_per_million is not null;
```

The following graph shows the 5 countries with the most cases per million. Nonetheless, analyzing the cases per million of some of these countries may not even be logical. Why?
For instance, it seems like Seychelles has a population of around 90,000 people. What?! hahaha Apparently the biggest country of these is Czech Republic with around 10,700,000 citizens. 

*The output*

![image](https://user-images.githubusercontent.com/88570786/135373295-f748e9ff-bf7c-491b-9208-94bf805abdcc.png)

![image](https://user-images.githubusercontent.com/88570786/135374660-e1f81661-6953-43cb-912a-d00919584baa.png)

This is the size of Seychelles just in case (as me) didn't know.

![image](https://user-images.githubusercontent.com/88570786/136625635-01df4fc5-4fd9-4613-a2c8-f415b8332598.png)


# Deadliest countries during covid times!

There were some inconscistencies while going through this dataset. For instance, the following graph shows some of the deadliest countries to live in during the pandemic. Being from Mexico, I can assure you how bad the situation was in my country. And even though we could argue that smaller countries could have a worse 'per million' scenario than other big countries, it seems fair to say that it also depends a lot in hou accurate the data was reported by each of the countries. Some of the countries may be more willing than others to share the actual amount of COVID cases. 

![image](https://user-images.githubusercontent.com/88570786/135385590-90c58245-2290-49ad-826e-c5d6c5cfe2ae.png)

# Is the death rate related to the amount of beds available for citizens by country?

Another struggle I had during the analysis is that, it doesn't matter how effective the countries were facing the pandemic. Vaccines and beds in hospitals could only do so much to lower the growth rate of cases around the world. So, even if vaccines and beds had a positive impact in the death rate. you wouldn't be able to see any relationship between one variable vs another. It was like trying to stop a big truck at full speed putting small bumps in front of it. It helps, but it is not enough.

The same applies to the relationship between vaccines applied vs the amount of daily cases and the relationship between vaccines applied and daily cases growth rate. Vaccines, beds, lockdown... it all helps to a certain point, but it is still not enough. 

```sql
select location, max(date), max(total_deaths_per_million) , 
hospital_beds_per_thousand from owid_covid_data_csv ocdc 
where hospital_beds_per_thousand is not null group by location;
```

![image](https://user-images.githubusercontent.com/88570786/135940766-26a1e1ec-8037-4bf1-a25f-9d81fc835d22.png)

# Conclusions

I believe this was a great exercise for me to practice and have a general idea of the work involved in analyzing datasets and the unexpected 'bumps' that we can find during the analysis.

There's a concept (I don't even know if it's a concept, but I believe it is hahaha) called 'data cleaning'. During the analysis of this dataset I found many small but significant differences in the data from some countries. This differences made me obtain results that were not completely reliable. For instance, some of the amounts in the 'total_cases' column instead of having a ',' showing a number like 70,000 had a '.' showing the number as this 70.000. As I said before, this may be a small but really significant mistake in the dataset that instead of showing a country with 70,000 cases would show you the same country with a total of 70 cases. 

I believe the right approach to analyzing a dataset must be:
1) Get a general understanding of the whole dataset
2) Get to know how the data was obtained, is it reliable?
3) Having a reliable source from where the data was obtained doesn't mean the dataset is free of mistakes. Has the dataset being analyzed and have the mistakes being corrected?

Once we have an answer for these questions we can take a decision of wether we should start doing the analysis or not.

# Vaccines applied per day vs the cases growth rate?

```sql
drop table tablatemporal1;

create table tablatemporal1(fila integer, location text, date text, 
total_vaccinations decimal(10,0), total_cases integer);

insert into tablatemporal1(fila, location, date, total_vaccinations, total_cases)
select row_number() over(partition by location order by date) as fila,
location, date, total_vaccinations, total_cases from owid_covid_data_csv ocdc 
where total_vaccinations is not null and total_cases is not null and location = 'Africa';

select * from tablatemporal1;

SELECT fila, location, date, total_vaccinations, total_cases, sum(total_cases)
  OVER (ORDER BY fila ROWS BETWEEN CURRENT ROW AND 1 FOLLOWING) sum
FROM tablatemporal1;
```

