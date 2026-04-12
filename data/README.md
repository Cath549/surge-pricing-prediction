# Data

## Sources

- **NYC TLC Trip Record Data**: https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page
- **Weather data**: To be determined during project setup
- **Holiday calendar**: To be engineered from public holiday lists

## Notes

- Raw data files are NOT committed to this repository (too large)
- See `.gitignore` for excluded paths
- To reproduce: download data using instructions in the notebooks

## Problem Statement

We predict whether ride-hailing demand in a given city zone will surge beyond its historical normal level within the next 15–30 minutes. The operations team uses these predictions to proactively reposition nearby idle drivers toward high-demand zones and optionally notify riders of expected price increases. False negatives (missing a real surge) are costlier than false positives (false alarm), because a missed surge causes long rider wait times, customer churn, and lost revenue — while a false positive only causes minor driver repositioning overhead.
