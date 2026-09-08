# Oclona — a Multi-Tenant B2B HR SaaS Platform

> **Product Engineering Case Study**

> **Note:** This document explains Oclona's product flows, architecture and engineering decisions for reference. It may become outdated as the application evolves and should not be treated as the source of truth. For the current behavior, always refer to the application and its implementation.

---

## 1. Overview

**Oclona** is a multi-tenant B2B HR SaaS platform designed to help organizations manage core workforce operations from a single application.

The platform brings together:

- Employee management
- Team management
- Employee onboarding
- Employee profiles
- Leave management
- Invitations
- Dashboards and workforce insights
- Role-based workflows
- Secure employee document management

The product was engineered with a focus on:

- Multi-tenant architecture
- Role-based access control (RBAC)
- Database-enforced authorization
- Secure tenant isolation
- Complex employee workflows
- Reusable frontend architecture
- Consistent product UX
- Maintainable business rules

The goal is to establish a **production-oriented SaaS foundation** capable of supporting organizations with different roles, permissions, workflows and data boundaries.

### Product at a glance

| Area           | Oclona                                |
| -------------- | ------------------------------------- |
| Product        | B2B HR SaaS                           |
| Architecture   | Multi-tenant                          |
| Primary focus  | Frontend & Product Engineering        |
| Authentication | Supabase Auth                         |
| Database       | PostgreSQL                            |
| Security       | RLS + RBAC                            |
| Storage        | Supabase Storage                      |
| Key domains    | Members, Onboarding, Leave, Dashboard |

### 📸 Oclona Application Dashboard

![Oclona dashboard](../assets/case-study/screenshots/dashboard-full.png)

---

## 2. Product Context

Organizations often manage employee information and workforce workflows processes across multiple, separate tools.

Common workflows include:

- Team management
- Employee invitations
- Employee onboarding
- Employee profiles
- Leave requests
- Leave approvals
- Leave balances
- Workforce dashboards and insights

Oclona approaches these as interconnected parts of one workforce platform.

### 📐 Oclona product domains

Oclona is organized around several connected workforce domains that share a common foundation for identity, tenant membership, roles, authorization and data access.

![Product domain map](../assets/case-study/diagrams/product-domain-map.svg)

---

## 3. My Role & Technology Stack

I worked on Oclona through a **Product Engineering Engagement**, contributing primarily to frontend and product engineering while working across the application stack where required to build complete and secure workflows.

### Frontend engineering

My frontend work included:

- React application architecture
- Feature-based module organization
- Role-aware UI behavior
- Reusable UI components
- Responsive interfaces
- Forms and validation
- Data tables
- Dashboard components
- Loading, error, empty and forbidden states

### Product engineering

The work also involved:

- Translating business requirements into workflows
- Designing role-aware experiences
- Modeling employee workflows
- Refining domain boundaries
- Improving maintainability
- Aligning frontend behavior with backend authorization

### Backend and data integration

Where required, I worked with:

- Supabase Auth
- PostgreSQL
- Row Level Security
- Supabase Storage
- Edge Functions
- Database authorization helpers
- Database triggers
- Audit fields
- Tenant isolation

The engineering focus remained **frontend & product engineering**, while extending into the backend when the product workflow or security model required it.

### Technology stack

| Layer          | Technologies            |
| -------------- | ----------------------- |
| Frontend       | React, Vite, JavaScript |
| UI             | Tailwind CSS, shadcn/ui |
| Routing        | React Router            |
| Forms          | React Hook Form, Yup    |
| Tables         | TanStack Table          |
| Charts         | Recharts                |
| Backend        | Supabase                |
| Database       | PostgreSQL              |
| Authentication | Supabase Auth           |
| Authorization  | RLS + RBAC              |
| Storage        | Supabase Storage        |
| Deployment     | Vercel + Supabase       |

The Oclona marketing website is maintained separately using **Next.js + TypeScript**.

---

## 4. Application Architecture

Oclona follows a **feature-oriented frontend architecture** rather than organizing the entire application purely by technical type.

Conceptually:

