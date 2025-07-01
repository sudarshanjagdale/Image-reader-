Subject: Steps Followed for Time Category Analysis – June Data

Hi [Manager's Name],

As discussed, here’s the approach I followed for preparing the time category data for June:

1. Download data from production – approx. 10–15 minutes (depending on volume).


2. Remove unwanted columns and keep only required fields.


3. Add calculated columns:

Time Taken (in minutes) using:
=(C2-B2)*1440

Time Category using:

=IF(D2<=15,"Within 15 min",IF(D2<=30,"Between 15-30 min",IF(D2<=60,"Between 30-60 min","More than 60 min")))



4. Insert Pivot Table:

Rows: Time Category

Values: Count of Submission IDs



5. Add percentage using:
=Count / Grand Total * 100



Since formulas and structure are already ready, it takes only about 5 minutes after downloading.
Overall estimated time: 15–20 minutes.

Let me know if you’d like the file or if anything else is needed.

Thanks,
Sudarshan