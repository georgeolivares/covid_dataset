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

Last date recorded in data set - Sept 24th 2021

Average days reported by country: 510.6078

```sql
with group1
	as(select location, max(total_cases_per_million) as cases_per_million, count(distinct(date))
	as unique_dates from owid_covid_data_csv ocdc group by location order by cases_per_million DESC),
group2 as(select avg(unique_dates) as average from
	(select count(distinct(date)) as unique_dates
	from owid_covid_data_csv ocdc group by location) mytable)
select * from group1 join group2 where unique_dates > average and cases_per_million is not null;
```

*The output*

![image](https://user-images.githubusercontent.com/88570786/135373295-f748e9ff-bf7c-491b-9208-94bf805abdcc.png)

![image](https://user-images.githubusercontent.com/88570786/135374660-e1f81661-6953-43cb-912a-d00919584baa.png)

