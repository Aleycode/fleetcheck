# Consistency checker for ambulance maintenance and mileage records

SEN0414 Advanced Programming · 2026-2027 Fall · Proposal · Method: BRSF software-development profile · Self-assessed grade 99.5/100

## Team, roles and work distribution

| Member | Student number | E-mail | Administrative roles | Development roles | Planned share |
| --- | --- | --- | --- | --- | --- |
| Aleyna Cebeci | 2100002803 | 2100002803@stu.iku.edu.tr | Team lead, Customer liaison, Work-log steward, Quality lead, Repository and release manager, Documentation lead | Requirements analyst, Architect, Knowledge engineer, Back-end developer, Test engineer, Release and operations | 100% |

### Modules and features

| Module or feature | Kind | What it does | Delivers epic | Responsible member | Second member |
| --- | --- | --- | --- | --- | --- |
| Export importer and graph builder | Module | Reads the vehicle, maintenance, fault and mileage-history exports from the Portal database with openpyxl and pandas, anonymises them and turns every row into RDF with RDFLib | Bring the Portal's vehicle records into one checkable graph | Aleyna Cebeci · 2100002803 |  |
| Rule set and validator | Module | Holds the SHACL shapes for the consistency rules and the SKOS scheme of fault types, runs pySHACL and collects every violation with its severity | Find and report every contradictory record | Aleyna Cebeci · 2100002803 |  |
| Report and correction proposal | Feature | Writes the violation report as HTML and CSV, the correction-proposal file and the PROV-O record of each run | Find and report every contradictory record | Aleyna Cebeci · 2100002803 |  |
| Early-warning panel | Feature | FastAPI page that shows violations per vehicle, the monthly trend and the vehicles whose periodic maintenance is due soon | Warn before a periodic maintenance is missed | Aleyna Cebeci · 2100002803 |  |

### Process leads

- Mission analysis: Aleyna Cebeci
- Stakeholder needs and requirements: Aleyna Cebeci
- System/software requirements definition: Aleyna Cebeci
- Architecture definition: Aleyna Cebeci
- Design definition: Aleyna Cebeci
- System analysis: Aleyna Cebeci
- Implementation: Aleyna Cebeci
- Integration: Aleyna Cebeci
- Verification: Aleyna Cebeci
- Transition: Aleyna Cebeci
- Validation: Aleyna Cebeci
- Operation: Aleyna Cebeci
- Maintenance: Aleyna Cebeci
- Disposal: Aleyna Cebeci

### Work log

2 entries; chain intact; head fingerprint 17aed2a9e422a40b5d9b479a73c74deb7d2ea50e8b24e44b4f97db5c35a7768d.

| No. | Time (UTC) | Member | Process | Hours | What | Evidence |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | 2026-10-17 18:43 | Aleyna Cebeci | Mission analysis | 2 | Chose the project topic, prepared the proposal draft in the Workbench and created the fleetcheck repository | https://github.com/Aleycode/fleetcheck |
| 2 | 2026-10-17 19:13 | Aleyna Cebeci | Stakeholder needs and requirements | 2 | Met the maintenance unit head, exported the September fault list from the Portal, computed baseline numbers and completed the proposal | https://github.com/Aleycode/fleetcheck |

## 1. Mission

### Customer

| Customer (organisation or role) | Contact person or role | How and when you reached them | Relation | What they get from it |
| --- | --- | --- | --- | --- |
| İstanbul İl Ambulans Servisi Başhekimliği, Araç Bakım Onarım Birimi (Vehicle Maintenance and Repair Unit) | Head of the vehicle maintenance and repair unit (bakım onarım birim sorumlusu) | Face-to-face at the unit office on 17 October 2026; the developer works with the unit daily | Owner of the system | an automatic check of the vehicle records its clerks enter into the Portal, a monthly violation report and an early warning for periodic maintenance, so corrections stop depending on one developer reading the database by hand |

### End users

