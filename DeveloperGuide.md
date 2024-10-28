---
  layout: default.md
  title: "Developer Guide"
  pageNav: 3
---

# AB-3 Developer Guide

<!-- * Table of Contents -->
<page-nav-print />

--------------------------------------------------------------------------------------------------------------------

## **Acknowledgements**

_{ list here sources of all reused/adapted ideas, code, documentation, and third-party libraries -- include links to the original source as well }_

--------------------------------------------------------------------------------------------------------------------

## **Setting up, getting started**

Refer to the guide [_Setting up and getting started_](SettingUp.md).

--------------------------------------------------------------------------------------------------------------------

## **Design**

### Architecture

<puml src="diagrams/ArchitectureDiagram.puml" width="280" />

The ***Architecture Diagram*** given above explains the high-level design of the App.

Given below is a quick overview of main components and how they interact with each other.

**Main components of the architecture**

**`Main`** (consisting of classes [`Main`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/Main.java) and [`MainApp`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/MainApp.java)) is in charge of the app launch and shut down.
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

Each of the four main components (also shown in the diagram above),

* defines its *API* in an `interface` with the same name as the Component.
* implements its functionality using a concrete `{Component Name}Manager` class (which follows the corresponding API `interface` mentioned in the previous point.

For example, the `Logic` component defines its API in the `Logic.java` interface and implements its functionality using the `LogicManager.java` class which follows the `Logic` interface. Other components interact with a given component through its interface rather than the concrete class (reason: to prevent outside component's being coupled to the implementation of a component), as illustrated in the (partial) class diagram below.

<puml src="diagrams/ComponentManagers.puml" width="300" />

The sections below give more details of each component.

### UI component

The **API** of this component is specified in [`Ui.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/Ui.java)

<puml src="diagrams/UiClassDiagram.puml" alt="Structure of the UI Component"/>

The UI consists of a `MainWindow` that is made up of parts e.g.`CommandBox`, `ResultDisplay`, `StudentListPanel`, `StatusBarFooter` etc. All these, including the `MainWindow`, inherit from the abstract `UiPart` class which captures the commonalities between classes that represent parts of the visible GUI.

The `UI` component uses the JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files that are in the `src/main/resources/view` folder. For example, the layout of the [`MainWindow`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/MainWindow.java) is specified in [`MainWindow.fxml`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/resources/view/MainWindow.fxml)

The `UI` component,

* executes user commands using the `Logic` component.
* listens for changes to `Model` data so that the UI can be updated with the modified data.
* keeps a reference to the `Logic` component, because the `UI` relies on the `Logic` to execute commands.
* depends on some classes in the `Model` component, as it displays `Student` object residing in the `Model`.

### Logic component

**API** : [`Logic.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/logic/Logic.java)

Here's a (partial) class diagram of the `Logic` component:

<puml src="diagrams/LogicClassDiagram.puml" width="550"/>

The sequence diagram below illustrates the interactions within the `Logic` component, taking `execute("delete 1")` API call as an example.

<puml src="diagrams/DeleteSequenceDiagram.puml" alt="Interactions Inside the Logic Component for the `delete 1` Command" />

<box type="info" seamless>

**Note:** The lifeline for `DeleteCommandParser` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline continues till the end of diagram.
</box>

How the `Logic` component works:

1. When `Logic` is called upon to execute a command, it is passed to an `AddressBookParser` object which in turn creates a parser that matches the command (e.g., `DeleteCommandParser`) and uses it to parse the command.
1. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `DeleteCommand`) which is executed by the `LogicManager`.
1. The command can communicate with the `Model` when it is executed (e.g. to delete a student).<br>
   Note that although this is shown as a single step in the diagram above (for simplicity), in the code it can take several interactions (between the command object and the `Model`) to achieve.
1. The result of the command execution is encapsulated as a `CommandResult` object which is returned back from `Logic`.

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<puml src="diagrams/ParserClasses.puml" width="600"/>

How the parsing works:
* When called upon to parse a user command, the `AddressBookParser` class creates an `XYZCommandParser` (`XYZ` is a placeholder for the specific command name e.g., `AddCommandParser`) which uses the other classes shown above to parse the user command and create a `XYZCommand` object (e.g., `AddCommand`) which the `AddressBookParser` returns back as a `Command` object.
* All `XYZCommandParser` classes (e.g., `AddCommandParser`, `DeleteCommandParser`, ...) inherit from the `Parser` interface so that they can be treated similarly where possible e.g, during testing.

### Model component
**API** : [`Model.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/model/Model.java)

<puml src="diagrams/ModelClassDiagram.puml" width="450" />


The `Model` component,

* stores the address book data i.e., all `Student` objects (which are contained in a `UniqueStudentList` object).
* stores the currently 'selected' `Student` objects (e.g., results of a search query) as a separate _filtered_ list which is exposed to outsiders as an unmodifiable `ObservableList<Student>` that can be 'observed' e.g. the UI can be bound to this list so that the UI automatically updates when the data in the list change.
* stores a `UserPref` object that represents the user’s preferences. This is exposed to the outside as a `ReadOnlyUserPref` objects.
* does not depend on any of the other three components (as the `Model` represents data entities of the domain, they should make sense on their own without depending on other components)

<box type="info" seamless>

**Note:** An alternative (arguably, a more OOP) model is given below. It has a `Tag` list in the `AddressBook`, which `Student` references. This allows `AddressBook` to only require one `Tag` object per unique tag, instead of each `Student` needing their own `Tag` objects.<br>

<puml src="diagrams/BetterModelClassDiagram.puml" width="450" />

</box>


### Storage component

**API** : [`Storage.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/storage/Storage.java)

<puml src="diagrams/StorageClassDiagram.puml" width="550" />

The `Storage` component,
* can save both address book data and user preference data in JSON format, and read them back into corresponding objects.
* inherits from both `AddressBookStorage` and `UserPrefStorage`, which means it can be treated as either one (if only the functionality of only one is needed).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects that belong to the `Model`)

