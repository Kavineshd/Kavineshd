# logic Diagram: `commonFn.utils.js` Full System

This document contains a comprehensive sequence and logic diagram showing exactly how the student engagement engine operates from start to finish.

## 1. Complete System logic Chart
This diagram tracks a student's journey from answering a question to seeing their updated status on the dashboard.

```mermaid
flowchart TD
    subgraph "Phase 1: Interaction (Mutation)"
        A[Student Submits Answer] --> B{Is Correct?}
        B -->|Yes| C[saveStudentLeague: Add XP]
        B -->|No| D[saveStudentLeague: Deduct XP]
        
        C & D --> E[Check Achievement Eligibility]
        E --> F[Fetch Student totalxp]
        F --> G{Hits Milestone?}
        G -->|Yes| H[Insert New Achievement Records]
        G -->|No| I[Continue]
    end

    subgraph "Phase 2: Background Processing"
        H & I --> J[Normalise Dates to IST]
        J --> K[Calculate New Streak Count]
    end

    subgraph "Phase 3: Dashboard Aggregation (Read)"
        L[Dashboard Request] --> M[getDashboardLayoutData]
        M --> N{Parallel Queries}
        
        N --> O1[Fetch Hearts & Stats]
        N --> O2[Calculate Subject % Progress]
        N --> O3[Fetch Sector & Badge List]
        N --> O4[Calculate Daily 10XP Progress]
        N --> O5[Fetch League Rank & Info]
        
        O1 & O2 & O3 & O4 & O5 --> P[Aggregate Data Object]
    end

    H -->|Badge Earned| L
    K -->|Streak Updated| L
    P --> Q[Send to UI]
```

## 2. Sequence Diagram (Data Flow)
This diagram shows the order of operations between the Application, the Utility Functions, and the Database.

```mermaid
sequenceDiagram
    participant App as Frontend / Mobile
    participant Utils as commonFn.utils
    participant DB as MySQL Database

    Note over App, DB: Student Interaction Flow
    App->>Utils: saveStudentLeague(sid, xp, isCorrect)
    Utils->>DB: SELECT current_league
    DB-->>Utils: league_data
    Utils->>DB: UPDATE studentleagues (new XP)
    
    Utils->>Utils: saveStudentAchievements(sid)
    Utils->>DB: SELECT totalxp, earned_badges
    DB-->>Utils: xp_info, badgelist
    Utils->>DB: INSERT NEW badges (if eligible)

    Note over App, DB: Dashboard Refresh Flow
    App->>Utils: getDashboardLayoutData(sid)
    par Parallel Queries
        Utils->>DB: Fetch Stats & XP
        Utils->>DB: Fetch Subjects & Progress
        Utils->>DB: Fetch Achievement History
        Utils->>DB: Fetch League Standing
    end
    DB-->>Utils: All Data Results
    Utils->>Utils: Calculate Overall Streak
    Utils->>Utils: Get Weekly Day-Map
    Utils-->>App: Return Full Dashboard JSON
```

---

## 3. Key Logic Branching
| Event | Trigger | Condition | Result |
| :--- | :--- | :--- | :--- |
| **League Update** | `saveStudentLeague` | `iscorrect = true` | `XP = XP + Rewards` |
| **League Penalty** | `saveStudentLeague` | `iscorrect = false` | `XP = max(0, XP - Rewards)` |
| **Achievement** | `saveStudentAchievements` | `minxp <= totalxp <= maxxp` | New row in `studentachievements` |
| **Streak Break** | `getOverallStreak` | `PrevDay` missing in Activity | Streak resets/stops at current count |
| **Daily Goal** | `getDashboardLayoutData` | `TodayXP >= 10` | `dailyProgressPercent = 100%` |