| User group (short name) | Who exactly | What they do with the system | How often | Device or channel |
| --- | --- | --- | --- | --- |
| Maintenance clerk | the clerks of the vehicle maintenance unit who enter maintenance, fault and mileage records for the ambulance fleet into the Portal | uploads the Portal export, reads the violation list and corrects the records it names in the Portal | Weekly | office desktop, browser and Excel |
| Fleet supervisor | the supervisor of the vehicle maintenance unit who answers for the fleet's maintenance schedule | reads the monthly summary and the list of vehicles whose periodic maintenance is due | Monthly | office desktop browser |

### Problem

The vehicle maintenance unit of the İstanbul provincial health directorate records every fault, maintenance visit and mileage reading of the ambulance fleet in the Portal's maintenance module (Bakım Onarım). The module stores what is typed but checks nothing against the other records of the same vehicle. A vehicle's current mileage, the mileage written on a fault record and the mileage history are three separate fields, and they drift apart. Between 1 September and 1 October 2026 the unit entered 1,702 fault records for 536 of the fleet's 577 active vehicles. In 49 of those records the mileage written on the fault is lower than the mileage of the same vehicle's previous fault, so the vehicle's mileage runs backwards; 45 vehicles are affected, about 8 percent of the vehicles that had a fault that month. Two more records jump by over 30,000 km between consecutive faults. None of this is flagged by the module. Today an error is found only when the unit notices a wrong maintenance due date or a fault that cannot be opened because the stored mileage is too high, and it is corrected by a developer reading the database by hand, about 5 hours a month; the rest stay in the records. On 17 October 2026 the module's own dashboard shows 45 vehicles past their periodic maintenance mileage and 17 approaching it, with no warning sent to anyone. Each uncorrected error delays or hides a periodic maintenance, which for an ambulance means a vehicle that may be off the road when it is needed.

### Mission statement

Give the Maintenance clerk and the Fleet supervisor of the vehicle maintenance unit a tool that reads the Portal's vehicle export, checks every maintenance, fault and mileage record against a written set of consistency rules, lists each violation with the rule it breaks, and warns which vehicles are approaching their periodic maintenance, so that record errors are found the week they are made rather than when a maintenance is missed, and nobody has to read the database by hand.

Origin: Owner stated

### Outcome

A maintenance clerk uploads the weekly export and within a minute sees which records contradict each other and why; the fleet supervisor opens the panel and sees which ambulances are due for periodic maintenance in the next two weeks, without asking a developer to query the database.

## 2. Scope

### Areas

| Area | Where it lives | How big it is today |
| --- | --- | --- |
| Import and knowledge graph | importer/ and graph/ | four Portal tables, 1,702 fault records in September 2026 for 577 active vehicles, no checks today |
| Rules, report and correction proposals | rules/ and report/ | 49 backward-mileage records in September 2026, found by hand if at all |
| Panel and early warning | web/ | 45 vehicles past periodic maintenance and 17 approaching it on 17 October 2026; no forecast or warning today |

### Deliverables

| Deliverable | Area | Layer | Kind |
| --- | --- | --- | --- |
| Importer that reads the vehicle, maintenance, fault and mileage-history exports and anonymises plates and names | Import and knowledge graph | Product layer | Functional |
| Fleet ontology with a SKOS scheme of fault types and an RDF graph of all records | Import and knowledge graph | Product layer | Functional |
| SHACL rule set with violation and warning severities, checked with pySHACL | Rules, report and correction proposals | Product layer | Functional |
| Violation report (HTML and CSV), correction-proposal file and PROV-O record per run | Rules, report and correction proposals | Product layer | Functional |
| A monthly export of about 1,700 fault records checked in under one minute | Rules, report and correction proposals | Product layer | Non-functional |
| Web panel with violations per vehicle, monthly trend and periodic-maintenance forecast | Panel and early warning | Product layer | Functional |
| Test suite, continuous integration and tagged releases on GitHub | Rules, report and correction proposals | Work layer | Non-functional |

### Exclusions

