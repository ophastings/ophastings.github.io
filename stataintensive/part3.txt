clear all

// In the workshop we downloaded this state-year file of fertility rates
// from http://wonder.cdc.gov/natality.html
import delimited "Natality, 2007-2012.txt", clear
drop notes // don't actually need this line
drop in 358/419 // don't actually need this line
drop if year == . // this will do
keep state statecode year fertilityrate 

gen lagyear = year-1 // create this for the merege below

save "fertilitydata.dta", replace // save our clean dataset

// explore that data
scatter fert year // not that helpful
scatter fert year, by(state) // interesting

tab year, sum(fert) // the mean of the state fertility rates
graph bar fert, over(year)

preserve
collapse fert, by(year)
scatter fert year, connect(-)
restore

use StateURmonthly.dta, clear // use unemployment data

/* This is a long way to do what can be done in one line
gen year1 = cm_ / 12
gen year2 = int(year1)
gen year3 = 1900 + year2
*/

gen year = int(cm_/12) + 1900
rename year lagyear // because we want to merge with next years fertility
rename State statecode

collapse (mean) ur, by(lagyear statecode)

save StateURyearly.dta, replace

use "fertilitydata.dta", clear // we could start with the fertility or UR dataset

merge 1:1 lagyear statecode using "StateURyearly.dta"
keep if _merge == 3
drop _m

reg fertility ur
reg fertility ur i.statecode // state fixed effects
reg fertility ur i.statecode i.year // state and year fixed effects

xtset statecode year // tell stata we have panel data
xtset fertility ur, fe // fixed effects
xtset fertility ur, re // random effects

reg fertility l.ur // the lag operator, so we didn't have to make lagyear...

drop lagyear
reshape wide fertility ur, i(statecode) j(year)
reshape long fertility ur, i(statecode) j(year)

reshape wide
reshape long
