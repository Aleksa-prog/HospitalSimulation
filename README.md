TIES481 — SIMULATION (2023)
UNIVERSITY OF JYVÄSKYLÄ

Workshop 2
by Oleksandra Ivanova, Shaswato Sarker and Tasnim Zahan

Google doc link: https://docs.google.com/document/d/14nUtUIbizoJPDYQ6YP7-S2c91f2VppOMQytv_JcTg78/edit

Goal: to try and implement the simulation of the hospital operation process, using the process-based method and programming language Python. To simulate patient processing in a hospital setting, focusing on patient flow through various phases and resource utilization.

I. DEFINITIONS
Key Components: Patients, Queue, Phases, Resources, Process Monitor.
Processes: Patient, Queue (Q), Monitor
Patient Process: Manages the journey of patients through different phases.
Queue Process: Handles the generation and scheduling of patients.
Process Monitor: Oversees the coordination between different phases and resources, managing timings and patient flow.
Phases for Patient: in P, in OT, in R
Resources: Preparatikn room (P), operation theater (OT), restoration room (R).
Throughput time — the total amount of time the patient is in the system from being generated in Queue until termination (so after termination we should have the whole time).
Facility time — actual amount of time the patient was processed in each facility phase in the life cycle (ex. : the time that patient was being prepared in the P).
Scheduled patients — patients with priority 1, they are considered non-dangerous ones, in other words they planned their operations with the doctors and are in Q (timetable) from the start.
Emergency patients — patients with priority 2, that weren’t scheduled to the Q (timetable); they never planned to have an operation, but due to some accident they have to get it. There appear randomly in the Q and go right to the start of it.

