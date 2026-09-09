# Student Help Desk Management System

![Salesforce](https://img.shields.io/badge/Platform-Salesforce-00A1E0?logo=salesforce&logoColor=white)
![Project](https://img.shields.io/badge/Project-Student%20Help%20Desk-blue)
![Status](https://img.shields.io/badge/Status-Completed-success)
![Edition](https://img.shields.io/badge/Edition-Developer%20Edition-orange)

## Project Overview

The **Student Help Desk Management System** is a Salesforce-based CRM application developed to manage student support requests in a structured and automated way.

The system provides a centralized process for:

- Maintaining student information
- Registering and tracking student help requests
- Prioritizing support requests
- Assigning requests to support users
- Managing request status and resolution
- Collecting feedback after resolution
- Automating repetitive support processes
- Monitoring help-desk activity through reports and dashboards

The project was developed using **Salesforce Lightning Experience / Developer Edition** and demonstrates practical Salesforce Administrator skills such as custom objects, custom fields, lookup relationships, validation rules, record-triggered flows, reports, and dashboards.

---

## Business Problem

Educational institutions receive many student issues related to academics, IT systems, examinations, libraries, ID cards, certificates, hostels, and other services.

When these requests are handled manually, common problems include:

- Requests being missed or delayed
- Lack of priority management
- Difficulty tracking resolution status
- Repeated manual data entry
- No centralized feedback mechanism
- Limited visibility into support workload

This project addresses these problems by implementing a centralized Salesforce help-desk solution.

---

## Objectives

1. Create a centralized student support management system.
2. Maintain structured student records.
3. Track help requests from creation to resolution.
4. Automatically assign high priority to IT Support requests.
5. Ensure that a resolution is entered before closing the resolution process.
6. Automatically create a feedback record when a request is resolved.
7. Provide reports for operational analysis.
8. Provide a dashboard for management-level visibility.

---

## Technology Stack

| Technology | Usage |
|---|---|
| Salesforce CRM | Application platform |
| Salesforce Lightning Experience | User interface |
| Custom Objects | Data model |
| Lookup Relationships | Object relationships |
| Validation Rules | Data quality and business rules |
| Record-Triggered Flow | Automation |
| Reports | Data analysis |
| Dashboards | Visual analytics |
| Salesforce Developer Edition | Development environment |

---

## System Architecture

```text
                         STUDENT HELP DESK
                                |
             +------------------+------------------+
             |                  |                  |
             v                  v                  v
        STUDENT           HELP REQUEST          FEEDBACK
             |                  |                  |
             |                  |                  |
             |          +-------+-------+          |
             |          |               |          |
             |       Priority         Status       |
             |          |               |          |
             |          v               v          |
             |      Automation      Resolution     |
             |                          |           |
             +--------------------------+-----------+
                                        |
                                        v
                                  REPORTS & DASHBOARD
```

---

# Data Model

## 1. Student

The **Student** custom object stores the student's personal and academic information.

### Fields

| Field | Type | Description |
|---|---|---|
| Student ID | Text / Record Name | Unique student identifier |
| First Name | Text | Student first name |
| Last Name | Text | Student last name |
| Email | Email | Student email |
| Phone | Phone | Contact number |
| Department | Picklist | CSE, ECE, EEE, MECH, CIVIL, AI & DS |
| Year | Picklist | 1st Year to 4th Year |
| Section | Text | Student section |
| Student Status | Picklist | Active, Inactive, Graduated |
| Admission Date | Date | Admission date |

### Sample Record

- Student ID: `STU001`
- Name: `Rahul Kumar`
- Department: `CSE`
- Year: `3rd Year`
- Section: `A`
- Status: `Active`

---

## 2. Help Request

The **Help Request** custom object stores student support issues.

### Fields

| Field | Type | Description |
|---|---|---|
| Request Number | Auto Number | Format `REQ-{0000}` |
| Student | Lookup → Student | Student raising the request |
| Request Type | Picklist | Academic, IT Support, Examination, Library, ID Card, Certificates, Hostel, Other |
| Subject | Text | Request title |
| Description | Long Text Area | Detailed issue |
| Priority | Picklist | Low, Medium, High, Urgent |
| Status | Picklist | New, In Progress, Waiting for Student, Resolved, Closed |
| Assigned To | Lookup → User | Support person |
| Resolution | Long Text Area | Resolution details |

### Sample Records

| Request | Type | Priority | Status |
|---|---|---|---|
| REQ-0001 | IT Support | High | New |
| REQ-0002 | IT Support | High | Resolved |

---

## 3. Feedback

The **Feedback** custom object stores feedback related to resolved requests.

### Fields

| Field | Type | Description |
|---|---|---|
| Feedback Number | Auto Number | Format `FB-{0000}` |
| Student | Lookup → Student | Student providing feedback |
| Help Request | Lookup → Help Request | Related request |
| Rating | Picklist | 1 - Very Poor to 5 - Excellent |
| Comments | Long Text Area | Feedback comments |

---

# Relationships

```text
Student
   |
   +----< Help Request
   |
   +----< Feedback

Help Request
   |
   +----< Feedback
```

- One Student can have multiple Help Requests.
- One Student can have multiple Feedback records.
- One Help Request can have related Feedback.
- Help Request also contains an `Assigned To` lookup to the Salesforce User object.

---

# Automation

## Automation 1 — Auto High Priority for IT Support

### Purpose

Automatically set the priority of a new IT Support request to **High**.

### Configuration

| Setting | Value |
|---|---|
| Flow Type | Record-Triggered Flow |
| Object | Help Request |
| Trigger | A record is created |
| Entry Condition | Request Type = IT Support |
| Optimization | Fast Field Updates |
| Assignment | `$Record.Priority = "High"` |

### Business Flow

```text
New Help Request
       |
       v
Request Type = IT Support?
       |
      Yes
       |
       v
Priority = High
```

This is implemented as a before-save record-triggered flow because the automation updates a field on the record that triggered the flow.

---

## Automation 2 — Create Feedback When Resolved

### Purpose

Automatically create a Feedback record when a Help Request is resolved.

### Configuration

| Setting | Value |
|---|---|
| Flow Type | Record-Triggered Flow |
| Object | Help Request |
| Trigger | A record is updated |
| Condition | Status = Resolved |
| Optimization | Actions and Related Records |
| Action | Create Feedback record |
| Student | Triggering Help Request → Student |
| Help Request | Triggering Help Request → Record ID |

### Business Flow

```text
Help Request Updated
        |
        v
Status = Resolved?
        |
       Yes
        |
        v
Create Feedback Record
        |
        +---- Student = Request Student
        |
        +---- Help Request = Current Request
```

The after-save approach is appropriate because the flow creates a related record after the Help Request has been saved.

---

# Validation Rule

## Resolution Required When Resolved

### Rule Name

`Resolution_Required_When_Resolved`

### Formula

```text
AND(
    ISPICKVAL(Status__c, "Resolved"),
    ISBLANK(Resolution__c)
)
```

### Error Message

> Please enter the Resolution before marking the request as Resolved.

### Purpose

This validation prevents users from marking a Help Request as **Resolved** without documenting how the issue was resolved.

---

# Reports

The project contains operational reports for help-desk analysis.

## 1. Help Requests by Status

Groups Help Requests by:

- New
- In Progress
- Waiting for Student
- Resolved
- Closed

## 2. Help Requests by Type

Groups requests by:

- Academic
- IT Support
- Examination
- Library
- ID Card
- Certificates
- Hostel
- Other

## 3. Help Requests by Priority

Groups requests by:

- Low
- Medium
- High
- Urgent

---

# Dashboard

## Student Help Desk Dashboard

The dashboard provides a consolidated view of help-desk activity.

### Dashboard Components

1. **Requests by Status**
2. **Requests by Type**
3. **Requests by Priority**

### Dashboard Preview

![Student Help Desk Dashboard](screenshots/dashboard.png)

---

# Project Evidence / Screenshots

The following screenshots are included as implementation proof.

## 1. Student Record

This screenshot demonstrates the configured Student record and student information.

![Student Record](screenshots/student-record.png)

---

## 2. Help Request Validation / Record

This screenshot demonstrates the Help Request configuration and validation behavior.

![Help Request Validation](screenshots/help-request-validation.png)

---

## 3. Help Request Record

The record demonstrates a real Help Request with:

- Student
- Request Type
- Subject
- Description
- Priority
- Status
- Resolution

![Help Request Record](screenshots/help-request-record.png)

---

## 4. Feedback Record

This screenshot demonstrates the Feedback object and its relationship to the Student and Help Request.

![Feedback Record](screenshots/feedback-record.png)

---

## 5. Feedback Automation Flow

The Salesforce Flow Builder screenshot provides visual evidence of the automation that creates a Feedback record.

![Feedback Flow](screenshots/feedback-flow.png)

---

## 6. Create Feedback Record Configuration

This screenshot shows the `Create Records` element used to create the Feedback record and populate the Student and Help Request lookup fields from the triggering Help Request.

![Create Feedback Flow](screenshots/create-feedback-flow.png)

---

## 7. Help Requests by Type Report

The report groups requests by Request Type and provides a visual chart.

![Help Requests by Type](screenshots/help-requests-by-type.png)

---

## 8. Final Dashboard

The final dashboard combines the operational reports into a single management view.

![Student Help Desk Dashboard](screenshots/dashboard.png)

---

# Testing

The application was tested using sample Salesforce records.

| Test Case | Test | Expected Result | Status |
|---|---|---|---|
| TC-01 | Create Student STU001 | Student created successfully | Passed |
| TC-02 | Create IT Support request | Priority automatically becomes High | Passed |
| TC-03 | Set Status = Resolved without Resolution | Save blocked by validation | Passed |
| TC-04 | Set Status = Resolved with Resolution | Record saves successfully | Passed |
| TC-05 | Resolve Help Request | Feedback record automatically created | Passed |
| TC-06 | Open Feedback list | New feedback record is visible | Passed |
| TC-07 | Open Dashboard | Status, Type and Priority charts are displayed | Passed |

---

# End-to-End Process

```text
1. Create Student
       |
       v
2. Create Help Request
       |
       v
3. IT Support?
       |
      Yes
       |
       v
4. Automatically set Priority = High
       |
       v
5. Support staff work on request
       |
       v
6. Enter Resolution
       |
       v
7. Set Status = Resolved
       |
       v
8. Automatically create Feedback
       |
       v
9. Student provides Rating + Comments
       |
       v
10. Reports and Dashboard provide insights
```

---

# Key Salesforce Features Demonstrated

- Custom Objects
- Custom Fields
- Text, Email, Phone, Date and Long Text Area fields
- Picklist fields
- Auto Number fields
- Lookup Relationships
- Validation Rules
- Record-Triggered Flows
- Before-Save / Fast Field Updates
- After-Save / Actions and Related Records
- `$Record` triggering record variable
- Create Records Flow element
- Salesforce Reports
- Report Grouping
- Report Charts
- Salesforce Dashboard
- Testing and validation

---

# Skills Demonstrated

This project demonstrates practical understanding of:

### Salesforce Administration

- Data modeling
- Object configuration
- Field configuration
- Relationships
- Business rules
- Automation
- Reports
- Dashboards
- Testing

### Problem Solving

The project converts a real-world student support process into a structured CRM workflow and reduces repetitive manual work through automation.

---

# Future Enhancements

The following features can be added in future versions:

1. **Auto-Assignment**
   - Automatically assign requests to support users based on Request Type.

2. **Email Notifications**
   - Notify students when request status changes.

3. **Urgent Request Alerts**
   - Notify support staff immediately when Priority is Urgent.

4. **Student Portal**
   - Allow students to submit and track requests through Experience Cloud.

5. **SLA Management**
   - Track response and resolution deadlines.

6. **Escalation Automation**
   - Automatically escalate overdue requests.

7. **Advanced Dashboard**
   - Add average resolution time, open-request aging, and SLA performance.

8. **Duplicate Request Detection**
   - Detect similar requests before creating a new record.

---

# Project Structure

Recommended GitHub repository structure:

```text
Student-Help-Desk-Management-System/
│
├── README.md
│
├── screenshots/
│   ├── student-record.png
│   ├── help-request-validation.png
│   ├── help-request-record.png
│   ├── feedback-record.png
│   ├── feedback-flow.png
│   ├── create-feedback-flow.png
│   ├── help-requests-by-type.png
│   └── dashboard.png
│
└── documentation/
    └── Student_Help_Desk_Management_System_Project_Documentation.docx
```

---

# Conclusion

The **Student Help Desk Management System** demonstrates how Salesforce can be configured to manage a complete student-support lifecycle.

The project combines structured data management, business validation, automation, feedback collection, reporting, and dashboard visualization into a single Salesforce application.

It is suitable as a **Salesforce Administrator portfolio project** and can be extended with more advanced automation, security, Experience Cloud, notifications, and SLA functionality.

---

## Author

**Salesforce Project — Student Help Desk Management System**

Built using **Salesforce Developer Edition**.

