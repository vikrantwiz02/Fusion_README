# LMS (Lecture Management System) - Integration Guide

## Module Overview

The LMS module manages lecture/course content delivery, including course materials, attendance tracking, quizzes, assignments, and student evaluations.

Note: From this module, only the `Student_grades` table is used in production. Other tables (attendance, quizzes, assignments, forums, etc.) are NOT actively used in production.

---

## Tables (Sync with Production)

### Table: `online_cms_student_grades` (PRODUCTION TABLE)

This is the PRIMARY table for storing student course grades in Fusion ERP.

| Column | Type | Description |
|--------|------|-------------|
| `id` | integer | Primary Key |
| `semester` | integer | Semester number (1-8) |
| `year` | integer | Calendar year |
| `roll_no` | text | Student roll number |
| `grade` | text | Grade (A, A-, B+, B, B-, C+, C, C-, D, F) |
| `batch` | integer | Batch year |
| `course_id_id` | integer | Course reference |
| `reSubmit` | boolean | Resubmission flag |
| `remarks` | character varying | Remarks |
| `verified` | boolean | Grade verification status |
| `academic_year` | character varying | Academic year (2024-25) |
| `semester_type` | character varying | Odd/Even/Summer |

```sql
SELECT * FROM public.online_cms_student_grades ORDER BY id ASC
```

### Model Definition

```python
class Student_grades(models.Model):
    """
    PRIMARY GRADES TABLE: online_cms_student_grades
    
    This is the ONLY production-relevant table from online_cms module.
    All other tables (attendance, quizzes, etc.) are NOT in production use.
    """
    course_id = ForeignKey(Courses, on_delete=CASCADE)
    semester = IntegerField(default=1)
    year = IntegerField(default=2016)
    roll_no = TextField(max_length=2000)
    grade = TextField(max_length=2000)
    batch = IntegerField(default=2021)
    reSubmit = BooleanField(default=False)
    remarks = CharField(max_length=255, null=True)
    verified = BooleanField(default=False)
    academic_year = CharField(max_length=9, null=True)
    semester_type = CharField(max_length=20)
    
    class Meta:
        db_table = 'online_cms_student_grades'
```

---

## Dependencies - Tables to Sync With

### From `programme_curriculum` Module

| Table | Key Columns | Usage |
|-------|-------------|-------|
| `Course` | id, code, name, credit | Course reference for grades |

### From `academic_information` Module

| Table | Key Columns | Usage |
|-------|-------------|-------|
| `Student` | id (roll_no), batch_id | Student identification |

### From `globals` Module

| Table | Key Columns | Usage |
|-------|-------------|-------|
| `ExtraInfo` | id, user, user_type | User identification |

---

## Integration Functions

### Get Student Grades

```python
from applications.online_cms.models import Student_grades

def get_student_grades(roll_no):
    """Get all grades for a student by roll number"""
    return Student_grades.objects.filter(
        roll_no=roll_no
    ).order_by('year', 'semester')
```

### Get Verified Grades Only

```python
def get_verified_grades(roll_no):
    """Get only verified grades for a student"""
    return Student_grades.objects.filter(
        roll_no=roll_no, 
        verified=True
    ).order_by('year', 'semester')
```

### Get Course Grades

```python
def get_course_grades(course_id, semester, year):
    """Get all grades for a course in a semester"""
    return Student_grades.objects.filter(
        course_id=course_id, 
        semester=semester, 
        year=year
    )
```

### Get Semester Grades

```python
def get_semester_grades(roll_no, semester, academic_year):
    """Get grades for a specific semester"""
    return Student_grades.objects.filter(
        roll_no=roll_no,
        semester=semester,
        academic_year=academic_year
    )
```

---

## Common Queries

### Get All Grades for a Batch
```python
Student_grades.objects.filter(batch=2021, verified=True)
```

### Get Grades by Semester Type
```python
Student_grades.objects.filter(
    roll_no='21BCS001',
    semester_type='Odd'
)
```

### Count Students per Grade in a Course
```python
from django.db.models import Count

Student_grades.objects.filter(
    course_id=course_id,
    semester=semester,
    year=year
).values('grade').annotate(count=Count('grade'))
```

---

## Integration with Other Modules

| Module | How to Use Grades |
|--------|-------------------|
| Placement Cell | Filter students by CGPA calculated from grades |
| Scholarships | Verify academic performance for eligibility |
| Academic Procedures | Check prerequisites, backlog status |
| Examination | Grade submission and verification |
| Dashboards | Display student performance |

---

## Important Notes

1. Only `Student_grades` is used in production - Ignore other online_cms tables
2. Roll number is stored as text - Use exact string matching
3. Grades must be verified - Check `verified=True` for official grades
4. This table stores individual course grades - Not aggregate scores

---

## Tables NOT in Production Use

The following tables exist in online_cms but are NOT actively used:

- Modules, CourseDocuments
- AttendanceFiles, Attendance
- Quiz, QuestionBank, Question, QuizResult
- Assignment, StudentAssignment
- Forum, ForumReply
- GradingScheme
- Topics, Courses (local copy)

Do not integrate with these tables - they are legacy/unused.