| Left out | Why |
| --- | --- |
| Writing corrections back into the Portal database | the Portal is a live system used by the whole service; the tool proposes corrections and a clerk applies them in the Portal |
| Driver, patient and personnel data | the rules need only vehicle, mileage and date fields, and personal data would bring the KVKK duties into the project |
| Replacing the Portal's vehicle module or its screens | the unit keeps entering records where it does today; the tool runs beside the Portal on an export |
| Predicting faults from sensor or telematics data | the fleet has no telematics feed; the forecast uses only the mileage history already recorded |

### Domain entities

| Entity | What it is | Created? | Used? | Retired? |
| --- | --- | --- | --- | --- |
| Vehicle | an ambulance of the fleet, identified by an anonymised vehicle code, with its current mileage and periodic maintenance mileage | Created — no | Used — yes | Retired — no |
| Mileage reading | one dated mileage value of a vehicle from the mileage history | Created — no | Used — yes | Retired — no |
| Fault record | a reported fault of a vehicle with its date, mileage and fault type from the SKOS scheme | Created — no | Used — yes | Retired — no |
| Maintenance visit | a periodic or corrective maintenance of a vehicle with its date, mileage and the fault it repaired, if any | Created — no | Used — yes | Retired — no |
| Violation | one record that breaks one rule, with the rule, the severity and the values that contradict each other | Created — yes | Used — yes | Retired — yes |
| Check run | one execution of the checker on one export, with its date, rule-set version and PROV-O record | Created — yes | Used — yes | Retired — no |

### Project kinds

Data quality and validation › Shape-based validation service. Stack layers they need: Engineering practice, Reasoning, Validation.

### Semantic technologies, by stack layer

- Data model: RDFLib
- Engineering practice: SHACL continuous-integration gates
- Governance and compliance: KVKK personal-data regime
- Provenance and metadata: PROV-O provenance ontology
- Query: SPARQL 1.1 and 1.2
- Serialisation: RDFLib
- Storage and serving: Oxigraph
- Validation: SHACL shapes, pySHACL, SHACL continuous-integration gates
- Vocabularies and taxonomies: SKOS concept schemes

RDFLib turns each export row into RDF statements about vehicles, readings, faults and visits, so the four Portal tables become one graph in which a vehicle's records can be compared. SHACL shapes, checked with pySHACL, state each consistency rule once, in a form the unit can read: a mileage reading may not be lower than the one before it, a repair may not be dated before its fault, a periodic maintenance mileage that is passed without a visit is a warning. A SKOS scheme holds the fault types, so 'motor' and 'MOTOR' are one concept. SPARQL queries over an Oxigraph store produce the report tables and the per-vehicle mileage rate used by the forecast. PROV-O records which export, which rule-set version and which run produced every report, so a correction can always be traced. The KVKK regime sets what the importer must strip before anything reaches the repository, and SHACL continuous-integration gates run the rules on every commit.

## 3. Goals

| Goal | Facing | Covers area | Why this goal |
| --- | --- | --- | --- |
| Every contradictory maintenance, fault or mileage record is listed with the rule it breaks within a week of being entered | Mission-facing | Rules, report and correction proposals | this is the change the maintenance unit asked for: errors found when made, not when a maintenance is missed |
| The graph holds every vehicle, reading, fault and visit of the export, with each fault typed from the scheme | Scope-facing | Import and knowledge graph | a record missing from the graph is a record no rule can check |
| The supervisor sees which vehicles are due for periodic maintenance and the monthly violation trend without a developer | Scope-facing | Panel and early warning | the panel is what turns a report into a weekly habit of the unit |
| The tool never writes to the Portal and no personal data enters the graph, the reports or the repository | Containment-facing | Rules, report and correction proposals | the Portal is live and the KVKK regime applies; the checker must stay a reader |

### Specification

