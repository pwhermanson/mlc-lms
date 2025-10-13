# Phase 3 Scope

This document defines the exact scope, deliverables, and implementation requirements for **Phase 3: Teacher Experience & Assignment Creation** of the MLC LMS rebuild.

> **Context**: See [initial-project-phases.md](../00-foundations/initial-project-phases.md) for strategic overview and phasing philosophy. Phase 3 builds directly on the foundation established in [phase-1-scope.md](./phase-1-scope.md) and the game integration completed in [phase-2-scope.md](./phase-2-scope.md).

## Purpose

Phase 3 completes the core learning loop by enabling teachers to create and assign learning content to students, then monitor student progress. This phase transforms the platform from a student-only game player into a functional Learning Management System where teachers can guide student learning and track mastery.

**What this enables:**
- Teachers can create simple assignments by selecting games and stages
- Teachers can assign games directly to individual students or classes
- Teachers can view basic student progress (games played, scores achieved)
- Students receive assignments on their dashboard (building on Phase 2)
- Complete end-to-end workflow: Teacher assigns → Student plays → Teacher sees results
- Foundation for advanced features like sequences, analytics, and bulk operations in Phase 4+

**Why it exists:**
- **Value delivery**: Creates the first complete teacher-student learning workflow
- **Pedagogical validation**: Proves the assignment and progress tracking systems work
- **User feedback**: Enables real teachers to test the core teaching experience
- **Foundation building**: Establishes patterns for advanced assignment features
- **Market readiness**: Delivers minimum viable product for early adopter teachers

**Success means**: Real teachers can log in, create a simple assignment with 2-3 games, assign it to students, and see when students complete those games with their scores—all without critical bugs or confusion.

---

## Scope

### What IS Included in Phase 3

#### 1. Teacher Dashboard Enhancement
- Assignment list view (active, scheduled, completed)
- Quick actions: "Create Assignment", "View Student Progress"
- Recent student activity feed (games played, scores achieved)
- Class roster overview with student status indicators
- Navigation to assignment builder and reports

#### 2. Simple Assignment Builder
- **Basic creation flow**: Name assignment, select students/class, choose games
- **Game selection interface**: Browse or search games from integrated library
- **Stage selection**: Choose which stages to include (Learn, Play, Quiz, Review)
- **Basic configuration**:
  - Assignment name and description
  - Target students or classes
  - Optional due date
  - Optional mastery threshold (e.g., "Quiz score ≥ 80%")
- **Save as draft or publish immediately**
- **Edit unpublished drafts**

**NOT in Phase 3 (deferred to Phase 4+):**
- ❌ Sequence-based assignments (use individual games only)
- ❌ Advanced scheduling (recurring, conditional release)
- ❌ Prerequisites or gating between games
- ❌ Bulk assignment to multiple classes simultaneously
- ❌ Assignment templates or library
- ❌ Drag-and-drop reordering
- ❌ Clone existing assignments

#### 3. Assignment Data Model (Core Schema)
- **assignments** table
- **assignment_steps** table (joins assignments to games/stages)
- **student_assignments** table (tracks which students have which assignments)
- **assignment_progress** table (tracks completion per student per step)

#### 4. Student Assignment View (Enhancement)
- Display assigned games prominently on student dashboard
- "Assignments" tab showing active, upcoming, and completed assignments
- Progress indicators per assignment (e.g., "2 of 5 games complete")
- Due date display and overdue warnings
- Launch game directly from assignment view
- Mark assignment as complete when all games meet criteria

#### 5. Teacher Progress View (Basic Reports)
- **Class progress table**: List students with assignment completion %
- **Individual student view**: See games completed, scores, timestamps
- **Assignment summary**: How many students completed, average scores
- **Export to CSV** (basic student list with scores)

**NOT in Phase 3 (deferred to Phase 4+):**
- ❌ Advanced analytics (trends, time-on-task, retry patterns)
- ❌ Comparison charts or visualizations
- ❌ Mastery predictions or recommendations
- ❌ Detailed event logs or audit trails
- ❌ Real-time live updates (use refresh button instead)

#### 6. Class Management (Minimal)
- **Create class**: Name, grade level, description
- **Add students to class**: Select from existing students
- **View class roster**: List of students in class
- **Remove students from class**

**NOT in Phase 3:**
- ❌ Bulk student CSV import (Phase 4+)
- ❌ Student groups within classes
- ❌ Class cloning or templates
- ❌ Class schedules or time slots

