# handy-time
Custom template for Home Assistant with some handy time macros - mainly smart_time which gives an English readable string for a time in the past or the future to the nearest 30 seconds for use on dashboards (which typically update every minute in HA), it gets more accurate as the time delta from now becomes smaller - just as you would say it!

I wrote this as a light and simple macro to deal with easily displaying a timestamp on the Lovelace dashboard that could handle times both in the past and in the future (unlike ```time_since``` and ```relative_time```).

If you are looking for something more comprehensive to manipulate time values check out [easy-time-jinja](https://github.com/Petro31/easy-time-jinja).

# Install
Copy the handy_time.jinja file into your config/custom_templates directory in Home Assistant, and then either restart Home Assistant or in the ```Developers Tools->Actions``` run the ```Reload custom Jinja2 template``` action.

Once loaded, anywhere you want to use one or more of the macros, import them like this:
```
{%- from 'handy_time.jinja' import long_time, short_time, smart_time,
   delta_mins, delta_hours, delta_days, delta_duration, format_hms, format_hours -%}
```

# How to Use
You can try this in the ```Developers Tools->Template``` to see how it behaves:
```
{%- from 'handy_time.jinja' import smart_time -%}

{% set a = now() - timedelta(days=5) %}
Date: {{ a | as_timestamp | timestamp_custom('%a %-d %b %H:%M', true, 0)  }}
Smart time: {{ smart_time(a) }}
```
... and you should see:
```
Date: Wed 10 Sep 00:40
Smart time: 5d ago
```

If you want to see everything, paste this in:
```
{%- from 'handy_time.jinja' import long_time, short_time, smart_time,
   delta_mins, delta_hours, delta_days, delta_duration, format_hms, format_hours -%}

{% set a = now() - timedelta(days=5) %}
Date: {{ a | as_timestamp | timestamp_custom('%a %-d %b %H:%M', true, 0)  }}
Smart time: {{ smart_time(a) }}
Short time: {{ short_time(a) }}
Long time: {{ long_time(a) }}
Delta Min: {{ delta_mins(a) }}
Delta Hours: {{ delta_hours(a) }}
Delta days: {{ delta_days(a) }}
{% set a = now() - timedelta(hours=5, seconds=55) %}
Date: {{ a | as_timestamp | timestamp_custom('%a %-d %b %H:%M', true, 0)  }}
Smart time: {{ smart_time(a) }}
Short time: {{ short_time(a) }}
Long time: {{ long_time(a) }}
Delta Min: {{ delta_mins(a) }}
Delta Hours: {{ delta_hours(a) }}
Delta days: {{ delta_days(a) }}
{% set a = now() + timedelta(hours=5, seconds=55) %}
Date: {{ a | as_timestamp | timestamp_custom('%a %-d %b %H:%M', true, 0)  }}
Smart time: {{ smart_time(a) }}
Short time: {{ short_time(a) }}
Long time: {{ long_time(a) }}
Delta Min: {{ delta_mins(a) }}
Delta Hours: {{ delta_hours(a) }}
Delta days: {{ delta_days(a) }}
{% set a = now() + timedelta(days=100) %}
Date: {{ a | as_timestamp | timestamp_custom('%a %-d %b %H:%M', true, 0)  }}
Smart time: {{ smart_time(a) }}
Short time: {{ short_time(a) }}
Long time: {{ long_time(a) }}
Delta Min: {{ delta_mins(a) }}
Delta Hours: {{ delta_hours(a) }}
Delta days: {{ delta_days(a) }}
{% set a = now() - timedelta(days=100) %}
Date: {{ a | as_timestamp | timestamp_custom('%a %-d %b %H:%M', true, 0)  }}
Smart time: {{ smart_time(a) }}
Short time: {{ short_time(a) }}
Long time: {{ long_time(a) }}
Delta Min: {{ delta_mins(a) }}
Delta Hours: {{ delta_hours(a) }}
Delta days: {{ delta_days(a) }}
{% set b = a - timedelta(hours=3, minutes=4, seconds=4) %}
A Date: {{ a | as_timestamp | timestamp_custom('%a %-d %b %H:%M:%S', true, 0)  }}
B Date: {{ b | as_timestamp | timestamp_custom('%a %-d %b %H:%M:%S', true, 0)  }}
A->B Delta duration: {{ delta_duration(a, b) }}
B->A Delta duration: {{ delta_duration(b, a) }}
{% set b = a - timedelta(hours=3, seconds=4) %}
A Date: {{ a | as_timestamp | timestamp_custom('%a %-d %b %H:%M:%S', true, 0)  }}
B Date: {{ b | as_timestamp | timestamp_custom('%a %-d %b %H:%M:%S', true, 0)  }}
A->B Delta duration: {{ delta_duration(a, b) }}
B->A Delta duration: {{ delta_duration(b, a) }}
{% set b = a - timedelta(hours=3) %}
A Date: {{ a | as_timestamp | timestamp_custom('%a %-d %b %H:%M:%S', true, 0)  }}
B Date: {{ b | as_timestamp | timestamp_custom('%a %-d %b %H:%M:%S', true, 0)  }}
A->B Delta duration: {{ delta_duration(a, b) }}
B->A Delta duration: {{ delta_duration(b, a) }}

Formating 34.2 hours: {{ format_hours(34.2) }}
Formating 34.223 hours: {{ format_hours(34.223) }}
Formating 4:23:23: {{ format_hms('4:23:23') }}
Formating 4:0:23: {{ format_hms('4:0:23') }}
Formating 4:0:0: {{ format_hms('4:0:0') }}
```
... and you should see:
```
Date: Wed 10 Sep 00:39
Smart time: 5d ago
Short time: 5d ago
Long time: 5 days ago
Delta Min: -7200
Delta Hours: -120
Delta days: -5

Date: Sun 14 Sep 19:38
Smart time: 5h 30s ago
Short time: 5h 30s ago
Long time: Today
Delta Min: -301
Delta Hours: -5
Delta days: 0

Date: Mon 15 Sep 05:39
Smart time: in 5h 1m
Short time: in 5h 1m
Long time: Tomorrow
Delta Min: 301
Delta Hours: 5
Delta days: 0

Date: Wed 24 Dec 00:39
Smart time: in 3 months
Short time: in 100d 1h
Long time: in 3 months
Delta Min: 144060
Delta Hours: 2401
Delta days: 100

Date: Sat 7 Jun 00:39
Smart time: 3 months ago
Short time: 100d ago
Long time: 3 months ago
Delta Min: -144000
Delta Hours: -2400
Delta days: -100

A Date: Sat 7 Jun 00:39:00
B Date: Fri 6 Jun 21:34:56
A->B Delta duration: +3h 4m 4s
B->A Delta duration: -3h 4m 4s

A Date: Sat 7 Jun 00:39:00
B Date: Fri 6 Jun 21:38:56
A->B Delta duration: +3h 4s
B->A Delta duration: -3h 4s

A Date: Sat 7 Jun 00:39:00
B Date: Fri 6 Jun 21:39:00
A->B Delta duration: +3h 
B->A Delta duration: -3h 

Formating 34.2 hours: 34h 12m
Formating 34.223 hours: 34h 13m 23s
Formating 4:23:23: 4h 23m 23s
Formating 4:0:23: 4h 23s
Formating 4:0:0: 4h
```

```long_time``` displays timestamps in days, weeks and months; ```short_time``` in days, hours, minutes and seconds.

```smart_time``` displays timestamps 14 days before or after now() using ```long_time```, otherwise it uses ```short_time```.

```delta_days```, ```delta_hours```, ```delta_mins``` all return the time delta from now() in either days, hours or minutes, using negative values for in the past.

```delta_duration``` takes two timestamps and returns the days, hours, mins and seconds between them as a string.

```format_hms``` takes a string of the formation h:m:s and turns it into 'h m s' with zero values removed - try it you'll see, it's good in dashboards!

```format_hours``` takes a decimal hours and formats it into 'h m s' format, again with zero values remove.

# Examples
In an entity-row in a lovelace dashboard (using [custom:template-entity-row](https://github.com/thomasloven/lovelace-template-entity-row)):

```
      - type: custom:template-entity-row
        entity: sensor.daylight_savings_days_until
        icon: mdi:calendar-clock
        name: Next Clock Change
        state: |-
          {%- from 'handy_time.jinja' import smart_time -%} {{
          smart_time(states('sensor.daylight_savings_next')) }}
```

Here with secondary text in the entity-row:
```
      - type: custom:template-entity-row
        entity: input_datetime.last_power_cut
        name: Last Power Cut
        state: |-
          {%- from 'handy_time.jinja' import smart_time -%}
          {{ smart_time(states('input_datetime.last_power_cut')) }}
        secondary: >-
          Last cut on {{ states('input_datetime.last_power_cut') | as_timestamp | timestamp_custom('%a %-d %b %H:%M', true, 0)  }}
```
...which looks like this:

<img width="480" height="65" alt="Screenshot 2025-09-14 at 23 07 26" src="https://github.com/user-attachments/assets/824910e9-77e0-45b8-b0e2-847b7f5785fb" />


...or...
```
      - entity: sensor.system_monitor_last_boot
        type: custom:template-entity-row
        name: Alfred Uptime
        state: |-
          {%- from 'handy_time.jinja' import smart_time -%}
          {{ smart_time(states('sensor.system_monitor_last_boot')) }}
        secondary: >-
          ▲ {{ states('sensor.system_monitor_last_boot') | as_datetime |
          as_local | as_timestamp | timestamp_custom('%d %b %G %H:%M:%S', true,
          0) }}

      - entity: sensor.uptime
        type: custom:template-entity-row
        name: Home Assistant {{ states('sensor.current_version') }} Uptime
        state: |-
          {%- from 'handy_time.jinja' import smart_time -%}
          {{ smart_time(states('sensor.uptime')) }}
        secondary: >-
          ▲ {{ states('sensor.uptime') | as_datetime | as_local | as_timestamp |
          timestamp_custom('%d %b %G %H:%M:%S', true, 0) }}
```
... which looks like this:

<img width="460" height="110" alt="Screenshot 2025-09-14 at 23 11 56" src="https://github.com/user-attachments/assets/87db7d16-07b4-460a-a8b0-543987919d40" />

...or from my Formula 1 sensor:

<img width="476" height="64" alt="Screenshot 2025-09-14 at 23 07 45" src="https://github.com/user-attachments/assets/41a131fe-3b4c-4e79-9266-9f6a5eb4c450" />

You can use the macros in scripts and automations as well - anywhere a template can be used - they return strings like all Jinja2 macros.
