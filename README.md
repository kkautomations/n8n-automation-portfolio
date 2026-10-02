# n8n Automation Portfolio

Production-oriented n8n workflows built around data synchronisation,
input validation, error handling and LLM integration.

---

## 1. Contact pipeline with validation and error handling

**Problem:** A business collects enquiries through a web form. Contacts
need to be stored without duplicates, invalid data must be kept out of
the database, and failures need to be visible rather than silent.

**How it works**

- Form submission triggers the workflow
- Input is validated before anything is written
- Valid submissions are searched for in Airtable by email
  - Existing contact → record updated, submission count incremented
  - New contact → record created
- Invalid submissions are written to a separate Rejections table with
  the reason and a timestamp, and an alert email is sent
- Every node that calls an external service retries three times
  before failing
- A separate error-handler workflow emails on any unexpected failure

**Design decisions**

- Search-before-write rather than blind create, to prevent duplicate
  contacts on repeat submissions
- Rejections stored in their own table so they remain queryable, rather
  than only existing as an email
- Retries set to three attempts at two-second intervals to absorb
  transient API outages
- Error handling split into two layers: expected failures (validation)
  handled inline, unexpected failures caught by a global error workflow

**Next steps**

- Duplicate-submission protection (idempotency key)
- Normalising email case before matching
- Reporting on rejection reasons over time

**Files:** `06 - airtable upsert.json`, `99 - error handler.json`

---

## 2. AI enquiry classifier

**Problem:** Incoming enquiries arrive as free text and need routing to
the right team without someone reading each one.

**How it works**

- Form collects an email address and a free-text message
- Google Gemini classifies the message into sales, support or spam,
  assigns an urgency level, and returns a one-line summary
- The model is prompted to return JSON only
- A Code node strips any markdown formatting and parses the response,
  falling back to an `unknown` classification if parsing fails
- A Switch node routes each category to its own branch, with a fallback
  output for unmatched or unparseable results

**Design decisions**

- The LLM only converts unstructured text into structured data; all
  routing decisions are made deterministically afterwards
- Defensive parsing, because model output is probabilistic and will
  occasionally deviate from the requested format
- Fallback output on the Switch so unclassifiable messages are surfaced
  rather than silently dropped

**Next steps**

- Logging low-confidence classifications for human review
- Measuring classification accuracy against a labelled sample

**Files:** `08 - AI classifier.json`

---

## Stack

n8n (self-hosted), Airtable, Google Sheets, Gmail API, Google Gemini,
JavaScript