#### 7. API Endpoints (Teacher Functions)
- `POST /api/assignments` - Create new assignment
- `PUT /api/assignments/:id` - Update draft assignment
- `POST /api/assignments/:id/publish` - Publish assignment to students
- `DELETE /api/assignments/:id` - Delete draft assignment
- `GET /api/assignments/:id` - Get assignment details
- `GET /api/teacher/assignments` - List teacher's assignments
- `GET /api/teacher/classes/:id/progress` - Get class progress for assignment
- `GET /api/teacher/students/:id/progress` - Get individual student progress
- `POST /api/classes` - Create class
- `PUT /api/classes/:id` - Update class
- `GET /api/classes/:id/students` - Get class roster
- `POST /api/classes/:id/students` - Add student to class
- `DELETE /api/classes/:id/students/:studentId` - Remove student from class

#### 8. Notifications (Basic)
- Student receives in-app notification when assigned new assignment
- Teacher receives notification when student completes assignment
- Email notifications (optional, use dev console logging if no email provider)

**NOT in Phase 3:**
- ❌ Digest emails (daily/weekly summaries)
- ❌ Push notifications
- ❌ Custom notification preferences
- ❌ Reminder notifications for due dates

#### 9. Permissions and Authorization
- Teachers can only see their own students and classes
- Teachers can only create assignments for their students
- Teachers can only view progress for their students
- System Admin can view all assignments and progress (read-only)
- Students can only see their own assignments and cannot view others

#### 10. Testing and Quality
- Unit tests for assignment creation and progress tracking logic
- Integration tests for teacher → student assignment workflow
- E2E test: Create assignment → Assign to student → Student completes → Teacher views result
- Performance test: Handle class of 30 students with 10 assignments each

---

### What is EXCLUDED from Phase 3

**NOT in Phase 3** (deferred to later phases):

- ❌ Sequence-based assignments (individual games only)
- ❌ Bulk operations (mass assign to multiple classes, CSV import)
- ❌ Advanced analytics and reporting (trends, predictions, detailed logs)
- ❌ Messaging system (teacher-student communication)
- ❌ Grading or score overrides
- ❌ Assignment versioning or rollback
- ❌ Collaborative assignments or group work
- ❌ Adaptive assignments (adjusting difficulty based on performance)
- ❌ Teacher-to-teacher sharing or assignment marketplace
- ❌ Mobile-optimized teacher dashboard (responsive web only)
- ❌ Advanced scheduling (recurring, conditional release, prerequisites)
- ❌ Integration with external LMS (LTI, SCORM)
- ❌ Billing or subscription management (still Phase 4+)
- ❌ Advanced admin tools (subscriber admin and teacher admin roles)

**Rationale:** Phase 3 focuses on delivering the minimum viable teacher experience to complete the core learning loop. Advanced features that require complex data models, additional infrastructure, or sophisticated UX are deferred to Phase 4+ after validating the basic workflow with real users.

---

## Data Model

### Phase 3 Core Schema

See [data-model-erd.md](../18-architecture/data-model-erd.md) for complete schema. Phase 3 adds the following entities on top of Phase 1 and Phase 2 foundations:

#### Classes Table
```
classes
  - id: uuid (PK)
  - teacherId: uuid (FK -> users.id)
  - name: varchar(200)
  - gradeLevel: varchar(50), nullable
  - description: text, nullable
  - status: enum(active, archived)
  - createdAt: timestamp
  - updatedAt: timestamp
```

#### Class Members Table (Join)
```
class_members
  - id: uuid (PK)
  - classId: uuid (FK -> classes.id)
  - studentId: uuid (FK -> users.id)
  - enrolledAt: timestamp
  - status: enum(active, inactive)
```

#### Assignments Table
```
assignments
  - id: uuid (PK)
  - teacherId: uuid (FK -> users.id)
  - title: varchar(200)
  - description: text, nullable
  - status: enum(draft, published, archived)
  - dueAt: timestamp, nullable
  - createdAt: timestamp
  - updatedAt: timestamp
  - publishedAt: timestamp, nullable
```

#### Assignment Steps Table
```
assignment_steps
  - id: uuid (PK)
  - assignmentId: uuid (FK -> assignments.id)
  - gameId: uuid (FK -> games.id)
  - stage: enum(learn, play, quiz, challenge, review)
  - orderIndex: integer
  - masteryThreshold: integer, nullable (score required to pass, e.g., 80)
  - createdAt: timestamp
```

#### Student Assignments Table
```
student_assignments
  - id: uuid (PK)
  - assignmentId: uuid (FK -> assignments.id)
  - studentId: uuid (FK -> users.id)
  - status: enum(not_started, in_progress, completed)
  - assignedAt: timestamp
  - startedAt: timestamp, nullable
  - completedAt: timestamp, nullable
  - progressPercentage: integer (0-100)
```

