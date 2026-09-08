# Oclona – Multi-Tenant HR SaaS (Technical Overview)

Oclona is a multi-tenant HR SaaS platform designed to bring workforce operations into a unified, role-aware system with strict tenant-level data isolation.

It is built using a React-based frontend and Supabase (Postgres, Auth, Row Level Security, Storage and Edge Functions).

The system emphasizes backend-enforced security, controlled onboarding flows, workflow-driven operations and predictable multi-tenant behavior without relying on client-side trust.

This repository provides a technical overview of the **product's architecture, engineering decisions, security model and key workflows.**

---

## Links

- **Case Study:** https://sk-shahnawazkhan.github.io/oclona-summary/case-study/
- **Live Product:** https://app.oclona.com/
- **Marketing Website:** https://oclona.com/
- **Repository:** https://github.com/sk-shahnawazkhan/oclona-summary

---

## Table of Contents

1. [Problem Space](#problem-space)
2. [System Overview](#system-overview)
3. [Architecture Overview](#-architecture-overview)
4. [Core Engineering Challenges](#core-engineering-challenges)
5. [Key Design Decisions](#-key-design-decisions)
6. [Important Flows](#important-flows)
7. [Security & Data Isolation](#-security--data-isolation)
8. [Email & Communication Infrastructure](#email--communication-infrastructure)
9. [Trade-offs & Limitations](#trade-offs--limitations)
10. [Future Improvements](#-future-improvements)
11. [Project History](#project-history)

---

## Problem Space

Multi-tenant systems introduce challenges around data isolation, authorization, workflows and lifecycle management:

- Enforcing strict **tenant-level data isolation**
- Designing onboarding flows without exposing internal identifiers
- Managing **role-based access** across multiple modules
- Supporting workflow-driven operations such as invitations, approvals and lifecycle states
- Handling data-heavy interfaces with consistent filtering, sorting, pagination and updates

Oclona is structured to address these constraints through clear application boundaries, database-enforced authorization and predictable product workflows.

---

## System Overview

- Multi-tenant architecture (**tenant = company**)
- Invite-based employee onboarding
- Role-based access control:
  - Super Admin
  - Owner
  - Admin
  - HR
  - Manager
  - Employee
- Core product domains:
  - Team & member management
  - Employee onboarding & profiles
  - Leave management
  - Workforce dashboards & insights
- Data-intensive interfaces using tables, filters, forms and analytics

---

## 📊 Architecture Overview

- **Frontend:** React SPA with modular feature architecture, hooks, context and reusable components
- **Backend:** Supabase
  - PostgreSQL
  - Authentication
  - Storage
  - Edge Functions
- **Authorization:** PostgreSQL Row Level Security (RLS)
- **Server Logic:** Supabase Edge Functions for privileged operations
- **Data Isolation:** Tenant-scoped records enforced at the database layer using `tenant_id`
- **Deployment:** Vercel + Supabase

---

## Core Engineering Challenges

### Tenant Isolation Without Client Trust

Users must never access data outside their authorized tenant scope.

**Impact:**

A tenant isolation failure would compromise the core SaaS security boundary.

**Approach:**

- Enforced through PostgreSQL RLS policies
- Policies evaluate tenant membership and role
- Client-side checks are treated as UX controls rather than security boundaries

---

### Invite-Based Onboarding

Users should be able to join a tenant through a controlled invitation flow.

**Impact:**

A weak invitation system could allow unauthorized tenant access or incorrect tenant association.

**Approach:**

- Token-based invitation flow
- Email-driven onboarding
- Controlled tenant association during signup
- Invite lifecycle tracking:
  - Pending
  - Accepted
  - Expired
  - Revoked

---

### Initial Tenant Setup

The first user must establish both the tenant and initial owner context.

**Approach:**

- User signup
- Company setup
- Tenant and owner/member context created together

---

### Data-Heavy UI Complexity

HR workflows require tables and interfaces that handle filtering, sorting, pagination, forms and state changes consistently.

**Approach:**

- TanStack Table for data-heavy interfaces
- Reusable table, drawer, form and dialog patterns
- Feature-based frontend structure
- Predictable data flow across views

---

### Secure Privileged Operations

Certain operations require elevated server-side capabilities, such as transactional email delivery.

**Approach:**

- Supabase Edge Functions
- Restricted server-side execution
- Service-role access isolated from the client

---

## 🔑 Key Design Decisions

### Why Supabase?

- Unified platform for Auth, PostgreSQL, RLS, Storage and Edge Functions
- Reduces the amount of custom backend infrastructure
- Provides strong database-level authorization capabilities
- Enables faster product iteration

---

### Why RLS Over Client/API-Level Authorization?

- Security is enforced at the data layer
- Database access remains protected even when client-side logic changes
- Tenant-scoped access rules can be expressed close to the data
- Reduces reliance on application-layer authorization alone

---

### Why Invite-Based Onboarding?

- Provides controlled tenant entry
- Prevents arbitrary tenant association
- Supports explicit role assignment
- Enables a clear member lifecycle

---

### Why React SPA?

- Well suited to dynamic, workflow-heavy product interfaces
- Provides fine control over UI and state
- Supports reusable component and feature patterns
- Works well with Supabase-backed applications

---

## Important Flows

### Multi-Tenant Onboarding

`Owner Signup` → `Email Verification` → `Company Setup` → `Invite Member` → `Accept Invite` → `Member Signup` → `Tenant Linking` → `Access System`

**Why this matters:**

The flow establishes tenant membership through controlled onboarding rather than trusting client-provided tenant identifiers.

---

### Team Management

- Owner/Admin manages team members
- Authorized users send invitations
- Invitations move through lifecycle states
- Accepted members become part of the tenant
- Member access is controlled through role and tenant scope

---

### Employee Onboarding

- Employee completes profile information
- Profile completion state controls onboarding experience
- Employee information is associated with the correct tenant/member
- Authorized roles can manage employee profiles according to access rules

---

### Leave Management

- Leave types define available leave categories
- Entitlements define employee leave allocations
- Employees submit leave requests
- Authorized users review requests
- Approved requests update leave accounting
- Requests move through controlled lifecycle states

---

## 🔐 Security & Data Isolation

- Tenant data protected using **PostgreSQL Row Level Security (RLS)**
- Access evaluated using tenant membership and role
- Sensitive operations handled through **Supabase Edge Functions**
- Storage access is scoped to authorized application flows
- Database constraints protect structural data integrity
- Database triggers handle selected automated behaviors such as timestamps and leave accounting

The security model separates:

**Authentication → Authorization → Data Integrity → Database Automation**

---

## Email & Communication Infrastructure

Oclona separates business communication from transactional email delivery to improve maintainability, deliverability and domain reputation management.

### Domain Structure

- `oclona.com` → public marketing website and business communication
- `app.oclona.com` → multi-tenant SaaS application
- `mail.oclona.com` → transactional email infrastructure

### Business Communication

Custom domain-based business mailboxes are managed through Zoho Mail for operational and customer communication.

Example identities:

- `hello@oclona.com`
- `support@oclona.com`
- `shahnawaz@oclona.com`

### Transactional Email Architecture

Transactional emails are delivered using Resend and Supabase Edge Functions through a dedicated email subdomain.

Current use cases include:

- Invite-based employee onboarding
- Tenant member invitations
- Onboarding communications
- Workflow notifications
- Future authentication and system-generated emails

Example identities:

- `notifications@mail.oclona.com`
- `noreply@mail.oclona.com`

### Email Security & Deliverability

The infrastructure uses standard email authentication mechanisms:

- SPF
- DKIM
- DMARC

These help support authenticated delivery, spoofing protection, inbox placement and long-term domain reputation.

### Operational Monitoring

Email and deployment workflows are monitored through:

- Resend delivery analytics
- Supabase Edge Function logs
- Vercel deployment monitoring

---

## Trade-offs & Limitations

- Supabase provides speed and integrated infrastructure but offers less flexibility than a fully custom backend
- RLS introduces additional complexity in policy design and debugging
- RBAC is currently role-based rather than fully permission-based
- Edge Functions are used selectively rather than as a complete backend layer
- Real-time updates are limited to selected workflows

---

## 🚀 Future Improvements

- Migration to TypeScript
- Fine-grained permission-level authorization
- Background jobs using scheduled or queue-based processing
- Expanded audit logging for critical actions
- Enhanced analytics and reporting
- Additional workforce and HR domains

---

## Project History

- Initially started as a form-based HR tool
- Evolved into a multi-tenant SaaS platform
- Expanded with authentication, onboarding, team management, leave workflows, dashboards and role-based access
- Architecture progressively refined around tenant isolation, backend-enforced security and maintainable product engineering practices

---

## Case Study

For the full product walkthrough, screenshots, architecture diagrams, engineering decisions and detailed workflows:

**[View the Oclona Case Study](https://sk-shahnawazkhan.github.io/oclona-summary/case-study/)**

---

## Notes

This repository is a **technical overview and demonstration of Oclona**, a production-oriented HR SaaS product.

The primary application implementation repository is private. This repository intentionally focuses on architecture, engineering decisions, product workflows, trade-offs and selected technical context rather than exposing the complete production codebase.

Documentation may evolve as the product changes. For current product behavior, refer to the deployed application and implementation.

---

## Author

[Shahnawaz Khan](https://shahnawazkhan.vercel.app/), Senior Frontend Engineer  
Focused on building scalable SaaS products and reliable product experiences.

---

## License

This repository is provided for **demonstration and evaluation purposes only**.

**Reuse, redistribution, or commercial** use is not permitted.
