---
theme: default
title: "Backend Web Programming (PHP + Laravel)"
info: |
  Course plan + weekly roadmap + project milestones
  Backend focus: PHP, Laravel, SQL, Auth, REST APIs, Testing, Deployment
paginate: true
drawings: false
---

# Backend Web Programming
## PHP + Laravel (Course Plan)

**Audience:** Students who know SQL + OOP (with a short refresher)  
**Outcome:** Build and deploy a real backend application with auth, roles, APIs, tests

---

## What you will be able to do by the end
- Build Laravel apps using MVC + Eloquent
- Design relational DB schemas + migrations
- Implement **validation + authorization** correctly
- Create REST APIs + API resources
- Write feature tests for critical flows
- Deploy a simple production release

---

## Tools (Install in Week 1)
- PHP 8.x
- Composer
- Laravel (via Composer)
- Database: MySQL or PostgreSQL
- Git + GitHub/GitLab
- VS Code
- Postman (or Insomnia)

---

## Course Rules (How we build every feature)
Every feature must include:
1) Migration / DB constraint
2) Model + relationship (if needed)
3) Request validation
4) Authorization (policy/gate)
5) Output (Blade or API)
6) At least 1 feature test (later weeks)

---

## Semester Project (Choose one)
Pick one project and grow it weekly:
- **Clinic Appointment System**
- **Internship/Company Portal**
- **Library Management**
- **Simple LMS / Course Enrollment**
- **Inventory + Sales (small POS backend)**

> We’ll implement the same “core modules” regardless of topic.

---

## Project Modules (Incremental)
- Users + Auth
- Roles/Permissions (Admin/Staff/User)
- CRUD + Search + Pagination
- Relationships (1–M, M–M)
- Reports (basic)
- REST API endpoints
- Testing + Deployment

---

## Assessment (Final Distribution = 100%)

**Project (40%)**
- Milestones (through semester): **20%**
- Final practical (demo + code): **20%**

**Final Exam (50%)**
- Written exam: **30%**
- Practical exam: **20%** *(same as the Project final practical)*

**Remaining (30%)**
- Midterm exam: **20%**
- Quizzes: **10%**


---

# Weekly Roadmap

---

## Weeks 1–2: PHP Refresher (fast)
**Goal:** Learn PHP style + what’s different in OOP

- Syntax, variables, constants, arrays
- Functions, type hints, null-safety
- OOP: classes, interfaces, traits, namespaces, exceptions
- Composer + autoloading
- Tiny “no-framework” web flow (request/response idea)

---

## Weeks 3–4: Laravel Fundamentals
- Laravel project structure
- Routing + controllers
- Request lifecycle + middleware basics
- Validation
- Blade (minimal, enough to test flows)

**Deliverable:** CRUD module (with validation)

---

## Weeks 5–6: Database + Eloquent Deepening
- Migrations, seeders, factories
- Eloquent relationships
- Query builder, pagination
- Filtering/search patterns

**Deliverable:** 2–3 related models + listing with filters

---

## Weeks 7–8: Auth + Authorization (real backend)
- Authentication (login/register)
- Authorization with **policies/gates**
- Ownership checks (user can only edit own resource)
- Secure route protection

**Deliverable:** Role-based access + protected admin screens/endpoints

---

## Weeks 9–10: REST APIs
- REST conventions + status codes
- API Resources (transformers)
- Token or session-based API (your choice)
- Rate limiting basics
- Why real-time? polling vs push
- Laravel Broadcasting concepts: Events, Channels, Authorization
- WebSocket server: Laravel Reverb (first-party)
- Client listening (minimal): Laravel Echo (or simple JS)
- 
**Deliverable:** API for core entities + Postman collection

---

## Week 11: File Uploads + Storage + Logging
- Upload validation
- Storage (local/public)
- Logging + basic error handling patterns


**Deliverable:** Attachment/image upload for one entity

---

## Week 12: Testing (minimum viable, very practical)
- Feature tests (HTTP)
- Testing auth + authorization
- Database refresh + factories

**Deliverable:** Tests for the top 3 critical flows

---

## Week 13: Deployment + Production Basics
- Environment config
- Migrations in production
- Basic security checklist
- Deploy to shared hosting or VPS

**Deliverable:** Live demo URL (or LAN server) + deployment notes

---

## Week 14: Final Project Demo
- Final polishing
- Demo + code review
- Project report (architecture + DB + endpoints + tests)

---

# Milestones (Project Timeline)

---

## Milestone 1 (End of Week 4)
- Project skeleton + one CRUD module
- Validation included
- Git repo with clean commits

---

## Milestone 2 (End of Week 8)
- Auth + roles/permissions
- At least 2 relationships implemented
- Admin vs user flows working

---

## Milestone 3 (End of Week 12)
- REST API endpoints + Postman collection
- Feature tests for critical flows
- Basic logging + upload feature (if applicable)

---

## Final (Week 14)
- Deployed + demo-ready
- Clean code structure (controllers thin, logic in services if needed)
- Report + endpoint documentation

---

# Next: Weekly Slides (Imported)

---

## How we’ll manage the content
- This `slides.md` stays as the master plan deck
- Each week gets its own file in `/weeks`
- We import week decks into one big Slidev presentation

---

---
src: ./weeks/01-php-basics.md
---

---
src: ./weeks/02-php-oop.md
---

---
src: ./weeks/03-laravel-routing-mvc.md
---

---
src: ./weeks/04-laravel-validation-blade.md
---

---
src: ./weeks/05-eloquent-relationships.md
---

---
src: ./weeks/06-filtering-pagination.md
---

---
src: ./weeks/07-auth.md
---

---
src: ./weeks/08-authorization-policies.md
---

---
src: ./weeks/09-rest-api.md
---

---
src: ./weeks/10-api-resources-rate-limit.md
---

---
src: ./weeks/11-uploads-logging.md
---

---
src: ./weeks/12-testing.md
---

---
src: ./weeks/13-deployment.md
---

---
src: ./weeks/14-final-demo.md
---
