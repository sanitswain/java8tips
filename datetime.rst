Evolution of date time
======================
Working with dates in Java was challenging tasks from the day one. Java 1.0 started with ``java.util.Date`` class to support date functionality but it had several problems and limitations. Despite its name, this class doesn't represent a date but a specific instant in time with millisecond precision. Its hazy design decision of using offsets: the year starts from 1900 and months are zero index based were misleading to the users. As an example if you want to represent March 18, 2014, we have to create instance of Date as follows.

Date date = new Date(114, 2, 18);

Here in the year field we have to pass year as 114 (2014-1900) and 2 as 3rd month which are quite confusing. Date had some of getXXX, setXXX methods to interpret dates as year, month, day, hour, minute, and second values and a ``Date(String)`` for parsing of date strings. Unfortunately, the API for these functions were not easy for internationalization. Another problem could be the arguments given to the Date API methods don't fall within any specific ranges; for example, a date may be specified as January 32 and is interpreted as meaning February 1. There is no explicit control over it.

To overcome all these limitations many of Date class methods were deprecated and ``java.util.Calendar`` class was introduced in Java 1.1, but it still couldn't meet the expectations. It solved some of Date class issues; internally handling offset values: passing 2014 as year rather than passing 114, daling with localization etc, but it has introduced some other problems. Calendar has similar problems and design flaws given below that lead to the error prone code.

* Constants were added in Calendar class but still month is zero index based.
* Calendar class is mutable so thread safety is always a question for it.
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

If you run the above code multiple time then you will see unexpcetd behaviors. Some of the third-party libraries, such as `Joda-Time` showed his interest to overcome issues with both Date and Calendar class and it become so popular that it won the attention of Java core team to include similar features to the Java core API.

Working with time zone
----------------------






Non ISO calendars
------------------