```text
src/
│
├── features/
│ ├── dashboard/
│ ├── leave/
│ ├── profile/
│ ├── team/
│ └── ...
│
├── components/
├── contexts/
├── hooks/
├── services/
└── ...
```

Domain-specific UI, hooks, services, validation and business behavior can therefore evolve together.

### Architecture principles

- Keep domain logic close to its feature
- Reuse shared UI patterns
- Keep data access in services
- Keep cross-feature state in appropriate contexts
- Separate presentation from persistence
- Avoid scattering business rules throughout components

### 📐 Application architecture

![Application architecture](../assets/case-study/diagrams/application-architecture.svg)

This structure allows the product to grow without turning the frontend into one large collection of unrelated screens and utilities.

---

## 5. Multi-Tenancy, Authentication & Security

Multi-tenancy is one of Oclona's core architectural requirements.

Each organization operates within its own tenant boundary.

Tenant-scoped domains include:

- Members
- Invitations
- Employee onboarding
- Leave Types
- Leave Entitlements
- Leave Requests
- Employee-related data

The system does not rely solely on the frontend to enforce this boundary.

### Authentication

Supabase Auth provides user identity.

The authenticated user is connected to the application's `members` model.

```text
Auth User
↓
Member
├── tenant_id
├── role
└── reporting relationship
```

Current roles include:

- Super Admin
- Owner
- Admin
- HR
- Manager
- Employee

Authorization can depend on more than the role alone.

Depending on the operation, the database can consider:

- Tenant membership
- Role
- Target member
- Manager relationship
- Record state
- Ownership/self-service rules
- Demo tenant status

### Database as the security boundary

The frontend controls the experience, but it is **not treated as the final security boundary**.

### 📐 Authentication → RLS → Database

![Authentication RLS database](../assets/case-study/diagrams/authentication-rls-database.svg)

This layered approach means that hiding a UI action is not considered sufficient authorization.

---

## 6. Team & Member Management

The Team domain separates two related but different concepts:

```text
Team
├── Members
└── Invitations
```

### Members

The member model contains information such as:

- Name
- Primary email
- Role
- Status
- Avatar
- Job information
- Department
- Manager
- Profile completion
- First login
- Last login
- Audit information

### Invitations

Invitations have their own lifecycle:

```text
Pending
↓
Accepted
│
├── Expired
└── Revoked
```

Separating invitations from members prevents an invitation from being treated as an employee before identity creation is complete.

### 📸 Members

![Members](../assets/case-study/screenshots/members.png)

The Team experience brings member information, roles, status and role-aware actions into a single operational view.

### 📸 Invitation workflow

![Invitation workflow](../assets/case-study/screenshots/invitation-workflow.png)

The Team module also separates **self-service operations** from **management operations**, allowing authorization rules to remain explicit.

---

## 7. Employee Onboarding & Profile

Employee identity and employee profile information are deliberately separated.

The `members` domain represents the application's organizational identity.

The employee onboarding domain contains the richer employee information required by HR workflows.

**Employee domain model**

### 📐 Employee domain model

![Employee domain model](../assets/case-study/diagrams/employee-domain.svg)

This keeps the member model relatively focused while allowing employee onboarding to contain substantially more information.

### Onboarding includes areas such as:

- Personal information
- Employment information
- Address
- Compensation
- Banking
- Emergency contacts
- Employee documents

### 📸 Employee Profile Details View

![Employee profile details view](../assets/case-study/screenshots/employee-profile.png)

The profile experience supports create/edit workflows, automatic email population, member synchronization, document handling and appropriate loading/error/not-found/forbidden states.

The product also uses profile completion to transition the user from onboarding into the normal profile experience.

Before completion

```text
Dashboard
↓
Complete Profile
↓
Start Onboarding
```

After completion

```text
Dashboard
↓
My Profile
```

---

## 8. Leave Management Domain

Leave is one of the more sophisticated domains in Oclona because it combines:

- Configuration
- Employee entitlements
- Leave requests
- Approval
- Manager relationships
- Balance accounting
- Database-side business behavior

### 📐 Leave lifecycle

![Leave domain lifecycle](../assets/case-study/diagrams/leave-lifecycle.svg)

### Leave Types

Leave Types define what kinds of leave an organization supports.

