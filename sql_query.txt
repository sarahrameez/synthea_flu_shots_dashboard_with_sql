-- tracking how many patients got flu shots in 2022, 
-- stratified by age, race, county (on a map) and overall
-- running total of flu shots over the course of 2022 (month by month)
-- total of flu shots given in 2022
-- list of patients that shows whether or not they have received flu shots

-- Requirement: people must be active in the healthcare system
-- kick out inactive patients and retain active patients

-- common table expression
with active_patients as
(
select distinct patient
	from encounters as e
	join patients as pat
		on e.patient = pat.id
	where start between '2020-01-01 00:00' and '2022-12-31 23:59'
		and pat.deathdate is null
		and extract(month from age('2022-12-31', pat.birthdate)) >= 6
),


flu_shot_2022 as
(
select patient, min(date) as earliest_flu_shot_2022
from immunizations
where code = '5302'
	and date between '2022-01-01 00:00' and '2022-12-31 23:59'
group by patient
)

select pat.birthdate
	,pat.race
	,pat.county
	,pat.id
	,pat.first
	,pat.last
	,flu.earliest_flu_shot_2022
	,flu.patient
	,case when flu.patient is not null then 1
	else 0
	end as flu_shot_2022
from patients as pat
left join flu_shot_2022 as flu
	on pat.id = flu.patient
where 1 = 1
	and pat.id in (select patient from active_patients)
