get_events_between
==================

MySQL procedure for determining dates and times when a recurring event will occur. This is a MySQL version of https://github.com/bakineggs/recurring_events_for, but without timezones implemented.

== Usage

  The recurring_events_for function takes 4 parameters:

    1. Range Start (TIMESTAMP)
      This parameter specifies the date and time before which no recurrences
      should be returned. If a recurrence starts before this time but ends after
      it, the recurrence will be returned. This should be in UTC.
  
    2. Range End (TIMESTAMP)
      This parameter specifies the date and time after which no recurrences
      should be returned. If a recurrence ends after this time but starts before
      it, the recurrence will be returned. This should be in UTC.
  
    3. Time Zone (CHARACTER VARYING)
      This specifies the time zone which should be used when determining if an
      all day event occurs within the given range.

    4. Event Limit (INTEGER)
      As an optimization over running a query such as
        SELECT * from get_events_between(...) LIMIT x
      you can include a parameter to limit the recurrences that are returned,
      instead of returning them all and then limiting what was selected. Pass
      in NULL for this parameter if you do not want a limit.

  The function returns one row for each occurrence of an event. The events
  table columns starts_on/ends_on or starts_at/ends_at will be the date/time
  of that occurrence. starts_at/ends_at will be adjusted for DST such that
  the event will always start at the same time in the event's time zone. This
  means that the utc time of the event will change over DST boundaries.

== How Events And Recurrence Patterns Are Stored

  events table:

    starts_on (DATE), ends_on (DATE):
      The first (or only) date that an event occurs should be stored in the
      starts_on column. For multi-day events, set the ends_on column to
      the final day of the event.

    starts_at (TIMESTAMP), ends_at (TIMESTAMP):
      The first (or only) timespan that an event occurs should be stored in
      the starts_at and ends_at columns. These columns should not be used if
      the starts_on or ends_on columns are also in use.

    frequency (CHARACTER VARYING):
      This column specifies the frequency at which this event recurs. If no
      specific rules for this event are given in the event_recurrences table,
      the event will recur on this frequency after the original date/time.
      Possible values are 'once', 'daily', 'weekly', 'monthly', and 'yearly'.

    separation (INTEGER):
      The number of intervals at en event's frequency in between occurrences of
      the event. For instance, if an event occurs every other week, it has a
      frequency of weekly and a separation of 2 because there are 2 weeks in
      between occurrences. This column defaults to 1.

    count (INTEGER):
      Specifies a limit number of times the event will occur. Set this column to
      NULL for no limit.

    until (TIMESTAMP):
      Specifies a limiting date and time after which no recurrences will be
      generated for this event. Set this column to NULL for no limit.

  event_recurrences table:

    for daily recurring events:
      This table is not used for daily recurring events. Adding entries to this
      table for a daily recurring event will cause unspecified results.

    for weekly recurring events:

      day (INTEGER):
        The day of the week the event occurs.
        0 = Sunday, 1 = Monday, ..., 6 = Saturday

      week (INTEGER), month (INTEGER):
        These columns should be set to NULL for weekly recurring events.
        Setting these columns to non-NULL values will cause unspecified results.

    for monthly recurring events:

      week (INTEGER):
        If non-NULL, this specifies the week of the month in which the event
        occurs. Positive numbers indicate the week from the start of the month.
        1 = 1st week of the month, 2 = 2nd week of the month, etc.
        Negative numbers indicate the week before the end of the month.
        -1 = last week of the month, -2 = 2nd to last week of the month, etc.

      day (INTEGER):
        If the week column is NULL, the day column specifies the day of the
        month that the event occurs. If the week column is non-NULL, the day
        column specifies the day of the week that the event occurs in that week
        of the month.

      month (INTEGER):
        This column should be set to NULL for monthly recurring events.
        Setting this column to a non-NULL value will cause unspecified results.

    for yearly recurring events:

      month (INTEGER):
        If the month column is non-NULL, it specifies the month for which this
        pattern should be used. If it is NULL, this pattern will be for the
        month of the original date/time of the event.

      week (INTEGER), day (INTEGER):
        The usage for the week and day columns of a yearly recurring event are
        exactly the same as their usage for monthly recurring events.

  event_cancellations table:

    date (DATE):
      The date of the recurrence of an event which should be cancelled. If the
      event spans multiple days, this column should be set to the first date on
      which the recurrence to be cancelled falls.


== Credits

  This function was originally posted on David Wheeler (justatheory)'s blog:
  http://www.justatheory.com/computers/databases/postgresql/recurring_events.html

  It was converted by Dan Barry (bakineggs.com) in a GitHub repository:
  http://github.com/danbarry/recurring_events_for

  I converted it to a MySQL implementation in another GitHub repository:
  https://github.com/SBShane/get_events_between
  
  I did not fork this from the original because I will not be relying on the PostgreSQL version for updates.