### Common classes

Classes used by multiple components are in the `seedu.address.commons` package.

--------------------------------------------------------------------------------------------------------------------

## **Implementation**

This section describes some noteworthy details on how certain features are implemented.

### \[Proposed\] Undo/redo feature

#### Proposed Implementation

The proposed undo/redo mechanism is facilitated by `VersionedAddressBook`. It extends `AddressBook` with an undo/redo history, stored internally as an `addressBookStateList` and `currentStatePointer`. Additionally, it implements the following operations:

* `VersionedAddressBook#commit()` — Saves the current address book state in its history.
* `VersionedAddressBook#undo()` — Restores the previous address book state from its history.
* `VersionedAddressBook#redo()` — Restores a previously undone address book state from its history.

These operations are exposed in the `Model` interface as `Model#commitAddressBook()`, `Model#undoAddressBook()` and `Model#redoAddressBook()` respectively.

Given below is an example usage scenario and how the undo/redo mechanism behaves at each step.

Step 1. The user launches the application for the first time. The `VersionedAddressBook` will be initialized with the initial address book state, and the `currentStatePointer` pointing to that single address book state.

<puml src="diagrams/UndoRedoState0.puml" alt="UndoRedoState0" />

Step 2. The user executes `delete 5` command to delete the 5th student in the address book. The `delete` command calls `Model#commitAddressBook()`, causing the modified state of the address book after the `delete 5` command executes to be saved in the `addressBookStateList`, and the `currentStatePointer` is shifted to the newly inserted address book state.

<puml src="diagrams/UndoRedoState1.puml" alt="UndoRedoState1" />

Step 3. The user executes `add n/David …​` to add a new student. The `add` command also calls `Model#commitAddressBook()`, causing another modified address book state to be saved into the `addressBookStateList`.

<puml src="diagrams/UndoRedoState2.puml" alt="UndoRedoState2" />

<box type="info" seamless>

**Note:** If a command fails its execution, it will not call `Model#commitAddressBook()`, so the address book state will not be saved into the `addressBookStateList`.

</box>

Step 4. The user now decides that adding the student was a mistake, and decides to undo that action by executing the `undo` command. The `undo` command will call `Model#undoAddressBook()`, which will shift the `currentStatePointer` once to the left, pointing it to the previous address book state, and restores the address book to that state.

