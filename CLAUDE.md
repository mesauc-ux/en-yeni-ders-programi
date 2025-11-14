# CLAUDE.md - AI Assistant Guide

**Project**: Özel Ders Programı Yönetim Sistemi (Private Course Schedule Management System)
**Language**: Turkish (TR)
**Framework**: Flask (Python)
**Last Updated**: 2025-11-14

## 📋 Project Overview

This is a comprehensive course scheduling system designed for private tutoring centers. It manages teachers, students, schedules, and automatically generates conflict-free 4-week course schedules with intelligent slot allocation.

### Key Features
- **Teacher Management**: Add/edit/delete teachers with availability schedules and blocked time slots
- **Student Management**: Manage students with priorities, restrictions, and manual lesson assignments
- **Smart Schedule Generation**: 4-week automatic scheduling with conflict detection
- **Class Lessons**: Pre-defined group lessons for entire classes
- **Conflict Detection**: Multi-level conflict checking with auto-fix suggestions
- **Export Capabilities**: Excel, HTML, and PDF export functionality
- **Schedule History**: Save, load, and manage multiple schedule versions
- **Dark Mode**: Modern UI with light/dark theme support

## 🏗️ Architecture

### Single-File Application Structure
This is a **monolithic single-file Flask application** (`flask_app.py`, ~17,660 lines) that includes:
- Backend logic (Flask routes and business logic)
- Frontend UI (embedded HTML template with CSS and JavaScript)
- Database schema and initialization

### File Structure
```
en-yeni-ders-programi/
├── flask_app.py          # Main application (all-in-one)
├── ders_programi.db      # SQLite database
├── .git/                 # Git repository
└── CLAUDE.md            # This file
```

## 🗄️ Database Schema

### Tables

#### 1. `teachers`
- `id`: Primary key
- `name`: Teacher first name
- `surname`: Teacher last name
- `branch`: Subject/branch (e.g., Matematik, İngilizce)
- `schedule`: JSON array of availability (day, start_time, end_time)
- `blocked_slots`: JSON array of blocked time slots

#### 2. `students`
- `id`: Primary key
- `name`: Student first name
- `surname`: Student last name
- `class`: Student's class/grade
- `restrictions`: JSON - time restrictions/unavailable slots
- `priorities`: JSON - priority teachers/subjects
- `manual_lessons`: JSON - manually assigned lessons
- `teacher_blocks`: JSON - teachers to avoid

#### 3. `saved_schedules`
- `id`: Primary key
- `name`: Schedule name
- `created_at`: Timestamp
- `schedule_data`: JSON - complete schedule
- `teachers_snapshot`: JSON - teacher data at save time
- `students_snapshot`: JSON - student data at save time
- `start_date`: Schedule start date

#### 4. `class_lessons`
- `id`: Primary key
- `class_name`: Class identifier
- `teacher_id`: Foreign key to teachers
- `day`: Day of week
- `start_time`: Lesson start time
- `end_time`: Lesson end time
- `weeks`: JSON - which weeks this lesson applies to
- `is_group`: Boolean - group lesson flag
- `is_forced`: Boolean - force schedule flag
- `created_at`: Timestamp

## 🔧 Core Functions

### Database Functions
**Location**: Lines 14-120

- `get_db()`: Returns SQLite connection with row factory
- `init_db()`: Initializes all tables and adds missing columns

### Schedule Generation
**Location**: Lines 14012-14536

#### `create_four_week_schedule(teachers, students, class_lessons=[])`
The main scheduling algorithm that:
1. Processes manual lesson assignments
2. Applies class lessons to all students in the class
3. Distributes lessons across 4 weeks
4. Prevents conflicts:
   - No double-booking (same student/teacher at same time)
   - Daily limit: Max 1 lesson per teacher per student per day
   - Respects student restrictions and teacher blocked slots
   - Priority lessons: 2 per week, normal lessons: 1 per week

**Key Algorithm Features**:
- Priority-based assignment (processes prioritized teachers first)
- Slot tracking to prevent overlaps
- Daily teacher-student limits
- Branch distribution tracking (e.g., Math lessons per week)

