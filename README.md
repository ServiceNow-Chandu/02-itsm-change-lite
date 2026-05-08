# Change Management – Change Lite (ServiceNow)

## Project Overview

The Change Lite project is a production-style Change Management implementation developed in ServiceNow to simulate enterprise ITIL Change Management processes. The project demonstrates risk assessment automation, CAB approval workflows, blackout window validation, emergency change handling, Incident-to-Change integration, and client-side governance controls.

The implementation was designed using traditional ServiceNow administration and development practices using Global Scope and Update Sets to replicate real-world enterprise development standards.

---

# Business Requirement

The organization required a centralized Change Management solution capable of:

* Managing Normal, Emergency, and Standard Changes
* Automating change risk calculation
* Implementing CAB approval governance
* Restricting deployments during blackout windows
* Allowing emergency fast-track approvals
* Linking Incidents to Changes
* Enforcing implementation documentation standards
* Supporting ITIL-based change lifecycle management

---

# Technologies Used

| Technology        | Purpose                       |
| ----------------- | ----------------------------- |
| ServiceNow ITSM   | Change Management             |
| Business Rules    | Server-side automation        |
| Script Includes   | Risk calculation engine       |
| GlideSchedule API | Blackout validation           |
| UI Policies       | Dynamic form controls         |
| Client Scripts    | Field validations             |
| UI Actions        | Incident-to-Change automation |
| ATF               | Automated Testing Framework   |
| Update Sets       | Version control and migration |

---

# Project Scope

| Item              | Value          |
| ----------------- | -------------- |
| Project Name      | Change Lite    |
| Table Used        | change_request |
| Development Scope | Global         |
| Deployment Method | Update Sets    |

---

# Features Implemented

# 1. Change Types Configuration

Configured enterprise-standard change models using Out-of-Box Change Types.

### Change Types Used

| Type      | Purpose                         |
| --------- | ------------------------------- |
| Normal    | Standard approval-based changes |
| Emergency | Critical production fixes       |
| Standard  | Pre-approved repetitive changes |

---

# 2. Risk Calculation Engine

A reusable Script Include named `ChangeRiskCalc` was developed to dynamically calculate risk scores for Change Requests.

---

## Risk Factors Considered

| Factor   | Logic                                  |
| -------- | -------------------------------------- |
| Impact   | High impact increases risk             |
| Urgency  | High urgency increases risk            |
| Category | Network/Security changes increase risk |

---

## Script Include – ChangeRiskCalc

```javascript id="k2s2ow"
var ChangeRiskCalc = Class.create();

ChangeRiskCalc.prototype = {

  initialize: function () {},

  calculate: function (chg) {

    var score = 0;

    // Impact
    if (chg.impact == 1) score += 40;
    else if (chg.impact == 2) score += 25;
    else score += 10;

    // Urgency
    if (chg.urgency == 1) score += 40;
    else if (chg.urgency == 2) score += 25;
    else score += 10;

    // Category
    if (chg.category == 'network') score += 20;

    if (chg.category == 'security') score += 30;

    return score;
  },

  riskLevel: function (score) {

    if (score >= 80)
      return 'High';

    if (score >= 50)
      return 'Moderate';

    return 'Low';
  },

  type: 'ChangeRiskCalc'
};
```

---

# 3. Business Rule – Automated Risk Calculation

A Before Insert/Update Business Rule was implemented to automatically calculate Risk Score and Risk Level during Change creation or modification.

---

## Functionalities

* Executes before Insert and Update
* Calls reusable Script Include
* Automatically updates:

  * Risk Score
  * Risk Level

---

## Business Rule Script

```javascript id="tt3mb3"
(function executeRule(current, previous) {

  var calc = new ChangeRiskCalc();

  var score = calc.calculate(current);

  current.risk_score = score;

  current.risk = calc.riskLevel(score);

})(current, previous);
```

---

# 4. CAB Approval Governance

Configured approval policies for enterprise governance and compliance.

---

## Approval Policy Logic

### Conditions

| Field       | Value            |
| ----------- | ---------------- |
| Change Type | Normal           |
| Risk        | High OR Moderate |

---

## Approvers

* Change Manager
* CAB Group

---

# 5. Emergency Change Fast-Track

Implemented dynamic UI Policies for Emergency Changes.

---

## Functionalities

| Feature            | Behavior            |
| ------------------ | ------------------- |
| Justification      | Mandatory           |
| Approval Field     | Read-only           |
| Emergency Handling | Fast-track approval |

---

## Business Value

* Supports production outage handling
* Improves emergency response
* Maintains audit governance

---

# 6. Blackout Window Enforcement

Implemented deployment blackout validation using GlideSchedule API.

---

## Blackout Schedule

