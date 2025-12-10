# Energy Peaks Per Month Average-EPPMA
Home assistant - electricity (eg Ellevio) consumption daily peaks per month calculations for hopefully load shedding/balancing

#### Home assistant community posting here:
> https://community.home-assistant.io/t/energy-peaks-per-month-average-eppma-electricity-eg-ellevio-consumption-daily-peaks-per-month-calculations-for-hopefully-load-shedding-balancing-help-to-test-and-improve/861977/1

## purpose
This is a roughly right script to determine the daily average power peaks per month as used in Sweden (and other places maybe including Beligium?, Netherlands?). I used the method defined by Ellevio in Sweden here: 
https://www.ellevio.se/abonnemang/ny-prismodell-baserad-pa-effekt/

After a number of months of use, it works well (as well as the input sensor is capable) and aligns well with our grid suppliers data.

I use this code today for some simpler load shedding / balancing / planning by being able to set a maximum allowed energy use each hour (based on the usage peaks and prices and monthly averages) to keep the peaks and prices lower.

## output
The script provides a number of sensors to be able to support deeper understanding and debugging, they are numbered from 1 and up prefixed by EPPMA so they are easy to find and understand how they flow! 
Within the script you will find more info regarding how it works, the process and the actual sensors.

Graphically, here is an early example in Apex charts, Apex is great for checking/debug:

<img src="/images/eppma_apex_0.png" width="400" />

And here is another example also quite early on without too much data as I had just cleared things after an update but including monthly comparison:

<img src="/images/eppma_apex_1.png" width="400" />



## How to get it working 

### pre-requisites
An import power (W) sensor is the only sensor needed - IF you have a sensor already in kWh and in a good enough time period then you should be able to just put it into step 3! Mine did not run so often (like every 5-10min) so I went to import power which updates every few seconds which then required the integration (step 2)

### configuration/install instructions
#### 1. put the following line(s) in your configuration.yaml in the appropriate place (you can change the directory but you need to change it then appropriately):

```
homeassistant:  
    packages:
      eppma_energy_peaks_per_month_average: !include packages/eppma_energy_peaks_per_ month_average.yaml
```
#### 2. put the yaml in your package directory "packages", name as described in config.yaml (1)

find yaml here: [yaml code](eppma_energy_peaks_per_month_average.yaml) 

#### 3. Replace the import power sensor name in the code to your actual sensor name in the yaml code! (~row 245)

#### 4. restart!
now it should be running!

#### 5. verify it is working! Either via sensors or Apex-charts graphical interface, code I use is [here](#apex-code)
>##### a. after 5-10min check values in eppma_1 & 2 after a few moments (so long as you are importing data you will see something)
>##### b. after 1st hour check values in eppma_3 & _4 for values from previous hour
>##### c. after 1st day after 00:00 check eppma_5 & _6  and even 7 for previous days values
>##### d. after 1st of month after 00:00ish check eppma _8 for results from previous month

--- 

## help, advice, tips, improvements?
I can certainly learn more when it comes to jinja/yaml so I guess there are some improvements to be made here, some things I have identified   :upside_down_face: which you can find in the open issues in the TODO section of the header within the code. If you find something or have ideas, let me know!

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
