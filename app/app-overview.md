# Application Overview 


## Application Name 

**E-SMAT** 

---

## Application Type

Bilingual Microsoft Power Apps application for the standardized registration of safety and environmental inspections. 

---

## Business Context 

Different operational sites require different inspection methodologies, questionnaires, location structures, and languages. Developing an independent application for each site would increase maintenance effort, duplicate business logic, and make process standardization more difficult. 

E-SMAT was therefore designed as a single configurable application capable of adapting its behavior according to the selected site, inspected location, applicable methodology, and language. 

---

## Supported Inspection Methodologies 

### SMAT **Safety Management Audit Training** 

Used for inspections focused on occupational safety requirements and risk prevention practices. 

### EMAT **Environment Management Audit Training** 

Used for inspections focused on environmental requirements, operational controls, and environmental risk management. 

### ESMAT 

An integrated inspection methodology combining safety and environmental requirements in a single workflow. This methodology is used only in operational sites with specific integrated inspection requirements. 

---

## Supported Languages 

The application provides localized interfaces for: 
- Brazilian Portuguese (`pt-BR`); 
- Argentine Spanish (`es-AR`). 

The Argentine Spanish version supports the operational site located in Argentina. 

---

## Application Scope 

E-SMAT supports: 
- Six operational sites in Brazil and Argentina; 
- Site-specific inspection methodologies; 
- Different location hierarchies; 
- Different questionnaire structures; 
- Safety and environmental question categories; 
- Bilingual interface content; 
- Temporary inspection records; 
- Final inspection submissions. 

---

## Main User Journey 

The user starts the inspection by identifying the operational location. 

The application progressively filters the available options until the inspected location is fully identified. Based on the selected site and location, E-SMAT evaluates the applicable business rules and loads the appropriate SMAT, EMAT, or ESMAT questionnaire. 

During the inspection, the user can register responses and related information. The inspection may either be temporarily saved for later completion or validated and submitted as a final record. 

---

## Design Approach 

E-SMAT follows a configuration-driven approach. Instead of creating an independent flow for every site, the application uses reference data and business rules to control: 

- Available location options; 
- Interface language; 
- Inspection methodology; 
- Questionnaire selection; 
- Applicable questions; 
- Question order; 
- Required fields; 
- Navigation behavior; 
- Record status. 

This approach improves scalability, consistency, and maintainability. 

---

## Functional Components 

### Location Identification 

Collects the information necessary to identify where the inspection is being performed. 

### Cascading Filtering 

Restricts each location field according to the selections made in previous fields. 

### Questionnaire Routing 

Determines whether the inspection should use an SMAT, EMAT, or ESMAT questionnaire. 

### Dynamic Question Loading 

Retrieves the questions associated with the selected site, methodology, questionnaire version, and active status. 

### Localization 

Loads the appropriate interface content in Brazilian Portuguese or Argentine Spanish. 

### Temporary Save 

Stores an incomplete inspection so that the user can resume it later. 

### Final Submission 

Validates the required information and stores the inspection as a completed record. 

---

## Technical Components 
- Microsoft Power Apps; 
- SharePoint Lists; - Power Fx; 
- Configurable reference data; 
- Conditional navigation; 
- Cascading filters; 
- Dynamic galleries and forms; 
- Input validation; 
- Temporary and final record management. 

---

## Portfolio Representation 

The portfolio version focuses on the design decisions, application logic, data flow, and technical capabilities demonstrated by the project. Production packages, connections, requirements, records, and internal identifiers are not included.