The system reads the vehicle, maintenance, fault and mileage-history exports of the Portal as Excel or CSV files, replaces plates and names with codes, and builds an RDF graph of vehicles, mileage readings, fault records and maintenance visits, with fault types taken from a SKOS scheme. It validates the graph against a SHACL rule set in which each rule has a severity: mileage going backwards, a fault mileage above the vehicle's current mileage, a repair dated before its fault, a missing fault type and a periodic maintenance mileage that has been passed without a visit. It writes a report that names each violating record, the rule, the severity and the contradicting values, writes a correction-proposal file for the clerk, and records the run in PROV-O with the export's hash and the rule-set version. It computes each vehicle's average daily mileage from the history and forecasts the date its periodic maintenance falls due. A web panel shows violations per vehicle, the monthly trend and the vehicles due within a chosen number of days. When an export is missing a table or a required column, the tool stops and names the missing part instead of checking a partial graph.

### Test scenarios

| Kind | Scenario (Given / When / Then) |
| --- | --- |
| Nominal | Given an export in which vehicle A017 has a mileage reading of 84,300 followed by a fault recorded at 83,900, when the clerk runs the check, then the report lists the fault record under the rule 'mileage may not go backwards' with both values |
| Rejection | Given an export in which the mileage-history table is missing, when the clerk runs the check, then the tool produces no report and says which table is missing |
| Boundary | Given a vehicle whose periodic maintenance mileage equals its current mileage exactly, when the check runs, then the vehicle appears as due today and not as overdue |
| Absent | Given a vehicle with no mileage reading at all, when the check runs, then the forecast for that vehicle says 'no history' instead of a date |

### Use cases

Maintenance clerk: upload export, read report, open a violation, download the correction proposal. Fleet supervisor: open panel, read monthly summary, read the due-soon list. Developer (me): add a rule, run the tests, tag a release.

## 4. Objectives

| Objective | Goal | Measured by | How exactly it is measured | Now | Target | Direction |
| --- | --- | --- | --- | --- | --- | --- |
| Contradictory records found within a week of entry | Every contradictory maintenance, fault or mileage record is listed with the rule it breaks within a week of being entered | Derived | violations found by the weekly run divided by the corrections the unit made in the same month, from the report files and the unit's correction list | 49 contradictory fault records in September 2026, found only when noticed | 100% of them found by the weekly run | Increase |
| Records of the export present in the graph | The graph holds every vehicle, reading, fault and visit of the export, with each fault typed from the scheme | Counted | rows in the four export tables compared with the subjects of the matching type in the graph, by a SPARQL count after each import | 0% (no graph today) | 100% | Increase |
| Hours a developer spends reading the database to find record errors | Every contradictory maintenance, fault or mileage record is listed with the rule it breaks within a week of being entered | Judged | hours per month recorded by the developer for manual correction work, before and after the tool | 5 hours a month | 1 hour a month | Decrease |
| Vehicles whose periodic maintenance is passed without a visit | The supervisor sees which vehicles are due for periodic maintenance and the monthly violation trend without a developer | Counted | vehicles in the monthly report whose current mileage exceeds their periodic maintenance mileage with no visit recorded | 45 vehicles on 17 October 2026 | 0 | Decrease |
| Personal data items and Portal writes | The tool never writes to the Portal and no personal data enters the graph, the reports or the repository | Counted | a regex scan of the repository, the graph and the reports for plates, names and identity numbers, and the absence of any database write statement in the code | 0 | 0 | Hold |

### Activity model

The clerk exports the four tables from the Portal and uploads them. The importer checks that every table and required column is present; if one is missing it stops and names it. Otherwise it anonymises plates and names, builds the graph and loads it into the store. The validator runs the SHACL rules; each violation is collected with its severity. The reporter writes the HTML and CSV report and the correction proposal, and records the run in PROV-O. The forecaster computes each vehicle's daily mileage rate and its due date. The panel reads the latest run and shows the summary; the clerk corrects the named records in the Portal and the next weekly run shows whether the violations are gone.

### Sequence model

The clerk sends the export files to the command-line tool. The tool asks the importer to parse and anonymise them and receives a graph. It sends the graph to the Oxigraph store, then asks pySHACL to validate the graph against the rule set and receives a validation report. It passes the report to the reporter, which queries the store with SPARQL for the contradicting values and writes the files. The reporter asks the provenance recorder to store the run. The panel, when opened by the supervisor, queries the store for the latest run, the per-vehicle counts and the forecast, and renders the page.

