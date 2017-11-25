## `format`

In JSON documents, dates are represented as strings. Elasticsearch uses a set of preconfigured formats to recognize and parse these strings into a long value representing _milliseconds-since-the-epoch_ in UTC.

Besides the [built-in formats](mapping-date-format.html#built-in-date-formats "Built In Formats"), your own [custom formats](mapping-date-format.html#custom-date-formats "Custom date formats") can be specified using the familiar `yyyy/MM/dd` syntax:
    
    
    PUT my_index
    {
      "mappings": {
        "my_type": {
          "properties": {
            "date": {
              "type":   "date",
              "format": "yyyy-MM-dd"
            }
          }
        }
      }
    }

Many APIs which support date values also support [date math](common-options.html#date-math "Date Mathedit") expressions, such as `now-1m/d` — the current time, minus one month, rounded down to the nearest day.

![Tip](images/icons/tip.png)

The `format` setting must have the same setting for fields of the same name in the same index. Its value can be updated on existing fields using the [PUT mapping API](indices-put-mapping.html "Put Mapping").

### Custom date formats

Completely customizable date formats are supported. The syntax for these is explained [in the Joda docs](http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html).

### Built In Formats

Most of the below dates have a `strict` companion dates, which means, that year, month and day parts of the week must have prepending zeros in order to be valid. This means, that a date like `5/11/1` would not be valid, but you would need to specify the full date, which would be `2005/11/01` in this example. So instead of `date_optional_time` you would need to specify `strict_date_optional_time`.

The following tables lists all the defaults ISO formats supported:

`epoch_millis`
     A formatter for the number of milliseconds since the epoch. Note, that this timestamp is subject to the limits of a Java `Long.MIN_VALUE` and `Long.MAX_VALUE`. 
`epoch_second`
     A formatter for the number of seconds since the epoch. Note, that this timestamp is subject to the limits of a Java `Long.MIN_VALUE` and `Long. MAX_VALUE` divided by 1000 (the number of milliseconds in a second). 
`date_optional_time` or `strict_date_optional_time`
     A generic ISO datetime parser where the date is mandatory and the time is optional. [Full details here](http://www.joda.org/joda-time/apidocs/org/joda/time/format/ISODateTimeFormat.html#dateOptionalTimeParser--). 
`basic_date`
     A basic formatter for a full date as four digit year, two digit month of year, and two digit day of month: `yyyyMMdd`. 
`basic_date_time`
     A basic formatter that combines a basic date and time, separated by a _T_ : `yyyyMMdd'T'HHmmss.SSSZ`. 
`basic_date_time_no_millis`
     A basic formatter that combines a basic date and time without millis, separated by a _T_ : `yyyyMMdd'T'HHmmssZ`. 
`basic_ordinal_date`
     A formatter for a full ordinal date, using a four digit year and three digit dayOfYear: `yyyyDDD`. 
`basic_ordinal_date_time`
     A formatter for a full ordinal date and time, using a four digit year and three digit dayOfYear: `yyyyDDD'T'HHmmss.SSSZ`. 
`basic_ordinal_date_time_no_millis`
     A formatter for a full ordinal date and time without millis, using a four digit year and three digit dayOfYear: `yyyyDDD'T'HHmmssZ`. 
`basic_time`
     A basic formatter for a two digit hour of day, two digit minute of hour, two digit second of minute, three digit millis, and time zone offset: `HHmmss.SSSZ`. 
`basic_time_no_millis`
     A basic formatter for a two digit hour of day, two digit minute of hour, two digit second of minute, and time zone offset: `HHmmssZ`. 
`basic_t_time`
     A basic formatter for a two digit hour of day, two digit minute of hour, two digit second of minute, three digit millis, and time zone off set prefixed by _T_ : `'T'HHmmss.SSSZ`. 
`basic_t_time_no_millis`
     A basic formatter for a two digit hour of day, two digit minute of hour, two digit second of minute, and time zone offset prefixed by _T_ : `'T'HHmmssZ`. 
`basic_week_date` or `strict_basic_week_date`
     A basic formatter for a full date as four digit weekyear, two digit week of weekyear, and one digit day of week: `xxxx'W'wwe`. 
`basic_week_date_time` or `strict_basic_week_date_time`
     A basic formatter that combines a basic weekyear date and time, separated by a _T_ : `xxxx'W'wwe'T'HHmmss.SSSZ`. 
`basic_week_date_time_no_millis` or `strict_basic_week_date_time_no_millis`
     A basic formatter that combines a basic weekyear date and time without millis, separated by a _T_ : `xxxx'W'wwe'T'HHmmssZ`. 
`date` or `strict_date`
     A formatter for a full date as four digit year, two digit month of year, and two digit day of month: `yyyy-MM-dd`. 
`date_hour` or `strict_date_hour`
     A formatter that combines a full date and two digit hour of day: `yyyy-MM-dd'T'HH`. 
`date_hour_minute` or `strict_date_hour_minute`
     A formatter that combines a full date, two digit hour of day, and two digit minute of hour: `yyyy-MM-dd'T'HH:mm`. 
`date_hour_minute_second` or `strict_date_hour_minute_second`
     A formatter that combines a full date, two digit hour of day, two digit minute of hour, and two digit second of minute: `yyyy-MM-dd'T'HH:mm:ss`. 
`date_hour_minute_second_fraction` or `strict_date_hour_minute_second_fraction`
     A formatter that combines a full date, two digit hour of day, two digit minute of hour, two digit second of minute, and three digit fraction of second: `yyyy-MM-dd'T'HH:mm:ss.SSS`. 
`date_hour_minute_second_millis` or `strict_date_hour_minute_second_millis`
     A formatter that combines a full date, two digit hour of day, two digit minute of hour, two digit second of minute, and three digit fraction of second: `yyyy-MM-dd'T'HH:mm:ss.SSS`. 
`date_time` or `strict_date_time`
     A formatter that combines a full date and time, separated by a _T_ : `yyyy-MM-dd'T'HH:mm:ss.SSSZZ`. 
`date_time_no_millis` or `strict_date_time_no_millis`
     A formatter that combines a full date and time without millis, separated by a _T_ : `yyyy-MM-dd'T'HH:mm:ssZZ`. 
`hour` or `strict_hour`
     A formatter for a two digit hour of day: `HH`
`hour_minute` or `strict_hour_minute`
     A formatter for a two digit hour of day and two digit minute of hour: `HH:mm`. 
`hour_minute_second` or `strict_hour_minute_second`
     A formatter for a two digit hour of day, two digit minute of hour, and two digit second of minute: `HH:mm:ss`. 
`hour_minute_second_fraction` or `strict_hour_minute_second_fraction`
     A formatter for a two digit hour of day, two digit minute of hour, two digit second of minute, and three digit fraction of second: `HH:mm:ss.SSS`. 
`hour_minute_second_millis` or `strict_hour_minute_second_millis`
     A formatter for a two digit hour of day, two digit minute of hour, two digit second of minute, and three digit fraction of second: `HH:mm:ss.SSS`. 
`ordinal_date` or `strict_ordinal_date`
     A formatter for a full ordinal date, using a four digit year and three digit dayOfYear: `yyyy-DDD`. 
`ordinal_date_time` or `strict_ordinal_date_time`
     A formatter for a full ordinal date and time, using a four digit year and three digit dayOfYear: `yyyy-DDD'T'HH:mm:ss.SSSZZ`. 
`ordinal_date_time_no_millis` or `strict_ordinal_date_time_no_millis`
     A formatter for a full ordinal date and time without millis, using a four digit year and three digit dayOfYear: `yyyy-DDD'T'HH:mm:ssZZ`. 
`time` or `strict_time`
     A formatter for a two digit hour of day, two digit minute of hour, two digit second of minute, three digit fraction of second, and time zone offset: `HH:mm:ss.SSSZZ`. 
`time_no_millis` or `strict_time_no_millis`
     A formatter for a two digit hour of day, two digit minute of hour, two digit second of minute, and time zone offset: `HH:mm:ssZZ`. 
`t_time` or `strict_t_time`
     A formatter for a two digit hour of day, two digit minute of hour, two digit second of minute, three digit fraction of second, and time zone offset prefixed by _T_ : `'T'HH:mm:ss.SSSZZ`. 
`t_time_no_millis` or `strict_t_time_no_millis`
     A formatter for a two digit hour of day, two digit minute of hour, two digit second of minute, and time zone offset prefixed by _T_ : `'T'HH:mm:ssZZ`. 
`week_date` or `strict_week_date`
     A formatter for a full date as four digit weekyear, two digit week of weekyear, and one digit day of week: `xxxx-'W'ww-e`. 
`week_date_time` or `strict_week_date_time`
     A formatter that combines a full weekyear date and time, separated by a _T_ : `xxxx-'W'ww-e'T'HH:mm:ss.SSSZZ`. 
`week_date_time_no_millis` or `strict_week_date_time_no_millis`
     A formatter that combines a full weekyear date and time without millis, separated by a _T_ : `xxxx-'W'ww-e'T'HH:mm:ssZZ`. 
`weekyear` or `strict_weekyear`
     A formatter for a four digit weekyear: `xxxx`. 
`weekyear_week` or `strict_weekyear_week`
     A formatter for a four digit weekyear and two digit week of weekyear: `xxxx-'W'ww`. 
`weekyear_week_day` or `strict_weekyear_week_day`
     A formatter for a four digit weekyear, two digit week of weekyear, and one digit day of week: `xxxx-'W'ww-e`. 
`year` or `strict_year`
     A formatter for a four digit year: `yyyy`. 
`year_month` or `strict_year_month`
     A formatter for a four digit year and two digit month of year: `yyyy-MM`. 
`year_month_day` or `strict_year_month_day`
     A formatter for a four digit year, two digit month of year, and two digit day of month: `yyyy-MM-dd`. 