#### Assignment Progress Table
```
assignment_progress
  - id: uuid (PK)
  - studentAssignmentId: uuid (FK -> student_assignments.id)
  - stepId: uuid (FK -> assignment_steps.id)
  - status: enum(not_started, in_progress, completed, mastered)
  - score: integer, nullable
  - bestScore: integer, nullable
  - attempts: integer, default 0
  - firstAttemptAt: timestamp, nullable
  - lastAttemptAt: timestamp, nullable
  - completedAt: timestamp, nullable
```

**Indexes:**
- `classes.teacherId` (for teacher class lookup)
- `class_members.classId, studentId` (unique together)
- `assignments.teacherId, status` (for teacher assignment lists)
- `assignment_steps.assignmentId, orderIndex` (for ordered game lists)
- `student_assignments.studentId, status` (for student dashboard)
- `student_assignments.assignmentId, studentId` (unique together)
- `assignment_progress.studentAssignmentId, stepId` (unique together)

**Foreign Key Constraints:**
- All FK constraints with appropriate ON DELETE cascades or SET NULL based on entity relationships
- Cannot delete a teacher if they have published assignments (business rule)
- Cannot delete a game if it's in published assignments (business rule)

---

## Behavior and Rules

### Assignment Creation Flow

1. Teacher navigates to `/teacher/assignments` → clicks "Create Assignment"
2. Provides: `title`, optional `description`, optional `dueAt`
3. Selects target: Choose students (individual) OR class (all current members)
4. **Game selection**:
   - Browse games by category OR search by name/concept
   - Click game → Select stages (checkboxes for Learn, Play, Quiz, Review)
   - Optionally set mastery threshold per stage (e.g., Quiz ≥ 80)
   - Add game to assignment (shows in ordered list)
   - Repeat to add more games (limit: 20 games per assignment in Phase 3)
5. **Review assignment**: See summary of selected games and target students
6. **Save as draft** OR **Publish immediately**
   - Draft: Saved to database, visible to teacher only, can be edited
   - Publish: Creates `student_assignments` records for all target students, sends notifications

**Error handling:**
- No students selected → "Please select at least one student or class"
- No games selected → "Please add at least one game to the assignment"
- Game unavailable or archived → "This game is no longer available. Please select another."
- Duplicate game/stage combination → "This game stage is already in the assignment"

### Assignment Publishing Rules

**When assignment is published:**
1. Validate all games are active and accessible
2. For each target student:
   - Create `student_assignments` record with `status: not_started`
   - Create `assignment_progress` record for each step with `status: not_started`
   - Send in-app notification to student
3. Set assignment `status: published`, record `publishedAt` timestamp
4. Log `assignment.published` event

**Business rules:**
- Published assignments cannot be edited (future: create versioning system)
- Teachers can archive published assignments to hide from student view
- Removing student from class does not remove their assignment (explicit removal required)
- Adding student to class does not auto-assign past assignments (explicit action required)

### Student Assignment Workflow

**Student views assignments:**
1. Navigate to `/student/dashboard` or `/student/assignments`
2. See list of assigned assignments with status badges:
   - 🔵 Not Started (0% complete)
   - 🟡 In Progress (1-99% complete)
   - 🟢 Completed (100% complete)
3. Click assignment → See list of games with individual progress:
   - ⚪ Not Started
   - 🟡 In Progress (started but not completed)
   - ✅ Completed (met mastery threshold or no threshold set)

**Student plays assignment game:**
1. Click game from assignment view → Launches game (same as Phase 2)
2. Play game stage (Learn, Play, Quiz, etc.)
3. On stage completion:
   - Record score to `assignment_progress`
   - Update `student_assignments.progressPercentage` (completed steps / total steps × 100)
   - If all steps completed → Set `student_assignments.status: completed`, record `completedAt`
   - Send notification to teacher if assignment is now complete

**Mastery threshold logic:**
- If step has `masteryThreshold`, student must achieve score ≥ threshold to mark step as `completed`
- If no threshold, any attempt marks step as `completed`
- Student can retry step any number of times (best score is recorded)

### Teacher Progress View

**View class progress:**
1. Navigate to `/teacher/classes/:id/assignments/:assignmentId`
2. See table of students with columns:
   - Student name
   - Status (Not Started, In Progress, Completed)
   - Progress % (e.g., "3/5 games complete")
   - Last activity timestamp
   - Actions: View details
3. Click student → See individual game scores and timestamps

**View individual student:**
1. Navigate to `/teacher/students/:id/assignments/:assignmentId`
2. See table of games with columns:
   - Game name + stage
   - Status (Not Started, In Progress, Completed)
   - Score (best score achieved)
   - Attempts
   - Last played timestamp

**Export to CSV:**
- Teacher clicks "Export" → Downloads CSV with student names, progress %, scores
- File format: `assignment-{assignmentId}-{timestamp}.csv`