<puml src="diagrams/UndoRedoState3.puml" alt="UndoRedoState3" />


<box type="info" seamless>

**Note:** If the `currentStatePointer` is at index 0, pointing to the initial AddressBook state, then there are no previous AddressBook states to restore. The `undo` command uses `Model#canUndoAddressBook()` to check if this is the case. If so, it will return an error to the user rather
than attempting to perform the undo.

</box>

The following sequence diagram shows how an undo operation goes through the `Logic` component:

<puml src="diagrams/UndoSequenceDiagram-Logic.puml" alt="UndoSequenceDiagram-Logic" />

<box type="info" seamless>

**Note:** The lifeline for `UndoCommand` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.

</box>

Similarly, how an undo operation goes through the `Model` component is shown below:

<puml src="diagrams/UndoSequenceDiagram-Model.puml" alt="UndoSequenceDiagram-Model" />

The `redo` command does the opposite — it calls `Model#redoAddressBook()`, which shifts the `currentStatePointer` once to the right, pointing to the previously undone state, and restores the address book to that state.

<box type="info" seamless>

**Note:** If the `currentStatePointer` is at index `addressBookStateList.size() - 1`, pointing to the latest address book state, then there are no undone AddressBook states to restore. The `redo` command uses `Model#canRedoAddressBook()` to check if this is the case. If so, it will return an error to the user rather than attempting to perform the redo.

</box>

Step 5. The user then decides to execute the command `list`. Commands that do not modify the address book, such as `list`, will usually not call `Model#commitAddressBook()`, `Model#undoAddressBook()` or `Model#redoAddressBook()`. Thus, the `addressBookStateList` remains unchanged.

<puml src="diagrams/UndoRedoState4.puml" alt="UndoRedoState4" />

Step 6. The user executes `clear`, which calls `Model#commitAddressBook()`. Since the `currentStatePointer` is not pointing at the end of the `addressBookStateList`, all address book states after the `currentStatePointer` will be purged. Reason: It no longer makes sense to redo the `add n/David …​` command. This is the behavior that most modern desktop applications follow.

<puml src="diagrams/UndoRedoState5.puml" alt="UndoRedoState5" />

The following activity diagram summarizes what happens when a user executes a new command:

<puml src="diagrams/CommitActivityDiagram.puml" width="250" />

#### Design considerations:

**Aspect: How undo & redo executes:**

* **Alternative 1 (current choice):** Saves the entire address book.
  * Pros: Easy to implement.
  * Cons: May have performance issues in terms of memory usage.

* **Alternative 2:** Individual command knows how to undo/redo by
  itself.
  * Pros: Will use less memory (e.g. for `delete`, just save the student being deleted).
  * Cons: We must ensure that the implementation of each individual command are correct.

_{more aspects and alternatives to be added}_

### \[Proposed\] Data archiving

_{Explain here how the data archiving feature will be implemented}_


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

* NUS SOC tutors who want to track their student's progress

**Value proposition**: manage students' tasks and attendance faster than a typical mouse/GUI driven app


### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority  | As a …​                         | I want to …​                                                                                       | So that I can…​                                                                          |
|-----------|---------------------------------|----------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------|
| `* * *`   | tutor                           | add student data                                                                                   | I can add student information efficiently                                                |
| `* * *`   | tutor                           | delete student data                                                                                | I can delete student information efficiently                                             |
| `* * *`   | tutor                           | categorise students based on tutorial classes                                                      | I can find out which students are in my class                                            |
| `* * *`   | course coordinator              | record student attendance daily through a CLI command                                              | I can track student participation                                                        |
| `* * *`   | assignment-focused instructor   | create assignments for students                                                                    | I can assign tasks to students                                                           |
| `* *`     | tutor                           | edit student data                                                                                  | I can manage student information efficiently                                             |
| `* *`     | tutor                           | undo previous commands                                                                             | I can undo mistakes made in previous commands                                            |
| `* *`     | independent tutor               | manage my own data files independently                                                             | I don’t have to worry about multi-user access conflicts                                  |
| `* *`     | power user                      | use command shortcuts for frequent operations                                                      | I can save time                                                                          |
| `* *`     | data-cautious professional      | back up my data to a local file                                                                    | I can restore it if something goes wrong                                                 |
| `* *`     | solo practitioner               | optimise my application for single-user functionality                                              | I can focus on my tasks without distractions                                             |
| `* *`     | assignment-focused instructor   | mark assignments as completed or pending                                                           | I can track the progress of each student                                                 |
| `* *`     | deadline-aware tutor            | have a reminder command to alert me about overdue assignments                                      | I can follow up with students                                                            |
| `* *`     | grading instructor              | enter grades and calculate final scores using a CLI command                                        | I can manage grading efficiently                                                         |
| `* *`     | class performance analyst       | see a summary of grades for a class in a single command                                            | I can evaluate overall performance                                                       |
| `* *`     | student mentor                  | generate predictive insights based on historical data to identify students who may need extra help | I can provide timely interventions                                                       |
| `* *`     | parent communication officer    | use a command to generate a progress report for each student                                       | I can share it during parent-teacher meetings                                            |
| `* *`     | long-term performance evaluator | compare student performance across different terms                                                 | I can assess improvement or decline                                                      |
| `* *`     | student progress tracker        | tag students with custom labels                                                                    | I can quickly identify those needing special attention                                   |
| `* *`     | cross-platform user             | run the application on any platform (Windows, Linux, OS X) without any OS-specific dependencies    | I can use it anywhere                                                                    |
| `* *`     | convenience-seeking educator    | use the application without an installer                                                           | I can use it directly from the downloaded JAR file                                       |
| `* *`     | data visualization enthusiast   | use ASCII-based bar charts and progress bars for a quick visual representation of data             | I can easily grasp trends                                                                |
| `* *`     | self-learning user              | use interactive help commands                                                                      | I can understand how to use the application without referring to external documentation. |
| `*`       | seasoned CLI user               | have a command history feature                                                                     | I can reuse previous commands without retyping them                                      |
| `*`       | advanced user                   | store all data in a local, human-editable text file                                                | I can manually edit it when needed                                                       |
| `*`       | meticulous user                 | check data integrity when loading files                                                            | I am alerted of any corrupt or inconsistent data                                         |
| `*`       | privacy-focused educator        | ensure that all data are user-specific and not shared                                              | My data remains private                                                                  |
| `*`       | custom-evaluation designer      | define my own grading scale via a text file                                                        | I can adapt it to different evaluation criteria                                          |
| `*`       | security-conscious tutor        | secure sensitive student data with encryption in local files                                       | Unauthorized access is prevented                                                         |
| `*`       | safety-focused user             | automate daily backups with encryption                                                             | My data is safe from accidental loss                                                     |
| `*`       | personalization enthusiast      | customize the CLI interface with different themes and fonts                                        | I can have a more comfortable user experience                                            |
| `*`       | course manager                  | configure the grading system and attendance rules via a configuration text file                    | I can tailor the application to my needs                                                 |


### Use cases

(For all use cases below, the **System** is the `AddressBook` and the **Actor** is the `user`, unless specified otherwise)

**Use case: Delete a student**

**MSS**

1.  User requests to list students
2.  AddressBook shows a list of students
3.  User requests to delete a specific student in the list
4.  AddressBook deletes the student

    Use case ends.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is invalid.

    * 3a1. AddressBook shows an error message.

      Use case resumes at step 2.

**Use case: Add a student**

**MSS**

1.  User requests to list students
2.  AddressBook shows a list of students
3.  User requests to add a specific student to the list
4.  AddressBook adds the student

    Use case ends.

**Extensions**

* 2a. The list is full.
  * AddressBook shows an error message
    Use case ends.

* 3a. The given index is invalid.

    * 3a1. AddressBook shows an error message.

      Use case resumes at step 2.

**Use case: Categorise student based on tutorial class**

**MSS**

1.  User requests to list students
2.  AddressBook shows a list of students
3.  User requests to categorise the tutorial class of a specific student to the list
4.  AddressBook checks if the tutorial class is valid. 
5.  AddressBook edits the student's tutorial class

    Use case ends.

**Extensions**

* 1a. The list is empty.

    Use case ends.

* 3a. The given student index is invalid.

    * 3a1. AddressBook shows an error message.

      Use case resumes at step 3.

