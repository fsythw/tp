---
layout: default.md
title: "Developer Guide"
pageNav: 3
---
# BloodNet Developer Guide

--------------------------------------------------------------------------------------------------------------------

## **Acknowledgements**

- Blood donation eligibility criteria in Singapore were based on guidelines from the Health Sciences Authority
<!-- Can add more, if used -->
--------------------------------------------------------------------------------------------------------------------

## **Setting up, getting started**

Refer to the guide [_Setting up and getting started_](SettingUp.md).

--------------------------------------------------------------------------------------------------------------------

## **Design**

### Architecture

<puml src="diagrams/ArchitectureDiagram.puml" width="280" />

The ***Architecture Diagram*** shown above explains the high-level design of the App.

Given below is a quick overview of main components and how they interact with each other.

**Main components of the architecture**

**`Main`** (consisting of classes [`Main`](https://github.com/AY2526S1-CS2103T-T16-4/tp/tree/master/src/main/java/bloodnet) and [`MainApp`](https://github.com/AY2526S1-CS2103T-T16-4/tp/blob/master/src/main/java/bloodnet/MainApp.java)) is in charge of the app launch and shut down.
* At app launch, it initializes the other components in the correct sequence, and connects them up with each other.
* At shut down, it shuts down the other components and invokes cleanup methods where necessary.

The bulk of the app's work is done by the following four components:

* [**`UI`**](#ui-component): The UI of the App.
* [**`Logic`**](#logic-component): The command executor.
* [**`Model`**](#model-component): Holds the data of the App in memory.
* [**`Storage`**](#storage-component): Reads data from, and writes data to, the hard disk.

[**`Commons`**](#common-classes) represents a collection of classes used by multiple other components.

**How the architecture components interact with each other**

The *Sequence Diagram* below shows how the components interact with each other for the scenario where the user issues the command `delete 1`.

<puml src="diagrams/ArchitectureSequenceDiagram.puml" width="574" />

Each of the four main components (also shown in the above diagram),

* defines its *API* in an `interface` with the same name as the Component.
* implements its functionality using a concrete `{Component Name}Manager` class (which follows the corresponding API `interface` mentioned in the previous point.

For example, the `Logic` component defines its API in the `Logic.java` interface and implements its functionality using the `LogicManager.java` class which follows the `Logic` interface. Other components interact with a given component through its interface rather than the concrete class (reason: to prevent outside component's being coupled to the implementation of a component), as illustrated in the (partial) class diagram below.

<puml src="diagrams/ComponentManagers.puml" width="300" />

The sections below give more details of each component.

### UI component

The **API** of this component is specified in [`Ui.java`](https://github.com/AY2526S1-CS2103T-T16-4/tp/blob/master/src/main/java/bloodnet/ui/Ui.java)

<puml src="diagrams/UiClassDiagram.puml" alt="Structure of the UI Component"/>

The UI consists of a `MainWindow` that is made up of parts e.g.`CommandBox`, `ResultDisplay`, `PersonListPanel`, `StatusBarFooter` etc. All these, including the `MainWindow`, inherit from the abstract `UiPart` class which captures the commonalities between classes that represent parts of the visible GUI.

The `UI` component uses the JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files that are in the `src/main/resources/view` folder. For example, the layout of the [`MainWindow`](https://github.com/AY2526S1-CS2103T-T16-4/tp/blob/master/src/main/java/bloodnet/ui/MainWindow.java) is specified in [`MainWindow.fxml`](https://github.com/AY2526S1-CS2103T-T16-4/tp/blob/master/src/main/resources/view/MainWindow.fxml)

The `UI` component,

* executes user commands using the `Logic` component.
* listens for changes to `Model` data so that the UI can be updated with the modified data.
* keeps a reference to the `Logic` component, because the `UI` relies on the `Logic` to execute commands.
* depends on some classes in the `Model` component, as it displays `Person` object residing in the `Model`.

### Logic component

**API** : [`Logic.java`](https://github.com/AY2526S1-CS2103T-T16-4/tp/blob/master/src/main/java/bloodnet/logic/Logic.java)

Here's a (partial) class diagram of the `Logic` component:

<puml src="diagrams/LogicClassDiagram.puml" width="550"/>

The sequence diagram below illustrates the interactions within the `Logic` component, taking `execute("delete 1")` API call as an example.

<puml src="diagrams/DeleteSequenceDiagram.puml" alt="Interactions Inside the Logic Component for the `delete 1` Command" />

<box type="info" seamless>

**Note:** The lifeline for `DeleteCommandParser` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline continues till the end of diagram.
</box>

How the `Logic` component works:

1. When `Logic` is called upon to execute a command, it is passed to an `BloodNetParser` object which in turn creates a parser that matches the command (e.g., `DeleteCommandParser`) and uses it to parse the command.
2. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `DeleteCommand`) which is executed by the `LogicManager`.
3. The command can communicate with the `Model` when it is executed (e.g. to delete a person).<br>
   Note that although this is shown as a single step in the diagram above (for simplicity), in the code it can take several interactions (between the command object and the `Model`) to achieve.
4. The result of the command execution is encapsulated as a `CommandResult` object which is returned back from `Logic`.

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<puml src="diagrams/ParserClasses.puml" width="600"/>

How the parsing works:
* When called upon to parse a user command, the `BloodNetParser` class creates an `XYZCommandParser` (`XYZ` is a placeholder for the specific command name e.g., `AddCommandParser`) which uses the other classes shown above to parse the user command and create a `XYZCommand` object (e.g., `AddCommand`) which the `BloodNetParser` returns back as a `Command` object.
* All `XYZCommandParser` classes (e.g., `AddCommandParser`, `DeleteCommandParser`, ...) inherit from the `Parser` interface so that they can be treated similarly where possible e.g, during testing.

### Model component
**API** : [`Model.java`](https://github.com/AY2526S1-CS2103T-T16-4/tp/blob/master/src/main/java/bloodnet/model/Model.java)

<puml src="diagrams/ModelClassDiagram.puml" width="450" />


The `Model` component,

* stores the BloodNet data i.e., all `Person` objects (which are contained in a `UniquePersonList` object).
* stores the currently 'selected' `Person` objects (e.g., results of a search query) as a separate _filtered_ list which is exposed to outsiders as an unmodifiable `ObservableList<Person>` that can be 'observed' e.g. the UI can be bound to this list so that the UI automatically updates when the data in the list change.
* stores a `UserPref` object that represents the user’s preferences. This is exposed to the outside as a `ReadOnlyUserPref` objects.
* does not depend on any of the other three components (as the `Model` represents data entities of the domain, they should make sense on their own without depending on other components)


### Storage component

**API** : [`Storage.java`](https://github.com/se-edu/BloodNet-level3/tree/master/src/main/java/bloodnet/storage/Storage.java)

<puml src="diagrams/StorageClassDiagram.puml" width="550" />

The `Storage` component,
* can save both bloodnet data and user preference data in JSON format, and read them back into corresponding objects.
* inherits from both `BloodNetStorage` and `UserPrefStorage`, which means it can be treated as either one (if only the functionality of only one is needed).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects that belong to the `Model`)

### Common classes

Classes used by multiple components are in the `bloodnet.commons` package.

--------------------------------------------------------------------------------------------------------------------

## **Implementation**

This section describes some noteworthy details on how certain features are implemented.

### \[Proposed\] Undo/redo feature

#### Proposed Implementation

The proposed undo/redo mechanism is facilitated by `VersionedBloodNet`. It extends `BloodNet` with an undo/redo history, stored internally as an `BloodNetStateList` and `currentStatePointer`. Additionally, it implements the following operations:

* `VersionedBloodNet#commit()` — Saves the current bloodnet state in its history.
* `VersionedBloodNet#undo()` — Restores the previous bloodnet state from its history.
* `VersionedBloodNet#redo()` — Restores a previously undone bloodnet state from its history.

These operations are exposed in the `Model` interface as `Model#commitBloodNet()`, `Model#undoBloodNet()` and `Model#redoBloodNet()` respectively.

Given below is an example usage scenario and how the undo/redo mechanism behaves at each step.

Step 1. The user launches the application for the first time. The `VersionedBloodNet` will be initialized with the initial BloodNet state, and the `currentStatePointer` pointing to that single BloodNet state.

<puml src="diagrams/UndoRedoState0.puml" alt="UndoRedoState0" />

Step 2. The user executes `delete 5` command to delete the 5th person in BloodNet. The `delete` command calls `Model#commitBloodNet()`, causing the modified state of BloodNet after the `delete 5` command executes to be saved in the `BloodNetStateList`, and the `currentStatePointer` is shifted to the newly inserted BloodNet state.

<puml src="diagrams/UndoRedoState1.puml" alt="UndoRedoState1" />

Step 3. The user executes `add n/David …​` to add a new person. The `add` command also calls `Model#commitBloodNet()`, causing another modified BloodNet state to be saved into the `BloodNetStateList`.

<puml src="diagrams/UndoRedoState2.puml" alt="UndoRedoState2" />

<box type="info" seamless>

**Note:** If a command fails its execution, it will not call `Model#commitBloodNet()`, so the BloodNet state will not be saved into the `BloodNetStateList`.

</box>

Step 4. The user now decides that adding the person was a mistake, and decides to undo that action by executing the `undo` command. The `undo` command will call `Model#undoBloodNet()`, which will shift the `currentStatePointer` once to the left, pointing it to the previous BloodNet state, and restores the BloodNet to that state.

<puml src="diagrams/UndoRedoState3.puml" alt="UndoRedoState3" />


<box type="info" seamless>

**Note:** If the `currentStatePointer` is at index 0, pointing to the initial BloodNet state, then there are no previous BloodNet states to restore. The `undo` command uses `Model#canUndoBloodNet()` to check if this is the case. If so, it will return an error to the user rather
than attempting to perform the undo.

</box>

The following sequence diagram shows how an undo operation goes through the `Logic` component:

<puml src="diagrams/UndoSequenceDiagram-Logic.puml" alt="UndoSequenceDiagram-Logic" />

<box type="info" seamless>

**Note:** The lifeline for `UndoCommand` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.

</box>

Similarly, how an undo operation goes through the `Model` component is shown below:

<puml src="diagrams/UndoSequenceDiagram-Model.puml" alt="UndoSequenceDiagram-Model" />

The `redo` command does the opposite — it calls `Model#redoBloodNet()`, which shifts the `currentStatePointer` once to the right, pointing to the previously undone state, and restores the bloodnet to that state.

<box type="info" seamless>

**Note:** If the `currentStatePointer` is at index `BloodNetStateList.size() - 1`, pointing to the latest bloodnet state, then there are no undone BloodNet states to restore. The `redo` command uses `Model#canRedoBloodNet()` to check if this is the case. If so, it will return an error to the user rather than attempting to perform the redo.

</box>

Step 5. The user then decides to execute the command `list`. Commands that do not modify BloodNet, such as `list`, will usually not call `Model#commitBloodNet()`, `Model#undoBloodNet()` or `Model#redoBloodNet()`. Thus, the `BloodNetStateList` remains unchanged.

<puml src="diagrams/UndoRedoState4.puml" alt="UndoRedoState4" />

Step 6. The user executes `clear`, which calls `Model#commitBloodNet()`. Since the `currentStatePointer` is not pointing at the end of the `BloodNetStateList`, all BloodNet states after the `currentStatePointer` will be purged. Reason: It no longer makes sense to redo the `add n/David …​` command. This is the behavior that most modern desktop applications follow.

<puml src="diagrams/UndoRedoState5.puml" alt="UndoRedoState5" />

The following activity diagram summarizes what happens when a user executes a new command:

<puml src="diagrams/CommitActivityDiagram.puml" width="250" />

#### Design considerations:

**Aspect: How undo & redo executes:**

* **Alternative 1 (current choice):** Saves the entire BloodNet.
    * Pros: Easy to implement.
    * Cons: May have performance issues in terms of memory usage.

* **Alternative 2:** Individual command knows how to undo/redo by
  itself.
    * Pros: Will use less memory (e.g. for `delete`, just save the person being deleted).
    * Cons: We must ensure that the implementation of each individual command are correct.

_{more aspects and alternatives to be added}_

--------------------------------------------------------------------------------------------------------------------

## **Documentation, logging, testing, configuration, dev-ops**

* [Documentation guide](Documentation.md)
* [Testing guide](Testing.md)
* [Logging guide](Logging.md)
* [Configuration guide](Configuration.md)
* [DevOps guide](DevOps.md)

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Requirements**

### Product scope

**Target user profile**:

* has a need to manage a database of blood donors with detailed profile information
* requires quick access to donor personal information
* prefer desktop apps over other types of apps
* is able to type fast
* prefers typing to mouse interactions
* is reasonably comfortable using CLI apps
* wants to filter profiles such as by blood type

**Value proposition**: manage blood donor profiles more efficiently as opposed to a typical mouse driven app

### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority | As a …      | I want to …                                                                           | So that …                                                                                                                            |
| -------- | ----------- |---------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------|
| * * *    | admin staff | add a donor’s contact & blood type                                                    | the blood bank can keep in touch with the donor if more information is needed                                                        |
| * * *    | admin staff | add a donor’s date of birth                                                           | the blood bank knows can determine a person's eligibility in donating blood                                                          |
| * * *    | admin staff | search donors by name                                                                 | I can find their contact information if I need to contact them                                                                       |
| * * *    | admin staff | modify a donor’s contact information                                                  | I can fix the stored contact information if it was keyed in wrongly previously                                                       |
| * * *    | admin staff | modify a donor’s date of birth                                                        | I can fix the stored date of birth if it was keyed in wrongly                                                                        |
| * * *    | admin staff | modify a donor’s blood type                                                           | I can fix the stored blood type if it was keyed in wrongly previously                                                                |
| * * *    | admin staff | delete a donor (soft-delete / archive)                                                | Remove donors who have passed away or are no longer eligible for donation                                                            |
| * * *    | admin staff | list all donors in the system                                                         | Have a quick overview of all the people who have agreed to donate blood to us                                                        |
| * * *    | admin staff | find all donors of a particular blood type                                            | If we have a shortage of a particular blood type, we can contact these people and ask them for donations                             |
| * * *    | admin staff | record a blood donation by a contact                                                  | I can track how many donations each contact has made, and the details of those donations                                             |
| * * *    | admin staff | modify a blood donation record                                                        | I can modify wrongly keyed in records                                                                                                |
| * * *    | admin staff | add the volume and donation date associated with a donation record                    | the blood bank is aware of the details associated with each donation record                                                          
| * * *    | admin staff | delete a blood donation record                                                        | I can remove wrongly keyed in records                                                                                                |
| * * *    | admin staff | find all eligible donors given a blood type (based on age and last donation interval) | I can determine who I can call if blood is needed                                                                                    |
| * *      | admin staff | find a donor based on contact information                                             | I can link their name and contact information together                                                                               |
| * *      | admin staff | detect duplicate donors                                                               | I can quickly identify duplicate data in the system and reconcile it to reduce data pollution                                        |
| * *      | admin staff | detect duplicate donation records associated with the same person                     | I am able to quickly identify duplicate data in the BloodNet system and reconcile it to reduce data pollution                        |
| *        | admin staff | record how much blood was donated by a donor in a session                             | I can recommend donors who have been very active for appreciation awards, to incentivise more donors                                 |
| *        | admin staff | record when a donor donated blood in a session                                        | I can maintain accurate records of each donor’s donation history |


### Use cases

(For all use cases below, the **System** is `BloodNet`, unless specified otherwise)

---

### **Use case: UC01 - Add a donor’s contact & blood type**

**Actor**: Admin staff

**MSS**

1. Admin staff requests to add a donor and provides the donor's information.
2. BloodNet creates a new donor entry with the provided details.

Use case ends.

**Extensions**

* 1a. One or more of the provided information is invalid
    * 1a1. BloodNet shows an error message.
    * Use case returns to step 1.

---

### **Use case: UC02 - Update a donor's information**

**Actor**: Admin staff

**MSS**

1. Admin staff lists all donors ([UC05](#use-case-uc05---list-all-donors-in-the-system
   )).
2. Admin staff requests to update a specified donor, and provides the fields to update and values to update them with.
3. BloodNet updates fields of the donor for which values were supplied.

Use case ends.

**Extensions**
* 1a. The list is empty.
    * Use case ends.

* 2a. Donor ID not found.
    * 3a1. BloodNet shows an error message.
    * Use case returns to step 3.

* 2b. One or more invalid values provided.
    * 3b1. BloodNet shows an error message.
    * Use case returns to step 3.

---

### **Use case: UC03 - Search donors by name**

**Actor**: Admin staff

**MSS**

1. Admin staff requests to find a donor by their name.
2. BloodNet searches and displays donors with matching names.

Use case ends.

---

### **Use case: UC04 - Delete a donor**

**Actor**: Admin staff

**MSS**

1. Admin staff lists all donors ([UC05](#use-case-uc05---list-all-donors-in-the-system)).
2. Admin staff requests to delete a donor.
3. BloodNet deletes a donor.

Use case ends.

**Extensions**

* 1a. The list is empty.
    * Use case ends.

* 2a. Donor ID is invalid.
    * 3a1. BloodNet shows an error message.
    * Use case returns to step 3.

---

### **Use case: UC05 - List all donors in the system**

**Actor**: Admin staff

**MSS**

1. Admin staff requests to list all donors.
2. BloodNet displays a list of all non-deleted donors.

Use case ends.

---

### **Use case: UC06 - Find all eligible donors of a particular blood type**

**Actor**: Admin staff

**MSS**

1. Admin staff requests to find donors of a particular blood type.
2. BloodNet searches the entire donor list and applies the eligibility rules, such as date of birth and days since last donation.
3. BloodNet displays all donors who match the given blood type and the eligibility rules. 

Use case ends.

**Extensions**

* 1a. Invalid blood type entered. 
  * 1a1. BloodNet shows an error message. 
  * Use case relates back to step 1, prompting the user to re-enter a blood type.

---

### **Use case: UC07 - Record a blood donation by a donor**

**Actor**: Admin staff

**MSS**

1. Admin staff requests to find a donor by their name.
2. BloodNet searches and displays donors with matching names.
3. Admin staff requests to record a blood donation by a donor.
4. BloodNet records the donation details (including date and volume).

Use case ends.

**Extensions**

* 2a. No donors match the search name.
    * Use case ends.

* 3a. Donor ID is invalid.
    * 3a1. BloodNet shows an error message.
    * Use case returns to step 3.

* 3b. Date or volume format is invalid.
    * 3b1. BloodNet shows an error message.
    * Use case resumes at step 3.

---

### **Use case: UC08 - List all blood donations by a donor**

**Actor**: Admin staff

**MSS**

1. Admin staff requests to find a donor by their name.
2. BloodNet searches and displays donors with matching names.
3. Admin staff requests to list all blood donations by a specified donor.
4. BloodNet returns a list of blood donations, sorted by donation date in descending order.

Use case ends.

**Extensions**

* 2a. No donors match the search name.
    * Use case ends.

* 3a. Donor ID is invalid.
    * 3a1. BloodNet shows an error message.
    * Use case returns to step 3.

---

### **Use case: UC09 - Modify a blood donation record**

**Actor**: Admin staff

**MSS**

1. Admin staff requests to find a donor by their name.
2. BloodNet searches and displays donors with matching names.
3. Admin staff requests to list all blood donations by a specified donor.
4. BloodNet returns a list of blood donations, sorted by donation date in descending order.
5. Admin staff requests to update a specified blood donation record.
6. BloodNet updates the donation record.

Use case ends.

**Extensions**

* 2a. No donors match the search name.
    * Use case ends.

* 3a. Donor ID is invalid.
    * 3a1. BloodNet shows an error message.
    * Use case returns to step 3.

* 5a. Donation ID is invalid.
    * 1a1. BloodNet shows an error message.
      Use case returns to step 5.

* 5b. Invalid data format for date or volume.
    * 5b1. BloodNet shows an error message.
      Use case returns to step 5.

---

### **Use case: UC10 - Delete a blood donation record**

**Actor**: Admin staff

**MSS**

1. Admin staff requests to find a donor by their name.
2. BloodNet searches and displays donors with matching names.
3. Admin staff requests to list all blood donations by a specified donor.
4. BloodNet returns a list of blood donations, sorted by donation date in descending order.
5. Admin staff requests to delete a specified blood donation record.
6. BloodNet deletes the donation record.

Use case ends.

**Extensions**

* 2a. No donors match the search name.
    * Use case ends.

* 3a. Donor ID is invalid.
    * 3a1. BloodNet shows an error message.
    * Use case returns to step 3.

* 5a. Donation ID is invalid.
    * 1a1. BloodNet shows an error message.
      Use case returns to step 5.

---


### Non-Functional Requirements

1.  Should work on any _mainstream OS_ as long as it has Java `17` or above installed.
2.  Should be able to hold up to 1000 persons and the program should still be responsive with response time less than 1 second for each operation.
3.  A user with above average typing speed for regular English text (i.e. not code, not system admin commands) should be able to accomplish most of the tasks faster using commands rather than using the mouse.
4. Should start within 5 seconds on a typical user machine (4 core CPU, 8GB RAM, SSD).
5. User guide should be written with easy-to-understand English that is comprehensible to users without technical background.
6. The user interface should be intuitive enough for users who are not IT-saavy.
7. Should not be required to have beautiful visual design or animations since it is for administrative use.
8. Should have user confirmation for _destructive operations_.
9. Should provide error message, in response to an invalid operation, that details what went wrong and how to fix it for a non-technical user.
10. Should be designed for a single user within a single computer.
11. Should back up contacts data within a human-editable file such that it persists across instances of the program.
12. Should work without requiring an installer.
13. Should be built into a JAR file of no larger than 100MB.


### Glossary

* **Blood Type**: The blood types supported are A+, A-, B+, B-, AB+, AB-, O+ and O-
* **Donor**: Person who donates blood to others
* **Destructive operation**: An action that leads to permanent removal of data
* **Mainstream OS**: Windows, Linux, Unix, MacOS
* **Private contact detail**: A Singaporean (+65) contact detail that is not meant to be shared with others

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Instructions for manual testing**

Given below are instructions to test the app manually.

<box type="info" seamless>

**Note:** These instructions only provide a starting point for testers to work on;
testers are expected to do more *exploratory* testing.

</box>

### Launch and shutdown

1. Initial launch

    1. Download the jar file and copy into an empty folder

    2. Double-click the jar file Expected: Shows the GUI with a set of sample contacts. The window size may not be optimum.

2. Saving window preferences

    1. Resize the window to an optimum size. Move the window to a different location. Close the window.

    2. Re-launch the app by double-clicking the jar file.<br>
       Expected: The most recent window size and location is retained.

### Deleting a person

1. Deleting a person while all persons are being shown

    1. Prerequisites: List all persons using the `list` command. Multiple persons in the list.

    2. Test case: `delete 1`<br>
       Expected: First contact is deleted from the list. Details of the deleted contact shown in the status message. Timestamp in the status bar is updated.

    3. Test case: `delete 0`<br>
       Expected: No person is deleted. Error details shown in the status message. Status bar remains the same.

    4. Other incorrect delete commands to try: `delete`, `delete x`, `...` (where x is larger than the list size)<br>
       Expected: Similar to previous.

2. _{ more test cases …​ }_

### Saving data

1. Dealing with missing/corrupted data files

    1. _{explain how to simulate a missing/corrupted file, and the expected behavior}_

2. _{ more test cases …​ }_
