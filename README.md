# handy-time
Custom template for Home Assistant with some handy macros to format time for display on a dashboard, in a text message or an audio announcement.

This isn't a macro set to manipulate time values, but instead to display or print them in a human readable way.

I wrote this as a light and simple macro to deal with easily displaying a timestamp on the Lovelace dashboard that could handle times both in the past and in the future.

There is a self-documenting template below that you can paste into the ```Developers Tools->Template``` to see exactly what is going on.

# Install
Copy the handy_time.jinja file into your config/custom_templates directory in Home Assistant, and then either restart Home Assistant or in the ```Developers Tools->Actions``` run the ```Reload custom Jinja2 template``` action.

Once loaded, anywhere you want to use one or more of the macros, import them like this (just list the ones you want to use):
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
... and you should see (with your date obviously!):
```
Date: Wed 10 Sep 00:40
Smart time: 5d ago
```

If you want to see everything, paste this in to the ```Developers Tools->Template```:
```
{# ---------------------------------------------------------------------------
  handy_time.jinja — User guide & examples (Home Assistant template output)
  Paste into: Developer Tools → Template

  Notes about HA built-ins:
    - time_since(x) is meaningful for past times (future times are not useful)
    - time_until(x) is meaningful for future times (past times are not useful)
--------------------------------------------------------------------------- #}

{%- from 'handy_time.jinja' import long_time, short_time, smart_time,
   delta_mins, delta_hours, delta_days, delta_duration, format_hms, format_hours -%}

{%- set n = now() -%}

{%- macro dt(t) -%}
{{ t | as_timestamp | timestamp_custom('%a %-d %b %H:%M:%S', true, 0) }}
{%- endmacro -%}

{%- macro rel_block(title, ts) -%}
{{ title }}
When:       {{ dt(ts) }}
smart_time: {{ smart_time(ts) }}
short_time: {{ short_time(ts) }}
long_time:  {{ long_time(ts) }}
delta_mins: {{ delta_mins(ts) }}
delta_hours:{{ delta_hours(ts) }}
delta_days: {{ delta_days(ts) }}
{%- endmacro -%}

{%- macro builtin_block_past(ts) -%}
HA time_since: {{ time_since(ts) }}
{%- endmacro -%}

{%- macro builtin_block_future(ts) -%}
HA time_until: {{ time_until(ts) }}
{%- endmacro -%}

{%- macro num_block(title, ts) -%}
{{ title }}
Target:     {{ dt(ts) }}
delta_mins: {{ delta_mins(ts) }}
delta_hours:{{ delta_hours(ts) }}
delta_days: {{ delta_days(ts) }}
{%- endmacro -%}

{%- macro duration_block(title, a, b) -%}
{{ title }}
A: {{ dt(a) }}
B: {{ dt(b) }}
A→B default:                 {{ delta_duration(a, b) }}
A→B show_plus=false:         {{ delta_duration(a, b, false) }}
A→A default zero:            {{ delta_duration(a, a) }}
A→A zero_text="now":         {{ delta_duration(a, a, true, "now") }}
A→A no plus, zero_text="-":  {{ delta_duration(a, a, false, "-") }}
B→A default:                 {{ delta_duration(b, a) }}
{%- endmacro -%}

handy_time.jinja — examples
Now: {{ dt(n) }}

================================================================================
1) Relative time for UI labels
================================================================================
{%- set ts = n + timedelta(hours=5, minutes=12, seconds=8) %}
{{ rel_block('Near future (compact countdown)', ts) }}
{{ builtin_block_future(ts) }}
---
{%- set ts = n - timedelta(hours=3, minutes=4, seconds=4) %}
{{ rel_block('Near past ("last seen")', ts) }}
{{ builtin_block_past(ts) }}
---
{%- set ts = n + timedelta(days=52, hours=1, minutes=1) %}
{{ rel_block('Mid-range future (weeks/months wording)', ts) }}
{{ builtin_block_future(ts) }}
---
{%- set ts = n - timedelta(days=430) %}
{{ rel_block('Far past (months/years wording)', ts) }}
{{ builtin_block_past(ts) }}

================================================================================
2) short_time stability (30-second buckets)
================================================================================
   Dashboards often refresh ~1 minute; bucketing reduces “jitter”.
   Behaviour:
     - <30s  => "now"
     - 30s+  => bucketed to 30s steps
     - seconds omitted once >= 1 day 
{% set ts = n + timedelta(seconds=20) %}
{{ rel_block('Future +20s (shows "now")', ts) }}
{{ builtin_block_future(ts) }}
---
{%- set ts = n + timedelta(seconds=30) %}
{{ rel_block('Future +30s (shows "in 30s")', ts) }}
{{ builtin_block_future(ts) }}
---
{%- set ts = n - timedelta(seconds=29) %}
{{ rel_block('Past -29s (shows "now")', ts) }}
{{ builtin_block_past(ts) }}
---
{%- set ts = n - timedelta(seconds=30) %}
{{ rel_block('Past -30s (shows "30s ago")', ts) }}
{{ builtin_block_past(ts) }}

================================================================================
3) smart_time threshold (short vs long)
================================================================================
   <14 days => short_time, >=14 days => long_time
{% set ts = n + timedelta(days=13, hours=23) %}
{{ rel_block('13d 23h away (uses short_time)', ts) }}
{{ builtin_block_future(ts) }}
---
{%- set ts = n + timedelta(days=14) %}
{{ rel_block('14d away (uses long_time)', ts) }}
{{ builtin_block_future(ts) }}

================================================================================
4) Numeric deltas (rounded to nearest, half-up)
================================================================================
   Sign convention: future +, past -
   Rounding: half-up (2.5 -> 3, -2.5 -> -3)
{{ num_block('Future +2h 24m (2.4h -> 2)', n + timedelta(hours=2, minutes=24)) }}
---
{{ num_block('Future +2h 30m (2.5h -> 3)', n + timedelta(hours=2, minutes=30)) }}
---
{{ num_block('Past -2h 30m (-2.5h -> -3)', n - timedelta(hours=2, minutes=30)) }}

================================================================================
5) Duration between two timestamps (delta_duration)
================================================================================

{%- set a = n - timedelta(hours=1, minutes=2, seconds=3) -%}
{% set b = n - timedelta(hours=4, minutes=6, seconds=7) %}
{{ duration_block('delta_duration(ts1, ts2, show_plus=true, zero_text="0s")', a, b) }}

================================================================================
6) Formatting helpers
================================================================================
format_hours(decimal_hours) — decimal hours → "Xh Ym Zs"
format_hours(1.5):     {{ format_hours(1.5) }}
format_hours(0.0167):  {{ format_hours(0.0167) }}
format_hours(-2.5083): {{ format_hours(-2.5083) }}
---
format_hms("h:m:s" | "h:m" | "s") → "Xh Ym Zs"
Convention: for m:s, pass 0:m:s

format_hms('4:23:23'): {{ format_hms('4:23:23') }}
format_hms('4:3'):     {{ format_hms('4:3') }}
format_hms('0:4:3'):   {{ format_hms('0:4:3') }}
format_hms('90'):      {{ format_hms('90') }}
format_hms('1:120'):   {{ format_hms('1:120') }}
```
... and you should see (again your date will be different):
```
handy_time.jinja — examples
Now: Tue 13 Jan 02:02:08

================================================================================
1) Relative time for UI labels
================================================================================
Near future (compact countdown)
When:       Tue 13 Jan 07:14:16
smart_time: in 5h 12m
short_time: in 5h 12m
long_time:  Today
delta_mins: 312
delta_hours:5
delta_days: 0
HA time_until: 5 hours
---
Near past ("last seen")
When:       Mon 12 Jan 22:58:04
smart_time: 3h 4m ago
short_time: 3h 4m ago
long_time:  Today
delta_mins: -184
delta_hours:-3
delta_days: 0
HA time_since: 3 hours
---
Mid-range future (weeks/months wording)
When:       Fri 6 Mar 03:03:08
smart_time: in 7 weeks
short_time: in 52d 1h
long_time:  in 7 weeks
delta_mins: 74941
delta_hours:1249
delta_days: 52
HA time_until: 2 months
---
Far past (months/years wording)
When:       Sat 9 Nov 02:02:08
smart_time: 14 months ago
short_time: 430d ago
long_time:  14 months ago
delta_mins: -619200
delta_hours:-10320
delta_days: -430
HA time_since: 1 year

================================================================================
2) short_time stability (30-second buckets)
================================================================================
   Dashboards often refresh ~1 minute; bucketing reduces “jitter”.
   Behaviour:
     - <30s  => "now"
     - 30s+  => bucketed to 30s steps
     - seconds omitted once >= 1 day 

Future +20s (shows "now")
When:       Tue 13 Jan 02:02:28
smart_time: now
short_time: now
long_time:  Today
delta_mins: 0
delta_hours:0
delta_days: 0
HA time_until: 20 seconds
---
Future +30s (shows "in 30s")
When:       Tue 13 Jan 02:02:38
smart_time: now
short_time: now
long_time:  Today
delta_mins: 0
delta_hours:0
delta_days: 0
HA time_until: 30 seconds
---
Past -29s (shows "now")
When:       Tue 13 Jan 02:01:39
smart_time: now
short_time: now
long_time:  Today
delta_mins: 0
delta_hours:0
delta_days: 0
HA time_since: 29 seconds
---
Past -30s (shows "30s ago")
When:       Tue 13 Jan 02:01:38
smart_time: 30s ago
short_time: 30s ago
long_time:  Today
delta_mins: -1
delta_hours:0
delta_days: 0
HA time_since: 30 seconds

================================================================================
3) smart_time threshold (short vs long)
================================================================================
   <14 days => short_time, >=14 days => long_time

13d 23h away (uses short_time)
When:       Tue 27 Jan 01:02:08
smart_time: in 13d 22h 59m
short_time: in 13d 22h 59m
long_time:  in 14 days
delta_mins: 20100
delta_hours:335
delta_days: 14
HA time_until: 14 days
---
14d away (uses long_time)
When:       Tue 27 Jan 02:02:08
smart_time: in 13d 23h 59m
short_time: in 13d 23h 59m
long_time:  in 14 days
delta_mins: 20160
delta_hours:336
delta_days: 14
HA time_until: 14 days

================================================================================
4) Numeric deltas (rounded to nearest, half-up)
================================================================================
   Sign convention: future +, past -
   Rounding: half-up (2.5 -> 3, -2.5 -> -3)
Future +2h 24m (2.4h -> 2)
Target:     Tue 13 Jan 04:26:08
delta_mins: 144
delta_hours:2
delta_days: 0
---
Future +2h 30m (2.5h -> 3)
Target:     Tue 13 Jan 04:32:08
delta_mins: 150
delta_hours:2
delta_days: 0
---
Past -2h 30m (-2.5h -> -3)
Target:     Mon 12 Jan 23:32:08
delta_mins: -150
delta_hours:-3
delta_days: 0

================================================================================
5) Duration between two timestamps (delta_duration)
================================================================================
delta_duration(ts1, ts2, show_plus=true, zero_text="0s")
A: Tue 13 Jan 01:00:05
B: Mon 12 Jan 21:56:01
A→B default:                 +3h 4m 4s
A→B show_plus=false:         3h 4m 4s
A→A default zero:            0s
A→A zero_text="now":         now
A→A no plus, zero_text="-":  -
B→A default:                 -3h 4m 4s

================================================================================
6) Formatting helpers
================================================================================
format_hours(decimal_hours) — decimal hours → "Xh Ym Zs"
format_hours(1.5):     1h 30m
format_hours(0.0167):  1m
format_hours(-2.5083): -2h 30m 30s
---
format_hms("h:m:s" | "h:m" | "s") → "Xh Ym Zs"
Convention: for m:s, pass 0:m:s

format_hms('4:23:23'): 4h 23m 23s
format_hms('4:3'):     4h 3m 
format_hms('0:4:3'):   4m 3s
format_hms('90'):      1m 30s
format_hms('1:120'):   3h
```

```long_time``` displays timestamps in days, weeks and months; ```short_time``` in days, hours, minutes and seconds.

```smart_time``` displays timestamps 14 days before or after now() using ```long_time```, otherwise it uses ```short_time```.

```delta_days```, ```delta_hours```, ```delta_mins``` all return the time delta from now() in either days, hours or minutes, using negative values for in the past.

```delta_duration``` takes two timestamps and returns the days, hours, mins and seconds between them as a string.

```format_hms``` takes a string of the formation h:m:s and turns it into 'h m s' with zero values removed - try it you'll see, it's great in dashboards!

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