### Conflict Detection
**Location**: Lines 15194-15772 (v1), 15772-16004 (v2)

#### `detect_all_conflicts(schedule_data, teachers, students)`
Detects various conflict types:
- **Teacher conflicts**: Same teacher, multiple students at same time
- **Student conflicts**: Same student, multiple lessons at same time
- **Restriction violations**: Student lessons during restricted times
- **Teacher unavailability**: Lessons outside teacher's schedule
- **Blocked slots**: Lessons in teacher-blocked time slots

#### `detect_conflicts_v2(schedule_data, teachers, students)`
Enhanced version with:
- Same conflict checks as v1
- Improved performance
- Better conflict categorization

### Time Utilities
**Location**: Lines 15176-15194

- `time_to_minutes(time_str)`: Converts "HH:MM" to minutes
- `check_time_overlap(start1, end1, start2, end2)`: Checks if time ranges overlap

## 🛣️ API Routes

### Core CRUD Operations

#### Teachers
- `POST /add_teacher`: Add new teacher
- `POST /update_teacher`: Update teacher details
- `POST /delete_teacher`: Delete teacher
- `GET /get_teachers`: Fetch all teachers

#### Students
- `POST /add_student`: Add new student
- `POST /update_student`: Update student details
- `POST /delete_student`: Delete student
- `GET /get_students`: Fetch all students

#### Class Lessons
- `POST /save_class_lesson`: Create class lesson
- `GET /get_class_lessons`: Fetch all class lessons
- `POST /update_class_lesson`: Update class lesson
- `POST /delete_class_lesson/<lesson_id>`: Delete class lesson

### Schedule Operations
- `GET /generate_schedule`: Generate 4-week schedule
- `POST /save_current_schedule`: Save current schedule
- `GET /get_saved_schedules`: List saved schedules
- `GET /load_schedule/<schedule_id>`: Load saved schedule
- `POST /delete_schedule/<schedule_id>`: Delete saved schedule
- `POST /rename_schedule/<schedule_id>`: Rename saved schedule

### Conflict Management
- `POST /check_conflicts`: Check for conflicts (v1)
- `POST /check_conflicts_v2`: Enhanced conflict checking
- `POST /suggest_alternative_slots`: Suggest conflict resolutions
- `POST /auto_fix_conflicts`: Automatically fix conflicts
- `POST /swap_lessons`: Swap two lessons

### Export Features
- `GET /export_excel`: Export schedule to Excel
- `GET /export_html`: Export schedule to HTML
- `GET /export_weekly_pdf_server`: Export single week to PDF
- `GET /export_all_weeks_pdf_server`: Export all weeks to PDF
- `POST /export_conflict_report`: Export conflict report

### Timeline Views
- `GET /get_teacher_timeline`: Get teacher's weekly schedule
- `GET /get_student_timeline`: Get student's weekly schedule

### Utility Routes
- `GET /`: Main page (renders HTML template)
- `GET /get_unique_classes`: Get list of all classes
- `GET /get_students_by_class`: Get students in a class

## 🎨 Frontend Structure

The frontend is embedded in the `HTML_TEMPLATE` variable (lines 125-13417).

### Technologies Used
- **Vanilla JavaScript**: No framework dependencies
- **CSS3**: Modern styling with CSS variables
- **Responsive Design**: Mobile-friendly interface
- **Dark Mode**: Theme switching capability

### Key UI Components
1. **Navigation Tabs**: Teacher management, student management, schedule view, etc.
2. **Data Tables**: Display teachers, students, schedules
3. **Forms**: Add/edit teachers and students with validation
4. **Schedule View**: 4-week calendar grid view
5. **Conflict Viewer**: Visual conflict detection and resolution interface

### Important CSS Variables (Line 144-168)
```css
--primary-color: #667eea
--success-color: #10b981
--warning-color: #f59e0b
--danger-color: #ef4444
```

## 🔑 Key Conventions

### Turkish Language
- All UI text is in Turkish
- Database content uses Turkish characters (UTF-8)
- Days of week: Pazartesi, Salı, Çarşamba, Perşembe, Cuma, Cumartesi, Pazar

