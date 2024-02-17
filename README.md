# handy-time
Custom template for Home Assistant with some handy time macros - mainly smart_time which gives an english readable string for a time in the past or the future to the nearest 30 seconds for use on dashboards, it gets more accurate as the time delta from now becomes smaller - just as you would say it was the aim!
# Install
Copy the handy_time.jinja file into your config/custom_templates directory in Home Assistant, then anywhere you want to use one or more of the macros in some Jinja template code use:
```
{%- from 'handy_time.jinja' import long_time, short_time, smart_time,
   delta_mins, delta_hours, delta_days -%}
```
# To play with
In Developers Tools->Template try pasting this:
```
{%- from 'handy_time.jinja' import smart_time -%}
{%- set a = now() - timedelta(days=5) %}
{{ smart_time(a) }}
{%- set a = now() - timedelta(hours=5, seconds=55) %}
{{ smart_time(a) }}
{%- set a = now() + timedelta(hours=5, seconds=55) %}
{{ smart_time(a) }}
{%- set a = now() + timedelta(days=100) %}
{{ smart_time(a) }}
{%- set a = now() - timedelta(days=100) %}
{{ smart_time(a) }}
```
and you should see:
```
5d ago
5h 30s ago
in 5h 1m
in 3 months
3 months ago
```

# Examples
In an entity-row in a lovelace dashboard (using custom:template-entity-row):

```
      - type: custom:template-entity-row
        entity: input_datetime.last_power_cut
        name: Last Power Cut
        state: |-
          {%- from 'handy_time.jinja' import smart_time -%}
          {{ smart_time(states('input_datetime.last_power_cut')) }}
```