## 5. Backlog

Kind of development: Initial development

### Epics

| Epic | Pursues objective |
| --- | --- |
| Bring the Portal's vehicle records into one checkable graph | Records of the export present in the graph |
| Find and report every contradictory record | Contradictory records found within a week of entry |
| Warn before a periodic maintenance is missed | Vehicles whose periodic maintenance is passed without a visit |

### Stories

| Epic | As a | I want | So that | Acceptance (Given / When / Then) | Category | Design concern | Value 1–10 | Time 1–10 | Risk 1–10 | Size 1–10 | Priority |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Bring the Portal's vehicle records into one checkable graph | Maintenance clerk (end user) | to upload the four Portal exports in one step | I do not have to prepare anything by hand before the check | Given the four export files of a month, when I run the tool with the folder, then every row of every table is loaded and the tool prints how many vehicles, readings, faults and visits it read | New capability | Interface | 8 | 7 | 6 | 4 | 5.3 |
| Bring the Portal's vehicle records into one checkable graph | İstanbul İl Ambulans Servisi Başhekimliği, Araç Bakım Onarım Birimi (Vehicle Maintenance and Repair Unit) (customer) | plates, driver names and personnel names removed before anything is stored | the unit's data can be used without KVKK exposure | Given an export with plates and names, when the importer runs, then the graph and every report contain only vehicle codes and no plate, name or identity number | Compliance | Data | 7 | 8 | 9 | 3 | 8 |
| Find and report every contradictory record | Maintenance clerk (end user) | every record whose mileage goes backwards listed with both values | I can see at once which of the two readings is wrong | Given a vehicle whose fault mileage is lower than its previous reading, when the check runs, then the report lists that fault with the rule name, the previous reading and the fault mileage | New capability | Data | 10 | 8 | 7 | 5 | 5 |
| Find and report every contradictory record | Maintenance clerk (end user) | a repair dated before its fault and a fault without a type to be flagged | the two most common typing errors are caught the same week | Given a maintenance visit dated before the fault it repairs, when the check runs, then the report lists the visit under the date rule as a violation and lists any untyped fault as a warning | New capability | Data | 8 | 6 | 6 | 4 | 5 |
| Find and report every contradictory record | Maintenance clerk (end user) | a correction-proposal file I can follow in the Portal | I correct records without asking a developer | Given a report with violations, when the run finishes, then a CSV lists each record, the field to change and the proposed value, and the Portal itself is never written to | New capability | Interface | 9 | 6 | 5 | 4 | 5 |
| Find and report every contradictory record | İstanbul İl Ambulans Servisi Başhekimliği, Araç Bakım Onarım Birimi (Vehicle Maintenance and Repair Unit) (customer) | every report to say which export and which rule version produced it | a disputed correction can be traced months later | Given a finished run, when the supervisor opens its record, then it shows the export's hash, the rule-set version, the date and the number of violations | New capability | Architecture | 6 | 4 | 6 | 3 | 5.3 |
| Warn before a periodic maintenance is missed | Fleet supervisor (end user) | to see which vehicles are due for periodic maintenance in the next 14 days | I plan the workshop before a vehicle goes overdue | Given each vehicle's mileage history and periodic maintenance mileage, when I open the panel, then vehicles are listed with their forecast due date and those within 14 days are marked | New capability | User interface | 9 | 7 | 7 | 6 | 3.8 |
| Warn before a periodic maintenance is missed | Fleet supervisor (end user) | a monthly trend of violations per vehicle | I can see whether the unit's records are getting better | Given at least two monthly runs, when I open the panel, then it shows violations per month and the five vehicles with the most violations | New capability | User interface | 6 | 4 | 4 | 4 | 3.5 |

### Definition of done

