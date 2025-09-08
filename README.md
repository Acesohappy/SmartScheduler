# SMART SCHEDULER

# Project summary

Build a system that automatically generates optimized weekly timetables for a school/college, assigning courses to timeslots and classrooms while respecting constraints (teacher availability, room capacity, course clashes, preferred times, equipment needs). Optionally add intelligence: clash-prediction, demand-aware room assignment, and a UI for manual edits.

---

# Goals (what it should do)

* Generate valid timetables that satisfy **hard constraints** (no teacher or student group double-booked, room capacity, required equipment).
* Optimize **soft constraints** (teacher preferred times, minimize gaps for students/teachers, balanced room usage).
* Allow manual adjustments and re-run optimization.
* Export timetables (PDF/CSV) and show interactive calendar UI per student/teacher/classroom.
* Optional: learn scheduling preferences from past timetables to improve scores.

---

# Core features (MVP → v2)

MVP:

1. Input management: add courses, teachers, rooms, batches/groups, constraints.
2. Auto-generate weekly timetable via an algorithm.
3. View timetables per batch, teacher, and room.
4. Basic conflict-checker and manual edit UI.

v1 / Advanced:

* Soft-constraint scoring and optimization (minimize gaps, respect preferences).
* Multiple schedule versions, comparison, rollback.
* Notifications (email) for changes.
* Export (PDF/CSV/iCal).
* ML: predict preferred times or detect likely conflicts based on historic edits.

---

# Tech stack suggestions

Frontend: React (or Next.js), FullCalendar for visual.
Backend: Node.js + Express (or Python Flask/FastAPI if you prefer Python for optimization).
DB: PostgreSQL (good for relations & constraints) or MongoDB.
Optimization:

* Simple: greedy / graph coloring (JS/Python).
* Advanced: Integer Linear Programming (ILP) with OR-Tools or PuLP (Python).
* Heuristics: Genetic Algorithm (GA), Simulated Annealing.
  Deployment: Docker, Heroku/AWS/GCP.

---

# Data model (core entities)

* Teacher(id, name, email, available\_timeslots\[], preferred\_times\[])
* Room(id, name, capacity, equipment\[])
* Course(id, name, required\_equipment\[], weekly\_hours, teacher\_id, batch\_id)
* Batch/Class (id, name, student\_count, required\_courses\[])
* Timeslot(id, day, start\_time, end\_time) — e.g. Mon 09:00-10:00
* TimetableEntry(id, course\_id, teacher\_id, room\_id, timeslot\_id)

Simple ERD idea:
Teachers —< Courses —> Batches
Rooms independent, assigned to TimetableEntry
Timeslots referenced by TimetableEntry

---

# Constraints (hard & soft)

Hard constraints (must hold):

* A teacher cannot have >1 class in the same timeslot.
* A batch/student group cannot have >1 class in same timeslot.
* Room capacity ≥ batch size.
* Room has required equipment for the course.
* Course weekly hours must be scheduled.

Soft constraints (optimize for):

* Respect teacher preferred times.
* Minimize gaps (free periods) for batches/teachers.
* Distribute classes evenly across days.
* Avoid back-to-back different rooms for a teacher (walking time).

---

# Algorithm ideas (practical approaches)

## 1) Greedy / Graph Coloring (fast MVP)

* Build conflict graph where nodes = course-session (each required hour block), edge when same teacher or same batch.
* Color graph with timeslot colors using greedy coloring ordered by degree.
* For each colored timeslot, assign a room that fits and has equipment. If none, try swapping or mark unschedulable.

Pros: Simple, fast.
Cons: May produce many soft-constraint violations.

## 2) ILP formulation (optimal but heavier)

Decision variables: x\_{c,t,r} ∈ {0,1} meaning course-session c at timeslot t in room r.

Objective: maximize Σ (weights \* soft\_constraints\_satisfaction)
Subject to:

* Σ\_{t,r} x\_{c,t,r} = required\_sessions(c) (each course gets needed sessions)
* For each teacher and timeslot, Σ\_{c,r} x\_{c,t,r} ≤ 1
* For each batch and timeslot, Σ\_{c,r} x\_{c,t,r} ≤ 1
* If room r lacks equipment or capacity < size, x\_{c,t,r}=0
  Use OR-Tools CBC or Gurobi (if available).

## 3) Genetic Algorithm / Simulated Annealing

* Represent a timetable as an array mapping sessions → timeslot+room.
* Define fitness = - (hard\_violations \* big\_penalty) - soft\_violation\_penalties.
* Run GA with crossover/mutation to improve fitness.

Pros: Good at multi-objective and large problem sizes.
Cons: Tuning required.

---

# Pseudocode — Greedy scheduler (simple)

```
sessions = expand each course into required sessions (courseId, teacherId, batchId)
sort sessions by descending degree (conflicts count)

for each session in sessions:
  for each timeslot in all_timeslots:
    if teacher free at timeslot and batch free at timeslot:
      find a room with capacity & equipment free at timeslot
      if found:
        assign session -> timeslot,room
        mark teacher/batch/room occupied
        break
  if not assigned:
    add to unscheduled list
```

---

# Sample APIs

* `POST /api/rooms` — add room.
* `POST /api/teachers` — add teacher/availability.
* `POST /api/courses` — add course info.
* `POST /api/schedule/generate` — run scheduler (body: strategy, params).
* `GET /api/schedule/{batch|teacher|room}/{id}` — fetch timetable.
* `POST /api/schedule/resolve` — attempt fixes for unscheduled entries.
* `PUT /api/schedule/entry/{id}` — manual edit.

---

# UI ideas

* Admin dashboard: forms to add teachers/rooms/courses.
* Visual timetable: FullCalendar with per-batch/teacher/room views.
* Conflict-highlighting overlay and suggestions (swap with X to resolve).
* One-click regenerate with different strategies or weights slider for soft constraints.

---

# Evaluation & metrics

* Hard violations count (should be 0).
* Soft score: weighted sum (teacher prefs satisfied, avg gaps per teacher, etc.).
* % scheduled classes vs total required.
* Time to generate schedule.

---

# Sample timeline (2–6 weeks)

Week 1: Data model + CRUD for teachers/rooms/courses + small static UI.
Week 2: Implement greedy scheduler + display timetables + basic conflict check.
Week 3: Add room assignment improvements, export feature, manual edit UI.
Week 4: Implement scoring, soft constraints, and allow strategy selection.
Week 5–6: Add ILP/GA backend, notifications, auth, deploy.

---

# Extensions / Smart ideas

* Use past manual edits to learn teacher preferences (ML classification/regression for preferred slot probability).
* Auto-suggest swaps when admin edits a slot (search small neighborhood for better solution).
* Support multiple campuses/timezones.
* Mobile-friendly timetable per student/teacher with push notifications.

---

# Quick repo README snippet (starter)

```md
# Smart Timetable Scheduler

MVP: auto-generate weekly timetables for batches, teachers, rooms with hard constraint enforcement.

Features:
- CRUD for teachers, rooms, courses
- Greedy scheduler (fast)
- Timetable views per teacher/batch/room
- Manual edit + re-run scheduler

Tech: React + Node/Express + PostgreSQL
Run: `docker-compose up` (backend + postgres)
API: POST /api/schedule/generate to auto-generate timetable
```

---

