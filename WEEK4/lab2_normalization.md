# VUA-SEN307 Week 4: Normalization of the Registration Sheet

Name: Ogah Christopher Matric No: VUG/SEN/24/11932

## 0. Unnormalized table

| matric_no | student_name | student_dept | course_codes | course_titles | lecturers | lecturer_depts | session |
|---|---|---|---|---|---|---|---|
| 20231234 | Ada Obi | Software Engineering | SEN307, SEN301 | Database Systems, Software Architecture | Dr A. Okafor, Dr M. Bello | Software Eng., Software Eng. | 2026/2027 |
| 20231235 | John Musa | Computer Science | SEN307, COS301 | Database Systems, Operating Systems | Dr A. Okafor, Mrs F. Eze | Software Eng., Computer Sci. | 2026/2027 |
| 20231236 | Grace Okon | Software Engineering | SEN301 | Software Architecture | Dr M. Bello | Software Eng. | 2026/2027 |

Problem: several cells hold more than one value, and the same facts are repeated.

## 1. First Normal Form (1NF)

**Table: Registration**  PK = (matric_no, course_code, session)

| matric_no | student_name | student_dept | course_code | course_title | lecturer | lecturer_dept | session |
|---|---|---|---|---|---|---|---|
| 20231234 | Ada Obi | Software Engineering | SEN307 | Database Systems | Dr A. Okafor | Software Eng. | 2026/2027 |
| 20231234 | Ada Obi | Software Engineering | SEN301 | Software Architecture | Dr M. Bello | Software Eng. | 2026/2027 |
| 20231235 | John Musa | Computer Science | SEN307 | Database Systems | Dr A. Okafor | Software Eng. | 2026/2027 |
| 20231235 | John Musa | Computer Science | COS301 | Operating Systems | Mrs F. Eze | Computer Sci. | 2026/2027 |
| 20231236 | Grace Okon | Software Engineering | SEN301 | Software Architecture | Dr M. Bello | Software Eng. | 2026/2027 |

**Fix:** I split the multi-valued cells so every cell holds one value, giving each student–course pair its own row, and chose the composite key (matric_no, course_code, session) because no single column is unique.

## 2. Second Normal Form (2NF)

Partial dependencies on the composite key:

- `matric_no` → `student_name`, `student_dept` (depends only on the student)
- `course_code` → `course_title` (depends only on the course)
- (`course_code`, `session`) → `lecturer`, `lecturer_dept` (depends on the course and session, not on the student)

**Student** (matric_no PK, student_name, student_dept)

| matric_no | student_name | student_dept |
|---|---|---|
| 20231234 | Ada Obi | Software Engineering |
| 20231235 | John Musa | Computer Science |
| 20231236 | Grace Okon | Software Engineering |

**Course** (course_code PK, course_title)

| course_code | course_title |
|---|---|
| SEN307 | Database Systems |
| SEN301 | Software Architecture |
| COS301 | Operating Systems |

**Course_Lecturer** (course_code, session, lecturer PK together; lecturer_dept)

| course_code | session | lecturer | lecturer_dept |
|---|---|---|---|
| SEN307 | 2026/2027 | Dr A. Okafor | Software Eng. |
| SEN301 | 2026/2027 | Dr M. Bello | Software Eng. |
| COS301 | 2026/2027 | Mrs F. Eze | Computer Sci. |

**Enrolment** (matric_no, course_code, session PK together)

| matric_no | course_code | session |
|---|---|---|
| 20231234 | SEN307 | 2026/2027 |
| 20231234 | SEN301 | 2026/2027 |
| 20231235 | SEN307 | 2026/2027 |
| 20231235 | COS301 | 2026/2027 |
| 20231236 | SEN301 | 2026/2027 |

**Fix:** I moved each attribute that depended on only part of the key into its own table, so student details, course details and lecturer assignments are stored once instead of on every enrolment row.

## 3. Third Normal Form (3NF)

Transitive dependencies:

- `student_dept` describes a department, not the student. The same department is even spelled two ways ("Software Engineering" vs "Software Eng."), and renaming a department would mean editing many rows.
- `lecturer_dept` depends on the lecturer, not on the course/session (lecturer → lecturer_dept).

So I created **Department** and **Lecturer** tables, replaced text names with surrogate keys, and used foreign keys.

**Department** (dept_id PK, dept_name)

| dept_id | dept_name |
|---|---|
| 1 | Software Engineering |
| 2 | Computer Science |

**Lecturer** (lecturer_id PK, full_name, dept_id FK)

| lecturer_id | full_name | dept_id |
|---|---|---|
| 1 | Dr A. Okafor | 1 |
| 2 | Dr M. Bello | 1 |
| 3 | Mrs F. Eze | 2 |

**Student** (student_id PK, matric_no, full_name, dept_id FK)

| student_id | matric_no | full_name | dept_id |
|---|---|---|---|
| 1 | 20231234 | Ada Obi | 1 |
| 2 | 20231235 | John Musa | 2 |
| 3 | 20231236 | Grace Okon | 1 |

**Course** (course_id PK, course_code, course_title)

| course_id | course_code | course_title |
|---|---|---|
| 1 | SEN307 | Database Systems |
| 2 | SEN301 | Software Architecture |
| 3 | COS301 | Operating Systems |

**Course_Lecturer** (course_id FK, lecturer_id FK, session; PK together). This records which lecturer teaches which course in which session.

| course_id | lecturer_id | session |
|---|---|---|
| 1 | 1 | 2026/2027 |
| 2 | 2 | 2026/2027 |
| 3 | 3 | 2026/2027 |

**Enrolment** (enrolment_id PK, student_id FK, course_id FK, session)

| enrolment_id | student_id | course_id | session |
|---|---|---|---|
| 1 | 1 | 1 | 2026/2027 |
| 2 | 1 | 2 | 2026/2027 |
| 3 | 2 | 1 | 2026/2027 |
| 4 | 2 | 3 | 2026/2027 |
| 5 | 3 | 2 | 2026/2027 |

**Fix:** I removed non-key columns that depended on other non-key columns by moving department details into their own table and linking with foreign keys, so a department rename is now a single-row update.

## 4. Final schema

```
Department (dept_id PK, dept_name)
Student (student_id PK, matric_no, full_name, dept_id FK → Department)
Lecturer (lecturer_id PK, full_name, dept_id FK → Department)
Course (course_id PK, course_code, course_title)
Enrolment (enrolment_id PK, student_id FK → Student, course_id FK → Course, session)
Course_Lecturer (course_id FK → Course, lecturer_id FK → Lecturer, session; PK = course_id + lecturer_id + session)
```

## 5. Relationships

- Department 1 : N Student
- Department 1 : N Lecturer
- Student M : N Course, resolved by **Enrolment**
- Course M : N Lecturer, resolved by **Course_Lecturer**
