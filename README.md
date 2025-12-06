# Energy Peaks Per Month Average-EPPMA
Home assistant - electricity (eg Ellevio) consumption daily peaks per month calculations for hopefully load shedding/balancing

#### Home assistant community posting here:
> https://community.home-assistant.io/t/energy-peaks-per-month-average-eppma-electricity-eg-ellevio-consumption-daily-peaks-per-month-calculations-for-hopefully-load-shedding-balancing-help-to-test-and-improve/861977/1

## purpose
the purpose of this is to roughly right determine the daily average power peaks per month as used in Sweden (and other places maybe including Beligium?, Netherlands?). I used the method defined by Ellevio in Sweden here: 
https://www.ellevio.se/abonnemang/ny-prismodell-baserad-pa-effekt/
(previously I called this code by Ellevio but have renamed it to a general name EPPMA and did some serious refactoring (beware)) 

The hope is to ultimately use this for load shedding / balancing / planning by being able to set a maximum allowed energy use each hour (based on the usage peaks and prices) to keep the peaks and prices lower.

After a few days, the data collection is all now seemingly working pretty well.

graphically in apex charts it is looking like this for checking/debug (previously):
![image|485x500, 50%](upload://dxdMdgDrPSEHhZSh4WYuYqXhoCZ.png) 
latest albeit without much data at the moment as I just cleared things in the update(s):
![image|588x500](upload://nTduWyfATzzgKaTHaizr0mKJTlJ.png)


It is quite a few steps and  I am using Apex to present it so far. It has recovered well with various hiccups and I have now (230521) checked this vs Ellevio in detail and it has been preforming well but it will not be 100% as the import data I have is not 100%, again the idea is to get roughly right.
if you want to try have a look, see below install and process to understand

## help, advice, tips, improvements?
I am guessing as I am not so great at jinja/yaml that there are some improvements to be made here, some things I have identified   :upside_down_face:  some things I see you can find in the open issues in the TODO section of the header



# give it a try!

## pre-requisites 
import power is needed and as it comes in it is integrated to kWh.. if you have it already in kWh in a good enough time period you should be able to just put it into step 3! Mine did not run so often so I went to import power.

## NOTES
see notes in code incl process flow and TODOs (improvements?)

## configuration/install instructions
#### 1. (of 4) put in the line in your configuration.yaml:

```
homeassistant:  
    packages:
      eppma_energy_peaks_per_month_average: !include packages/eppma_energy_peaks_per_ month_average.yaml
```
#### 2a. (of 4) put the package in your package directory "package" with the name described in yaml (you can change director but you need to change it the config.yaml above) "eppma_energy_peaks_per_ month_average.yaml"
#### 2b. apex charts - if using, load the code found at the bottom [Apex code](#Apex code)
#### 3. after 5-10min check values in eppma_1 & 2 after a few moments (so long as you are importing data you will see something)
#### 4. after 1st hour check values in eppma_3 & _4 for previous hour
#### 5. after 1st day after 00:00 check eppma_5 & _6  and even 7 for previous day
#### 6. after 1st of month after 00:00ish check eppma _8 for result from previous month

--- 

### Apex code
```
type: custom:apexcharts-card
update_interval: 2.5min
header:
  show: true
  title: EPPMA energy peak /mth ave - dashboard
  show_states: true
  colorize_states: true
experimental:
  brush: true
brush:
  selection_span: 1d
apex_config:
  chart:
    height: 300px
graph_span: 70d
span:
  start: day
  offset: "-69d"
yaxis:
  - id: first
    decimals: 2
  - id: second
    opposite: true
    decimals: 1
series:
  - entity: sensor.eppma_1_import_power_to_energy_spent_integral
    name: 1. importPower integrated to energy
    yaxis_id: second
    stroke_width: 1
    extend_to: false
    show:
      in_header: false
  - entity: sensor.eppma_2_energy_per_hour
    name: 2. energy hrly (utility meter max/hr)
    type: area
    curve: stepline
    yaxis_id: first
    stroke_width: 1
    extend_to: false
    show:
      in_header: true
    float_precision: 2
    opacity: 0.4
  - entity: sensor.eppma_3_energy_per_hour_max
    name: 3. Max energy/hr
    curve: stepline
    type: line
    yaxis_id: first
    stroke_width: 1
    extend_to: false
    show:
      in_header: false
    float_precision: 2
  - entity: >-
      sensor.eppma_4_energy_hourly_value_adjusted_for_reductions_during_non_peak_hours
    name: 4. adjusted for non peak hours
    type: area
    curve: stepline
    yaxis_id: first
    stroke_width: 3
    extend_to: false
    show:
      in_brush: true
      in_header: true
    float_precision: 2
    opacity: 0.6
  - entity: sensor.eppma_5_energy_adjusted_daily_max_peak_value
    name: 5. daily max (running)
    curve: stepline
    yaxis_id: first
    stroke_width: 3
    extend_to: end
    show:
      in_brush: true
    float_precision: 2
  - entity: sensor.eppma_6_energy_peak_per_day
    name: 6. peak/d
    curve: stepline
    yaxis_id: first
    stroke_width: 6
    extend_to: end
    show:
      in_brush: true
      in_header: false
    float_precision: 2
  - entity: sensor.eppma_7_peaks_per_month_average_per_day
    name: 7. peaks/mth ave this mth
    curve: stepline
    yaxis_id: first
    stroke_width: 8
    stroke_dash: 10
    extend_to: end
    show:
      in_brush: false
      in_header: true
    float_precision: 2
  - entity: sensor.eppma_8_peaks_per_month_average_last_months_values
    name: 8. peaks/mth last mth
    curve: stepline
    yaxis_id: first
    stroke_width: 3
    extend_to: false
    show:
      in_brush: true
      in_header: true
    float_precision: 2

```

[![yellow-button|690x193, 20%](upload://lZnuL5e63jbNUbCDpjpz6Wqe4Du.png)](https://buymeacoffee.com/patrch)
