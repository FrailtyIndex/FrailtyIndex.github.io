---
layout: post
title: First meeting of the site informatics teams.
---

Today at 3pmEST/2pmCST, Nicholas Pajewski (WFU), Adam Moses (WFU), George Golovko (UTMB), Gerald Brown (UTMB), Kathleen Stevens (UTHealth SA), and Alex Bokov (UTHealth SA) met by phone to learn from the WFU team some of the technical details of how their process works.

### Q&A

This is a highly abbreviated summary, in a question-answer format, with lots of paraphrasing and probably errors, all of them the fault of me, @AB (Alex). :-)

<dl> 
  <dt><b>Q: What are the main steps in the processing of Clarity data by WFU?</b></dt>
  <dd>A: There is a weekly process that dumps certain Clarity tables which get extracted by @AM's SQL scripts and the resulting staging tables are then processed by @NP's SAS scripts. The final result is a two-column flat file. The first column is each patient's MRN and the second column is their eFI. This file is uploaded back into Epic via a datalink job and used to populate a custom, patient-level SmartData element. In Epic, SmartData elements can be inserted almost anyplace. Currently the eFI scores are inserted into the scheduling view for outpatient visits. Efforts are underway to incorporate them into the Clinical Overview, a patient-specific report.</dd>
  <dt><b>Q: How long (calendar time) did it take for the Epic team to set up their part of this process, and how many hours of effort would you guesstimate it took them once this got to the top of their queue?</b></dt>
  <dd>A: One year to roll out, roughly 40 hours of building and testing.</dd>
  <dt><b>Q: Are all the scripts required for other sites?</b></dt>
  <dd>A: No, `Provider_Department_Mapping`, `Patient_Registries`, and `HAR_Records` are specific to WFU and can be skipped.</dd>
  <dt><b>Q: What does the 'EPT 30' comment in the `Patient_Problems` script refer to?</b></dt>
  <dd>A: EPT is the abbreviation for the patient master file in Epic Chronicles, and EPT 30 is the data item in that file which shows what types of enounters exist.</dd>
  <dt><b>Q: Why don't you use the MED_HX diagnoses?</b></dt>
  <dd>A: Because those aren't always reliable. Even though they are coded, they are provided by the patient themselves. Also, once such a diagnosis is added, it tends to remain there whether or not it still applies.</dd>
  <dt><b>Q: What is the overall behavior of all the SQL scripts and what are the script-specific features that were commented on?</b></dt>
  <dd>A: Each script is for a different category of data element, indicated by its name. Each script does a join on several tables usually including patient encounter, the tables specific to the scripts data element/s, and some mapping tables (with the `ZC_` prefix) which link numeric codes to human-readable names. Then the result-set is filtered for the desired date-interval, encounter type (to exclude purely administrative encounters where the patient is not physically present), and appointment status (to insure that only completed appointments are pulled).</dd>
  <dt><b>Q: In `Patient_Meds` how do we avoid over-reporting polypharmacy due to different versions of the (nearly) same drug having different IDs?</b></dt>
  <dd>A: This is a work in progress. There is a manually curated table that helps mitigate this.</dd>
  <dt><b>Q: For `Patient_Labs` might it make things easier to use Epic's value-flags instead of hard-coded lab-specific thresholds?</b></dt>
  <dd>A: Maybe.</dd>
</dl>

### Script-specific notes

* **AWV_Data**: In addition to date, encounter type, and appointment status also filters on specific flow measure IDs. Data from only the latest annual wellness visit (not all AWVs from last 2 years).
* **Patient_Problems**: These are problem-list diagnoses. The `Noted_Date` column is to document retroactive addition of diagnoses by the provider after the visit.
* **Patient_Meds**: Only filtered on date. Over preceding _3 months_ (not 2 years) to latest visit, the number of simultaneously active prescriptions for that patient is calculated (for each day?) taking into account the start and end of each prescription. The maximum such number is used to determine whether they are in a condition of polypharmacy (not sure I got this right)
* **Patient_Labs**: Inpatient and ED labs are filtered out to avoid values that are non-representative of the patients' normal state.
* **Patient_Encounters**: These are the encounter diagnoses (as opposed to `Patient_Problems`). Also smoking and vitals.
* **Patient_Demographics**: 

### Next steps

* **WFU team:** Kindly agreed to share a mapping table of flowsheet IDs and labels to allow other sites to figure out what their corresponding flowsheet IDs should be, as well as a copy of the medication mapping table.
* **@AB:** Will send out the files previously shared by @NP and @AM to UTMB team, and write up this meeting summary. Will schedule a follow-up meeting to go over the SAS portion of the process.