### Time Format
- 24-hour format: "HH:MM" (e.g., "09:00", "14:30")
- Time ranges: "HH:MM-HH:MM" (e.g., "09:00-10:00")

### JSON Data Structures

#### Teacher Schedule
```json
[
  {
    "day": "Pazartesi",
    "start_time": "09:00",
    "end_time": "17:00"
  }
]
```

#### Student Restrictions
```json
[
  {
    "day": "Salı",
    "start_time": "14:00",
    "end_time": "16:00"
  }
]
```

#### Manual Lessons
```json
[
  {
    "week": 1,
    "day": "Pazartesi",
    "teacher_id": 5,
    "time": "10:00-11:00"
  }
]
```

### Day Ordering
Days are sorted using this order:
```python
day_order = ['Pazartesi', 'Salı', 'Çarşamba', 'Perşembe', 'Cuma', 'Cumartesi', 'Pazar']
```

## 🚀 Development Workflow

### Starting the Application
```bash
python flask_app.py
```

Default runs on `http://localhost:5000` (check the file for actual host/port config).

### Database Initialization
Database auto-initializes on application startup via `init_db()` at line 121.

### Making Changes

#### When Modifying Backend Logic:
1. **Locate the relevant function** using the function list in this doc
2. **Read surrounding context** - functions are interconnected
3. **Test schedule generation** after changes to ensure no conflicts
4. **Check database migrations** if schema changes are needed

#### When Modifying Frontend:
1. **Find HTML_TEMPLATE** (starts at line 125)
2. **JavaScript is inline** within `<script>` tags
3. **CSS is inline** within `<style>` tags
4. **Maintain responsive design** and dark mode compatibility

#### When Adding New Features:
1. **Add database columns** in `init_db()` with try/except (lines 46-114)
2. **Create API route** with appropriate decorator
3. **Add frontend UI** in HTML_TEMPLATE
4. **Add frontend JS** to call the API
5. **Test thoroughly** with real data

### Testing Workflow
Since there are no automated tests:
1. Test with sample data (create test teachers/students)
2. Generate schedules and check for conflicts
3. Export to Excel/PDF to verify formatting
4. Test conflict detection and resolution
5. Verify dark mode compatibility

## 📦 Dependencies

Based on imports (lines 1-9):
```python
flask                # Web framework
openpyxl            # Excel export
weasyprint          # PDF generation
sqlite3             # Database (built-in)
datetime            # Date handling (built-in)
json                # JSON handling (built-in)
random              # Random selection (built-in)
io                  # BytesIO for file handling (built-in)
```

### Required System Dependencies (for weasyprint):
- Cairo
- Pango
- GDK-PixBuf

## ⚠️ Important Notes

### Performance Considerations
- **Large file**: 17,660 lines in a single file
- **Memory**: Schedule generation can be memory-intensive with many students
- **Database**: SQLite may struggle with concurrent writes

### Limitations
- **Single-file design**: All code in one file makes navigation challenging
- **No authentication**: No user login system
- **No validation layer**: Validation is spread across frontend and backend
- **No API versioning**: Direct API changes can break frontend

### Database Path
The database path is hardcoded (line 15):
```python
conn = sqlite3.connect('/home/mesauc/mysite/ders_programi.db')
```
**Action Required**: Update this path for your environment!

## 🔍 Common Tasks

### Adding a New Teacher Field
1. Add column in `init_db()` with ALTER TABLE and try/except
2. Update `add_teacher()` route to accept new field
3. Update `update_teacher()` route to handle new field
4. Update frontend form in HTML_TEMPLATE
5. Update teacher list display

### Modifying Schedule Algorithm
1. Locate `create_four_week_schedule()` at line 14012
2. Understand the current slot tracking mechanisms
3. Make changes carefully - this is the core algorithm
4. Test with various teacher/student combinations
5. Run conflict detection after generation

### Adding Export Format
1. Create new route (e.g., `/export_csv`)
2. Query schedule data from `schedule_data` global or database
3. Format data for export
4. Return response with appropriate MIME type
5. Add frontend button to trigger export