### Class Management

**Create class:**
1. Navigate to `/teacher/classes` → "Create Class"
2. Provide: `name` (required), `gradeLevel` (optional), `description` (optional)
3. Save → Creates class with `status: active`

**Add students to class:**
1. Navigate to `/teacher/classes/:id` → "Add Students"
2. Search existing students by name or email
3. Select students → Click "Add to Class"
4. Creates `class_members` records with `status: active`

**Remove students from class:**
1. Navigate to `/teacher/classes/:id` → Student roster
2. Click "Remove" next to student name
3. Confirm → Updates `class_members.status: inactive` (soft delete)
4. Note: Does not affect existing assignments (student keeps those)

---

## UX Requirements

### Teacher Dashboard

**Layout:**
- Header: Logo, navigation (Dashboard, Classes, Assignments, Reports), user menu
- Main content:
  - Welcome message: "Welcome back, [Teacher Name]"
  - Quick stats: Total students, Active assignments, Recent completions
  - Quick action buttons: "Create Assignment", "View Classes", "View Reports"
  - Recent activity feed: Last 10 student actions (game completions, assignment completions)
- Sidebar (optional): Class list with quick links

**Responsive:**
- Desktop (1024px+): Full layout with sidebar
- Tablet (768-1023px): Stacked layout, collapsible sidebar
- Mobile (375-767px): Single column, hamburger menu

**Accessibility:**
- Keyboard navigation between sections
- Focus states on all interactive elements
- Screen reader labels for stats and actions

### Assignment Builder

**Steps:**
1. **Assignment Details**: Form with title, description, due date fields
2. **Select Students**: Radio buttons: "Individual students" or "Entire class" → Dropdown to select
3. **Add Games**: 
   - Search bar at top
   - Game cards with thumbnail, name, description
   - Click card → Modal with stage checkboxes and mastery threshold input
   - "Add to Assignment" button → Game appears in selected list
4. **Review**: Summary table of games, students, due date
5. **Save**: Buttons: "Save as Draft" and "Publish Assignment"

**Key states:**
- Loading: Skeleton screens while fetching games
- Empty: "No games selected yet. Search or browse to add games."
- Error: Inline error messages on form validation failures
- Success: Toast notification "Assignment published successfully!"

### Student Assignment View

**Assignment card:**
- Assignment title (e.g., "Week 1: Pitch Recognition")
- Due date (if set) with visual indicator (upcoming, due soon, overdue)
- Progress bar: "3 of 5 games complete" with visual % bar
- Status badge: Not Started, In Progress, Completed
- "View Details" button

**Assignment detail page:**
- Assignment description at top
- List of games with status icons:
  - Game name + stage name
  - Status icon (⚪ Not Started, 🟡 In Progress, ✅ Completed)
  - Score (if completed): "Score: 85/100"
  - "Play Now" button (launches game in Phase 2 player)

**Accessibility:**
- Clear heading hierarchy
- Status icons with text labels (not color-only)
- "Play Now" buttons with full context (e.g., "Play Pitch Recognition - Learn stage")

### Teacher Progress View

**Class progress table:**
| Student Name | Status | Progress | Last Activity | Actions |
|--------------|--------|----------|---------------|---------|
| Alice Smith  | In Progress | 60% (3/5) | 2 hours ago | View Details |
| Bob Johnson  | Not Started | 0% (0/5) | Never | View Details |
| Carol Lee    | Completed | 100% (5/5) | 1 day ago | View Details |

**Filters:**
- Status: All, Not Started, In Progress, Completed
- Sort by: Name, Progress, Last Activity

**Individual student detail:**
- Student name as page title
- Table of games with score, attempts, timestamps
- "Back to Class Progress" link

**Export button:**
- Icon + "Export to CSV"
- Downloads immediately, no confirmation modal needed

---

## Telemetry

See [event-model.md](../15-analytics-and-reporting/event-model.md) for complete event specifications. Phase 3 implements assignment lifecycle telemetry:

### Events Tracked

#### `assignment.created`
**When:** Teacher creates assignment (draft or published)  
**Where:** Server-side after assignment record created  
**Properties:**
- `assignmentId`: UUID
- `teacherId`: UUID
- `status`: "draft" | "published"
- `gameCount`: Number of games in assignment
- `studentCount`: Number of students assigned
- `hasDueDate`: Boolean
- `timestamp`: ISO 8601

#### `assignment.published`
**When:** Teacher publishes draft assignment  
**Where:** Server-side after student_assignments created  
**Properties:**
- `assignmentId`: UUID
- `teacherId`: UUID
- `studentCount`: Number of students assigned
- `gameCount`: Number of games
- `timestamp`: ISO 8601

