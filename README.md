# Research Laboratory Management System (RLMS)

A modern, responsive, enterprise-grade laboratory information and facility operations management web application built for scientific research institutions, multi-user core instrumentation facilities, and interdisciplinary project groups.

---

## 🌟 Institutional Overview

The **Research Laboratory Management System (RLMS)** automates and unifies the complex workflows of multi-million-dollar scientific research facilities:
- **Experiment Lifecycle Tracking**: Formulate protocols, submit scheduling requests, declare safety hazards, and track progress from proposal through data collection.
- **Strict Double-Booking & Conflict Prevention Engine**: Automatically validates proposed time slots against confirmed and pending bookings to eliminate instrumentation scheduling conflicts.
- **Lab Manager Review & Approval Queue**: Streamlined queue where core facility managers review requests, evaluate session requirements, and record decision notes.
- **Shared Master Experiment Calendar**: Interactive month and agenda schedules color-coded by operational status, filterable by project, instrument, and researcher.
- **Detector Data Library & Metadata Repository**: Decouples physical detector files from searchable instrument parameters (kV, resolution, exposure, temperature, detector type, run number, sample ID) with SHA-256 integrity digests.
- **Interdisciplinary Project Collaboration**: Interactive Kanban/checklist task management, collaborative lab notebook entries, protocol amendments, and team directories.
- **Technician Maintenance & Downtime Hub**: Instrument status overrides, preventative calibration logs, emergency repair records, spare parts tracking, and downtime telemetry.
- **Utilization & Activity Analytics**: Comprehensive facility capacity utilization heatmaps, runtime metrics, project throughput, and 1-click CSV report export.
- **Institutional Role Governance (RBAC)**: Complete user directory, dynamic role assignments, project charters, and equipment catalog management.

---

## 👥 User Roles & Permissions Matrix

| Capability / Privilege | Researcher | Lab Manager | Technician | Principal Investigator | Administrator |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Create & Edit Experiments** | ✅ | ❌ | ❌ | ✅ | ✅ |
| **Request Instrument Reservations** | ✅ | ✅ | ❌ | ✅ | ✅ |
| **Approve / Reject Bookings** | ❌ | ✅ | ❌ | ❌ | ✅ |
| **Log Decision Notes** | ❌ | ✅ | ❌ | ❌ | ✅ |
| **Register & Upload Detector Data** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Update Equipment Status** | ❌ | ✅ | ✅ | ❌ | ✅ |
| **Record Maintenance & Downtime** | ❌ | ✅ | ✅ | ❌ | ✅ |
| **Create Project Tasks & Notes** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Export Facility Reports & CSV** | ❌ | ✅ | ❌ | ✅ | ✅ |
| **Manage Users & Role Assignments** | ❌ | ❌ | ❌ | ❌ | ✅ |
| **Catalog Instruments & Projects** | ❌ | ❌ | ❌ | ❌ | ✅ |

### Authentication & Account Management
- **Default Entry**: Opening the application always presents the **Login Page** directly (no automatic login bypass or demo persona shortcuts).
- **Sign In Tab**: Enter registered institutional email and security password to log into the facility.
- **Create New Account (Registration)**: Self-service registration for new researchers, technicians, managers, PIs, or administrators with full name, institutional email, department, academic title, and password validation.
- **Session Logout**: Immediate **Sign Out** button in the top navigation bar, in the user profile menu, and in the sidebar footer. Logging out returns the user to the login screen.

---

## 🔬 Core Facility Instrumentation Included

The system comes pre-seeded with realistic fictional high-end scientific equipment:
1. **Titan Krios G4 300kV Cryo-TEM** (Room 102 - Cryo Suite) — Falcon 4i direct electron detector, Selectris X energy filter, 1.2Å Nyquist limit.
2. **Orbitrap Exploris 480 Mass Spectrometer** (Room 204 - Analytical Lab) — HESI & APCI sources, 480,000 FWHM resolution.
3. **Leica Stellaris 8 STED Super-Resolution Confocal** (Room 108 - Optical Imaging) — Sub-30nm lateral resolution, 5x HyD detectors.
4. **Rigaku SmartLab 9kW X-Ray Diffractometer** (Room 305 - XRD Suite) — 9kW rotating Cu anode, HyPix-3000 2D hybrid pixel detector.
5. **Quantum Design PPMS DynaCool Cryostat** (Room 012 - Cryogenics) — ±9 Tesla superconducting split coil, 1.8K to 400K range.
6. **Beckman Coulter Optima XPN-100 Ultracentrifuge** (Room 114 - Wet Lab) — 100,000 RPM (802,400 x g) induction drive.

---

## 🛡️ Equipment Double-Booking Prevention Engine

The conflict detection engine is implemented in `src/services/db.ts`:
```typescript
// Overlap formula: (NewStart < ExistingEnd) && (NewEnd > ExistingStart)
public checkEquipmentConflict(
  equipmentId: string,
  startTime: string,
  endTime: string,
  excludeBookingId?: string
): EquipmentConflict | null
```
- **Real-Time Validation**: Evaluates schedule conflicts during experiment request creation, editing, direct reservation, and manager approval.
- **Maintenance Lock**: Blocks booking requests immediately if the instrument status is marked as `under maintenance` or `unavailable`.
- **Descriptive Diagnostics**: Returns the exact conflicting experiment title, reserving researcher name, and occupied time window.

---

## 🚀 Getting Started & Setup Instructions

### Prerequisites
- Node.js (v18.0 or later recommended; tested on v20 and v26)
- npm or pnpm

### Installation
```bash
# Navigate to the project directory
cd research-lab-system

# Install dependencies
npm install

# Run the local development server
npm run dev

# Or build for production and preview
npm run build
npm run preview
```

The application will be served at `http://localhost:3000` (or `http://localhost:5173`).

---

## 🔌 Production Database & Backend Integration Guide

The demo data layer is decoupled and abstracted in `src/services/db.ts` using the singleton pattern. To connect a production backend:

1. **Replace `LocalStorage` with an API Client**:
   - Replace the internal `state` and `localStorage` calls with standard REST (`fetch` / `axios`) or GraphQL mutations.
2. **PostgreSQL Relational Schema Mapping**:
   - `users` ➔ `users` table (`id`, `name`, `email`, `role`, `department`, `avatar`)
   - `projects` ➔ `projects` table (`id`, `code`, `name`, `lead_id`, `budget_code`)
   - `equipment` ➔ `equipment` table (`id`, `name`, `category`, `status`, `lab_room`, `specs`)
   - `experiments` ➔ `experiments` table (`id`, `code`, `title`, `project_id`, `lead_id`, `schedule_start`, `schedule_end`, `status`)
   - `bookings` ➔ `bookings` table (`id`, `equipment_id`, `experiment_id`, `researcher_id`, `start_time`, `end_time`, `status`, `decision_notes`)
   - `detector_data` ➔ `detector_datasets` table (`id`, `run_number`, `sample_id`, `file_format`, `file_url`, `raw_data_checksum`, `instrument_parameters`)
   - `maintenance_records` ➔ `maintenance_records` table (`id`, `equipment_id`, `technician_id`, `type`, `downtime_hours`, `cost`, `notes`)
3. **Object Storage Integration**:
   - In `DetectorDataPage.tsx`, wire the dropzone directly to AWS S3, Google Cloud Storage, or MinIO pre-signed URLs.

---

## 📄 License
Designed for research institutions, core facilities, and universities. Released under the MIT License.
