# Covid-19-Analysis
Covid-Analysis - Data Exploration using Postgresql(pgAdmin4)

I have used PostgreSQL for the first time to perform my data exploration where in I have created database, schemas, imported data from csv files
Skills used: JOINS, AGGREGATE FUNCTION, CTE, TEMP TABLES, VIEWS, CHANGING DATA TYPES

Following was my query, after creating all the tables and importing the data from CSV files

-------Select the data we are going to use

select location, date, total_cases, new_cases, total_deaths, population
from "Covid"."CovidDeaths"
where continent is not null
order by 1,2

----Looking at total cases vs total deaths
-----IN postgre sql we need to cast the integer value to numeric to get the answer in decimal
---Shows likelihood of dying if you contract covid in your country
select location, date, total_cases, total_deaths, (total_deaths::NUMERIC/total_cases)*100 as DeathPercentage
from "Covid"."CovidDeaths"
where location like '%Kingdom%'
and where continent is not null
order by 1, 2;

--Looking at total cases va Population
-- shows what percentage of population got covid
select location, date, total_cases, population, (total_cases::NUMERIC/population)*100 as PercentageAffected
from "Covid"."CovidDeaths"
where location like '%Kingdom%'
and where continent is not null
order by 1, 2;

--Looking at Countries having the highest infection rate compared to population
select location, population,MAX(total_cases) as HighestInfectedCount, max((total_cases::NUMERIC/population))*100 as PercentageAffected
from "Covid"."CovidDeaths"
--where location like '%Kingdom%'
where continent is not null
group by location, population
order by PercentageAffected desc;

--showing countries with highest death count per population
select location, MAX(total_deaths) as TotalDeathCount
from "Covid"."CovidDeaths"
--where location like '%Kingdom%'
where continent is not null
group by location
order by TotalDeathCount desc;


--let's break things down by continent

select continent, MAX(total_deaths) as TotalDeathCount
from "Covid"."CovidDeaths"
where continent is not null
group by continent
order by TotalDeathCount desc;

--global numbers

select date, SUM(new_cases) as Total_new_cases, SUM(new_deaths) as Total_new_deaths, SUM(new_deaths)/SUM(new_cases)* 100 as DeathPercentage
from "Covid"."CovidDeaths"
where continent is not null
group by date
order by 1,2;

----Total cases globally

select SUM(new_cases) as Total_new_cases, SUM(new_deaths) as Total_new_deaths, SUM(new_deaths)/SUM(new_cases)* 100 as DeathPercentage
from "Covid"."CovidDeaths"
where continent is not null
order by 1,2;


----- let us now join the other table to get vaccination data aswell

select * 
from "Covid"."CovidDeaths" dea
join "Covid"."CovidVaccinations" vac
   on dea.location = vac.location
   and dea.date = vac.date
   
   
-------looking at total population vs vaccinations

--------New vaccination rooling count by location and date
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
  SUM(vac.new_vaccinations) 
  OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) as RollingPeopleVaccinated
from "Covid"."CovidDeaths" dea
join "Covid"."CovidVaccinations" vac
   on dea.location = vac.location
   and dea.date = vac.date
where dea.continent is  not null
 order by 1, 2, 3
 
----USE CTE
-----Calculate the rolling percentage of people vaccinated
-----We use CTE to apply aggregate function on the column created out of aggregate function

WITH PopvsVac(Continent, Location, Date, Population, New_vaccinations,RollingPeopleVaccinated)
AS
(
select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
  SUM(vac.new_vaccinations) 
  OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) as RollingPeopleVaccinated
from "Covid"."CovidDeaths" dea
join "Covid"."CovidVaccinations" vac
   on dea.location = vac.location
   and dea.date = vac.date
where dea.continent is  not null
--ORDER BY 1, 2
	)
	
select *, (RollingPeopleVaccinated/Population)*100 AS Rolling_percentageVaccinated
from PopvsVac

-----USE TEMTABLE

----We will use temp tables to calculate the rolling percentage 
----- of people who are fully vaccinated by Location
Drop table if exists percentage_fully-vaccinated
Create Temporary Table percentage_fully_vaccinated
(
continent character(100),
location character(100),
population bigint,
people_fully_vaccinated bigint,
rollingfullyvaccinated numeric
);
Insert into percentage_fully_vaccinated
(
select dea.continent, dea.location, dea.population, vac.people_fully_vaccinated,
SUM(vac.people_fully_vaccinated) OVER (PARTITION BY dea.location ORDER BY dea.location) as RollingFullyVaccinated
 from "Covid"."CovidDeaths" dea
join "Covid"."CovidVaccinations" vac
   on dea.location = vac.location
where dea.continent is  not null
ORDER BY 1, 2
);
select *, (rollingfullyvaccinated/population)*100 AS Rolling_percentage_fully_Vaccinated
from percentage_fully_vaccinated


----- create view to use it for our future analysis and visualisation

Create view "Covid"."percentage_fully_vaccinated" as
select dea.continent, dea.location, dea.population, vac.people_fully_vaccinated,
SUM(vac.people_fully_vaccinated) OVER (PARTITION BY dea.location ORDER BY dea.location) as RollingFullyVaccinated
 from "Covid"."CovidDeaths" dea
join "Covid"."CovidVaccinations" vac
   on dea.location = vac.location
where dea.continent is  not null
ORDER BY 1, 2


-----------------------------------------------------------------

The data set was quite vast and has lot more things to divein for future analysis. 

I am open for all the feebacks and improvement so that I can learn more. Hope this is helpfull to any new bee like me....
Happy coding :)
