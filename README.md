# Alerts-Overhaul

[VCF-Operations-Alerts-Export.xlsx](VCF-Operations-Alerts-Export.xlsx) is an export of the alert content in a VCF Operations instance. It was read through the VCF Operations suite API, and nothing on the instance was changed.

| Tab | Contents | Rows |
|---|---|---|
| Alerts | Alert definitions, with their symptoms and recommendations | 1,193 |
| Symptoms | Symptom definitions | 1,951 |
| Recommendations | Recommendations, with the alert definitions that use them | 931 |
| Actions | Action definitions | 147 |

## Notes on the data

- **Symptoms tab:** It combines all four symptom tabs in the VCF Operations UI (Metric/Property, Message Event, Fault and Logs). The UI shows each tab separately, so its Metric/Property tab alone has fewer rows than this tab.
- **Criticality "AUTO":** Many alerts show AUTO. This means the criticality is taken from their symptoms. Only alerts with a fixed criticality show a value such as CRITICAL.
- **Alert type and subtype:** The API returns these as numbers. They were mapped to names (Hardware, Availability and so on) by inference. Four alerts were spot-checked against the UI and match, but the mapping has not been verified for every alert. If a type or subtype looks wrong, compare it with the UI.
- **Not included:** The Defined By, Last Modified and Modified By columns shown in the UI are not available from the API, so they are not in the workbook.