#### `assignment.assigned_to_student`
**When:** Student receives assignment (at publish time)  
**Where:** Server-side during publish operation  
**Properties:**
- `assignmentId`: UUID
- `studentId`: UUID
- `studentAssignmentId`: UUID
- `gameCount`: Number of games in assignment
- `dueAt`: ISO 8601 or null
- `timestamp`: ISO 8601

#### `assignment.started`
**When:** Student plays first game in assignment  
**Where:** Client-side when student clicks first game  
**Properties:**
- `assignmentId`: UUID
- `studentId`: UUID
- `studentAssignmentId`: UUID
- `firstGameId`: UUID
- `timestamp`: ISO 8601

#### `assignment.game_completed`
**When:** Student completes a game within assignment  
**Where:** Server-side after game score recorded  
**Properties:**
- `assignmentId`: UUID
- `studentId`: UUID
- `studentAssignmentId`: UUID
- `stepId`: UUID (assignment_steps)
- `gameId`: UUID
- `stage`: "learn" | "play" | "quiz" | "review"
- `score`: Integer
- `masteryAchieved`: Boolean (met threshold or no threshold)
- `progressPercentage`: Integer (assignment completion %)
- `timestamp`: ISO 8601

#### `assignment.completed`
**When:** Student completes all games in assignment  
**Where:** Server-side after final game completed  
**Properties:**
- `assignmentId`: UUID
- `studentId`: UUID
- `studentAssignmentId`: UUID
- `completionTime`: Duration in seconds (assigned to completed)
- `averageScore`: Integer (across all games)
- `timestamp`: ISO 8601

#### `assignment.progress_viewed`
**When:** Teacher views class or student progress  
**Where:** Client-side when teacher loads progress page  
**Properties:**
- `teacherId`: UUID
- `assignmentId`: UUID
- `viewType`: "class" | "individual"
- `classId`: UUID or null
- `studentId`: UUID or null
- `timestamp`: ISO 8601

#### `class.created`
**When:** Teacher creates new class  
**Where:** Server-side after class record created  
**Properties:**
- `classId`: UUID
- `teacherId`: UUID
- `initialStudentCount`: Integer (if students added immediately)
- `timestamp`: ISO 8601

#### `class.student_added`
**When:** Student added to class  
**Where:** Server-side after class_members record created  
**Properties:**
- `classId`: UUID
- `teacherId`: UUID
- `studentId`: UUID
- `timestamp`: ISO 8601

#### `class.student_removed`
**When:** Student removed from class  
**Where:** Server-side after class_members updated  
**Properties:**
- `classId`: UUID
- `teacherId`: UUID
- `studentId`: UUID
- `timestamp`: ISO 8601

### Implementation

**Phase 3 telemetry storage:**
- Events written to `audit_logs` table (established in Phase 1)
- Events also written to dedicated `assignment_events` table for analytics
- Console logging in development
- Optional external analytics service integration (PostHog, Mixpanel)

**Future enhancements:**
- Real-time teacher dashboards showing live student activity
- Predictive analytics (which students at risk of not completing)
- A/B testing framework for assignment effectiveness

---

## Permissions

See [roles-matrix.md](../02-roles-and-permissions/roles-matrix.md) for complete permission specifications. Phase 3 implements role-based access for teacher features:

### Roles in Phase 3

- **Student**: Can view own assignments, play assigned games, see own progress
- **Teacher**: Can create assignments, create classes, view own students' progress
- **System Admin**: Can view all assignments and progress (read-only), cannot create assignments for other teachers

**NOT in Phase 3:**
- ❌ Teacher Admin role (Phase 4+)
- ❌ Subscriber Admin role (Phase 4+)
- ❌ Fine-grained permissions (all teachers have same capabilities)

### Permission Checks

**Route protection:**
- `/teacher/*` - Requires role: teacher or system_admin
- `/student/assignments` - Requires role: student
- API endpoints under `/api/teacher/*` - Requires teacher role
- API endpoints under `/api/assignments/*` - Requires teacher role (create/update) or student role (view own)

**Data access rules:**
- Teachers can only:
  - View/edit their own assignments
  - View their own classes and students
  - View progress for their own students
  - Create assignments only for their own students
- Students can only:
  - View their own assignments
  - Play games from their own assignments
  - View their own progress
- System Admin can:
  - View all assignments (read-only)
  - View all classes and progress (read-only)
  - Cannot create or modify assignments/classes

**Phase 3 Permission Logic:**
```typescript
// Simplified - detailed implementation in code
canAccessAssignment(user, assignment):
  if user.role == 'system_admin': return true // read-only access
  if user.role == 'teacher': return assignment.teacherId == user.id
  if user.role == 'student': return studentAssignmentExists(assignment.id, user.id)
  return false

canCreateAssignment(user):
  return user.role == 'teacher'

canViewStudentProgress(user, studentId):
  if user.role == 'system_admin': return true
  if user.role == 'teacher': return teacherHasStudent(user.id, studentId)
  if user.role == 'student': return user.id == studentId
  return false
```