Authorized roles can manage Leave Types according to the application's role rules.

A Leave Request must reference an applicable active Leave Type belonging to the same tenant.

### 📸 Leave Types

![Leave types](../assets/case-study/screenshots/leave-types.png)

### Leave Entitlements

Entitlements determine how much leave a member has available.

Conceptually:

```text
Total Days
│
├── Used Days
│
└── Remaining Days
```

The database also validates relationships such as:

- Target member belongs to the same tenant
- Leave Type belongs to the same tenant
- Restricted members cannot receive certain entitlement operations
- Demo tenants cannot be mutated

### 📸 Leave Entitlements

![Leave entitlements](../assets/case-study/screenshots/leave-entitlements.png)

---

## 9. Leave Request, Approval & Database Accounting

The Leave request lifecycle is:

```text
Employee
↓
Select Leave Type
↓
Submit Request
↓
Pending
↙ ↘
Review Cancel
↓ ↓
Review Cancelled
↙ ↘
Approved Rejected
↓
Update Entitlement
↓
Used Days
↓
Remaining Days
```

### 📸 Leave Request

![Leave requests](../assets/case-study/screenshots/leave-requests.png)

Approval authority depends on role and organizational relationship.

For example:

- Owner/Admin/HR can have tenant-level management authority
- Managers can operate within their reporting relationship
- Employees manage their own applicable workflows
- A requester cannot approve their own leave

### 📸 Leave Approval

![Leave approvals](../assets/case-study/screenshots/leave-approvals.png)

### Database-side accounting

One important engineering decision was to keep approval-related balance accounting in the database rather than relying on frontend calculations.

Conceptually:

```text
Leave status changes to approved
↓
handle_leave_approval()
↓
leave_entitlements
↓
used_days += days_count
```

This keeps the derived entitlement state consistent with the approval event.

---

## 10. Dashboard & Data-Heavy Product UX

The Dashboard provides role-aware workforce information and brings together operational information into a single product view.

The dashboard architecture was refined around a KPI registry and role-aware visibility.

Examples include:

- Active members
- Profile metrics
- Pending invitations
- Leave-related metrics
- Role-specific workforce information

The application also uses reusable patterns for:

- Data tables
- Charts
- Forms
- Loading states
- Empty states
- Error states
- Forbidden states
- Not-found states
- Responsive layouts

---

## 11. Secure Documents, Auditability & Database Automation

HR applications deal with employee information that requires more careful handling than ordinary application assets.

### Document storage

Supabase Storage is used for employee-related documents and assets.

Depending on the resource, the implementation supports public or private access patterns.

Private documents can be accessed through signed URLs rather than permanent public URLs.

Examples include:

- Resume
- Photo/identity documents
- Offer letter
- Educational certificates
- Address proof

### Auditability

Important records contain audit information such as:

```text
created_at
created_by
updated_at
updated_by
deleted_at
deleted_by
```

Soft-delete workflows preserve historical information rather than immediately removing the record.

### Centralized timestamps

Rather than requiring every frontend service to calculate `updated_at`, the database maintains it through a shared trigger function.

This reduces duplicated timestamp logic across services.

### 📐 Data infrastructure

![Data infrastructure](../assets/case-study/diagrams/data-storage-audit.svg)

This separates ordinary application data, security enforcement, automatic database behavior and document storage concerns.

---

## 12. Business Rules & Authorization Design

Oclona's authorization model became more explicit as the product evolved.

A key principle is that **role alone is not always enough**.

An operation may depend on:

```text
Role
↓
Tenant Scope
↓
Target Member
↓
Relationship
↓
Record State
↓
Business Rule
↓
Authorization Decision
```

Examples include:

### Self vs management

```text
Employee
└── Own record
```

```text
Manager
└── Direct reporting members
```

```text
HR / Admin / Owner
└── Authorized tenant members
```

### Authorization helper model

Reusable database helpers encapsulate common decisions such as:

- `fn_is_super_admin()`
- `fn_is_tenant_member()`
- `fn_get_role()`
- `fn_is_tenant_admin()`
- `fn_can_write_tenant()`
- `fn_can_manage_team()`
- `fn_can_view_tenant_members()`
- `fn_can_edit_member()`
- `fn_is_manager_of()`
- `fn_can_approve_leave()`

