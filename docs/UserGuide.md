---
layout: default.md
title: "User Guide"
pageNav: 3
---

# BloodNet User Guide

BloodNet is a **desktop app for tracking blood donors and their donations, optimized for use via a Command Line
Interface** (CLI) while still
having the benefits of a Graphical User Interface (GUI). If you can type fast, BloodNet can get your blood donation
management
tasks done faster than traditional GUI apps! BloodNet also does not require internet connectivity, making it perfect
for locations where internet service is unreliable!

--------------------------------------------------------------------------------------------------------------------

## Quick start

1. Ensure you have Java `17` or above installed in your Computer.<br>
   **Mac users:** Ensure you have the precise JDK version
   prescribed [here](https://se-education.org/guides/tutorials/javaInstallationMac.html).

2. Download the latest `.jar` file from [here](https://github.com/AY2526S1-CS2103T-T16-4/tp/releases).

3. Copy the file to the folder you want to use as the _home folder_ for your BloodNet application.

4. Open a command terminal, `cd` into the folder you put the jar file in, and use the `java -jar bloodnet.jar` command
   to run the application.<br>
   A GUI similar to the below should appear in a few seconds. Note how the app contains some sample data.<br>
   ![Ui](images/Ui.png)

5. Type the command in the command box and press Enter to execute it. e.g. typing **`help`** and pressing Enter will
   open the help window.<br>
   Some example commands you can try:

    * `list` : Lists all donors.

    * `add n/John Doe p/98765432 e/johnd@example.com b/A+ d/21-06-2003` : Adds a donor named `John Doe` to BloodNet.

    * `delete 3` : Deletes the 3rd donor shown in the current donor list.

    * `adddonation p/1 d/21-10-2025 v/500` : Adds a donation record for the 1st donor shown in the current donor
      list.

    * `findeligible O+ A+` : Finds all donors with the specified blood type(s) provided who are currently eligible to 
       donate based on their date of birth and number of days since last donation.

    * `exit` : Exits the app.

6. Refer to the [Features](#features) below for details of each command.

--------------------------------------------------------------------------------------------------------------------

## Command summary

 Action              | Format, Examples                                                                                                                                
-------------------------- |-------------------------------------------------------------------------------------------------------------------------------------------------
[**Add Donor**](#adding-a-donor-add)| `add n/NAME p/PHONE_NUMBER e/EMAIL b/BLOOD_TYPE d/DATE_OF_BIRTH` <br> e.g., `add n/James Ho p/22224444 e/jamesho@example.com b/A+ d/22-11-2004` 
[**List All Donors**](#listing-all-donors--list) | `list`                                                                                                                                          
[**Find Donor**](#locating-donors-by-name-find-)| `find KEYWORD [MORE_KEYWORDS]`<br> e.g., `find James Jake`     
[**Find Eligible Donors**](#finding-eligible-donors-based-on-blood-type-findeligible) | `findeligible BLOOD_TYPE(S)`<br> e.g., `findeligible A+ O+ B+`
[**Edit Donor**](#editing-a-donor--edit) | `edit DONOR_LIST_INDEX [n/NAME] [p/PHONE_NUMBER] [e/EMAIL] [b/BLOOD_TYPE] [d/DATE_OF_BIRTH]`<br> e.g.,`edit 2 n/James Lee e/jameslee@example.com`          
[**Delete Donor**](#deleting-a-donor--delete)  | `delete DONOR_LIST_INDEX `<br> e.g., `delete 3`    
[**Edit Donation Record**](#editing-a-donation-record--editdonation) | `editdonation DONATION_RECORD_LIST_INDEX `<br> e.g., `editdonation 1 v/350 d/20-02-2025 `
[**Delete All Data**](#clearing-all-entries--clear)  | `clear`
[**Help**](#viewing-help--help) | `help`                                                                                                                                          

--------------------------------------------------------------------------------------------------------------------

## Features

<box type="info" seamless>

**Notes about the command format:**<br>

* Words in `UPPER_CASE` are the parameters to be supplied by the user.<br>
  e.g. in `add n/NAME`, `NAME` is a parameter which can be used as `add n/John Doe`.

* Parameters can be in any order.<br>
  e.g. if the command specifies `n/NAME b/BLOOD_TYPE`, `b/BLOOD_TYPE n/NAME` is also acceptable.

* Extraneous parameters for commands that do not take in parameters (such as `help`, `list`, `exit` and `clear`) will be
  ignored.<br>
  e.g. if the command specifies `help 123`, it will be interpreted as `help`.

* If you are using a PDF version of this document, be careful when copying and pasting commands that span multiple lines
  as space characters surrounding line-breaks may be omitted when copied over to the application.
  </box>

### Adding a donor: `add`

Whenever someone signs up to be a blood donor, use this command to add them to the BloodNet system!

Format: `add n/NAME p/PHONE_NUMBER e/EMAIL b/BLOOD_TYPE d/DATE_OF_BIRTH`

* BLOOD_TYPE must be either O+, O-, A+, A-, B+, B-, AB+ or AB-
* DATE_OF_BIRTH must be in the *dd-MM-yyyy* format

Examples:

* `add n/John Doe p/98765432 e/johnd@example.com b/B+ d/04-11-1999`
* `add n/Betsy Crowe e/betsycrowe@example.com b/O- d/20-05-2004`

### Listing all donors : `list`

Shows a list of all donors in the BloodNet system.

Format: `list`

### Locating donors by name: `find `

Finds donors whose names contain any of the given keywords.

Format: `find KEYWORD [MORE_KEYWORDS]`

* The search is case-insensitive. e.g. `hans` will match `Hans`
* The order of the keywords does not matter. e.g. `Hans Bo` will match `Bo Hans`
* Only the name is searched.
* Persons matching at least one keyword will be returned (i.e. `OR` search).
  e.g. `Hans Bo` will return `Hans Gruber`, `Bo Yang`

Examples:

* `find John` returns `john` and `John Doe`
* `find alex david` returns `Alex Yeoh`, `David Li`<br>
  ![result for 'find alex david'](images/findAlexDavidResult.png)

### Find eligible donors based on blood type: `findeligible`

Finds all people who are eligible to donate blood for the specified blood type(s).

Format: `findeligible BLOOD_TYPE(S)`

* The search is case-insensitive. e.g. `O+` and `o+` will match the blood type of someone with blood type O+.
* Eligibility criteria are based on guidelines from the Health Sciences Authority. The donor’s date of birth and 
  how long it has been since their last blood donation are both considered when determining eligibility.

Examples:
To be added
<!-- Examples will be added soon -->

### Editing a donor : `edit`

Let's say you made a mistake and keyed in the wrong information when adding a donor. No worries! Use this command to
edit an existing donor in the BloodNet system.

Format: `edit DONOR_LIST_INDEX [n/NAME] [p/PHONE] [e/EMAIL] [b/BLOOD_TYPE] [d/DATE_OF_BIRTH]`

* Edits the person at the specified `DONOR_LIST_INDEX`. The index refers to the index number shown in the 
  displayed donor list. The specified index **must be a positive integer** 1, 2, 3, …​
* At least one of the optional fields must be provided.
* Existing values will be updated to the input values.

Examples:

* `edit 1 p/91234567 e/johndoe@example.com` Edits the phone number and email address of the 1st person to be `91234567`
  and `johndoe@example.com` respectively.
* `edit 2 n/Betsy Crower ` Edits the name of the 2nd person to be `Betsy Crower`.
* `find Betsy` followed by `edit 1 p/91234567` edits the phone number of the 1st donor in the result of the `find`
  command.

### Deleting a donor : `delete`

Let's say you added someone to BloodNet on accident. That's alright! This command can be used to delete a specified
donor from the BloodNet system.

Format: `delete DONOR_LIST_INDEX`

* Deletes the donor at the specified `DONOR_LIST_INDEX`.
* The index refers to the index number shown in the displayed donor list.
* The index **must be a positive integer** 1, 2, 3, …​

Examples:

* `list` followed by `delete 2` deletes the 2nd donor in the BloodNet system.
* `find Betsy` followed by `delete 1` deletes the 1st donor in the result of the `find` command.

### Editing a donation record : `editdonation`

If you made a mistake when adding the donation record particulars for a person, do not fret! Use this command to
edit an existing donor record in the BloodNet system.

Format: `editdonation DONATION_RECORD_LIST_INDEX [v/BLOOD_VOLUME] [d/DONATION_DATE]`

* Edits the person at the specified `DONATION_RECORD_LIST_INDEX`. The index refers to the index number shown in the
  displayed donation record list. The specified index **must be a positive integer** 1, 2, 3, …​
* At least one of the optional fields must be provided.
* Existing values will be updated to the input values.

Examples:

* `editdonation 1 v/200 ` Edits the blood volume of the 1st donation record list
* `editdonation 3 d/13-10-2024 ` Edits the donation date of the 3rd donation record list.
<!-- More examples will be added soon -->

### Clearing all entries : `clear`

Clears all entries from the BloodNet system.

Format: `clear`

<box type="warning" seamless>

**Caution:**
This operation is irreversible! Hence, it is recommended to take a backup of the data file before running this command.
</box>

### Exiting the program : `exit`

Exits the program.

Format: `exit`

### Viewing help : `help`

If you need a refresher on the formats of the various commands, use this command! It will provide a summary of the
formats of each command, as well as the link to this user guide.

Format: `help`

### Saving the data

BloodNet data are saved in the hard disk automatically after any command that changes the data. There is no need to save
manually.

### Editing the data file

BloodNet data are saved automatically as a JSON file `[JAR file location]/data/bloodnet.json`. Advanced users are
welcome to update data directly by editing that data file.

<box type="warning" seamless>

**Caution:**
If your changes to the data file makes its format invalid, BloodNet will discard all data and start with an empty data
file at the next run. Hence, it is recommended to take a backup of the file before editing it.<br>
Furthermore, certain edits can cause the BloodNet to behave in unexpected ways (e.g., if a value entered is outside the
acceptable range). Therefore, edit the data file only if you are confident that you can update it correctly.
</box>

### Archiving data files `[coming in v2.0]`

_Details coming soon ..._

--------------------------------------------------------------------------------------------------------------------

## FAQ

**Q**: How do I transfer my data to another Computer?<br>
**A**: Install the app in the other computer and overwrite the empty data file it creates with the file that contains
the data of your previous BloodNet home folder.

--------------------------------------------------------------------------------------------------------------------

## Known issues

1. **When using multiple screens**, if you move the application to a secondary screen, and later switch to using only
   the primary screen, the GUI will open off-screen. The remedy is to delete the `preferences.json` file created by the
   application before running the application again.
2. **If you minimize the Help Window** and then run the `help` command (or use the `Help` menu, or the keyboard
   shortcut `F1`) again, the original Help Window will remain minimized, and no new Help Window will appear. The remedy
   is to manually restore the minimized Help Window.
