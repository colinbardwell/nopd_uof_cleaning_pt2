Cleaning a snapshot of the NOPD Use of Force open dataset part 2: Manipulating data in Python with pandas

Data pulled on July 1st, 2024
Pre-cleaned and explored in Excel.

Data imported as a dataframe into Python for analysis after importing the following packages:

pandas (data analysis/manipulation)
numpy (array functions)
matplotlib.pyplot(visualization)
seaborn (streamlined viz)

PROFILING / INITIAL OBSERVATIONS

	- Dataframe shape matches Excel table with 3876 rows, 35 columns.
	- Data types are all object (aside from Officer/Subject counts, which are int64)

	- The following fields have the most null values by far:
		- Unit (772), Working Status (434), and Shift (603)
		- Subject Arrest Charges (3869)

	- The Subject Arrest Charges column is dropped since it contains next to no data.

	- Missing values within all fields are replaced with "Unknown"
		- 739 rows changed from "Unknown Working Status" to "Unknown" for consistency
	
	- Subject Count column is not reliable - different subject attribute fields contain different distributions of list lengths.


TRANSFORMATION

	- Strings are split to reformat the multi-value columns from "Value | Value" to ['Value', 'Value]
		- Number of Officers/Number of Subjects columns cast to string first to avoid errors
		- Duplicate single-value columns and original multi-value columns get dropped after appending new columns

	- Dataset shape is verified, a list of newly transformed columns is created

	- When trying to explode columns the following error occurs: "ValueError: columns must have matching element counts"
		- At least one row has mismatched value counts between columns (e.g., Officer Ethnicity: ['Black', 'Black'] and Officer Age: ['32'])

	- List lengths printed for each field to compare
		- Officer columns have the same distribution and can be exploded with no further reformatting
		- Subject columns have a few different distributions of list lengths and cannot be exploded without addressing inconsistencies

	- Dataframe is split into 3 different dataframes: officer_df, subject_df, and incident_df
		- Officer count and subject count not carried over due to inconsistencies in subject field

	- The new dataframe officer_df is exploded, and column names are reverted to original format

	- The following columns in subject_df have missing information for one or more subjects:
		- Subject Height, Subject Age, Subject Ethnicity, Distance Between, and Subject Influencing Factors

	- Padded list values in each row so that missing values (e.g., 4 subjects but only 3 ages listed) are now "Unknown"
		- Create function, apply across all rows, verify and fix PIB File Number column

	- Verified that all subject columns now contain the same distribution of list lengths by remaking value count tables
	
	- The now standardized subject_df exploded and column names reverted to original format

	- Newly exploded dataframes used to recalculate number of officers and subjects per incident
		- PIB File Number (incident ID) counted for each row in the newly exploded tables, where rows represent individual officers/subjects.
		- Number of Officers and Subjects added to incident table by joining on unique PIB File Number
	
	- Officer and Subject counts visualized via bar chart to compare to previously calculated numbers


EXPORTING TABLES/VALIDATION

	- CSVs loaded into Excel

	- Subjects/Officers Tables issues
		- Entries have excess spaces (e.g., "  No" instead of "No")
		- Used Excel's TRIM function to clean up data - seems to be unintended result of explode function

	- Compared dataframe shapes to imported tables to verify proper import

	- Pivot tables set up to represent newly exploded column value counts for cleaning and validation

	- Column by column review in Excel supported by Python scripts as needed


CLEANING OFFICERS TABLE

	- Use of Force Type

		- Some rows contain a variation on "- Hands (with injury) " that does not read properly in Excel. 
			- Python script used to identify values with 29 occurrences in this column and verify intended input
			- All 29 rows found/replaced with "Hands (with injury)"
		- https://nola.gov/getattachment/NOPD/NOPD-Consent-Decree/Chapter-1-3-Use-of-Force.pdf/
		- PowerBI visualization referenced to help inform format - columns copied over to compare
	
		- 48 rows changed from "- Hands (with injury)" to "Hands (with injury)
		- 1 row changed from "Rifle (Discharged)" to "Firearm (Discharged)"
		- 30 rows labeled "CEW", with no specification on whether deployed or exhibited
			- Every row has L4 as the Use of Force Level, suggesting "CEW Deployment" is the correct fix.
			- All 30 rows changed to "CEW Deployment"
		- 6 rows changed from "Baton/PR-24(NonStrk) to "Baton (non-strike)"
		- 3 rows changed from "Baton/PR-24 (Miss)" to "Baton (strike)"
		- 7 rows changed from "Baton/PR-24 (Strike)" to "Baton (strike)"
		- 2 rows changed from "Canine (Contact)" to "Canine (No Bite)"
		- 3 rows changed from "Shotgun (Pointed)" to "Firearm (Exhibited)"
		- 7 rows changed from "Vehicle as Weapon" to "Vehicle"
		- 11 rows changed from "VehPursuits w/Injury" to "Vehicle"

	- Officer Race/Ethnicity

		- 10 rows changed to "Unknown" from missing values
		- 86 rows changed from "Not Specified" to "Unknown"
		- 8 rows changed from "Not Applicable (Non-U.S.)" to "Unknown"

	- Officer Gender

		- 10 rows changed to "Unknown" from missing values

	- Officer Age

		- 13 rows changed to "Unknown" from missing values

	- Officer Years of Service

		- 26 rows changed to "Unknown" from missing values


CLEANING SUBJECTS TABLE

	- Subject Influencing Factors

		- 2 rows changed from no value to "Unknown"
		- 337 rows changed from "Alchohol" to "Alcohol"
		- 157 rows changed from "Alchohol and unknown drugs" to "Unknown Drugs"

	- Distance Between

		- 2 rows changed from no value to "Unknown"
		- 106 rows changed from "*Direct Contact" to "0 feet to 1 feet"

	- Subject Gender

		- 15 rows changed from no value to "Unknown"
		- 1 row changed from "m" to "Male"
		- 1 row changed from "femaale" to "Female"
		- 15 rows changed from "Sex-Unk" to "Unknown"

	- Subject Ethnicity

		- 33 rows changed from no value to "Unknown"
		- 22 rows changed from "Race-Unknown" to "Unknown"
		- 2 rows changed from "w" to "White"

	- Subject Age

		- 64 rows changed from no value to "Unknown"
		- 120 rows changed from "0" to "Unknown"
		- 1 row changed from "-1" to "Unknown"
		- 9 rows changed from "1" to "Unknown"
		- 1 row changed from "2" to "Unknown"
		- 1 row changed from "4" to "Unknown"
		- 1 row changed from "6" to "Unknown"
		- 4 rows changed from "8" to "Unknown"

	- Subject Build 
		
		-89 rows changed from "XLarge" to "Large"

	- Subject Injured/Hospitalized/Arrested

		- FTN2021-0378 missing values for every subject detail
			- Only instance of "Unknown" in these 3 columns


GPT PROMPTS

I have a dataframe titled subject_df with multiple fields that all contain information about at least one subject. Some rows only have one subject, but some instances represent multiple subjects at once. Each field value is a list representing the field's attribute for a particular subject. For example, field "Subject Age" may be [25, 26] in one row, while "Subject Ethnicity" is ['Black', 'White']. This represents one 25 year old 'Black' subject and one 26 year old 'White' subject. There are some instances where the list of values in one field does not match the length of the other fields, meaning there is missing data for one or more subjects in that instance. How do I go about inserting "Unknown" as the missing value? For example, if there is a row that represents 3 subjects in the "Subject Age" field, but there are 2 values for "Subject Age", I want the missing third value to be "Unknown".