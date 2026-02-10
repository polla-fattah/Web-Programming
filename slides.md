---
theme: default
title: "Backend Web Programming (PHP + Laravel)"
info: |
  Course plan + weekly roadmap + project milestones
  Backend focus: PHP, Laravel, SQL, Auth, REST APIs, Testing, Deployment
paginate: true
drawings: false
routerMode: hash
---

# Backend Web Programming
## PHP + Laravel (Course Plan)

**Audience:** Students who know SQL + OOP (with a short refresher)  
**Outcome:** Build and deploy a real backend application with auth, roles, APIs, tests

---

# Course Map

- [Week 1: PHP Basics](27)
- [Week 2: PHP OOP](54)
- [Week 3: Laravel Fundamentals](72)
- [Week 4: Validation & Blade](#week-04)
- [Week 5: Eloquent Relationships](#week-05)
- [Week 6: Filtering & Pagination](#week-06)
- [Week 7: Authentication](#week-07)
- [Week 8: Authorization & Policies](#week-08)
- [Week 9: REST API](#week-09)
- [Week 10: API Resources & Rate Limiting](#week-10)
- [Week 11: Uploads & Logging](#week-11)
- [Week 12: Testing](#week-12)
- [Week 13: Deployment](#week-13)
- [Week 14: Final Demo](#week-14)

---

## What you will be able to do by the end
- **Build Laravel apps** using MVC + Eloquent
- **Design relational DB schemas** + migrations
- **Implement validation + authorization** correctly
- **Create REST APIs** + API resources
- **Write feature tests** for critical flows
- **Deploy a simple production release**

---

## Tools (Install in Week 1)
- **PHP 8.x**
- **Composer**: PHP package manager
- **Laravel**: Installed via Composer
- **Database**: MySQL or PostgreSQL
- **Git + GitHub/GitLab**: Version control
- **VS Code**: Code editor
- **Postman (or Insomnia)**: API testing tool

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
- **Users + Auth**: Registration and login system
- **Roles/Permissions**: Admin/Staff/User access control
- **CRUD + Search + Pagination**: Create, read, update, delete with filtering
- **Relationships**: 1–to–Many, Many–to–Many associations
- **Reports**: Basic data summaries
- **REST API endpoints**: Structured API access
- **Testing + Deployment**: Quality assurance and production release

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

- **Syntax, variables, constants, arrays**: PHP data structures
- **Functions**: Type hints, null-safety, return types
- **OOP**: Classes, interfaces, traits, namespaces, exceptions
- **Composer + autoloading**: Package management and code organization
- **Tiny "no-framework" web flow**: Request/response concept

---

## Weeks 3–4: Laravel Fundamentals
- **Laravel project structure**: Folders, config, app layout
- **Routing + controllers**: HTTP handling and request processing
- **Request lifecycle + middleware basics**: How requests flow through Laravel
- **Validation**: Input verification and error handling
- **Blade**: Minimal templating, enough to test flows
- **Artisan CLI** *(intro)*: Scaffolding models, controllers, migrations, running commands
- **Laravel Debugbar**: Query profiling, request/response inspection, timing

**Deliverable:** CRUD module (with validation)

---

## Weeks 5–6: Database + Eloquent Deepening
- **Migrations, seeders, factories**: Database schema management and testing data
- **Eloquent relationships**: Defining model associations
- **Query builder**: Building dynamic queries
- **Pagination**: Limiting query results per page
- **Filtering/search patterns**: Finding specific records
- **Tinker** *(intro)*: Interactive shell for testing models and queries
- **Database GUI** (DBeaver/TablePlus): Visual data exploration alongside Tinker
- **Database Transactions**: Data consistency when multiple tables change together
- **Eager Loading**: Fix N+1 query problems
- **Collections**: Manipulating data sets

**Deliverable:** 2–3 related models + listing with filters

---

## **Authentication**: Login/register systems
- **Authorization with policies/gates**: Role-based access control
- **Ownership checks**: Users can only edit their own resources
- **Secure route protection**: Restrict unauthorized accesscan only edit own resource)
- Secure route protection

**Deliverable:** Role-based access + protected admin screens/endpoints

---

## **REST conventions + status codes**: Standard HTTP patterns
- **API Resources (transformers)**: Formatting data responses
- **Token or session-based API**: API authentication (your choice)
- **Rate limiting basics**: Prevent abuse
- **Email/Notifications**: Password resets, confirmations, alerts
- **Real-time concepts**: Polling vs push
- **Laravel Broadcasting**: Events, Channels, Authorization
- **WebSocket server**: Laravel Reverb (first-party)
- **Client listening** –everb (first-party)
- Client listening (minimal): Laravel Echo (or simple JS)

**Deliverable:** API for core entities + Postman collection

---

## Week 11: File Uploads + Storage + Logging + Error Handling
- **Upload validation**: File type and size checks
- **Storage**: Local and public file management
- **Logging + error handling patterns**: Tracking issues and graceful responses
- **Exception Handling**: Custom exceptions, graceful error responses
- **Queues** *(optional)*: Async tasks (emails, reports) for performance

**Deliverable:** Attachment/image upload for one entity

---

## Week 12: Testing (minimum viable, very practical)
- **Feature tests**: HTTP request and response testing
- **Testing auth + authorization**: Security flow validation
- **Database refresh + factories**: Test data setup

**Deliverable:** Tests for the top 3 critical flows

---

## Week 13: Deployment + Production Basics
- **Environment config**: Settings per environment
- **Migrations in production**: Safe database updates
- **Basic security checklist**: Key security measures
- **Deploy to shared hosting or VPS**: Live server setup

**Deliverable:** Live demo URL (or LAN server) + deployment notes

---

## Week 14: Final Project Demo
- **Final polishing**: Code cleanup and refinement
- **Demo + code review**: Presentation and technical evaluation
- **Project report**: Architecture, database schema, endpoints, test coverage

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
id: week-01
---

---
src: ./weeks/02-php-oop.md
id: week-02
---

---
src: ./weeks/03-laravel-routing-mvc.md
id: week-03
---

---
src: ./weeks/04-laravel-validation-blade.md
id: week-04
---

---
src: ./weeks/05-eloquent-relationships.md
id: week-05
---

---
src: ./weeks/06-filtering-pagination.md
id: week-06
---

---
src: ./weeks/07-auth.md
id: week-07
---

---
src: ./weeks/08-authorization-policies.md
id: week-08
---

---
src: ./weeks/09-rest-api.md
id: week-09
---

---
src: ./weeks/10-api-resources-rate-limit.md
id: week-10
---

---
src: ./weeks/11-uploads-logging.md
id: week-11
---

---
src: ./weeks/12-testing.md
id: week-12
---

---
src: ./weeks/13-deployment.md
id: week-13
---

---
src: ./weeks/14-final-demo.md
id: week-14
---
