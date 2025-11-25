# Document A — Chromatography Transition Analysis (Time-First BA Report)

## 1. Purpose
Evaluate the time savings achieved by moving Protein A chromatography transition analysis from post-run, file-based workflows to a centralized streaming analytics platform. Financial translation appears only at the end.

## 2. Scope & Assumptions
- 50 non-identical Protein A chromatography units.
- Per-second data streaming available from historian.
- Current manual transition analysis: 8–10 hours/week per unit.
- Future-state automated analysis: ~1 hour/week per unit.
- 2,000 working hours/year per FTE.
- No dollar conversion until the final section.

## 3. Current-State Activities & Time Burden (File-Based)
### Activities per unit:
1. End-of-run file extraction and verification.
2. Data cleaning, filtering, timestamp alignment.
3. Manual transition segmentation (load→wash→elute).
4. KPI calculation: HETP, asymmetry, peak width.
5. Manual plotting and trending across cycles.
6. Troubleshooting inconsistent or missing data.
7. Report assembly and validation.

**Time per unit:** 8–10 hrs/week  
**Across 50 units:** 400–500 hrs/week  
**Annual load:** 20,000–25,000 hrs/year

This is equivalent to **10–12.5 full-time persons** worth of manual analytics effort.

## 4. Future-State Activities & Time Burden (Streaming Platform)
### Activities per unit:
1. Automated ingestion of per-second data.
2. Automated transition detection.
3. Automated KPI generation and trending.
4. Exception review only.

**Time per unit:** ~1 hr/week  
**Across 50 units:** 50 hrs/week  
**Annual load:** 2,500 hrs/year  
Equivalent to **1.25 FTE**.

## 5. Time-Based Value Summary (Hours Only)
| Category | Current State | Future State | Time Saved |
|----------|----------------|--------------|-------------|
| Weekly | 400–500 hrs | 50 hrs | **350–450 hrs/week** |
| Annual | 20,000–25,000 hrs | 2,500 hrs | **17,500–22,500 hrs/year** |

At this stage, we have **massive time liberation** without discussing money:  
A reduction of approximately **9–11 FTEs** worth of work annually.

## 6. Financial Translation (Final Section Only)
Assume:
- Fully loaded cost: $50/hr (from $100k/year salary ÷ 2,000 hours/year)
- Batch value: $3M
- Likely avoidance of ≥1 failed batch/year

**Labor Savings**  
17,500–22,500 hrs/year × $50/hr → **$875k–$1.125M saved/year**

**Batch Failure Avoidance**  
1 avoided failure × $3M = **$3M/year**

**Total Financial Benefit**  
≈ **$3.9M–$4.1M/year**

## 7. Conclusion
The centralized streaming solution eliminates nearly all manual transition-analysis labor and enables earlier detection of chromatography failures. Time savings alone justify the transformation; the financial impact (~$4M/year) reinforces it as a high-value modernization initiative.
