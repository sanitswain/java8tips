Evolution of date time API
==========================
Working with dates in Java was challenging tasks from the day one. Java 1.0 started with ``java.util.Date`` class to support date functionality but it had several problems and limitations. Despite its name, this class doesn't represent a date but a specific instant in time with millisecond precision. Its hazy design decision of using offsets: the year starts from 1900 and months are zero index based were misleading to the users. As an example if you want to represent March 18, 2014, we have to create instance of Date as follows.

Date date = new Date(114, 2, 18);

Here in the year field we have to pass year as 114 (2014-1900) and 2 as 3rd month which are quite confusing. Date had some of getXXX, setXXX methods to interpret dates as year, month, day, hour, minute, and second values and a ``Date(String)`` for parsing of date strings. Unfortunately, the API for these functions were not easy for internationalization. Another problem could be the arguments given to the Date API methods don't fall within any specific ranges; for example, a date may be specified as January 32 and is interpreted as meaning February 1. There is no explicit control over it.

To overcome all these limitations many of Date class methods were deprecated and ``java.util.Calendar`` class was introduced in Java 1.1, but it still couldn't meet the expectations. It solved some of Date class issues; internally handling offset values: passing 2014 as year rather than passing 114, daling with localization etc, but it has introduced some other problems. Calendar has similar problems and design flaws given below that lead to the error prone code.

* Constants were added in Calendar class but still month is zero index based.
* Calendar class is mutable so thread safety is always a question for it.
* It is very complicated to do date calculations. In fact, there is no simple, efficient way to calculate the days between two dates.
* ``java.text.DateFormat`` were introduced for the purpose of parsing of date strings but it isn't thread-safe. Following example shows the serious problem can occure when DateFormat is used in multi thraded scenarios.

  .. code:: java

    SimpleDateFormat sdf = new SimpleDateFormat("ddMMyyyy");
    ExecutorService es = Executors.newFixedThreadPool(5);
    for (int i = 0; i < 10; i++) {
        es.submit(() -> {
            try {
               System.out.println(sdf.parse("15081947"));
            } catch (ParseException e) {
               e.printStackTrace();
            }
        });
    }
    es.shutdown();
	
	
    Output:
    -------
    Fri Aug 15 00:00:00 IST 1947
    Mon Aug 11 00:00:00 IST 1947
    Fri Aug 15 00:00:00 IST 1947
    Fri Aug 15 00:00:00 IST 1947

If you run the above code multiple times then you will see unexpected behaviors. The existing Java date and time classes are poor, mutable, and have unpredictable performance. Some of the third-party libraries, such as `Joda-Time` showed his interest to overcome the issues with both Date and Calendar classes and it become so popular that it won the attention of Java core development team to include similar features to the Java core API.

`JSR 310 <https://jcp.org/en/jsr/detail?id=310>`_ defines the specifications for new Date and Time API to tackle the problem of a complete date and time model, including dates and times (with and without time zones), durations and time periods, intervals, formatting and parsing. Project `ThreeTen <http://www.threeten.org/>`_ was created to integrate it into `JDK 8 <http://openjdk.java.net/projects/jdk8/>`_. The goals of new Date Time API are:

* Support standard time concepts including date, time, instant, and time-zone.
* Immutable implementations for thread-safety.
* Provide an effective API suitable for developer usability.
* Provide a limited set of calendar systems and be extensible to others in future.

Java 8 introduced a new package `java.time <https://docs.oracle.com/javase/8/docs/api/java/time/package-summary.html>`_ to provide a high quality date and time support in the native Java API.


java.time package
-----------------
`java.time <https://docs.oracle.com/javase/8/docs/api/java/time/package-summary.html>`_ package contains classes to represent basic date-time concepts: instants, durations, dates, times, time-zones and periods based on ISO calendar system. All the classes are immutable and thread-safe. Following are package level descriptions.

.. list-table::
   :header-rows: 1

   * - Package
     - Description

   * - `java.time.temporal <https://docs.oracle.com/javase/8/docs/api/java/time/temporal/package-summary.html>`_
     - Each date time instance is composed of fields. This package contains lower 
	 level access to those fields.

   * - `java.time.format <https://docs.oracle.com/javase/8/docs/api/java/time/format/package-summary.html>`_
     - Provides classes to print and parse dates and times. Instances are generally 
	 obtained from DateTimeFormatter, however DateTimeFormatterBuilder can be used if more power is needed.

   * - `java.time.chrono <https://docs.oracle.com/javase/8/docs/api/java/time/chrono/package-summary.html>`_
     - It contains the calendar neutral API ChronoLocalDate, ChronoLocalDateTime, 
	 ChronoZonedDateTime and Era. Actually the main API is build on ISO-8601 calendar system. 
	 However, there are other calendar systems: Hijrah Calendar, Japanese Calendar, 
	 Minguo Calendar, Thai Buddhist Calendar also exist for which this package provide support.
	 
   * - `java.time.zone <https://docs.oracle.com/javase/8/docs/api/java/time/zone/package-summary.html>`_
     - This package provides support for time-zones, their rules and the resulting gaps 
	 and overlaps in the local time-line typically caused by Daylight Saving Time.


Working with time zone
----------------------






Non ISO calendars
------------------