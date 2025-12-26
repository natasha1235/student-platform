
# Student Contract (Canonical)

## Purpose
The student is the primary actor. The system exists to preserve student agency while enabling conditional, revocable visibility to parents/guardians and additive oversight by editors.

This document defines the non-negotiable rules for student ownership, visibility, and access boundaries.

---

## Definitions
- **Student**: a user with `profiles.role = 'student'`.
- **Artifact**: a student-owned work item (volunteer, research, art, accomplishment, problem_solved).
- **Visibility**: an explicit state attached to each artifact.
- **Parent link**: a relationship in `parent_student_links` with `status = active`.
- **Editor**: a user with `profiles.role = 'editor'` with no ownership rights over student content.

---

## Invariants (Cannot Change Without Rewriting the System)
1. **Ownership**: A student owns their artifacts and projects. Ownership is the only default authority.
2. **Explicit visibility only**: Nothing becomes visible to anyone unless the student explicitly sets a visibility state that permits it.
3. **Revocation is immediate**: When visibility or linking conditions are removed, access disappears immediately (no grace, no cache).
4. **Parents do not automatically see student content**: Parent access is conditional and limited.
5. **Editors are additive only**: Editors can add metadata/feedback but cannot change student ownership or parent access.
6. **Roles are immutable**: Students cannot change roles and cannot escalate privileges.

---

## Student Capabilities

### Authentication & Landing
- On authentication, the student is routed to `/student`.
- The student can read their own profile row.

### Profile (Student-Owned)
The student can:
- Read and update student-controlled profile fields (e.g., display_name, avatar_url).
The student cannot:
- Change `profiles.role`.
- View other users’ profiles by default.

### Artifacts (Student-Owned)
The student can:
- Create artifacts.
- Read all their artifacts, regardless of visibility state.
- Update their artifacts (student-owned fields).
- Delete their artifacts.

The student cannot:
- Create or modify artifacts owned by others.
- Force others to see their artifacts without explicit visibility.

### Visibility (Student-Controlled)
Each artifact has exactly one visibility state:

- `private`: visible only to the student.
- `selected`: visible to linked parents (active link required).
- `public`: visible to the platform audience defined by product policy (at minimum: authenticated users, unless changed later).

Rules:
- Visibility is an explicit action by the student.
- Default visibility for any new artifact is `private`.
- Changing visibility is reversible.
- Switching to `private` removes access for all non-owners immediately.

### Projects (Private by Default)
Projects are private by default.
A student may optionally share a project via an explicit share record (if/when enabled in product scope).

Parents do not see projects unless explicit share exists.

---

## Parent Access Boundaries (Student-Centric)
Parents/guardians can see student artifacts only if ALL are true:
1. A parent-student link exists AND `status = active`
2. Artifact visibility is `selected` or `public`
3. The requesting user is the linked parent

Parents can never:
- Edit artifacts
- Change visibility
- Delete artifacts
- Access private artifacts
- Infer hidden artifacts from errors

If a link becomes `revoked`, parent access disappears immediately.

---

## Editor Access Boundaries (Student-Centric)
Editors may have read access to student artifacts if the platform grants it.
Editors can:
- Read authorized artifacts
- Write editor-scoped metadata/feedback (if those fields exist)
Editors cannot:
- Edit student content
- Change visibility
- Delete artifacts
- See parent relationships
- Escalate privileges

Editor actions must never alter access relationships.

---

## Non-Goals (Explicitly Not Built Yet)
The student cannot currently:
- Browse other students
- Social-follow or DM
- View parent identities by default
- View platform analytics about other users
- Access admin tooling

If any of these are added later, this contract must be amended.

---

## Correctness Tests (Must Hold)
1. Student creates artifact → visibility defaults to `private`.
2. Parent with no active link → sees nothing.
3. Student sets artifact to `selected`:
   - parent still sees nothing until link is active
4. After link is active:
   - parent sees only `selected`/`public` artifacts
   - parent never sees `private`
5. Student switches artifact back to `private`:
   - parent loses access immediately
6. Editor cannot modify student-owned fields.
7. Student cannot change `profiles.role`.

---

## Versioning
This is the canonical student contract v1.
Any change must be reviewed as a governance change, not a UI change.