**NOT in Phase 3:**
- ❌ Impersonation (deferred to Phase 4+)
- ❌ Delegation (teacher allowing another teacher to access their data)
- ❌ Student-to-student visibility (peer progress)
- ❌ Public sharing of assignments

---

## Supporting Documents Referenced

This Phase 3 scope specification draws from the following source documents:

- [Assignment Play Re-design.docx.txt](../00-ORIG-CONTEXT/Assignment%20Play%20Re-design.docx.txt) — Assignment creation workflows, teacher-student assignment flow, and progress tracking requirements
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Teacher role permissions, class management capabilities, and student progress visibility
- [System Admin Re-design.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design.docx.txt) — Class and roster management interfaces referenced for teacher experience design

## Dependencies

### Internal Dependencies

**Foundational documents:**
- [initial-project-phases.md](../00-foundations/initial-project-phases.md) - Strategic context and phasing philosophy
- [product-vision.md](../00-foundations/product-vision.md) - Overall product goals and principles
- [pedagogy-principles.md](../00-foundations/pedagogy-principles.md) - Educational approach and mastery model

**Phase 1 and Phase 2 foundations:**
- Phase 1 authentication system (users, sessions, roles)
- Phase 2 game integration (games table, game player, progress tracking)
- Phase 2 student dashboard (extends with assignment view)

**Teacher experience specs:**
- [teacher-dashboard.md](../04-teacher-experience/teacher-dashboard.md) - Teacher dashboard layout and navigation
- [assignment-builder.md](../04-teacher-experience/assignment-builder.md) - Detailed assignment creation UX
- [teacher-reports.md](../04-teacher-experience/teacher-reports.md) - Progress reporting requirements

**Student experience specs:**
- [student-dashboard.md](../03-student-experience/student-dashboard.md) - Student dashboard layout
- [student-assignments.md](../03-student-experience/student-assignments.md) - Assignment display and navigation
- [assignment-play.md](../03-student-experience/assignment-play.md) - Game playing experience

**Architecture:**
- [system-overview.md](../18-architecture/system-overview.md) - High-level architecture
- [data-model-erd.md](../18-architecture/data-model-erd.md) - Complete database schema
- [api-design-principles.md](../18-architecture/api-design-principles.md) - API conventions
- [rest-endpoints.md](../18-architecture/rest-endpoints.md) - API endpoint specifications

**Operations:**
- [test-strategy.md](../19-quality-and-operations/test-strategy.md) - Testing approach
- [release-management.md](../19-quality-and-operations/release-management.md) - Deployment process
- [observability.md](../19-quality-and-operations/observability.md) - Monitoring and logging

### External Dependencies

**Required services (carried over from Phase 1 & 2):**
- **Next.js** - Web application framework (v14+)
- **Vercel** - Hosting and deployment platform
- **Neon PostgreSQL** - Managed database (Serverless Postgres)
- **NextAuth.js** - Authentication library (v5+)
- **Tailwind CSS** - Styling framework (v3+)
- **Cloudflare R2** - Game asset storage (from Phase 2)

**New dependencies for Phase 3:**
- **React Hook Form** - Form handling for assignment builder
- **TanStack Query** - Data fetching and caching for teacher dashboards
- **PapaParse** - CSV export functionality (student progress)
- **React Table** - Table UI for progress views (optional, can use native HTML tables)

**NOT in Phase 3** (deferred):
- ❌ Email service provider (Resend, SendGrid) - Use console logging in dev
- ❌ Real-time updates (WebSocket service) - Use polling or manual refresh
- ❌ Advanced analytics platform (PostHog, Mixpanel) - Basic logging only
- ❌ Error monitoring (Sentry) - Console errors only

### Phase 4 Blockers

Phase 4 (Advanced Teacher Features & Sequences) cannot begin until Phase 3 delivers:
- ✅ Working assignment creation and publishing system
- ✅ Student assignment view integrated with dashboard
- ✅ Progress tracking for assigned games
- ✅ Teacher progress view with student list and scores
- ✅ Class management (create, add students, assign to class)
- ✅ Basic CSV export functionality

**Critical path:** Phase 3 establishes the assignment data model and workflow patterns. Phase 4 will extend with sequences, bulk operations, advanced analytics, and admin roles. The assignment system must be flexible enough to support future enhancements without breaking changes.

---

## Open Questions

### To Resolve Before Build

