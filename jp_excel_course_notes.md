# Advanced Excel Tips (LinkedIn Course with Dennis Taylor)

TIPS
CMD + ~ = show/hide formulas
Formula tracing / auditing… (find the hotkeys)

ALT + = —> auto sum

If you want to copy as values only, you can right-click-drag using the border/edge of a highlighted range

To highlight a contiguous column, highlight the top cell, hold SHIFT key, and double click the bottom edge

Paste special allows you to do math operations to ranges — paste the value you want to use in the operation!

See in-line calculations/evaluations in a formula by highlighting the section in an entered formula in a cell and hitting FN+F9. (F9)

Show all named ranges and cells with F3 key; can also use Name Manager in Formulas tab
Can create named ranges from highlighted cells, including col headers, using Create Names From Selection on Formulas tab

Use In Formula -> Paste Name -> Paste List — will paste all named ranges and their formulae

CTRL+ENTER when entering a formula does something — I think it fills in a newly entered formula into all highlighted cells upon entry

Double click the + in the bottom right corner of a cell to fill down a column

For formulas that use other tabs in a workbook, we can SHIFT click from the first to the last tab and include them all in the current formula (assuming all sheets being used have the same layout)

CTRL + period (.) takes you to the bottom of a column

LOOKUP FUNCTIONS
CHOOSE func can select a month from a date string, etc

SWITCH func can act like if/else in-line, or replace VLOOKUP in this fashion, too  (like an in-line dictionary)

MATCH func will return the row in a column or range that the item is found.  If using whole column, row = actual row.  If range, just the nth row in that range

INDEX func uses a row and col lookup to find values in a table based on the row and col index numbers

Can use a combination of MATCH (to find row num) and INDEX (to extract data from MATCHed row and whichever col you name) to do a left-hand VLOOKUP

FORMULATEXT func takes the hidden formula of a cell and prints it to text in a diff cell

Can use combo of string + formula by separating in the formula bar with an &. Example: =“<—“&SUM(A1:A3)

UNIQUE func makes a unique list of a range that is dynamic.  Updates on its own

In 2019, XLOOKUP func replaces VLOOKUP and HLOOKUP — more powerful and flexible

POWER FUNCTIONS
COUNTIF — can use wildcards like “*” etc.
SUMIF and AVERAGEIF use the same structure as COUNTIF but add a third parameter which is the data range to sum or average if the entry is counted to begin with

The plural versions of these functions (COUNTIFS, SUMIFS, AVERAGEIFS, MAXIFS, MINIFS) simply let us use multiple evaluation parameters when deciding which rows to tabulate

SUBTOTAL —> lets you pick an operation (sum, avg, …) to use on a range.  Then when you use a SUBTOTAL function over a range that has SUBTOTALs in it, the outer function ignores the SUBTOTAL cells.  Convenient with data that is a lot of constituent parts down a full range

STATISTICAL FUNCTIONS
LARGE func will give the nth highest value in a range (nice).
SMALL does the opposite

COUNTIFBLANK func counts all blank cells in a range

COUNT func counts numerical data
COUNTA func counts any data

MATH FUNCTIONS
ROUND, ROUNDUP, ROUNDDOWN let you control the direction you round…
MROUND is Modulo Round — round to the nearest multiple of a value
CEILING and FLOOR are the cousins to MROUND

ODD and EVEN choose the nearest odd and even value moving away from zero

Can use MOD func to do conditional formatting via formula: MOD(ROW(A1), 5)=0   …will color every 5th row..

RAND, RANDARRAY, RANDBETWEEN —> generate random values

CONVERT func will convert measurements

AGGREGATE is like a super function —> lets you choose an operation to apply to a range AND lets you choose things, like #ERROR or #VALUE, to ignore.  You can also skip hidden rows — this is a BIG help for summing filtered rows.  This is a big time saver instead of trying to track down a single error many times.

ROMAN and ARABIC convert between number systems (sweet!).  By law you have to use Times New Roman in these cells.

DATE FUNCTIONS
DATE, YEAR, MONTH, TIME, HOUR — all functions to construct date times
TODAY and NOW handle entering times of right now from the system clock
WEEKDAY converts a date to DOW (1=Sunday)
WORKDAY and NETWORKDAYS tally the amount of work days in a date range
DATEDIF does a time delta
EDATE and EOMONTH are for end date and end of month dates in a range

REFERENCE FUNCTIONS
OFFSET func can grab data from remote cells, include a range of data.  Starts with an anchor point.  Common to use COUNTA func with it to set termination point, which can be dynamic as a column grows

INDIRECT func lets you append string literals as part of a command to grab data from other tabs in the workbook, so you can use the same formula on one central sheet and it will grab data from all relevant tabs.  Very handy when needed.  Be aware of spaces in Sheet Names!

INDIRECT func can also be used with Data Validation to make a multi-tiered pick list (complex, must delete blanks)

TEXT FUNCTIONS
FIND is case dependent
SEARCH is not
They both locate only the first occurrence of specified value/str
Use them in conjunction with MID to extract data dynamically from a cell/str.
=MID(C2, FIND(“:”, C2) + 2, 5)   —> will find the first semi-colon in a cell, then go two positions past that (+2), then extract the next 5 characters (say, a zip code)

MID allows you to start a search in the middle of a string

FLASH FILL (button) can try to fill down using predictive text extraction (can work well, can fail)

LEFT and RIGHT extract characters from a string in a cell starting from the left or right boundary of the cell.
Can combine with FIND to dynamically change starting point of extraction

TRIM func removes spaces, and can also trim inner spaces if used on a whole range as part of the argument for an outer TRIM (so, a nested TRIM)

CONCAT func does the obvious for strings…  (Can use an ampersand to concat w/o the function!)
TEXTJOIN works like .join() in Python and joins items in a range with specified separators

UPPER, LOWER, PROPER do expected capitalizations

REPLACE changes text based on position
SUBSTITUTE changes text based on content

LEN gets the count of characters / digits in a cell
REPT repeats a specified character(s) a given number of times.  Use with LEN to ‘pad’ numbers or codes and concat to existing entry to form final value
Ex: REPT(“0”, 6-LEN(A1))&A1  —> pads entry in A1 up to 6 digits with zeroes.

VALUE func might be required to convert a cell(s) into a value that can be formatted by Excel

LET func is very nice when using nested IF or IFS.  It lets you assign a value to a temp variable, say “x”, just like a lambda in Python.  Then you can use that variable in the rest of the formula in that cell (e.g. with nested IF).  Space saver and makes formula way more readable.

CELL and INFO return a lot of information about cells…

All the ISERROR, IFERROR, etc…
Remember AGGREGATE will overlook errors!
