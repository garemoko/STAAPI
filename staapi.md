# Server-side Tracking and Assessment API (STAAPI) specification
## Introduction
The Server-side Tracking and Assessment API (STAAPI) specification does not exist except in idea form in this document. This document is not a complete, 
fully fleshed-out solution, but rather a description of a suggested high level approach as a starting point for discussion and further detail.

STAAPI is an attempt to define a common interface between packaged content and a Training Delivery System (TDS) such as a Learning Management System (LMS) such 
that marking of assessments and tracking of progress and success can be handled server side by the TDS following rules defined by the content package while retaining 
the portability of content packages enjoyed using SCORM.  

## Rationale
SCORM uses a JavaScript API to communicate between e-learning content and the TDS. Content packages are generally entirely self-contained such that all of the code 
relating to marking of assessments and sending of data relating to learner progress is downloaded to and executed on the learner’s computer. This is inherently insecure 
in that for any SCORM course, a learner (or an attacker with control of the learner’s computer) with sufficient technical knowledge is able to read the logic for marking 
the question, read tracking data and send their own false tracking data.

xAPI enables server-side marking of assessments and tracking of assessments, however this functionality has not seen widespread adoption. This is largely because widely 
used authoring tools developed to produce JavaScript based content cannot be easily adapted to produce server-side assessed and tracked content. Instructional designers 
like the ease and rapidity of rapid authority tools and developing content without them is normally prohibitively complex or expensive. Further, content produced with 
server side tracking and assessment is not easily portable between TDSs, a key benefit of SCORM packaged content. 

cmi5 defines a mechanism for the launch of learning experiences (including e-learning content packages) tracked via xAPI. However, since it expects the learning experience
to send tracking data to the LRS, this either requires a content package using client side JavaScript to send the data (which is as insecure as SCORM), or requires 
server-side content (which cannot be produced using common authoring tools and is not portable). 

This specification is an attempt to take the best of both worlds such that portable client side content can be tracked by the TDS using server-side tracking and such that 
learner responses to assessments contained within that content can also be marked server-side. This is achieved by the content package providing a metadata file describing 
tracking and assessment marking behaviour for the TDS to follow. 

For example, the STAAPI metadata file describes what the correct answers are for a given question and how that question is scored etc. Then, when the content package 
communicates to the TDS that the learner has given a particular answer to that question, the TDS can then return appropriate feedback to be displayed to the learner, update 
the learner’s score, and record the attempt at the question with an xAPI statement stored in the Learning Record Store (LRS). 

## Components
### STAAPI Metadata File
A JSON file containing all the information required by the TDS to mark learner responses, record progress and track interactions appropriately. This file is included in the 
content package but kept private on the server.

### API Endpoint
The TDS will make available to the content package a RESTful API endpoint with appropriate permissions for that learner and session. The content package is responsible for 
communicating to that endpoint what the learner does (e.g. the learner selected option B in response to question 4) and then the TDS is responsible for interpreting what 
that action means (e.g. that the learner got that question right) based on the definition metadata, and for tracking the outcome to the LRS via xAPI. 

#### Assessment Marking
The content package is able to communicate to the TDS the learner’s response to a question. In response the content package gets back:
* Success (true/false).
* Score (for this question and current total).
* Feedback text appropriate for the option selected.

The TDS also tracks the learner’s response and the result to the LRS in an xAPI statement. 
#### Variables and Calculated Marking
The definition metadata file can also define variables that can be modified by the content via API request (if permitted in the metadata) or as a result of particular
learner responses. These variables and mathematical operations can be used in calculations for the purposes of marking questions where the correct answer may not be fixed.
#### Progress and Completion Tracking
STAAPI can also be used to facilitate server side progress and completion tracking via the following mechanism:
* The metadata file defines a number of milestones and how they contribute to completion. For example, in a simple course each slide might be a milestone all of which need 
to be completed in order to complete the course and which contribute equally to the completion percentage. 
* The content package communicates to the TDS when each milestone is completed.
* The TDS keeps a record of progress and returns a progress % to the content package. 
* The TDS also records appropriate xAPI data for progress and completion to the LRS. 

In this way, while the learner could still falsify a report of completing individual milestones to the TDS, overall completion is measured and tracked server-side by the TDS 
and the learner does not have access to information about which milestones are required for completion.
#### xAPI Tracking
In addition to automatic xAPI tracking relating to assessment marking and progress described above (which will be sent following the cmi5 xAPI profile), the content package 
can also include an xAPI Profile JSON-LD file which defines additional statements that can be sent. These statements can be triggered directly by an API request from the 
content package or as a result of assessment result or progress as defined in the STAAPI Metadata File.
## Related specifications
### Dependencies
* xAPI
* cmi5
* xAPI Profile Specification 
* [xAPI cmi5 Profile](https://github.com/adlnet/xapi-authored-profiles/tree/master/cmi5/v1.0)
### Other specifications in a similar space
* IMS QTI
