CREATE VIEW v_final_SQL_JV_1 AS
WITH promenna_koeficient AS (
	SELECT 	
		cbd.date,
		cbd.country,
		ROUND(cbd.confirmed/lt.population/ct.tests_performed*100000*1000,2) as koeficient_poctu_nakazeni
	FROM covid19_basic_differences cbd 
	JOIN lookup_table lt 
		ON cbd.country = lt.country AND lt.province is NULL
	JOIN tmp_JV_covid_test_pr ct
		ON cbd.country = ct.country_new 
		AND cbd.`date` = ct.`date` 
	WHERE ct.tests_performed is NOT NULL AND cbd.confirmed is NOT NULL
),
casova_promenna AS (
	SELECT 
		cbd.date,
		ct.country_new,
		IF(WEEKDAY(cbd.date) IN (5,6),1,0) AS flag_weekend,
		CASE 
			WHEN MONTH(cbd.date) IN (1,2) AND DAY(cbd.date) <= '31' THEN 0 # 1.1. - 29.2.
			WHEN MONTH(cbd.date) = 3 AND DAY(cbd.date) between '1' and '20' THEN 0 # 1.3. - 20.3.
			WHEN MONTH(cbd.date) = 3 AND DAY(cbd.date) >'20' THEN 1 # 21.3. - 31.3.
			WHEN MONTH(cbd.date) IN (4,5) AND DAY(cbd.date) <= '31' THEN 1 # 1.4. - 31.5.
			WHEN MONTH(cbd.date) = 6 AND DAY(cbd.date) between '1' and '20' THEN 1 # 1.6. - 20.6.
			WHEN MONTH(cbd.date) = 6 AND DAY(cbd.date) >'20' THEN 2 # 21.6. - 30.6.
			WHEN MONTH(cbd.date) IN (7,8) AND DAY(cbd.date) <= '31' THEN 2 # 1.7. - 31.8.
			WHEN MONTH(cbd.date) = 9 AND DAY(cbd.date) between '1' and '22' THEN 2 # 1.9. - 22.9.
			WHEN MONTH(cbd.date) = 9 AND DAY(cbd.date) >'20' THEN 3 # 23.9. - 30.9.
			WHEN MONTH(cbd.date) IN (10,11) AND DAY(cbd.date) <= '31' THEN 3 # 1.10. - 30.11.
			WHEN MONTH(cbd.date) = 12 AND DAY(cbd.date) between '1' and '20' THEN 3 # 1.12. - 20.12.
			WHEN MONTH(cbd.date) = 12 AND DAY(cbd.date) >'20' THEN 0 # 21.12. - 31.12.
			END AS flag_season 
	FROM covid19_basic_differences cbd
	JOIN tmp_JV_covid_test_pr ct
		ON cbd.country = ct.country_new
		AND cbd.`date` = ct.`date` 
),
specificka_promenna AS (
	SELECT DISTINCT 
		c.capital_city,
		e.year,
		c.population_density AS hustota_zalidneni,
		c.country_new,
		e.mortaliy_under5 
	FROM tmp_JV_countries_pr c
	JOIN tmp_JV_economies_pr e
		ON c.country = e.country 
	WHERE e.mortaliy_under5 is not NULL 
),
GDP AS (
	SELECT DISTINCT 
		e.year,
		e.country_new,
		ROUND(e.GDP/e.population,2) AS HDP_na_obyvatele
	FROM tmp_JV_economies_pr e
	WHERE e.GDP is not NULL 
	GROUP BY e.country 
),
gini AS (
	SELECT 
		e.country_new,
		e.year,
		e.gini
	FROM tmp_JV_economies_pr e
	WHERE e.gini is not NULL
	GROUP BY e.country
),
umrtnost AS (
	SELECT
		e.year,
		e.country_new,
		e.mortaliy_under5 AS umrtnost
	FROM tmp_JV_economies_pr e
	WHERE e.mortaliy_under5 is not null
	GROUP BY e.country 
),
median AS (
	SELECT 
		c.median_age_2018 AS median,
		c.country_new
	FROM tmp_JV_countries_pr c
	WHERE c.median_age_2018 is not null
),
nabozenstvi AS (
	SELECT 
		r.country_new,
		r.religion, 
		COALESCE(ROUND(r.population/lt.population*100,2),0) AS podil_nabozenstvi
	FROM tmp_JV_religions_pr r 
	INNER JOIN lookup_table lt
		ON lt.country = r.country_new 
	WHERE lt.population is not NULL 
	GROUP BY r.country,r.religion
),
doziti AS (
	SELECT 
		le.iso3,
		c.country_new,
		MAX(CASE WHEN le.year = 1965 THEN le.life_expectancy END) AS life_expectancy_1965,
		MAX(CASE WHEN le.year = 1966 THEN le.life_expectancy END) AS life_expectancy_1966,
		MAX(CASE WHEN le.year = 1967 THEN le.life_expectancy END) AS life_expectancy_1967,
		MAX(CASE WHEN le.year = 1968 THEN le.life_expectancy END) AS life_expectancy_1968,
		MAX(CASE WHEN le.year = 1969 THEN le.life_expectancy END) AS life_expectancy_1969,
		MAX(CASE WHEN le.year = 1970 THEN le.life_expectancy END) AS life_expectancy_1970,
		MAX(CASE WHEN le.year = 1971 THEN le.life_expectancy END) AS life_expectancy_1971,
		MAX(CASE WHEN le.year = 1972 THEN le.life_expectancy END) AS life_expectancy_1972,
		MAX(CASE WHEN le.year = 1973 THEN le.life_expectancy END) AS life_expectancy_1973,
		MAX(CASE WHEN le.year = 1974 THEN le.life_expectancy END) AS life_expectancy_1974,
		MAX(CASE WHEN le.year = 1975 THEN le.life_expectancy END) AS life_expectancy_1975,
		MAX(CASE WHEN le.year = 1976 THEN le.life_expectancy END) AS life_expectancy_1976,
		MAX(CASE WHEN le.year = 1977 THEN le.life_expectancy END) AS life_expectancy_1977,
		MAX(CASE WHEN le.year = 1978 THEN le.life_expectancy END) AS life_expectancy_1978,
		MAX(CASE WHEN le.year = 1979 THEN le.life_expectancy END) AS life_expectancy_1979,
		MAX(CASE WHEN le.year = 1980 THEN le.life_expectancy END) AS life_expectancy_1980,
		MAX(CASE WHEN le.year = 1981 THEN le.life_expectancy END) AS life_expectancy_1981,
		MAX(CASE WHEN le.year = 1982 THEN le.life_expectancy END) AS life_expectancy_1982,
		MAX(CASE WHEN le.year = 1983 THEN le.life_expectancy END) AS life_expectancy_1983,
		MAX(CASE WHEN le.year = 1984 THEN le.life_expectancy END) AS life_expectancy_1984,
		MAX(CASE WHEN le.year = 1985 THEN le.life_expectancy END) AS life_expectancy_1985,
		MAX(CASE WHEN le.year = 1986 THEN le.life_expectancy END) AS life_expectancy_1986,
		MAX(CASE WHEN le.year = 1987 THEN le.life_expectancy END) AS life_expectancy_1987,
		MAX(CASE WHEN le.year = 1988 THEN le.life_expectancy END) AS life_expectancy_1988,
		MAX(CASE WHEN le.year = 1989 THEN le.life_expectancy END) AS life_expectancy_1989,
		MAX(CASE WHEN le.year = 1990 THEN le.life_expectancy END) AS life_expectancy_1990,
		MAX(CASE WHEN le.year = 1991 THEN le.life_expectancy END) AS life_expectancy_1991,
		MAX(CASE WHEN le.year = 1992 THEN le.life_expectancy END) AS life_expectancy_1992,
		MAX(CASE WHEN le.year = 1993 THEN le.life_expectancy END) AS life_expectancy_1993,
		MAX(CASE WHEN le.year = 1994 THEN le.life_expectancy END) AS life_expectancy_1994,
		MAX(CASE WHEN le.year = 1995 THEN le.life_expectancy END) AS life_expectancy_1995,
		MAX(CASE WHEN le.year = 1996 THEN le.life_expectancy END) AS life_expectancy_1996,
		MAX(CASE WHEN le.year = 1997 THEN le.life_expectancy END) AS life_expectancy_1997,
		MAX(CASE WHEN le.year = 1998 THEN le.life_expectancy END) AS life_expectancy_1998,
		MAX(CASE WHEN le.year = 1999 THEN le.life_expectancy END) AS life_expectancy_1999,
		MAX(CASE WHEN le.year = 2000 THEN le.life_expectancy END) AS life_expectancy_2000,
		MAX(CASE WHEN le.year = 2001 THEN le.life_expectancy END) AS life_expectancy_2001,
		MAX(CASE WHEN le.year = 2002 THEN le.life_expectancy END) AS life_expectancy_2002,
		MAX(CASE WHEN le.year = 2003 THEN le.life_expectancy END) AS life_expectancy_2003,
		MAX(CASE WHEN le.year = 2004 THEN le.life_expectancy END) AS life_expectancy_2004,
		MAX(CASE WHEN le.year = 2005 THEN le.life_expectancy END) AS life_expectancy_2005,
		MAX(CASE WHEN le.year = 2006 THEN le.life_expectancy END) AS life_expectancy_2006,
		MAX(CASE WHEN le.year = 2007 THEN le.life_expectancy END) AS life_expectancy_2007,
		MAX(CASE WHEN le.year = 2008 THEN le.life_expectancy END) AS life_expectancy_2008,
		MAX(CASE WHEN le.year = 2009 THEN le.life_expectancy END) AS life_expectancy_2009,
		MAX(CASE WHEN le.year = 2010 THEN le.life_expectancy END) AS life_expectancy_2010,
		MAX(CASE WHEN le.year = 2011 THEN le.life_expectancy END) AS life_expectancy_2011,
		MAX(CASE WHEN le.year = 2012 THEN le.life_expectancy END) AS life_expectancy_2012,
		MAX(CASE WHEN le.year = 2013 THEN le.life_expectancy END) AS life_expectancy_2013,
		MAX(CASE WHEN le.year = 2014 THEN le.life_expectancy END) AS life_expectancy_2014,
		MAX(CASE WHEN le.year = 2015 THEN le.life_expectancy END) AS life_expectancy_2015,
		MAX(CASE WHEN le.year = 2015 THEN le.life_expectancy END) - MAX(CASE WHEN le.year = 1965 THEN le.life_expectancy END) AS rozdil_doziti
	FROM tmp_JV_life_expectancy_pr le 
	JOIN tmp_JV_countries_pr c
		ON c.iso3 = le.iso3 
	GROUP BY le.iso3
),
teplota AS (
	SELECT 
		w.city_new,
		w.date,
		AVG(temp) AS prumerna_denni_teplota
	FROM tmp_JV_weather_pr w 
	WHERE w.time between '06:00' and '18:00'
	GROUP BY w.DATE,w.city_new
	ORDER BY w.date DESC 
),
hodiny_srazky AS (
	SELECT 
		w.city_new,
		w.date,
		w.time,
		w.rain,
		COUNT(time)*3 AS pocet_hodin
	FROM tmp_JV_weather_pr w 
	WHERE rain != '0.0 mm'
	GROUP BY w.date,w.city_new
	ORDER BY w.date DESC
),
naraz AS (
	SELECT
		w.city_new,
		w.date,
		MAX(CAST(gust AS INT)) AS naraz
	FROM tmp_JV_weather_pr w 
	GROUP BY w.date,w.city_new
	ORDER BY w.date DESC
)
SELECT 
	pk.date,
	pk.country,
	pk.koeficient_poctu_nakazeni,
	cp.flag_weekend,
	cp.flag_season,
	sp.hustota_zalidneni,
	GDP.HDP_na_obyvatele,
	gini.gini,
	umrtnost.umrtnost,
	median.median,
	nabozenstvi.podil_nabozenstvi,
	doziti.rozdil_doziti,
	teplota.prumerna_denni_teplota,
	hodiny_srazky.pocet_hodin,
	naraz.naraz
FROM promenna_koeficient pk
JOIN casova_promenna cp
	ON pk.country = cp.country_new
JOIN specificka_promenna sp
	ON pk.country = sp.country_new
JOIN GDP 
	ON pk.country = GDP.country_new
JOIN gini 
	ON pk.country = gini.country_new
JOIN umrtnost 
	ON pk.country = umrtnost.country_new
JOIN median                 
	ON pk.country = median.country_new
JOIN nabozenstvi 
	ON pk.country = nabozenstvi.country_new
JOIN doziti 
	ON pk.country = doziti.country_new
JOIN teplota 
	ON sp.capital_city = teplota.city_new
JOIN hodiny_srazky  
	ON sp.capital_city = hodiny_srazky.city_new
JOIN naraz 
	ON sp.capital_city = naraz.city_new


CREATE TABLE t_Jana_Velkoborska_projekt_SQL_final AS
SELECT 
	* 
FROM v_final_SQL_JV_1