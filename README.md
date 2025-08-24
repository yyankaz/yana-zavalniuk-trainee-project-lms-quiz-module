# Quiz module — LMS platform — Yana Zavalniuk
This document performs object-oriented analysis and design. The UML class diagram models the core features, entities, their operations and relationships for the Quiz module of an abstract LMS platform.

**It is no allowed to to write code, but the model should be realistic for later implementation in a Spring Boot service.**

![PNG UML diagram](docs/images/quiz-lms-yana-zavalniuk.png)

## This is the "Quiz" module inside LMS platform.
It:

- allows to create and edit questions (also with media),

- makes quizzes from questions,

- assigns quizzes to students,

- checks answers (including manual checking by mentor),

- supports different user roles (student, mentor, admin),

- keeps versioning for questions (published questions no change),

- and even has foundation for cheating protection.

## Important features in architecture
### Question versioning
- `Question` is like logical "cover" for question.
- `QuestionVersion` is actual version used in quizzes.
- After `QuestionVersion` is published — no changes allowed, must create new version.

This allows:
- to see history of changes.
- to use same questions in many quizzes.
- to protect data from accidental edits.

### Topic tree
- `QuestionTopic` means topics and subtopics.
- Each topic can have parent and child topics (recursive).
- So questions easy to sort by topics.

### Question types
- Types (`SINGLE_CHOICE`, `MATCHING`, `TEXT_LONG`, `FIX_THE_CODE`, etc.) are in enum `QuestionType`.
- It is easy to add new type without big code changes.

### Media files
- Each `QuestionVersion` can have media attached (pictures, video, etc.) via `MediaAttachment`.
- Media stored separately, file type is enum `FileType`.
- So media is safely attached and logic is not broken.

### Quizzes building
- `Quiz` is test with one or more `QuestionInstance`.
- `QuestionInstance` is particular `QuestionVersion` in quiz (to track order and parameters).
- Quiz has enum status `QuizStatus`(`DRAFT`, `PUBLISHED`, `CLOSED`) and timer.

### User roles
- User class is abstract, extended by:
    - Student: takes quizzes.
    - Mentor: creates/edits questions and quizzes, checks answers.
    - Admin: no duties now, can add later.
- All users stored in one table by JPA inheritance JOINED for flexible design.

### Security and protection
- Planned: 
    - Render questions as images (PNG) to prevent copy-paste.
    - Tokens for quiz access.
    - Timers support.
    - Block copy-paste and maybe anti-screenshot.

### Details
- The correct answers and question content can be stored as JSON, which allows flexible and complex formatting — for example, multiple correct options, matching pairs, code fixes, etc.
- Easy to add Spring Security for role access control.

## Potential problems
While designing this UML model, I tried to balance clarity with completeness. The structure already became quite complex, so I intentionally left some implementation-level details out of the diagram to keep it readable. However, I’m aware of these aspects and want to show that I’ve considered them:

- There is no clear mechanism to pick the active version of a question. In practice, it would be important to enforce only one active version at a time.
- The `QuestionInstance` class could include extra fields like time limit, shuffle option, or point weight. I left them out to keep the diagram clean, but I see their value.
- Roles are modeled using both inheritance and enums. This could be simplified later to avoid duplication.
- The `Admin` role is included but not connected to any real logic yet. It’s there as a placeholder for potential system-level actions in the future.
- Some answer types (like MATCHING or ORDERING) would need structured data formats, not just a string. In real code, I would likely store this as JSON.
- The model does not currently separate automatically-graded answers from those requiring manual review. This would be important for implementation.
- Submissions don’t have a `status` field (like draft or reviewed), which would be useful to track progress more clearly.
- Question protection mechanisms (such as image rendering or blur levels) are not represented visually here, but I plan for them in future development.
- The way media files are stored (file system, cloud, or DB) is also not shown, since it depends on infrastructure decisions.
- The relationship chain between `Answer`, `QuestionInstance`, and `QuestionVersion` could be too indirect in performance-critical scenarios like real-time grading.
- Role recognition and user authentication logic are not shown in the diagram but are assumed to be handled in the application layer (e.g., using Spring Security).
- I didn’t include technical annotations (like `@Lob` or JSON mapping), since these are closer to implementation and would overcomplicate the diagram.

These points are not mistakes but conscious omissions or simplifications for the sake of architectural focus. I highlight them here to demonstrate awareness and readiness to handle them during real development.