Aditionally

- Leave-management helpers
- Employee-onboarding authorization helpers
- Invitation acceptance authorization

The intention is to avoid duplicating the same fundamental authorization logic throughout individual RLS policies.

### RLS, constraints and triggers have different responsibilities

### 📐 RLS, constraints and triggers

![RLS constraints and triggers](../assets/case-study/diagrams/rls-constraints-triggers.svg)

This separation prevents RLS from becoming a general-purpose business-logic engine.

---

## 13. Engineering Challenges & Solutions

Several challenges required architectural rather than purely UI-level solutions.

| Challenge                   | Engineering approach                  |
| --------------------------- | ------------------------------------- |
| Multi-tenant authorization  | Tenant-aware RLS policies             |
| Complex role relationships  | Reusable authorization helpers        |
| Manager-specific scope      | Relationship-based authorization      |
| Leave balance consistency   | Database-side accounting              |
| Employee profile complexity | Separate onboarding domain            |
| Invitation lifecycle        | Separate invitation/member models     |
| Demo tenant protection      | Centralized tenant write guard        |
| Repeated timestamp logic    | Centralized database trigger          |
| Complex frontend workflows  | Feature-oriented architecture         |
| Consistent UX states        | Reusable loading/error/empty patterns |

The common theme was to avoid solving every new requirement independently.

Instead, reusable architectural patterns were introduced where they could support future product growth.

---

## 14. Key Engineering Decisions & Current Product State

### Key engineering decisions

#### Database as the security boundary

The frontend controls the user experience, while PostgreSQL independently enforces authorization.

#### Self-service vs management

Operations on one's own records are deliberately separated from operations performed on other members.

#### Tenant-aware authorization

Tenant membership is validated alongside role and target relationships instead of assuming the frontend will provide a valid tenant.

#### Domain separation

Employee identity, employee profile, invitations, Leave configuration and Leave transactions are treated as distinct concerns.

#### Database-side automation

Cross-cutting behavior such as timestamps and Leave accounting is handled centrally where appropriate.

### Current product state

Oclona currently provides a substantial product foundation including:

- Multi-tenant architecture
- Authentication
- RBAC
- Tenant isolation
- Team management
- Invitations
- Employee onboarding
- Employee profiles
- Leave Types
- Leave Entitlements
- Leave Requests
- Leave approval
- Dashboard KPIs
- Account profile
- Charts
- Data tables
- Document storage
- Audit fields
- Database triggers
- RLS-based authorization
- Demo tenant protection
- Responsive product UI

The architecture is designed to support additional workforce capabilities without abandoning the existing tenant, identity and authorization model.

---

## 15. Future Direction & What Oclona Demonstrates

Oclona provides a foundation for broader workforce operations.

```text
Oclona HR Core
│
├── People
├── Teams
├── Onboarding
├── Leave
├── Time
└── Attendance
```

Some capabilities can be introduced incrementally as organizational requirements become more sophisticated, such as richer Manager/direct-report workflows and department-based permissions.

### What this project demonstrates

Oclona represents more than implementation of individual UI screens.

It demonstrates the ability to work across the lifecycle of a SaaS product:

Product Requirements
↓
Domain Modeling
↓
UX / Workflow Design
↓
Frontend Architecture
↓
Data Modeling
↓
Authorization
↓
Database Security
↓
Business Rules
↓
Product Refinement

From an engineering perspective, the project demonstrates experience with:

- React product development
- SaaS architecture
- Multi-tenancy
- RBAC
- PostgreSQL
- Supabase
- RLS
- Complex business workflows
- Data modeling
- Database-side authorization
- Reusable frontend architecture
- Forms and validation
- Data-heavy interfaces
- Product UX
- Production-oriented engineering

### Final note

> **A SaaS product should not only work when everything goes right; its architecture should define what users are allowed to do, what data they can access, and how business rules remain consistent as the product grows.**

Oclona reflects that approach by combining **frontend product engineering with secure, tenant-aware backend architecture**, while keeping the product modular enough to continue evolving.
