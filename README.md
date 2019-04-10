# Create-useful-date-centric-service-programs-with-embedded-SQL.
Get the Day of the Week, Full Text Date Values, and More Within RPG Using SQL
THOMAS SNYDER  17 JANUARY 2012
 PRINT 
SQL
Create useful date-centric service programs with embedded SQL.

 

Welcome to 2012! All of our year-end processes are complete, and it's the beginning of a new year with a clean slate. 2011 was a very clean year date-wise because it ended on a Saturday. Typically, there is always special handling with the last week of the year for one reason or the other, which makes working with data a prevalent topic. For this article, I wanted to share a few handy little SQL functions that I've put into service programs to extend the existing RPG date resources. And this is all done with very minimal code in a reliable way that is highly reusable.

Some Simple Date Built-In Functions
RPG offers some useful built-in functions (BIFs) to make working with dates easy. Here are some common things you can do with the provided date BIFs:

 

%date—Returns the current date when you omit the parameter or converts a value to a date object with optional date formatting
%subdt—Allows you to pull the date apart into its pieces by specifying what part of the date you want
%diff—Returns the difference between two dates. The unit of the difference determines how the difference is represented.
 

     D today           S               D

     D future          S               D

     D dayDiff         S              4S 0

     D thisYear        S              4S 0

     D thisMonth       S              2S 0

     D thisDay         S              2S 0

     D displayBytes    S             52A

      /free

       today = %date();

       displayBytes = 'Today is: ' + %char(today);

       dsply displayBytes;

 

       thisYear = %subdt(today: *YEARS);

       thisMonth = %subdt(today: *MONTHS);

       thisDay = %subdt(today: *DAYS);

       displayBytes = 'Y: ' + %char(thisYear)

                   + ' M: ' + %char(thisMonth)

                   + ' D: ' + %char(thisDay);

       dsply displayBytes;

 

       future = %date('2012-07-01');

       dayDiff = %diff(future: today: *DAYS);

       displayBytes = 'Number of Days to July: '

                    + %char(dayDiff);

       dsply displayBytes;

 

 

       *inlr = *ON;

      /end-free                                              

 

This simple program demonstrates how to get the current date into a date variable and then pull the different parts of the date into separate variables. After that, the program initiates a future date to July 1, which could be a special date for which you want to know the number of days remaining before that date arrives. To determine how far away it is, you can compare to the current date and return the difference in days.

 

If you run the program, you'll see the following output:

 

DSPLY  Today is: 2012-01-18         

DSPLY  Y: 2012 M: 1 D: 18            

DSPLY  Number of Days to July: 165  

Things You Can't Easily Do with the RPG Date Variables
Even though you can do plenty of things with RPG date variables, there are a few things that aren't so easy in RPG without using some CEE* APIs, retrieving system variables, or writing the code yourself with some arithmetic. But some simple SQL date functions fill the gap and are also good to keep in mind for your embedded SQL code.

 

There are some redundant functions, of course, to get the Year, Month, and Date as illustrated above. Then there are functions that are easily accessible using SQL, such as Week of the Year, Day of the Year, Day of the Week, and Text Descriptions of the Day of the Week and Month of the Year.

Using the SYSIBM.SYSDUMMY1 Table
To take advantage of the date functionality available in SQL, you may want to access these functions without the use of data tables. To do this, you could use the SYSIBM.SYSDUMMY1 table provided with DB2. This is the same as the DUAL table provided with MySQL and Oracle. When using Microsoft SQL, you're not required to specify a table.

 

Here are several SQL date functions that I will be discussing:

 

SQL Function

Description

week

Gets the Week of the Year

dayOfYear

Gets the Day of the Year

dayOfWeek

Gets the Day of the Week

dayName

Gets the Readable Text Description for the Day of the Week

monthName

Gets the Readable Text Description for the Month of the Year

 

Note: There are also ISO versions for the week and dayOfWeek functions above called week_ISO and dayOfWeek_ISO, respectively.

 

With a single line of embedded SQL within your program, you can access the all of the functions listed above and more. For example, to get the day of the week in free-formatted RPG, you would execute the following, where argDate is a date variable and svReturn is a variable to receive the value: 

 

exec sql select dayOfWeek(:argDate) into :svReturn from SYSIBM/SYSDUMMY1;

getDateDOW
If you prefer to not use SQL all the time, you can simply wrap it up in a procedure like this:

 

      *-----------------------------------------------------------------

      * getDateDOW: Gets the Day of the Week for a Specified Date

      *-----------------------------------------------------------------

     P getDateDOW...

     P                 B                   EXPORT

     D getDateDOW...

     D                 PI             1S 0

     D   argDate                       D   const

     D svReturn        s              1S 0

      /free

        svReturn = *ZEROS;

        exec sql select dayOfWeek(:argDate)

             into :svReturn

             from SYSIBM/SYSDUMMY1;

        return svReturn;

      /end-free

     P                 E

 

Now anyone can have access to an easy-to-use procedure to get the day of the week from an RPG date. Another benefit to putting your code into procedures is that you can add more logic without having to change the calling program (Java programmers do the same thing). This would be useful if you were to add exception- handling to the procedure.

