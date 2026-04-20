# Oclona – Multi-Tenant HR SaaS (System Design Summary)

Oclona is a multi-tenant HR SaaS system designed to handle tenant-scoped onboarding, team management, and workflow-driven operations with strict data isolation.

It is built using Supabase (Postgres + Row Level Security) and a React-based frontend.

The system emphasizes backend-enforced security (RLS), controlled onboarding flows, and predictable multi-tenant behavior without relying on client-side trust.

This repository focuses on the **system design, architectural decisions, and engineering trade-offs** behind the product.

---

## Live Demo

Public demo will be added once the current production deployment is stabilized.

---

## Table of Contents

1. [Problem Space](#problem-space)
2. [System Overview](#system-overview)
3. [Architecture Overview](#-architecture-overview)
4. [Core Engineering Challenges](#core-engineering-challenges)
5. [Key Design Decisions](#-key-design-decisions)
6. [Important Flows](#important-flows)
7. [Security & Data Isolation](#-security--data-isolation)
8. [Trade-offs & Limitations](#trade-offs--limitations)
9. [Future Improvements](#-future-improvements)
10. [Project History](#project-history)

---

## Problem Space

Multi-tenant systems introduce challenges that go beyond standard CRUD applications:

- Enforcing strict **tenant-level data isolation**
- Designing onboarding flows without exposing internal identifiers
- Managing **role-based access** across multiple modules
- Supporting workflow-driven operations (invites, approvals, lifecycle states)
- Handling data-heavy interfaces with consistent performance

Oclona is structured to address these constraints in a predictable and scalable way.

---

## System Overview

- Multi-tenant architecture (**tenant = company**)
- Invite-based onboarding system
- Role-based access control (Admin, Employee)
- Core modules:
  - Leave management
  - Employee onboarding
  - Team management

- Data-intensive UI (tables, filters, analytics dashboards)

---

## 📊 Architecture Overview

- **Frontend:** React (SPA) with hooks, context, modular structure
- **Backend:** Supabase (Postgres, Auth, Storage)
- **Authorization:** Row Level Security (RLS)
- **Server Logic:** Supabase Edge Functions for privileged operations
- **Data Isolation:** Enforced at database level using `tenant_id`

---

## Core Engineering Challenges

### Tenant Isolation Without Client Trust

Users must never access data outside their tenant scope.

**Impact:** Any leakage here breaks the core SaaS boundary model.

**Approach:**

- Enforced via RLS policies scoped by `tenant_id`
- Eliminates reliance on client-side validation

---

### Invite-Based Onboarding

Users should be able to join a tenant securely without exposing tenant identifiers.

**Impact:** A weak invite system can allow unauthorized users to join tenants or expose internal tenant identifiers, breaking core access boundaries.

**Approach:**

- Token-based invite system
- Email-driven onboarding
- Tenant association handled during controlled signup
- Invite lifecycle tracking (Pending, Accepted, Expired)

---

### Initial Admin Setup

The first user must create both tenant and admin context.

**Approach:**

- Two-step flow:
  1. User signup
  2. Company setup

- Tenant and admin profile created together

---

### Data-Heavy UI Complexity

Tables require consistent handling of filtering, sorting, pagination, and updates.

**Approach:**

- TanStack Table with controlled state
- Reusable table and drawer patterns
- Predictable data flow across views

---

### Secure Privileged Operations

Certain actions require elevated permissions (e.g., sending invites).

**Approach:**

- Supabase Edge Functions using `service_role`
- Restricted execution for sensitive operations

---

## 🔑 Key Design Decisions

### Why Supabase?

- Unified layer for Auth, Database, RLS, and Storage
- Reduces backend surface area
- Enables faster iteration on product features

---

### Why RLS over API-Level Authorization?

- Security enforced at the data layer
- Removes dependency on API-level checks
- Scales naturally with multi-tenant access patterns

---

### Why Invite-Based Onboarding?

- Prevents unauthorized tenant access
- Ensures controlled user lifecycle
- Simplifies role assignment

---

### Why React SPA?

- Fine control over UI and state
- Supports dynamic, workflow-heavy interfaces
- Encourages reusable component patterns

---

## Important Flows

### Multi-Tenant Onboarding

`Admin Signup` → `Email Verification` → `Company Setup` → `Invite Member` → `Accept Invite` → `Member Signup` → `Tenant Linking` → `Access System`

**Why this matters:**
This flow ensures users can only enter a tenant through controlled invites, preventing unauthorized access and maintaining strict tenant boundaries.

---

### Team Management

- Admin sends invite
- Invite tracked (Pending / Accepted / Expired)
- Member joins via secure link
- Team state updates accordingly

---

### Leave Management

- Employee submits leave request
- Admin reviews
- Status updated (Approved / Rejected / Cancelled)
- Reflected in records and balances

---

### Flow Diagrams (Coming Soon)

---

## 🔐 Security & Data Isolation

- All tenant data secured via **Row Level Security (RLS)**
- Access restricted to tenant-scoped records
- Sensitive operations handled via **Edge Functions**
- Storage access controlled and scoped per tenant

---

## Trade-offs & Limitations

- Supabase limits flexibility compared to a fully custom backend
- RLS introduces additional complexity in debugging and query design
- RBAC is currently role-based (not permission-level)
- Edge Functions are used selectively, not as a full backend layer
- Real-time updates are limited to key workflows

---

## 🚀 Future Improvements

- Fine-grained RBAC (permission-level access)
- Background jobs (pg_cron or queue-based processing)
- Audit logs for critical actions
- Enhanced analytics and reporting

---

## Project History

- Initially started as a form-based HR tool
- Evolved into a multi-tenant SaaS system with onboarding, authentication, RBAC, and workflow modules
- Architecture refined around tenant isolation, security, and scalability

---

## Screenshots (Coming Soon)

---

## Architecture Diagram (Coming Soon)

---

## Flow Diagrams (Coming Soon)

---

## Notes

This repository represents a **system design summary** of a production-oriented application.

The primary implementation repository is private.

---

## Author

[Shahnawaz Khan](https://shahnawazkhan.vercel.app/), Senior Frontend Engineer  
Focused on building scalable SaaS products and system design

---

## License

This repository is provided for **demonstration and evaluation purposes only**.

**Reuse, redistribution, or commercial** use is not permitted.