### Debugging Conflicts
1. Use `/check_conflicts_v2` endpoint
2. Check console logs (app prints debug info)
3. Verify student restrictions JSON format
4. Verify teacher schedule JSON format
5. Check blocked_slots are properly formatted

## 🛡️ Security Considerations

### Current Issues
- **SQL Injection**: Using parameterized queries (✓ good)
- **No authentication**: Anyone can access/modify data
- **No CSRF protection**: POST routes are unprotected
- **No input validation**: Limited server-side validation
- **Hardcoded path**: Database path is hardcoded

### Recommendations for Future
- Add user authentication
- Implement CSRF tokens
- Add comprehensive input validation
- Use environment variables for configuration
- Add rate limiting
- Implement audit logging

## 📚 Code Reading Guide

### Understanding Flow for a New Developer

1. **Start with `init_db()`** (line 20): Understand data model
2. **Read `get_teachers()`** (line 13541): See how data is retrieved
3. **Study `create_four_week_schedule()`** (line 14012): Core algorithm
4. **Check `detect_conflicts_v2()`** (line 15772): Conflict logic
5. **Browse HTML_TEMPLATE** (line 125): UI structure

### Function Categories

**Database Operations**: Lines 14-120
**HTML/CSS/JS Frontend**: Lines 125-13417
**Flask Routes**: Lines 13418-17660
**Schedule Generation**: Lines 14012-14536
**Conflict Detection**: Lines 15194-16004
**Export Functions**: Lines 14537-16588
**Utility Functions**: Lines 15176-15194, 16243-16906

## 🔄 Git Workflow

This project uses feature branches. Current branch structure:
- **Main branch**: Production-ready code
- **Feature branches**: Named with `claude/` prefix

### Committing Changes
1. Make changes to `flask_app.py` or database
2. Test thoroughly
3. Commit with descriptive message
4. Push to feature branch
5. Create pull request to main

### Commit Message Style
Based on git history, keep messages simple:
- "Add files via upload"
- "Update schedule algorithm"
- "Fix conflict detection bug"

## 🎯 AI Assistant Guidelines

### When Making Changes:
1. **Always read before writing**: Use Read tool to see current implementation
2. **Preserve Turkish text**: Don't translate UI strings
3. **Maintain formatting**: Keep emoji comments (🔥, ✅, 🆕, etc.)
4. **Test data flow**: Verify backend → database → frontend flow
5. **Check both themes**: Ensure changes work in light and dark mode

### When Debugging:
1. **Check JSON formats**: Most bugs are JSON structure issues
2. **Verify time formats**: Ensure "HH:MM" format consistency
3. **Review conflict logs**: Conflict detection returns detailed info
4. **Test with real scenarios**: Create sample data that mimics real usage

### When Refactoring:
1. **Don't break single-file structure** (unless explicitly requested)
2. **Maintain backward compatibility**: Database changes need migrations
3. **Keep function signatures**: Frontend depends on API contracts
4. **Document changes**: Update this file when making architectural changes

### Best Practices:
- **Use existing patterns**: Follow the code style already present
- **Leverage utilities**: Use `time_to_minutes()` and `check_time_overlap()`
- **Respect constraints**: The scheduling logic has careful constraints
- **Update documentation**: Keep CLAUDE.md current
- **Test exports**: Excel/PDF generation is fragile

## 📞 Support and Resources

### Key Files to Reference:
- `flask_app.py`: Everything is here
- `ders_programi.db`: Sample data for testing
- This file (CLAUDE.md): Your guide

### Understanding Turkish Terms:
- **Öğretmen**: Teacher
- **Öğrenci**: Student
- **Ders**: Lesson/Course
- **Program**: Schedule
- **Sınıf**: Class
- **Hafta**: Week
- **Gün**: Day
- **Saat**: Hour/Time
- **Branş**: Subject/Branch
- **Çakışma**: Conflict

### When Stuck:
1. Search for similar implementations in the file
2. Check how existing routes handle similar data
3. Review the database schema in `init_db()`
4. Look at frontend API calls in HTML_TEMPLATE
5. Test with minimal example first

---

**Last Updated**: 2025-11-14
**Maintainer**: AI Assistant (Claude)
**Project Status**: Active Development