getDateWOY
The following procedure is to get the week of the year. This is the functionality that originally led me down this road. I wanted to group some billing history by the weeks.

 

      *-----------------------------------------------------------------

      * getDateWOY: Gets the Week of the Year for a Specified Date

      *-----------------------------------------------------------------

     P getDateWOY...

     P                 B                   EXPORT

     D getDateWOY...

     D                 PI             2S 0

     D   argDate                       D   const

     D svWoy           s              2S 0

      /free

        svWoy = *ZEROS;

        exec sql select week(:argDate)

             into :svWoy

             from SYSIBM/SYSDUMMY1;

        return svWoy;

      /end-free

     P                 E

getDateDOY
To get the day of the year, you can use the dayOfYear function as shown in this procedure:

 

      *-----------------------------------------------------------------

      * getDateDOY: Gets the Day of the Year for a Specified Date

      *-----------------------------------------------------------------

     P getDateDOY...

     P                 B                   EXPORT

     D getDateDOY...

     D                 PI             3S 0

     D   argDate                       D   const

     D svReturn        s              3S 0

      /free

        svReturn = *ZEROS;

        exec sql select dayOfYear(:argDate)

             into :svReturn

             from SYSIBM/SYSDUMMY1;

        return svReturn;

      /end-free

     P                 E

getDateDayName
Sometimes, it's desirable to display the full text name of the day of the week along with the date. It would be easy enough to use the getDateDOW value as an index into an array of values, but this functionality is easy enough through SQL.

 

      *-----------------------------------------------------------------

      * getDateDayName: Gets the Day Name for a Specified Date

      *-----------------------------------------------------------------

     P getDateDayName...

     P                 B                   EXPORT

     D getDateDayName...

     D                 PI            20A

     D   argDate                       D   const

     D svReturn        s             20A

      /free

        svReturn = *ZEROS;

        exec sql select dayName(:argDate)

             into :svReturn

             from SYSIBM/SYSDUMMY1;

        return svReturn;

      /end-free

     P                 E

getDateLongName
The getDateMonthName would be the same as getDateDayName using the monthName function instead. To make it more interesting, let's illustrate several functions together to make a function that will convert the date into a fully readable long date string as follows:

 

      *-----------------------------------------------------------------

      * getDateLongName: Gets the Full Long Text for a Specified Date

      *-----------------------------------------------------------------

     P getDateLongName...

     P                 B                   EXPORT

     D getDateLongName...

     D                 PI            40A

     D   argDate                       D   const

     D svReturn        s             40A

     D svDayName       s             20A

     D svMonthName     s             20A

     D svDayOfMonth    s              2S 0

     D svYear          s              4S 0

      /free

        svReturn = *ZEROS;

        exec sql select dayName(:argDate),

                        monthName(:argDate),

                        dayOfMonth(:argDate),

                        year(:argDate)

             into :svDayName, :svMonthName,

                  :svDayOfMonth, :svYear

             from SYSIBM/SYSDUMMY1;

        svReturn = %trim(svDayName) + ', '

                 + %trim(svMonthName) + ' '

                 + %char(svDayOfMonth) + ', '

                 + %char(svYear);

        return svReturn;

      /end-free

     P                 E

Testing It All Out
To put it all together, let's try a test program to review our results.

 

     D today           S               D

     D thisWOY         S              2S 0

     D thisDOY         S              3S 0

     D thisDOW         S              1S 0

     D thisDayName     S             20A

     D thisMonthName   S             20A

     D thisDateName    S             40A

     D displayBytes    S             52A

     D* Enter Prototypes Here…

      /free

       today = %date();

       displayBytes = 'Today is: ' + %char(today);

       dsply displayBytes;

 

       thisWOY = getDateWOY(today);

       displayBytes = 'The current week of the year is: '

                    + %char(thisWOY);

       dsply displayBytes;

 

       thisDOY = getDateDOY(today);

       displayBytes = 'The current day of the year is: '

                    + %char(thisDOY);

       dsply displayBytes;

 

       thisDOW = getDateDOW(today);

       displayBytes = 'The current day of the week is: '

                    + %char(thisDOW);

       dsply displayBytes;

 

       thisDayName = getDateDayName(today);

       displayBytes = 'The current day name is: '

                    + %trim(thisDayName);

       dsply displayBytes;

 

       thisMonthName = getDateMonthName(today);

       displayBytes = 'The current Month Name is: '

                    + %trim(thisMonthName);

       dsply displayBytes;

 

       thisDateName = getDateLongName(today);

       displayBytes = 'The current Date is: '

                    + %trim(thisDateName);

       dsply displayBytes;

       *inlr = *ON;

      /end-free

 

This main program will get the current date and put all of the new procedures to the test. When we run the program, we get the following results:

 

DSPLY  Today is: 2012-01-18    

DSPLY  The current week of the year is: 3              

DSPLY  The current day of the year is: 18              

DSPLY  The current day of the week is: 4               

DSPLY  The current day name is: Wednesday              

DSPLY  The current Month Name is: January              

DSPLY  The current Date is: Wednesday, January 18, 2012

 

I kept the procedures simple to focus on the objective. But you could take this a step further and create a single procedure that could accept some constant values like DATE_DOW, DATE_DOY, etc. to pass back the desired results to emulate %subdt if you could pass in *DOW, *DOY, etc.