**1. Assignment Editing**
- **Question:** Can teachers edit published assignments, or are they immutable once published?
- **Options:**
  - A) Immutable - Published assignments cannot be edited (simpler, Phase 3)
  - B) Versioning - Create new version when edited, students see updated version
  - C) Allow edits - Direct edits affect all students immediately
- **Recommendation:** Option A (immutable) for Phase 3 simplicity. Add versioning in Phase 4.
- **Owner:** Product Lead
- **Deadline:** Before Phase 3 kickoff

**2. Assignment Deletion**
- **Question:** Can teachers delete published assignments? What happens to student progress?
- **Options:**
  - A) Cannot delete published (only archive to hide)
  - B) Soft delete (mark deleted but retain data)
  - C) Hard delete with cascade (removes all student progress)
- **Recommendation:** Option A (archive only) to preserve student data
- **Owner:** Product Lead
- **Deadline:** Before Phase 3 kickoff

**3. Duplicate Game Prevention**
- **Question:** Can teachers add the same game multiple times to an assignment?
- **Options:**
  - A) Allow duplicates (same game, different stages is common)
  - B) Prevent exact duplicates (same game + same stage)
  - C) Warn but allow
- **Recommendation:** Option B (prevent exact duplicates) with clear error message
- **Owner:** UX Lead
- **Deadline:** Before Phase 3 UI work begins

**4. Assignment Sorting**
- **Question:** How do students see multiple assignments - sorted by due date, creation date, or manually?
- **Options:**
  - A) Due date ascending (soonest first)
  - B) Creation date descending (newest first)
  - C) Teacher-defined priority order
- **Recommendation:** Option A (due date) with fallback to creation date if no due date
- **Owner:** UX Lead
- **Deadline:** Before Phase 3 UI work begins

**5. Class Size Limits**
- **Question:** Is there a maximum number of students per class in Phase 3?
- **Options:**
  - A) No limit (handle via pagination)
  - B) Soft limit with warning (e.g., 50 students)
  - C) Hard limit (e.g., 100 students per class)
- **Recommendation:** Option A (no limit) with pagination for classes > 50 students
- **Owner:** Tech Lead
- **Deadline:** Before Phase 3 kickoff

**6. Progress Calculation**
- **Question:** How is assignment completion % calculated - by games completed or by mastery achieved?
- **Options:**
  - A) Simple: Completed games / Total games
  - B) Mastery-based: Games with score ≥ threshold / Total games
  - C) Weighted: Different stages have different weights
- **Recommendation:** Option B (mastery-based) to align with pedagogical principles
- **Owner:** Product Lead
- **Deadline:** Before Phase 3 kickoff

**7. Teacher Can See Own Assignments as Student**
- **Question:** If a teacher also has a student account, can they view teacher-side assignments?
- **Options:**
  - A) Separate accounts required (single role per account from Phase 1)
  - B) Role switcher (future Phase 4+)
  - C) Not supported
- **Recommendation:** Option A (separate accounts) based on Phase 1 decision
- **Owner:** Tech Lead
- **Deadline:** Before Phase 3 kickoff

### Future Considerations (Not Blocking Phase 3)

**8. Assignment Templates**
- Create reusable assignment templates or share with other teachers
- **Timing:** Phase 4+
- **Reference:** [assignment-builder.md](../04-teacher-experience/assignment-builder.md)

**9. Sequence-Based Assignments**
- Assign entire sequences instead of individual games
- **Timing:** Phase 4+
- **Reference:** [sequence-model.md](../09-sequences-and-curriculum-tools/sequence-model.md)

**10. Advanced Analytics**
- Time-on-task, retry patterns, mastery predictions
- **Timing:** Phase 4+
- **Reference:** [teacher-reports-kpis.md](../15-analytics-and-reporting/teacher-reports-kpis.md)

**11. Bulk Assignment Operations**
- Assign to multiple classes simultaneously, clone assignments
- **Timing:** Phase 4+
- **Reference:** [mass-assignments.md](../05-admin-subscriber-experience/mass-assignments.md)

**12. Real-Time Updates**
- Live progress updates on teacher dashboard as students play
- **Timing:** Phase 4+
- **Complexity:** Requires WebSocket infrastructure

---

## Phase 3 Timeline

**Total duration:** 3 weeks (15 business days)

### Week 1: Data Model and Assignment Creation
- **Days 1-2:** Database schema design, migrations, seed data
- **Days 3-5:** Assignment creation API endpoints and business logic

### Week 2: Student Assignment View and Progress Tracking
- **Days 6-7:** Student assignment display on dashboard
- **Days 8-10:** Progress tracking integration with game player from Phase 2

### Week 3: Teacher Progress View and Polish
- **Days 11-12:** Teacher progress view and class management UI
- **Days 13-14:** Testing, bug fixes, edge cases
- **Day 15:** Final validation, deployment, documentation

