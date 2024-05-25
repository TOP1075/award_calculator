# CS50 Final Project: Award Calculator (AC)

## Materials
The files and materials provided are:
- **The AC .py file.** This file was written to be used in a Google colab notebook, so should be uploaded to a Google Drive in order to be used
- **Two CSV files ("Example SMRs" and "Example SPRs").** These are example files which contain made up results and students (ten in total).
- And a [**Google Drive folder**](https://drive.google.com/drive/folders/1Gzugq_0Xzw1hcReTzJPXjhUeTfq-Atg5), here:  which contains Google sheets as follows:
    - **Internal checking output sheet**, which reports errors and provides a preview of calculation results
    - **Ratification report template and output folder**, which determines where and in what form generated reports are stored
    - **Programme routes data sheet**, which is read to provide the tool dynamically with programme and module information

For the video introduction to this programme, see: https://www.youtube.com/watch?v=ljo9wcrRCM0

## Running
To run the programme, navigate to the bottom of the notebook and execute the final code block. (By design, the code will be hidden; only the markdown and configuration widgets will be visible.)

The "Dependencies, definitions, and configuration" section above will now run. When run for the first time in a session, you will be asked to give access via your Google credentials.

You will then be asked to:
- upload your **SPRs** file
- upload your **SMRs** file
- choose your output type ("Internal checking" or "Board reporting")
- define the academic period, e.g. 23/24 P1 (this is only needed if your output type is "Board reporting")

Once you've completed to above, hit the run button to start the programme. Printing is used to report when the programme begins and completes.

> Note: the files provided via the shared Google Drive folder have already been configured to work with AC. There is no requirement to edit these. Outputs will be returned via the internal checking and ratification report folders, depending on the output type selected.

### Output Types

**Internal Checking** - selecting this option will return a list of errors and the calculated outcomes for the provided student records via the internal checking sheet. Existing data on the internal checking sheet will be wiped. *Use this option to check for errors and sense check calcuations - i.e. it is for internal use only.*

**Ratification Reports** - when selecting this option the programme will generate ratification reports (one per deparment), which will have a dynamic name depending on the academic period provided (or a placeholder if no period is given). These reports will be stored in the specified folder.
> If any errors are detected, the programme will default to "Internal checking", even if the user has specified "Board Report"; an alert notifies the user if this is the case.


## Documentation

> **The following is a substantial extract from the documentation I've produced as part of this project. It provides a summary of how the programme operates and how each function behaves.**


AC is divided into two sections: Section  1 - “Dependencies, definitions, and configuration”; and Section 2 - “AC - Run me!”.

Section 1 contains all class and function definitions as well as configuration rules and dependencies. It is divided into 4 code block:

- Dependencies
- Configuration
- Class definitions
- Function definitions

Section 2 is a basic trigger through which users execute the programme.

This summary is broadly in the order in which different sections are utilised. (This is not technically accurate, as all code blocks in Section 1 execute first automatically every time whenever running Section 2; however, the summary below is organised based on when functions are called.)

### Dependencies

A static list of libraries required for this programme. This block runs automatically. Two elements warrant specific mention.

#### gspread

The gspread library - used for GSheet manipulation - defaults to version 3.4.2 in a colaboratory environment. However, we need version 3.5 at a minimum, in order to make use of certain methods (e.g. ability to get spreadsheets and worksheets by id).

gspread versioning is enforced by `!pip install --upgrade gspread` which forces the library to uninstall and reinstalls the latest version (5.12.0 as of 24/11/23).

As reinstalling gspread is unnecessary where this has already been done within a given runtime, a try/except block is used to:

- check the gspread version
- if it the version is below 5.12.0 **or** gspread is undefined (returning an error), gspread is reinstalled to the most recent version

#### Google authentication

The gspread library requires authentication in order to access Google Drive. This is also handled in the Dependencies block, returning an authenticated gspread session in the variable `gc`. This variable is used later to make use of gspread functionality.

### User Trigger

The trigger uses the ipywidgets library (Jupyter widgets). It:

1. Defines three widgets (one dropdown, one free-text, one button)
2. Requests user inputs:
    - a file upload for a SPR .csv
    - another file upload for a SMR .csv
    - a choice of report types (an internal checking report or external Board report)
    - space to input the academic period, which is used when naming the Board report output spreadsheet
    - a click-able button

On selecting the “Run AC 😎” button, the programme will execute.

There is no mechanism to enforce the order of uploaded files (nor is it viable to add one as file names are at user discretion and file types the same). It is entirely possible to upload the SMR then the SPR, but doing so will cause an error when executing the programme.

### Configuration

Configured values are used throughout the programme. Where they are not directly defined by the user via the widgets described above, they are defined in the Configuration code block.

These configurable values are:

- `COV_COMPENSATION` and `NEW_COMPENSATION` (boolean values: true/false)

    These two variables define whether or not a third compensation rule is in use. This third rule is the requirement to have a best attempt average across compensatable modules of ≥50 in order to compensate.

    Two variables are used to differentiate between rulesets. **Note:** students eligible for the best of COVID and NEW, are governed for compensation purposes **by the NEW_COMPENSATION variable**.

    Where these variables are set to **True**, the third rule is applied.

    As of the current version, `COV_COMPENSATION = True` and `NEW_COMPENSATION = False`

    - `NEW_COMP_ATTR` is also defined here. It determines which attribute (i.e. which kind of mark) to use for NEW_COMPENSATION. Acceptable values are strictly: “best_attempt” or “module_mark”. The current default is “best_attempt”.

- `gex_SMR` (list [])

    This is a list containing string values for specific grades. This list of grades is used to determine where SMR records should **not** be included in award calculations. In practice, this means any SMR records with an agreed grade that matches one of the grades in the `gex_SMR` will be discarded.

    **However, these grades will not cause the exclusion of the SPR records** i.e. the student record will continue to be processed using all remaining SMRs.

- `gex_SPR` (regular expression)

    Using the re library, this variable contains a regular expression (regex). Regular expressions are used to match specific characters or strings.

    As of December 2023, the following regex is used: `gex_SPR = re.compile(r'^(P|F).?')`. Here re.compile calls the compile method from the re library in order to turn the provided string into a regex expression.

    - The expression `^(P|F).?` matches any P* and F* grades, including P and F themselves.

- Three file ids are specified here:
    - `prog_route_id` gives the file id for the programme route spreadsheet
    - `output_id` gives the file id for the static internal checking sheet
    - `board_template_id` gives the file id for the template used to generate new board report spreadsheets

- `covid_ols` and `old_ols` are lists of strings. Each string is in the format **YYYY/Y OLX** - i.e. a standard combination of academic year and OL. These lists contain all OLs for the COVID and pre-COVID periods respectively. They should never need editing, hence they are hardcoded.

- AIT checking is managed via three variables:
    - `AIT_CHECK` is a bool value that  governs whether to check AIT status or not; if `False` AIT is ignored
    - `ait_field` is the name of the AIT field on an SPR export (currently “User Defined Field 11”)
    - `ait_value` is the pass value to check (currently “PASS”)

### `run_ac`

This function acts as a master, calling defined functions and methods as required.

It interprets and stores user input for later use. (This includes entering a placeholder value for academic period if none was provided **and** report type is “Board report”.)

This function also reports to the user when the function begins and when it has completed.

### `retrieve_routes`

Before looking at student data, `retrieve_routes` is called. It populates and returns a dictionary{} (called `routes`).

It does this by:

- opening the programme routes spreadsheet by the id provided
- reading the “Routes” tab of this spreadsheet and initialising an instance of the `PROGRAMME_ROUTE` class for each entry on this tab. This class contains:
    - programme code
    - route code
    - department
    - short name
    - an empty dictionary called `route_modules`
    - a method `add_route_module`
- Each new `PROGRAMME_ROUTE` object is stored in the `routes` dictionary, with the key as the route code and the values as the object.
- For each entry in the `routes` dictionary, the relevant route tab is retrieved (using route codes, hence tab names must be route codes). For each module entry on the specific tab an instance of the `MODULE` class is initialised. This class contains:
    - module code
    - whether it is a cpm module (boolean value)
    - credit value
- Each `MODULE` object is added to the relevant `PROGRAMME_ROUTE` object via the `add_route_module` method.

The end result is a dictionary - `routes` - which contains a `PROGRAMME_ROUTE` object for each programme route, each of which contains `MODULE` objects for each module on the route.

### `store_SPRs`

Before calling the `store_SPRs` function, we need to access the uploaded data. The `files.upload()` function that is used to allow the user to unload the required CSV files returns a dictionary, which has the file name(s) as the key and the file content as the value. Therefore, we can decode and access this file as follows (**NB** the uploaded file is stored in the variable `fupload_SPRs`):

- `fname_SPRs = list(fupload_SPRs.keys())[0]`

    Retrieve the file name by listing the keys in the upload dictionary and taking the first one (as multiple file uploads are not permitted, the first key will always be our uploaded file)

- `fdata_SPRs = fupload_SPRs[fname_SPRs].decode('latin_1')`

    Call the decode method on the values uploaded, using the file name to identify the values required.

    Decode requires the user to provide the file encoding as an argument. Here we use ‘latin_1’, as - using the chardet library - I’ve identified the default SITS encoding to be iso-8859-1, an alias for latin_1.

- `fSPRs = io.StringIO(fdata_SPRs)`

    Using the io library’s StringIO class, this creates a file-like object from a string that can then be used as a file (i.e. read from, written to etc.). This takes place entirely within memory, without the need to use or store an external file. It also circumvents the need for the programme to access a real file via a path.

The end result of this process is a variable - `fSPRs` - which acts as a file containing our uploaded csv data.

The `store_SPRs` function then reads the uploaded SPR data. It uses a DictReader, which uses column headings - hence why header must be included in the exported file.

For each row of the SPR file, an instance of the `SPR_RECORD` class is initialised and stored in the `SPR_records` dictionary. The `SPR_RECORD` class contains a great deal of information; however, at this point, we are only populating:

- SPR code
- surname
- forename
- student status
- route code

This function thus returns a dictionary - `SPR_records` - which contains as keys SPR codes and as values the `SPR_RECORD` object associated with each code.

### `store_SMRs`

The uploaded SMR file is decoded and converted in exactly the same way as `store_SPRs`, putting it in a file-like format into the variable `fSMRs`.

The `store_SMRs` function then reads each line of the SMR file and initialises an instance of the SMR_RECORD class, storing in it:

- module code
- academic year code
- period slot code
- first attempt mark (four digit value) - **NB** this value (if greater than 0) is /100 and rounded to two decimal places
- agreed mark (four digit value) - **NB** this value (if greater than 0) is /100 and rounded to two decimal places
- agreed grade
- current attempt
- completed number
- process status

Each `SMR_RECORD` object is then stored in the `SMRs` dictionary contained within the relevant `SPR_RECORD` object (i.e. the one with the matching SPR code key). This is done with the `add_SMR_record` method.

NB - to avoid issues of duplication, the `SMRs` dictionary uses the module code, academic year code, and period slot code concatenated as a key.

Thus, the `SPR_records` dictionary now contains an `SPR_RECORD` object for each student, and - within each of these - a dictionary of SMR records, stored as `SMR_RECORD` objects.

### `check_ready`

The `check_ready` function creates a list (`errors`). It then calls the `ready_to_process` method on each entry in the `SPR_records` dictionary, before populating the `errors` list with the results.

All errors written to the `errors` list take the form of a tuple with three values: the student SPR code, the module/route code, and the error message.

The `ready_to_process` method checks the following for each SMR record in the `SMRs` dictionary in each `SPR_records` entry:

- grades that appear in the `gex_SMR` list are excluded, but the `SPR_record` is retained
    - where a module has a SD grade and no other non-SD SMRs for the same module code (i.e. the student only has one or more SDs for a given module), the error “All SMRs for this module have SD grades” is returned and the `SPR_record` is discarded
    - where a module has a WD and the best attempt mark is ≥39.5 (i.e. potentially credit bearing), the error “A WD grade is potentially credit-bearing for this module” is returned and the `SPR_record` discarded
    - all checks below this point ignore SMRs excluded by this initial check against the `gex_SMR` list
- SPRs that have any SMRs with a grade that does not match the regex expression in `gex_SPR` are discarded and the error message “Invalid grade {grade}” is returned. This error replaces “{grade}” with the specific grade in question; thus, where an SMR has a blank grade, the error message will read “Invalid grade “.
- SPRs with any duplicate SMRs - not counting those that have SD or WD grades, which have been excluded already - are excluded with the error message “Duplicate SMR”
- SPRs with any SMRs that do not belong to their route are excluded with the error message “Module not in route {route code}”
- SPRs with do not have SMRs for all modules on their route (i.e. those that have not complete their route) **and** which do not have the status WDN are excluded with the error message “Incomplete route, status {student status}”
- SPRs that have SMRs for all modules on their route (i.e. they have completed) and which are status WDN are excluded with error message “Student has completed, but status is WDN”.
- If `AIT_STATUS` is `True` then the attribute `ait_status` is checked against the provided `ait_value`. If it does **not** match the provided value and SPR status is not WDN, the SPR is excluded with message “Has not completed AIT. AIT status = {self.ait_status}; student status = {self.status}”

Once these checks are complete, the `check_ready` function calls `clear_errors` which compares the first value (i.e. the SPR code) in each entry of the `errors` list against the keys of the `SPR_records` dictionary and removes all matches from `SPR_records`.

The error list is then returned to the main function. At this point we now have a `SPR_records` dictionary containing only valid SPR records and SMR records, as well as a list of excluded SPRs and associated errors.

### Record processing

`SPR_RECORD` class methods and then called in sequence on each entry in the `SPR_records` dictionary. Each method, in the order called, is described below.

#### `record_department`

Helper function. Uses the stored student route to cross-reference with the `routes` dictionary and retrieve the department, which is stored in the `SPR_RECORD` object for later use.

#### `determine_ruleset`

Uses three boolean values - `covid_period`, `new_period`, and `old_period`, all of which start false.

Each SMR is examined as follows:

- The first four characters of the academic year are taken as `start_year`
- The academic year value and OL value are combined into an `ol_string` variable
- Then the following conditions are tested:
    - if the start year is ≥ 2023, `new_period` is set to true
    - else if the `ol_string` is found in the `covid_ols` list, `covid_period` is set to true
    - else if the `ol_string` is found in the `old_ols` list, `old_period` is set to true
    - and if none of these conditions are met, the `SPR_record` as a whole is discarded with the error message “Error in iteration {year} {ol}”

After these tests, the state of the three boolean values determines the ruleset. The following checks are performed **in sequence**.

- if `covid_period` and `new_period` are true, ruleset is set to “best”
- else if `covid_period` is true, ruleset is set to “covid”
- else if `new_period` or `old_period` are true, ruleset is set to”new”
- and if none of these conditions are met, the `SPR_record` is discarded with the error message “Cannot determine ruleset - check iterations”

The determined ruleset is stored in the `SPR_RECORD` object.

#### `calculate_module_mark`

This method calls the `SMR_RECORD` method `calculate_module_mark` on each SMR in the `SMRs` dictionary, which in turn determines module mark by checking in sequence:

- if the first attempt and best attempt marks are the same, the module mark is the first attempt
- else if the best attempt is greater than 50, the module mark is 50
- else the module mark is the best attempt mark

The calculated module mark is stored in the `SMR_RECORD` object.

#### `store_credit_value`

This method calls the `SMR_RECORD` method `store_credit_value` on each SMR in the `SMRs` dictionary. This simply writes the relevant credit value from the `routes` dictionary to each `SMR_RECORD` object

#### `store_cpm`

Similar to the above, this method calls the `SMR_RECORD` method `store_cpm` on each SMR in the SMRs dictionary, which checks against the `routes` dictionary and records for each `SMR_RECORD` if they are a CPM module.

#### `calculate_award`

This method works out the level of award achieved and the optimum combination of SMR records to use to achieve the requisite credit.

It first defines the appropriate mark to use based on ruleset. (For covid-only students, best attempt is used; for everyone else, module mark is used.)

Thereafter, each SMR is examined in order to count credits, with credit values split into passed-taught, passed-cpm, and marginal (i.e. potentially compensatable) credits.

The following calculations ********in sequence******** determine which award level has - potentially - been achieved:

- if a student has (1) passed ≥ 180 credits or (2) has passed ≥ 150 credits **and** has ≥ 180 credits in total, including marginal, they may be eligible for a MSCOL award
    - if required by compensation rules in force, their average taught mark is checked to see it is above 50; if not, the PGDIP eligibility is checked.
    - all `SMR_RECORD` objects are updated with credits passed (using the existing credit value attribute) and the award ‘MSCOL’ is stored in the `SPR_RECORD` object.

    Before moving on to PGDIP and PGCER checks, the list of SMRs we are using to calculate awards is adjusted to exclude CPM modules (as these are not usable for alternative qualifications).

- Based on the remaining SMRs, if a student has (1) passed ≥ 120 credits or (2) has passed ≥ 90 credits **and** has ≥ 120 credits total, they may be eligible for a PGDIP
    - at this point their SMR records are passed to the `award_PGDIP_PGCER` method, along with the parameters number of modules=8, max_marginal=2. See below for the details of this method.
    - assuming `award_PGDIP_PGCER` is able to find a valid combination of SMRs, the award ‘PGDIP’ is stored in the `SPR_RECORD` object.
    - if `award_PGDIP_PGCER` fails to identify a valid combination of SMRs, the student cannot receive a PGDIP and is considered for a PGCER instead.
- if a student has (1) passed ≥ 60 credits or (2) passed ≥ 45 credits **and** has ≥ 60 credits total, they may be eligible for a PGCER
    - at this point their SMR records are passed to the `award_PGDIP_PGCER` method, along with the parameters number of modules=4, max_marginal=1. See below for the details of this method.
    - assuming `award_PGDIP_PGCER` is able to find a valid combination of SMRs, the award ‘PGCER’ is stored in the `SPR_RECORD` object.
    - if `award_PGDIP_PGCER` fails to identify a valid combination of SMRs, the student cannot receive a PGCER and is reported as a fail error, as below.
- And if none of the criteria are met, the SPR is excluded and an error returned: “No award achievable?”. The error message also contains a list of different credit counts.

#### `award_PGDIP_PGCER`

This method is the most complex part of the programme.

Its purpose is to take a list of SMRs and return the very best combination of SMRs that meet the required criteria. The complexity here is that it needs to do this for all three possible rulesets, which means the criteria are dynamic.

It also is able to accommodate PGDIPs or PGCERs, as the method requires the arguments `num_mods` and `max_marg`, which specifies the number of modules required (8 for a PGDIP, 4 for a PGCER) and the maximum number of marginal marks permitted (2 for a PGDIP, 1 for a PGCER).

The method calls the `get_best` method once for the NEW ruleset and/or once for the COVID ruleset. The `get_best` method does the following:

- uses itertools combination method to generate all possible combinations of modules
- checks each combination using the `meets_criteria` method, storing those that return true. The `meets_criteria` method
    - calls `calc_averages` to return average values
    - checks the relevant average for compensation purposes, if required based on Configuration
    - counts the number of marginal best attempts
    - return true if the combination of SMRs meets one or both criteria, as required
- If no valid combinations of SMRs have been identified, none are returned/
- Otherwise, the combination with the highest average module mark (for the “new” ruleset) or highest average best attempt mark (for the “covid” ruleset) is returned.

The method then marks modules to make it clear which SMRs have been used to generate the optimum list(s). Those used for the best COVID list and the best NEW list are stored separately.

Where a student has ruleset “best”, two lists are generated: one based on COVID rules and one based on NEW rules. They are compared later to determine the optimum outcome.

#### `award_rank`

This method makes use of two additional methods - `award_rank_covid` and `award_rank_new` - which provide an award mark and award rank as per the specified ruleset. The behaviour of each of these methods is described below.

For students with the ruleset “new”, their award mark and rank is returned by the `award_rank_new` method. These values are stored in their `SPR_RECORD` as is the calculation method (”new”).

For students with the ruleset “covid”, their award mark and rank is returned by the `award_rank_covid` method. These values are stored in their `SPR_RECORD` as is the calculation method (”covid”).

For students with the ruleset “both”, both `award_rank_covid` and `award_rank_new` are called, with the results stored separately for comparison. This comparison takes place **in sequence** as follows:

- if the new rank and covid rank are the same, the award mark is set to the new award mark (as this is always the same or better than the covid award mark) and the method is set to “new”. The award rank is set to the new rank.
- else if the new rank is ‘Distinction’, new is used for rank, method, and mark
- else if the covid rank is ‘Distinction’, covid is use for rank, method, and mark
- else if the new rank is ‘Merit’, new is used for rank, method, and mark
- else covid is used for rank, method, and mark (and the rank will be ‘Merit’)

The final check this method performs is to see whether the stored award mark value is < 50. If it is, this value is overwritten with the lowest award mark consistent with a pass (50). If a student has their award mark uplifted to 50, this will be recorded in the am_uplift variable (”Yes”) and outputted in the results (Internal checking and Board report).

This concludes the calculation process and results are ready for export.

#### `award_rank_covid`

Uses first attempt marks.

Looking at SMRs that are in the optimum COVID-specific list only, this method calculates several credit weighted averages in order to ascertain award rank.

Award mark the a credit weighted average first attempt mark across all contributing modules, rounded to the nearest integer.

**NB** CPM mark is calculated as a credit weighted average mark across all modules marked as CPM. At present, this is always only the 30-credit CPM module.

For students with award ‘MSCOL’:

- if the award mark or taught average is ≥ 70

    **and** the CPM mark is ≥ 50

    **and** ≤ 15 credits with marks < 49.5

    **and** 0 credits with marks < 39.5

    Then the rank is ‘Distinction’

- if the award mark or taught average is ≥ 60

    **and** the CPM mark is ≥ 50

    **and** ≤ 30 credits with marks < 49.5

    **and** ≤ 15 credits with marks < 39.5

    Then the rank is ‘Merit’

- Otherwise, the rank is ‘Pass’

For students with award ‘PGDIP’:

- if the award mark is ≥ 70

    **and** ≤ 15 credits with marks < 49.5

    **and** 0 credits with marks < 39.5

    Then the rank is ‘Distinction’

- if the award mark is ≥ 60

    **and** ≤ 30 credits with marks < 49.5

    **and** ≤ 15 credits with marks < 39.5

    Then the rank is ‘Merit’

- Otherwise, the rank is ‘Pass’

For students with award ‘PGCER’, the rank is always ‘Pass’

#### `award_rank_new`

This method is simpler than `award_rank_covid` due to the greater simplicity of the award rules.

It uses module mark.

Award mark is calculated as a credit weight average of module mark across all contributing modules, rounded to the nearest integer.

******NB****** CPM mark is calculated as a credit weighted average module mark across all modules marked as CPM. At present, this is always only the 30-credit CPM module.

For students with award ‘MSCOL’:

- if the award mark is ≥ 70 and the CPM mark is ≥ 70, the rank is ‘Distinction’
- if the award mark is ≥ 60 and the CPM mark is ≥ 60, the rank is ‘Merit’
- else the rank is ‘Pass’

For students with award ‘PGDIP’:

- if the award mark is ≥ 70, the rank is ‘Distinction’
- if the award mark is ≥ 60, the rank is ‘Merit’
- else the rank is ‘Pass’

For students with award ‘PGCER’, the rank is always ‘Pass’

### Export results

Before exporting results, `clear_errors` is called to remove any `SPR_records` entries that were flagged as errors during the calculation process.

The function `output_results` is then called. This function organises SPR_records into a simple output format. The aim here is to minimise the number of write actions, as repeated write actions require API calls that create delay.

Thus a dictionary - `output_records` - is created which has departments as keys and stored `SPR_RECORD` objects as values. Recall each `SPR_RECORD` object contains all of the calculated outcomes.

`output_results` then determines the output type by checking **in sequence**:

- if the user has selected "Internal checking”, it calls the `int_output` function
- if the `errors` list is not empty, it calls the `int_output` function (a pop-up will let the user know)
- if the user has selected “Board reporting”, it calls the `ext_output` function

#### `int_output`

Opens the output sheet using the sheet id provided in the Configuration code block.

Takes a list of existing tabs in the sheet and creates new tabs - where required - for each department (i.e. for each key in the `output_results` dictionary).

And for each department it:

- takes the headers from the existing sheet
- to these headers it appends a list of data taken from the `SPR_RECORD` objects
    - the values specified at this point determine which values are append to the sheet
- clears the sheet of existing data
- and then append the new list of data

Then the “Errors” tab is located (or created). It is cleared and the errors reported are written to the sheet.

#### `ext_output`

The function retrieves the department tab of the programme routes spreadsheet. It reads the relevant departmental folder ids stored in that sheet to work out where to create new department-specific Board report spreadsheets.

For each department the function:

- creates a new spreadsheet in the given folder with the name “{Department} Board Report: {academic period}” - {academic period} is the string supplied by the user.
- splits the SPR_RECORD objects for the department into their specific routes and sorts the data by award, award rank, and then award mark
- creates a tab per route, using the route shortname, from the existing template

    Then for each tab, the function:

    - gathers and inserts a list of the modules on a given route into each route tab
    - creates a list of data to write to the sheet, matching `SPR_RECORD` SMRs to the relevant sheet columns
    - provides a list of contributing modules (for PGDIP and PGCER students only)
    - and appends the list of student data to the new tab

**At this point the programme is complete.** The user is informed that AC is complete.

No stored data is deleted. All colaboratory runtimes are deleted after a certain period of inactive or once closed. Therefore, the programme cannot retain data over the medium term.

---

### Addendum - rounding

#### Rounding behaviour

- All rounding rounds up from .5 (so 1.5 = 2, 2.5 = 3)
- Module marks are rounded to two decimal places.
- All averages - **including award mark** - are rounded to whole integers.
    - Therefore, all average checks also use whole integers (so ≥ 50, not 49.5)

#### Rounding method

Rounding cannot be achieved through the standard Python round() method. This standard method uses “bankers rounding” , which rounds to the nearest even value. So 1.5 rounds to 2, but 2.5 also rounds to 2.

Therefore, rounding is achieved via the decimal library. The `round_float` function is called wherever rounding is required. It takes as arguments the float value to round and a string containing the round pattern (e.g. ‘.01’, ‘1.’).

The `round_float` function:

- converts the float to a decimal
- rounds using the `quantize` method with the keyword argument `round=ROUND_HALF_UP`
- returns the rounded value as a float
