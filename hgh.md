# Calculation Logic: `commonFn.utils.js`

This document isolates and explains the mathematical formulas and logic used across the utility system for scoring, streaks, and progress.

---

## 1. Time & Week Range Calculation
Used in `getWeekRangeIST` and `getWeeklyCompletedLevelsActivity`.

### Logic Step:
1.  **Current Day Reference:** `day = now.getDay()` (0 for Sunday, 1 for Monday, etc.)
2.  **Offset to Monday:** 
    *   If Sunday (0): `diff = -6` (go back 6 days)
    *   Else: `diff = 1 - day`
3.  **Start Boundary:** `weekStart = now + diff days` at `00:00:00.000`
4.  **End Boundary:** `weekEnd = weekStart + 7 days`

**Formula:**
$$WeekStart = Date(Now - ((DayOfWeek == 0 ? 6 : DayOfWeek - 1) \times 24h))$$

---

## 2. Streak Calculation (Consecutive Days)
Used in `getTotalSteaks` and `getOverallStreak`.

### The "Day-Step" Algorithm:
The system uses the constant `ONE_DAY = 86,400,000` milliseconds.

1.  **Input:** A list of unique days (timestamps) sorted descending $[D_0, D_1, D_2, ... D_n]$.
2.  **Initial State:** `streak = 1`, `current = D_0`.
3.  **Iteration:** For every next day $D_i$:
    *   **Check:** Does $D_i == (current - 86,400,000)$?
    *   **If Yes:** `streak++`, `current = D_i`.
    *   **If No:** Break loop.

---

## 3. League XP Update logic
Used in `saveStudentLeague`.

### Conditional XP formula:
The change in XP depends on whether the student's answer was correct (`iscorrect`).

*   **If Correct:** $XP_{new} = XP_{existing} + XP_{reward}$
*   **If Incorrect:** $XP_{new} = \max(0, XP_{existing} - XP_{penalty})$

---

## 4. League Level Matching
Used in `getStudentLeague`.

### JSON Range Parsing:
The system parses a JSON array of point ranges $[R_0, R_1, ...]$. Each range has a `minxp` and `maxxp`.

**Matching Condition:**
Find Range $R$ where:
$$R.minxp \leq StudentXP \leq R.maxxp$$

---

## 5. Subject Progress Percentage
Used in `getDashboardLayoutData`.

### Formula:
The progress for a subject is the ratio of correctly answered unique questions to the total unique questions mapped to that subject.

$$Progress\% = \text{round}\left( 100 \times \frac{\sum \text{Correct Responses}}{\sum \text{Total Questions in Level Mapping}} \right)$$

---

## 6. Daily Challenge Completion
Used in `getDashboardLayoutData`.

### Goal Logic:
The target is fixed at **10 XP** per day.

1.  **Today's XP:** Sum of `xpearned` where `DATE(createdat) == TODAY`.
2.  **Completion Percent:**
    *   Formula: $Percent = \min(100, \text{round}((\text{todayXp} / 10) \times 100))$

---

## 7. Achievement Milestone Check
Used in `saveStudentAchievements`.

### Eligibility Logic:
A badge is awarded if the student's **Total XP** sits within the achievement's defined brackets.

**Logic:**
Select `achievementId` where:
1.  `minxp <= student.totalxp`
2.  `maxxp >= student.totalxp`
3.  `achievementId` NOT IN `earned_achievement_list`