* 4a. The given tutorial index is invalid.

  * 4a1. AddressBook shows an error message.

    Use case resumes at step 4.

**Use case: Record Student Attendance**

**MSS**

1.  User requests to list students
2.  AddressBook shows a list of students
3.  User requests to mark attendance of a specific student to the list
4.  AddressBook adds the date and status to the students attendance list

    Use case ends.

**Extensions**

* 1a. The list is empty.

  Use case ends.

* 3a. The given student index is invalid.

    * 3a1. AddressBook shows an error message.

      Use case resumes at step 3.

* 4a. The date given is invalid or after the current date.

    * 4a1. AddressBook shows an error message.

      Use case resumes at step 4.

**Use case: Add assignment**

**MSS**

1.  User requests to add an assignment 
2.  AddressBook creates the assignment
3.  AddressBook gets a list of students 
4.  AddressBook adds the copy of the assignment to the assignment list of every student in the list.

       Use case ends.

**Extensions**

* 1a. The list is empty.

  Use case ends.

### Non-Functional Requirements

1. Should work on any _mainstream OS_ as long as it has Java `17` or above installed.
2. Should be able to hold up to 1000 students without a noticeable sluggishness in performance for typical usage.
3. A user with above average typing speed for regular English text (i.e. not code, not system admin commands) should be able to accomplish most of the tasks faster using commands than using the mouse.
4. The application should start up in under 3 seconds on a standard machine with Java 17 installed.
5. Automated backups should be encrypted to secure data during storage and transfer. 
6. Customizable themes should include options for high contrast and font size adjustments for visual accessibility. 
7. The system is optimized for single-user operation and does not need to handle multi-user access.
8. The application should be optimized for low CPU and memory usage to run smoothly on standard hardware.
9. The application should be reliable enough for continuous use during working hours without the need for frequent restarts.
10. The application size should be less than 50mb to facilitate easy distribution and storage.

### Glossary

* **AB-3**: The code name for the AddressBook Level 3 application, which serves as the base framework for the student management system being developed.
* **NUS SOC**: National University of Singapore, School of Computing. The application is designed for tutors within this institution to manage student progress effectively.
* **Tutor**: The primary user of the application—an NUS School of Computing tutor responsible for tracking students' progress, attendance, and assignments.
* **Mainstream OS**: Windows, Linux, Unix, MacOS
* **CLI (Command Line Interface)**: A text-based interface where users interact with the application by typing commands rather than using graphical elements like buttons.
* **GUI (Graphical User Interface)**: A visual interface that allows users to interact with the application through graphical elements like windows, buttons, and menus.
* **JSON (JavaScript Object Notation)**: A lightweight data format used for data storage and transmission, typically for configuration or file storage.
* **Parser**: A component that interprets user commands and converts them into actions or objects that the system can process.
* **MSS (Main Success Scenario)**: The primary sequence of steps in a use case where everything proceeds as expected without any errors.
* **JAR File**: Java ARchive file; a package file format that bundles Java class files and associated metadata for distribution.

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

   1. Double-click the jar file Expected: Shows the GUI with a set of sample contacts. The window size may not be optimum.

1. Saving window preferences

   1. Resize the window to an optimum size. Move the window to a different location. Close the window.

   1. Re-launch the app by double-clicking the jar file.<br>
       Expected: The most recent window size and location is retained.

1. _{ more test cases …​ }_

### Deleting a student

1. Deleting a student while all students are being shown

   1. Prerequisites: List all students using the `list` command. Multiple students in the list.

   1. Test case: `delete 1`<br>
      Expected: First contact is deleted from the list. Details of the deleted contact shown in the status message. Timestamp in the status bar is updated.

   1. Test case: `delete 0`<br>
      Expected: No student is deleted. Error details shown in the status message. Status bar remains the same.

   1. Other incorrect delete commands to try: `delete`, `delete x`, `...` (where x is larger than the list size)<br>
      Expected: Similar to previous.

1. _{ more test cases …​ }_

### Saving data

1. Dealing with missing/corrupted data files

   1. _{explain how to simulate a missing/corrupted file, and the expected behavior}_

1. _{ more test cases …​ }_