**Milestone:** By end of Week 3, real teachers can create assignments with 2-3 games, assign to students, and see completion status and scores when students play those games.

---

## Success Criteria

Phase 3 is complete when:

✅ **Technical:**
- All database migrations run successfully in production
- All API endpoints pass integration tests
- Assignment creation and progress tracking work end-to-end
- No critical security vulnerabilities

✅ **Functional:**
- Teachers can create assignments with 1-20 games
- Teachers can select individual students or entire class as target
- Teachers can publish assignments (students receive notifications)
- Students see assignments on dashboard with progress indicators
- Students can play assigned games (using Phase 2 game player)
- Game completion updates assignment progress automatically
- Teachers can view class progress table with student names, completion %, scores
- Teachers can view individual student detail (games completed, scores, timestamps)
- Teachers can create classes and add/remove students
- Teachers can assign to classes (all current members receive assignment)
- Teachers can export progress to CSV

✅ **Quality:**
- All forms have proper validation and error messages
- All pages are responsive (mobile, tablet, desktop)
- Basic accessibility requirements met (keyboard nav, focus states, labels)
- No console errors or warnings
- Page load time < 3 seconds on 3G connection
- Teacher dashboard handles classes of 30+ students without performance issues

✅ **Documentation:**
- API endpoints documented with examples
- Database schema documented with relationships
- Teacher workflows documented with screenshots
- Setup instructions updated for Phase 3 features

**Phase 3 does NOT require:**
- ❌ Sequence-based assignments (individual games only)
- ❌ Advanced analytics or visualizations
- ❌ Bulk operations or CSV import
- ❌ Messaging system
- ❌ Email notifications (console logging acceptable)
- ❌ Real-time updates (polling or manual refresh acceptable)

---

## Next Phase

Upon successful completion of Phase 3, proceed to:
- **[phase-4-scope.md](./phase-4-scope.md)** - Advanced Teacher Features & Sequences (Future)

Phase 4 will build on this foundation to deliver:
- Sequence-based assignments (assign entire curriculum paths)
- Bulk operations (CSV import students, mass assignment)
- Advanced analytics and reporting (trends, predictions, detailed logs)
- Teacher Admin and Subscriber Admin roles
- Messaging system (teacher-student communication)
- Email notifications and reminders
- Assignment templates and sharing

---

## Related Documentation

**Foundational context:**
- [initial-project-phases.md](../00-foundations/initial-project-phases.md) - Strategic overview and phasing philosophy
- [product-vision.md](../00-foundations/product-vision.md) - Overall product vision and goals
- [pedagogy-principles.md](../00-foundations/pedagogy-principles.md) - Educational approach and mastery model

**Teacher experience:**
- [teacher-dashboard.md](../04-teacher-experience/teacher-dashboard.md) - Teacher dashboard layout and navigation
- [assignment-builder.md](../04-teacher-experience/assignment-builder.md) - Detailed assignment creation UX
- [teacher-reports.md](../04-teacher-experience/teacher-reports.md) - Progress reporting requirements
- [teacher-settings.md](../04-teacher-experience/teacher-settings.md) - Teacher preferences and configuration

**Student experience:**
- [student-dashboard.md](../03-student-experience/student-dashboard.md) - Student dashboard layout
- [student-assignments.md](../03-student-experience/student-assignments.md) - Assignment display and navigation
- [assignment-play.md](../03-student-experience/assignment-play.md) - Game playing experience
- [student-profile.md](../03-student-experience/student-profile.md) - Student profile and settings

**Architecture and design:**
- [system-overview.md](../18-architecture/system-overview.md) - High-level system architecture
- [data-model-erd.md](../18-architecture/data-model-erd.md) - Complete database schema (all phases)
- [api-design-principles.md](../18-architecture/api-design-principles.md) - API design conventions
- [rest-endpoints.md](../18-architecture/rest-endpoints.md) - API endpoint specifications

**Roles and permissions:**
- [roles-matrix.md](../02-roles-and-permissions/roles-matrix.md) - Complete role-permission matrix
- [session-and-auth-states.md](../02-roles-and-permissions/session-and-auth-states.md) - Authentication states

**Operations:**
- [test-strategy.md](../19-quality-and-operations/test-strategy.md) - Testing approach
- [release-management.md](../19-quality-and-operations/release-management.md) - Deployment process
- [observability.md](../19-quality-and-operations/observability.md) - Monitoring and logging

---

**Document Status:** ✅ Complete and ready for Phase 3 implementation  
**Last Updated:** 2025-10-13  
**Owner:** Product & Engineering  
**Reviewers:** Tech Lead, UX Lead, Product Manager