| Schedule            | Example                   |
| ------------------- | ------------------------- |
| Production Blackout | Friday 6 PM → Sunday 6 AM |

---

## Business Rule Logic

* Prevents non-emergency changes during blackout periods
* Displays validation error
* Aborts record transaction

---

## Blackout Validation Script

```javascript id="g8gh4q"
(function executeRule(current, previous) {

  var sched = new GlideSchedule();

  sched.load('PUT_BLACKOUT_SCHEDULE_SYS_ID_HERE');

  if (sched.isInSchedule(new GlideDateTime())) {

    gs.addErrorMessage(
      'Changes are not allowed during blackout window.'
    );

    current.setAbortAction(true);
  }

})(current, previous);
```

---

# 7. Incident to Change Integration

Developed a custom UI Action named `Create Change` on the Incident table.

---

## Functionalities

* Creates Change directly from Incident
* Transfers:

  * Description
  * Impact
  * Urgency
* Redirects user to Change form

---

## UI Action Script

```javascript id="7b08rw"
var chg = new GlideRecord('change_request');

chg.initialize();

chg.short_description =
'Change for Incident ' + current.number;

chg.description = current.description;

chg.impact = current.impact;

chg.urgency = current.urgency;

chg.insert();

action.setRedirectURL(chg);
```

---

# 8. Client-Side Validation

Implemented onChange Client Script for governance enforcement during implementation phase.

---

## Validation Logic

When State = Implement:

* Implementation Plan mandatory
* Test Plan mandatory
* Backout Plan mandatory

---

## Client Script

```javascript id="svs1be"
function onChange(control, oldValue, newValue) {

  if (newValue == 'Implement') {

    g_form.setMandatory(
      'implementation_plan',
      true
    );

    g_form.setMandatory(
      'test_plan',
      true
    );

    g_form.setMandatory(
      'backout_plan',
      true
    );
  }
}
```

---

# End-to-End Workflow

```text id="0t75yb"
User creates Change Request
        ↓
Business Rule calculates Risk Score
        ↓
Risk Level automatically assigned
        ↓
Approval Policy evaluates Change
        ↓
CAB approval triggered if required
        ↓
Blackout validation executes
        ↓
Emergency Changes bypass standard approval
        ↓
Implementation validations enforced
        ↓
Change progresses through lifecycle
```

---

# ITIL Concepts Demonstrated

| ITIL Concept             | Implemented |
| ------------------------ | ----------- |
| Change Lifecycle         | Yes         |
| Risk Assessment          | Yes         |
| CAB Governance           | Yes         |
| Emergency Change Process | Yes         |
| Blackout Management      | Yes         |
| Incident Integration     | Yes         |
| Client-side Governance   | Yes         |
| Reusable Business Logic  | Yes         |
| Automated Validation     | Yes         |

---

# Technical Challenges Solved

## Challenge 1

Automating enterprise risk assessment.

### Solution

Developed reusable `ChangeRiskCalc` Script Include.

---

## Challenge 2

Preventing deployments during maintenance windows.

### Solution

Implemented GlideSchedule blackout validation.

---

## Challenge 3

Emergency change handling without blocking operations.

### Solution

Configured fast-track approval governance.

---

## Challenge 4

Reducing duplicate manual data entry.

### Solution

Developed Incident-to-Change UI Action automation.

---

# Automated Testing (ATF)

Created Automated Test Framework (ATF) test scenarios.

---

## Test Cases

| Test                    | Validation               |
| ----------------------- | ------------------------ |
| High Risk Normal Change | CAB approval required    |
| Emergency Change        | Auto-approved            |
| Blackout Validation     | Change blocked           |
| Incident to Change      | UI Action creates Change |

---

# GitHub Repository Structure

```text id="k84g6c"
02-itsm-change-lite/
│
├── README.md
├── docs/
│   ├── architecture.md
│   ├── risk-logic.md
│   ├── blackout-validation.md
│   ├── approval-flow.md
│   └── testing.md
│
├── update_sets/
│   └── P02_Change_Lite_YYYYMMDD.xml
│
├── media/
│
└── scripts/
```

---

# Real-World Enterprise Benefits

| Benefit                 | Impact                        |
| ----------------------- | ----------------------------- |
| Automated Governance    | Reduced manual review effort  |
| CAB Enforcement         | Improved compliance           |
| Blackout Protection     | Reduced production outages    |
| Incident Integration    | Faster remediation process    |
| Dynamic Risk Assessment | Better operational visibility |

---

# Project Outcome

The Change Lite implementation successfully simulated a real-world enterprise Change Management process with governance automation, risk assessment, blackout protection, approval routing, and Incident integration.

The project improved:

* Change governance
* Operational compliance
* Risk visibility
* Deployment safety
* Process automation
