# Design Document: Old System Problems and New Solutions Integration

Documenting and integrating the 4 core architectural issues of the old system along with their solutions in the new LMS AI system.

## Proposed Layout: Option B (Integrated into Section 1)
As selected, the problems and solutions will be placed inside Section 1 (Overview) of the interactive dashboard `index.html` as a structured grid subsection, and also added to `so-do-chuc-nang-lms-ai.md`.

---

## 1. Specification of the 4 Issues & Technical Solutions

### Issue 1: Mixed Students from Different Programs in One Class
* **Problem**: Students from standard, high-quality, international, or retake programs attend the same class sessions. The old system binds scoring weights and curriculum directly to the physical Class entity, preventing different grading scales.
* **Technical Solution**:
  - Decouple **Class Session** (`ClassRoom`/`ClassSection`) from **Student Curriculum** (`StudentCurriculum`).
  - Calculate grades and check progress by joining the student's personal enrollment back to their specific `ProgramType` weight config instead of relying on class-level coefficients.

### Issue 2: Persistence of Class Historical Data After Each Course
* **Problem**: When a class completes a module and transitions to a new one, stats and the lecturer's identity are overwritten or lost.
* **Technical Solution**:
  - Introduce a historical ledger table `class_course_history`.
  - Upon course completion, write a read-only snapshot containing: `class_id`, `course_id`, `lecturer_id`, `semester_id`, `final_average_gpa`, `attendance_rate`, and completion timestamps.

### Issue 3: Retake and Transfer Students Following Correct Curriculums
* **Problem**: Students transfer programs (e.g., standard to high-quality) or retake courses and must adhere strictly to their curriculum rules, which is difficult to enforce when courses change.
* **Technical Solution**:
  - Implement a dynamic equivalents mapping `course_equivalents`.
  - Maintain a student-specific `IndividualStudyPlan` which tracks progress against the active mapped curriculum and raises visual alerts if a student enrolls in a course outside their program.

### Issue 4: Reconciling Subject Equivalence Across Different Program Cohorts (K24 vs K25)
* **Problem**: Course C for K24 (ID: 101) and Course C for K25 (ID: 202) are technically separate rows in the database, making automatically calculating retake replacement grades fail.
* **Technical Solution**:
  - Establish a global **Common Base Subject** (`base_subject_id`).
  - Both Course 101 and Course 202 reference the same `base_subject_id`.
  - The GPA engine aggregates grades by `base_subject_id` to automatically execute grade substitution (retake grade replaces previous failing grade).

---

## 2. Proposed Changes

### [MODIFY] [index.html](file:///c:/Users/ADMIN/Desktop/LMS_AI/index.html)
- Add a new layout block inside `<section id="tong-quan">` representing the **Khảo sát hiện trạng & Vấn đề cốt lõi** section.
- Use a 2-column or grid layout with glassmorphism styling, showing problems with high-contrast red indicator and the corresponding solution in green.

### [MODIFY] [so-do-chuc-nang-lms-ai.md](file:///c:/Users/ADMIN/Desktop/LMS_AI/so-do-chuc-nang-lms-ai.md)
- Insert a new section **1.2 Các vấn đề tồn tại ở hệ thống cũ & Giải pháp cải tiến** right under Section 1.

---

## 3. Verification Plan
- Deploy the update to Vercel and review in the browser.
- Verify the layout responsiveness and alignment under Section 1.