| Criterion | How it is checked |
| --- | --- |
| All tests pass in continuous integration | GitHub Actions run on every push, green before merge |
| The sample export validates with no unexpected violation | pySHACL report of the sample attached to the run |
| The story's acceptance criterion is shown working | a pytest test named after the story, or a recorded run |
| No personal data in the repository | regex scan of the repository runs in continuous integration |

### Class model

Vehicle has a code, a current mileage, a periodic maintenance mileage and many MileageReadings, FaultRecords and MaintenanceVisits. MileageReading has a date and a value. FaultRecord has a date, a mileage and a FaultType from the SKOS scheme. MaintenanceVisit has a date, a mileage, a kind (periodic or corrective) and an optional repaired FaultRecord. Rule has an identifier, a text, a severity and a SHACL shape. Violation links one record to one Rule with the contradicting values. CheckRun has a date, the export hash, the rule-set version and many Violations; it is a PROV-O activity that used the export and generated the report. Forecast belongs to a Vehicle and has a daily rate and a due date.

### Test cases

| For story | Test case | Test data |
| --- | --- | --- |
| to upload the four Portal exports in one step | load the sample month and count vehicles, readings, faults and visits against the row counts | anonymised sample export of one month |
| plates, driver names and personnel names removed before anything is stored | run the importer on a raw sample and scan the graph and reports for plates, names and identity numbers | a sample with ten synthetic plates and names |
| every record whose mileage goes backwards listed with both values | a fault with a mileage lower than the previous reading appears in the report with both values | vehicle A017 with readings 84,300 and a fault at 83,900 |
| a repair dated before its fault and a fault without a type to be flagged | a visit dated before its fault is a violation and an untyped fault is a warning | one visit dated two days before its fault; one fault with an empty type |
| a correction-proposal file I can follow in the Portal | the correction CSV lists each violation with field and proposed value, and the code has no database write | the report of the sample month |
| every report to say which export and which rule version produced it | the run record holds export hash, rule version, date and violation count | one finished run |
| to see which vehicles are due for periodic maintenance in the next 14 days | vehicles within 14 days of their forecast due date are marked on the panel | three vehicles with known daily rates |
| a monthly trend of violations per vehicle | two monthly runs give a two-point trend and a top-five list | two sample months |

### Architecture — Component diagram

Component diagram: Importer (openpyxl, pandas) feeds Anonymiser (regex) feeds GraphBuilder (RDFLib); Store (Oxigraph) holds the graph; Validator (pySHACL) reads the rule set and the graph and emits violations; Reporter (Jinja2 templates, csv) and ProvenanceRecorder (RDFLib, PROV-O) write the outputs; Forecaster reads the store over SPARQL; Panel (FastAPI) reads the store and the latest run. The command-line entry point orders them; the Portal is outside the system and only its export files cross the boundary.

### Data — Class diagram

Design-level class diagram: Vehicle, MileageReading, FaultRecord, MaintenanceVisit, FaultType, Rule, Violation, CheckRun and Forecast as in the class model. VehicleShape requires a code and a periodic maintenance mileage; ReadingShape requires a date and a non-negative value; FaultShape requires a FaultType from the scheme; VisitShape requires a date not earlier than its repaired fault; CheckRun is a prov:Activity that used the export entity and generated the report entity.

### Interface — Component diagram

Component diagram of the boundaries: the command-line tool takes a folder with the four export files (vehicles, readings, faults, visits as .xlsx or .csv with named columns) and writes report.html, report.csv, corrections.csv and run.ttl into a dated folder; the panel exposes GET / for the summary, GET /vehicle/{code} for one vehicle's violations and forecast, and GET /runs/{id} for a run's provenance as JSON-LD. Nothing is written to the Portal.

### User interface — design model

Panel: a header with the date of the latest run and the violation count; a table of vehicles with code, violations, forecast due date and a 'due within 14 days' mark, sortable by each column; a monthly trend chart below; clicking a vehicle opens its violations with the rule text and the contradicting values. One page, readable on an office desktop browser without training.

## 6. Course-specific

### Python libraries and course topics