II. IMPLEMENTATION DESCRIPTION
1. GENERAL IDEA DESCRIPTION
For the ease of implementation and trying to do the most realistic thing, we decided to change the patient generation idea. From now on we will treat the Queue (Q) as a timetable of scheduled operation appointments (not “scheduled” state, but real life planned medical operations on patients (surgery, for instance), that have been planned by the patients with doctors beforehand. If we treat it like a real life example, imagine that first patient will have their operation at 8 a.m., the second — at noon and the third one at 4 p.m. and so on. With that, we will treat our whole stream as if it was a flow of only one medical group working, because there is only 1 OT (operation theater, where we can operate at once on only one patient).
Treating the Q as a timetable, we can ensure that each patient, which we treat as a process, will know how much time they need to spend in each facility. This values will be given in certain ranges — for example, for preparation from 10 minutes to 1 hour, for operation from 1 hour to 8 hours and so on. 
Treating the Q as a timetable also helps us with how our patients will be generated — instead of always monitoring each of them to create a new one, we will generate blocks of patients at once (as of now it is 5 patients at on time, but in the future we can randomly try to generate one time 5, other time 4 and so on). This means that we will simulate a day for the operation team (doctors, etc.). What this means is that one block of patients can be seen in one day, so if there are 5 patients in a block, this medical team will operate on 5 scheduled patients that day. Only when the last of these 5 patients goes to a restoration room (phase), the new block of patients will be created. That means that the throughput time counting will not be starting in the Queue stage (seeing the Queue as a timetable, the patients that are in the Queue are basically at home), but rather in the preparation room, so as to have good, useful data.
While doing this task we found two main bottlenecks in this assignment — first was the condition for generating new patients, which we explored earlier, and the second one was the condition of when we can let other patients go to the preparation room and not let them wait there for OT to be available. The problem with this is that in reality the whole flow of preparation-operation-restoration must happen smoothly for a patient to be well in the end. When a patient od in the preparation room, they are prepared for the operation (surgery, for instance), so they are being filled with necessary medications for them not to feel pain and not to be conscious during the operation. But those medications have their length of time before they stop working, and giving another dose is a risk of harming a patient — for example, leading them into a coma — rather than helping them. With this we must ensure that patients NEVER are left waiting in any facility, because that will endanger them. For this to work, we have made a condition, that a new patient can enter the preparation room, ONLY if there's room to enter (we have two Ps for scheduled patients) and if the sum of preparation time and OT time of the patient that occupying one of the Ps is less or equal to the preparation time of the patient, who wants to enter the P, ONLY them they can enter the preparation room. With this method we can absolutely ensure that all the scheduled patients will go through the flow preparation-operation-restoration smoothly. This strategy has its flaw in a face of the blocking time that appears while doing the sum of P and OT times. P time starts ticking when entering the P room, so potentially there can be blocking time created, if some P time has already ticked out when the next patient wants to enter. But this strategy still appeared to be the winning one in our efforts to plan and implement this simulation. This process will be watched by the Monitor Process.
In this implementation we have 2 preparation rooms for scheduled patients, 1 operation theater and an infinite amount of restoration rooms.
2. PATIENTS GENERATOR (QUEUE) PROCESS
Active state of Q: The Q is technically a timetable. It generates patients in batches of 5 (where 5 is n that we can configure) ONLY and there are scheduled patients (priority 1). After generating the batch Q comes to its scheduled state. 
3. MONITOR PROCESS
Function: Monitors and calculates times between facilities, deciding when new patients should enter preparation.
Logic: Manages the flow of patients based on operation and preparation times.
Role: Ensures efficient utilization of resources and minimizes blocking time.
What it actually does is: calculate throughput time for each lifecycle, detects when to generate a new batch of patients.
It calculates times between facilities and when to put new patients into preparation. The Q generates 5 patients. The first patient (p1) immediately goes into P and we don’t let anybody in, unless the operation time in OT + P time for p1 is less than P time for next patient (p2). Otherwise, only let p2 into P when the remaining OT time for p1 is less than P time of p2.
4. PATIENT PROCESS
4.1 Patient Generation (Done in Queue Process):
— Trigger: New patient list creation (by Queue Process, triggered by Monitor) when Q is empty and OT is available.
— Initialization: Expected times for P, OT, R set to 0 initially (in classes, not instances).
— Process: Generate patients with random time values for P, OT, R within specified ranges. Timer starts upon the patient going to preparation and stops at termination after recovery.
— Patient Process Phases / States: Represented by a dictionary of states (e.g., 1: "active").
5. QUEUE PROCESS
— Active State: Q functions as a timetable, generating patients in batches (configurable size).
— Priority: Scheduled patients have priority 1, emergency patients have priority 2, are initialized randomly, NOT in the Q and way more rarely.
— Scheduling: After generating a batch, Q comes to a scheduled state.
III. PROGRAMMING STRUCTURE
Main File: Initiates the simulation, calls configuration.
Patients File: Contains the patient generator.
Facilities Files: Separate files for P, OT, R.
Monitor: Manages throughput time and compiles statistics.
Config: Location to set variable values.
Visualization: Console output for state changes and documentation.
This comprehensive structure should provide a clear framework for the development and implementation of the simulation. The inclusion of the "Process Monitor" as a distinct process emphasizes its importance in maintaining the efficiency and effectiveness of the patient flow within the hospital simulation model.
1. PSEUDOCODE
1.1 Generation of patients (Q-process)
We have a class ‘Patient’ along with these initial variables: P time, R time, OT time, Throughput time and Priority.
1. Queue generates 5 patient instances of ‘Patient’ class with priority 1 (queue becomes full).
2. Initialize all variables (P, OT, R times) with random times except Throughput time which is equal to zero.
1.2 Process Monitor
The Monitor comes into action. It goes through a set of conditions: 
IF P1 AND P2 AND OT are empty, then a patient goes to P. 
ELIF P1 OR P2 is empty then, 
SUM = sum(P_timepatient1, OT_timepatient1 ) 
IF P_timepatient2 >= SUM then patient2 goes to available preparation facility.
ELSE patient2 stays in Q.
When a patient enters P, the monitor starts to count throughput time.
1.3 In Preparation Phase (P)
1. When a patient enters P, preparation time for that patient is monitored.
2. After P time is over, 
IF  OT is available the patient goes to OT
ELSE patient stays in P 
1.4 In OT Phase
1. When a patient enters OT, OT time for that patient is monitored.
2. After OT time is over, the patient goes to R.
We are assuming that there are infinitely many recovery facilities, for that no need to check for availability.
1.5 In Recovery Phase (R)
1. When a patient enters R, R time for that patient is monitored.
2. After R time is over, the patient goes to the termination phase.
1.6 Termination
1. Throughput time calculation is triggered.
2. Patient process is terminated.
