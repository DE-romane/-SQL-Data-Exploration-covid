# COVID Insights Analyzer sql

## Overview
This project involves exploring and analyzing COVID-19 data to gain insights into the pandemic's impact across different countries and continents. Various SQL techniques such as joins, common table expressions (CTEs), temporary tables, window functions, aggregate functions, views, and data type conversions are utilized in this project.

## Project Files

1. `CovidDeaths.xlsx`: Contains data on COVID-19 cases and deaths.
2. `CovidVaccinations.xlsx`: Contains data on COVID-19 vaccinations.
3. `Data Exploration.sql` : SQL code for this project

## SQL Skills Demonstrated
- Joins
- Common Table Expressions (CTEs)
- Temporary Tables
- Window Functions
- Aggregate Functions
- Creating Views
- Converting Data Types

## Steps and SQL Queries

### 1. Initial Data Selection
Select and order the data from the `CovidDeaths` table to understand the initial dataset structure.
```sql
SELECT *
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
ORDER BY 3, 4;
```

### 2. Detailed Data Selection
Select specific columns from the `CovidDeaths` table to focus on key attributes.
```sql
SELECT Location, date, total_cases, new_cases, total_deaths, population
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
ORDER BY 1, 2;
```

### 3. Total Cases vs. Total Deaths
Calculate the death percentage to analyze the likelihood of dying if infected.
```sql
SELECT Location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 AS DeathPercentage
FROM PortfolioProject..CovidDeaths
WHERE location LIKE '%states%'
AND continent IS NOT NULL
ORDER BY 1, 2;
```

### 4. Total Cases vs. Population
Calculate the percentage of the population infected with COVID-19.
```sql
SELECT Location, date, Population, total_cases, (total_cases/population)*100 AS PercentPopulationInfected
FROM PortfolioProject..CovidDeaths
ORDER BY 1, 2;
```

### 5. Highest Infection Rate per Population
Identify countries with the highest infection rates.
```sql
SELECT Location, Population, MAX(total_cases) AS HighestInfectionCount, MAX((total_cases/population))*100 AS PercentPopulationInfected
FROM PortfolioProject..CovidDeaths
GROUP BY Location, Population
ORDER BY PercentPopulationInfected DESC;
```

### 6. Highest Death Count per Population
Identify countries with the highest death counts.
```sql
SELECT Location, MAX(CAST(Total_deaths AS int)) AS TotalDeathCount
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
GROUP BY Location
ORDER BY TotalDeathCount DESC;
```

### 7. Continent Breakdown: Death Counts
Show continents with the highest death counts.
```sql
SELECT continent, MAX(CAST(Total_deaths AS int)) AS TotalDeathCount
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
GROUP BY continent
ORDER BY TotalDeathCount DESC;
```

### 8. Global Numbers
Calculate global totals and percentages.
```sql
SELECT SUM(new_cases) AS total_cases, SUM(CAST(new_deaths AS int)) AS total_deaths, (SUM(CAST(new_deaths AS int))/SUM(New_Cases))*100 AS DeathPercentage
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
ORDER BY 1, 2;
```

### 9. Population vs. Vaccinations
Calculate the percentage of the population vaccinated at least once.

`OVER` clause specifies a window for the summation. Here, it's partitioned by `dea.Location`, meaning the sum is calculated for each location independently. Ordering by `dea.location` and `dea.Date` ensures the summation considers the chronological order of dates within each location.
```sql
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
    SUM(CONVERT(int, vac.new_vaccinations)) OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac ON dea.location = vac.location AND dea.date = vac.date
WHERE dea.continent IS NOT NULL
ORDER BY 2, 3;
```

### 10. Using CTE for Calculation
Use a CTE to simplify complex calculations.
first calculates rolling vaccination totals for each location and then adds a column showing the vaccination rate as a percentage of the population.
```sql
WITH PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated) 
AS (
    SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
        SUM(CONVERT(int, vac.new_vaccinations)) OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
    FROM PortfolioProject..CovidDeaths dea
    JOIN PortfolioProject..CovidVaccinations vac ON dea.location = vac.location AND dea.date = vac.date
    WHERE dea.continent IS NOT NULL
)
SELECT *, (RollingPeopleVaccinated/Population)*100
FROM PopvsVac;
```

### 11. Using Temporary Table for Calculation
Use a temporary table to store intermediate results.
```sql
DROP TABLE IF EXISTS #PercentPopulationVaccinated;

CREATE TABLE #PercentPopulationVaccinated (
    Continent NVARCHAR(255),
    Location NVARCHAR(255),
    Date DATETIME,
    Population NUMERIC,
    New_vaccinations NUMERIC,
    RollingPeopleVaccinated NUMERIC
);

INSERT INTO #PercentPopulationVaccinated
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
    SUM(CONVERT(int, vac.new_vaccinations)) OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac ON dea.location = vac.location AND dea.date = vac.date;

SELECT *, (RollingPeopleVaccinated/Population)*100
FROM #PercentPopulationVaccinated;
```

### 12. Creating a View
Create a view to store data for later visualizations.
```sql
CREATE VIEW PercentPopulationVaccinated AS
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
    SUM(CONVERT(int, vac.new_vaccinations)) OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac ON dea.location = vac.location AND dea.date = vac.date
WHERE dea.continent IS NOT NULL;
```

## Conclusion
This project demonstrates a variety of SQL techniques to explore and analyze COVID-19 data, providing insights into infection and vaccination rates across different regions. The use of advanced SQL features like CTEs, temporary tables, and views helps in organizing and simplifying complex queries for better data analysis.