openpyxl and pandas read the Excel and CSV exports; regular expressions anonymise plates and names and scan the outputs; dataclasses and typing describe vehicles, readings, faults and visits; generators stream rows into the graph; a decorator times and logs each stage; a context manager opens and closes the Oxigraph store; concurrent.futures loads the four tables in parallel; argparse builds the command-line tool; logging writes the run log; pytest and GitHub Actions test every rule; RDFLib, pySHACL and Jinja2 build, validate and report the graph; FastAPI serves the panel. Course topics applied: object-oriented design, iterators and generators, decorators and context managers, regular expressions, concurrency, testing, packaging with pyproject and a command-line interface.

## Plan and risks

| Date | Deliverable | Lead |
| --- | --- | --- |
| 2026-10-30 | Importer with anonymisation, fleet ontology, SKOS fault scheme and first graph of a sample month, with tests | Aleyna Cebeci |
| 2026-11-13 | SHACL rule set with severities, violation report (HTML and CSV) and PROV-O run record | Aleyna Cebeci |
| 2026-11-27 | Correction-proposal file, forecast and web panel with trend and due-soon list | Aleyna Cebeci |
| 2026-12-07 | Pilot run on a real anonymised month with the maintenance unit; findings recorded | Aleyna Cebeci |
| 2026-12-15 | Tagged release v1.0, test report from continuous integration, user guide and lessons learned | Aleyna Cebeci |

| Risk | Likelihood | Impact | Mitigation |
| --- | --- | --- | --- |
| The export's column names or layout change between months | Medium | Medium | the importer checks columns by name and stops with the missing column's name; a test per table layout |
| A sample export still contains personal data | Medium | High | anonymisation runs before anything is written; a regex scan runs in continuous integration and blocks the merge |
| The maintenance unit has no time for the pilot in December | Medium | High | book the pilot date in the first meeting and keep the pilot to one hour with a prepared export |
| One person: illness or workload stops the project for a week | Medium | High | the panel and the trend are the last stories by priority; the core checker is finished by mid-November, and the last week of the term is left free |
| Mileage history too sparse for a usable forecast | Low | Medium | the forecast falls back to the fleet's average daily mileage and says so on the panel |

Repository: https://github.com/Aleycode/fleetcheck — Confirmed on GitHub 2026-10-17 (public, created 2026-10-17)

## References

1. W3C (2017). Shapes Constraint Language (SHACL). W3C Recommendation. https://www.w3.org/TR/shacl/
2. W3C (2014). RDF 1.1 Concepts and Abstract Syntax. W3C Recommendation.
3. W3C (2009). SKOS Simple Knowledge Organization System Reference. W3C Recommendation.
4. W3C (2013). PROV-O: The PROV Ontology. W3C Recommendation.
5. W3C (2013). SPARQL 1.1 Query Language. W3C Recommendation.
6. RDFLib documentation, https://rdflib.readthedocs.io; pySHACL, https://github.com/RDFLib/pySHACL
7. Sweigart, A. (2025). Automate the Boring Stuff with Python, 3rd ed. No Starch Press.
8. Kişisel Verilerin Korunması Kanunu (KVKK), Law No. 6698, Official Gazette 7 April 2016.

## Self-assessment

| Part | Weight | Grade | Criteria met |
| --- | --- | --- | --- |
| Team and work | 10% | 95/100 | 11 of 12 |
| Mission | 20% | 100/100 | 7 of 7 |
| Scope | 15% | 100/100 | 8 of 8 |
| Goal | 15% | 100/100 | 7 of 7 |
| Objective | 10% | 100/100 | 3 of 3 |
| Backlog | 20% | 100/100 | 10 of 10 |
| Course-specific | 5% | 100/100 | 1 of 1 |
| Plan and references | 5% | 100/100 | 3 of 3 |
| Overall | 100% | 99.5/100 |  |

### Member grades (Proposal)

Project grade 100 (60%), team grade 95 (20%), individual contribution (20%).

| Member | Student number | Individual | Member grade |
| --- | --- | --- | --- |
| Aleyna Cebeci | 2100002803 | 100 | 99 |

