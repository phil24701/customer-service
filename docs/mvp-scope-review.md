# Customer Support Platform — MVP Scope Review

**Reviewer:** Alex, Veteran Customer Service & Support Operations Manager  
**Date:** August 13, 2026  
**Target:** Product Owner / Product Team  

---

### Instructions & Guidance
Each item below has been evaluated from a practical support operations perspective—focusing on **Agent Experience (AX)**, **Admin Overhead**, **Cross-Department Handoffs**, and **Data Integrity**.

---

### 1. Real-time agent collision prevention
* **Description:** When one agent is actively drafting a response or note, other agents see a real-time warning and cannot simultaneously edit the response.
* **MVP:** **MVP**
* **Comments:** Passive collision indicators (like Freshdesk's subtle eye icons) fail during high-volume triage. Active collision locking prevents embarrassing duplicate responses sent to customers and eliminates wasted agent effort.

---

### 2. Automatic Pending Customer → Open
* **Description:** An inbound customer communication automatically changes a case from `Pending Customer` to `Open`.
* **MVP:** **MVP**
* **Comments:** Mandatory automation. If agents have to manually flip this status back to `Open`, 30–50% of replied cases will sit trapped in `Pending Customer`, destroying your resolution SLAs and leaving customers ignored.

---

### 3. Send & Set to Pending Customer
* **Description:** Agents can send a response and put the case into `Pending Customer` with a single action.
* **MVP:** **MVP**
* **Comments:** Pure Agent Experience (AX) and throughput optimizer. Combining reply submission with a status change eliminates hundreds of unnecessary dropdown clicks per agent every single day.

---

### 4. System Catch-All Queue
* **Description:** A mandatory, protected `Unassigned / System Catch-All` queue receives cases when routing or rule evaluation fails. Administrative personnel are alerted when this occurs.
* **MVP:** **MVP**
* **Comments:** Crucial for the "3 AM On-Call Admin." Routing rules break; without a mandatory system fallback queue, failed cases vanish into thin air without anyone realizing it.

---

### 5. Safe configuration deactivation
* **Description:** The system prevents deactivation of a queue, category, or status that is currently associated with active cases until those cases are appropriately reassigned.
* **MVP:** **MVP**
* **Comments:** Prevents the dreaded "ghost case" phenomenon where active tickets become unassigned, invisible, and unworkable because an admin deactivated a queue or category mid-flight.

---

### 6. Internal Department / Owner
* **Description:** When a case is `Pending Internal`, the responsible internal department or owner must be identified (for example, Engineering, Billing, or L3 Operations).
* **MVP:** **MVP**
* **Comments:** Eliminates the cross-department "black hole." If you don't track *which* department holds the ball during an escalation, supervisors cannot manage queue bottlenecks or measure hold times.

---

### 7. Pinned Internal Case Summary
* **Description:** Support users can maintain a single prominent internal summary containing the current blocker, steps taken, and required action.
* **MVP:** **Not MVP**
* **Comments:** A fantastic capability for context preservation during long escalations, but for a v1.1 release, agents can survive using formatted top-of-thread internal notes. Promote to v1.2.

---

### 8. Resolution Data Validation
* **Description:** Before a case can be resolved, required information such as Root Cause, Category, and Final Priority must be populated and validated.
* **MVP:** **MVP**
* **Comments:** Essential for analytics integrity. Customers always pick "Urgent" or select the wrong category on intake. Forcing agents to validate root cause upon resolution keeps your reporting clean.

---

### 9. SLA Clock Pausing
* **Description:** SLA response/resolution timers automatically pause while a case is in `Pending Customer` and resume when it returns to `Open`.
* **MVP:** **MVP**
* **Comments:** Vital for operational honesty. You cannot accurately measure support handle or resolution performance if the SLA clock runs while waiting days for a customer to reply.

---

### 10. Canned Responses / Macros
* **Description:** Authorized users can apply preconfigured response templates that may also update case fields.
* **MVP:** **MVP**
* **Comments:** Essential throughput tool. Without macros, agents waste hours retyping routine instructions, driving up handle times and agent fatigue.

---

### 11. SLA Risk / Breach Visibility
* **Description:** Supervisors can identify cases approaching or exceeding their service-level targets.
* **MVP:** **MVP**
* **Comments:** Core supervisor feature. Managers need at-a-glance visibility into cases nearing or in breach so they can reassign and prioritize active queues effectively.

---

### 12. Cross-Department Handoff Tracking
* **Description:** The system can identify and report cases currently waiting on another department or internal owner.
* **MVP:** **Not MVP**
* **Comments:** While valuable long-term, requiring the `Internal Department / Owner` field on `Pending Internal` cases (Item #6) allows supervisors to filter standard lists without needing dedicated handoff reporting in v1.

---

### 13. Real-Time Case/Queue Updates
* **Description:** Relevant case and queue changes are reflected for other users without requiring a manual page refresh.
* **MVP:** **MVP**
* **Comments:** Triage efficiency collapses if agents have to manually mash F5/refresh to see if a ticket was picked up, updated, or reassigned by someone else in the queue.

---

### 14. Customer Satisfaction / CSAT
* **Description:** Customers can provide a satisfaction rating or feedback after case resolution.
* **MVP:** **Not MVP**
* **Comments:** Valuable metric, but for a strict internal MVP focused on core ticketing fundamentals, CSAT survey automation can wait for the immediate follow-up release.

---

### 15. Knowledge Base
* **Description:** Agents and/or customers can search a knowledge base for articles related to a case.
* **MVP:** **Not MVP**
* **Comments:** Knowledge base integrations add scope. First focus on getting case ingestion, state management, and resolution workflows rock-solid.

---

### 16. Email-to-Case
* **Description:** Incoming customer email can automatically create or update a case.
* **MVP:** **MVP**
* **Comments:** Fundamental channel ingestion. Unless this platform is strictly portal-only, email ingestion is mandatory for realistic operations.

---

### 17. AI-Assisted Support
* **Description:** AI can provide capabilities such as case classification, summarization, or suggested responses.
* **MVP:** **Not MVP**
* **Comments:** Total distraction for an initial build. AI capabilities add unexpected usage fees, API latency, and setup overhead—nail the core deterministic UI and routing engine first.

---

### Additional Recommendations

* **Additional item:** **Keyboard Shortcuts for Core Triage Actions (e.g., Submit, Toggle Note, Apply Macro)**
  * **MVP:** **MVP**
  * **Comments:** Significantly reduces agent wrist strain and speeds up high-volume triage by 15–20% with minimal backend development overhead.

* **Additional item:** **Mandatory Audit Log for Assignment & Status Changes**
  * **MVP:** **MVP**
  * **Comments:** Crucial for troubleshooting "who touched this ticket?" disputes when cases get bounce-passed between teams or accidentally closed.

* **Additional item:** **Basic Bulk Actions in Queue Views (Bulk Assign, Bulk Status Change)**
  * **MVP:** **MVP**
  * **Comments:** When an outage hits or a queue gets flooded, forcing agents to open 50 individual tickets to update them one-by-one destroys operational momentum.
