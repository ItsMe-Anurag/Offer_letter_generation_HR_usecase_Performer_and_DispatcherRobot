# Offer Letter Generation HR Use Case – Performer & Dispatcher Robot

A UiPath-based automation solution for generating offer letters in an HR workflow. This project splits responsibilities between a **Performer** robot (which does the actual document generation) and a **Dispatcher** robot (which loads, dispatches, and handles queueing logic).

---

## Table of Contents

- [Overview](#overview)  
- [Architecture & Components](#architecture--components)  
- [Folder Structure](#folder-structure)  
- [Prerequisites](#prerequisites)  
- [Setup & Configuration](#setup--configuration)  
- [How It Works](#how-it-works)  
- [Error Handling & Logging](#error-handling--logging)  
- [Testing](#testing)  
- [Limitations & Future Improvements](#limitations--future-improvements)  
- [License](#license)  

---

## Overview

When a candidate is marked as “Hired” in an Excel sheet or data source, the Dispatcher robot picks up that record, adds it into an Orchestrator queue (or equivalent transaction store). Then the Performer robot processes each transaction, generates the offer letter (based on template + input data), and writes output/status. The architecture allows scaling, retry logic, and separation of duties.

---

## Architecture & Components

| Component | Role |
|-----------|------|
| **Dispatcher_HR_Robot** | Reads from data source (Excel), filters “Hired” candidates, enqueues transactions into `OFFER_LETTER_QUEUE`. |
| **Performer_HR_OfferLetter** | Processes each candidate transaction: reads config, generates documents, writes output. Uses REFramework structure. |
| **Framework** (common to Performer) | ReUsable workflows: initialization, transaction handling, logging, error handling. |
| **Config & Assets** | Config.xlsx holds settings/constants; Orchestrator assets can override config values. |

This separation allows the dispatcher to focus solely on queue management, while the performer handles the heavy-lifting of document generation.

---

## Folder Structure
```
├── Dispatcher_HR_Robot
│ ├── Main.xaml
│ └── project.json
├── Performer_HR_OfferLetter
│ ├── Data
│ │ ├── Config.xlsx
│ │ ├── Input
│ │ ├── Output
│ │ └── Temp
│ ├── Documentation
│ ├── Exceptions_Screenshots
│ ├── Framework
│ ├── Main.xaml
│ ├── Tests
│ └── project.json
└── LICENSE
```

A few notes:

- `Data/Input`, `Data/Output`, `Data/Temp` are used for staging inputs and outputs.
- `Exceptions_Screenshots` is for storing screen captures in case of failures.
- `Framework` contains reusable workflows (Init, GetTransactionData, retries, etc.).
- `Tests` contains test cases / XAML workflows to validate modules.

---

## Prerequisites

- UiPath Studio (compatible version)  
- Excel activities & Mail activities packages  
- Access to Orchestrator (if queue-based approach used)  
- Permissions to read/write Excel and files  
- The repository’s config file (`Config.xlsx`) must be updated for your environment (sheet names, paths, Orchestrator queue name, retry counts, etc.)

---

## Setup & Configuration

1. Clone or download this repository.
2. Open the `Performer_HR_OfferLetter` and `Dispatcher_HR_Robot` projects in UiPath.
3. Update `Data/Config.xlsx`:
   - Define settings (e.g. retry counts, file paths, queue names, sheet names).
   - Use two sheets (e.g. `Settings`, `Constants`) as expected by the Init workflow.
4. If using Orchestrator:
   - Create a queue named as per `Config.xlsx` (e.g., `OFFER_LETTER_QUEUE`).
   - Define assets (if overriding config values).
5. In Dispatcher, configure the data source path (Excel) or source from which “Hired” rows will be picked.
6. Publish and deploy both robots to your UiPath Orchestrator (if using).
7. Run the Dispatcher first; then Performer will pick up the queued items.

---

## How It Works

### Dispatcher Robot

- Opens the Excel sheet containing candidate statuses.
- Reads all rows from a “Status” sheet.
- Filters rows where `Status == "Hired"`.
- Adds each filtered row as a transaction in the `OFFER_LETTER_QUEUE`.

### Performer Robot (REFramework-based)

1. **Init**  
   - Reads configuration from `Config.xlsx` and Orchestrator assets.
   - Initializes logging, applications, and required settings.

2. **Get Transaction Data**  
   - Retrieves a transaction from the queue.
   - If none left → process ends.
   - Extracts fields needed for document generation.

3. **Process**  
   - Uses the input data to generate the offer letter (e.g. populate a Word or PDF template).
   - Writes the generated file to `Data/Output`.
   - Updates status/metadata (maybe in Excel or via logs).

4. **Set Transaction Status**  
   - Marks transaction as successful or failed.
   - Logs errors, takes screenshot if needed.

5. **Close Applications**  
   - Cleanup: close opened applications, release resources.

---

## Error Handling & Logging

- Uses robust Try/Catch blocks and retry mechanisms (in REFramework).
- On exception, a screenshot is captured and stored under `Exceptions_Screenshots`.  
- Logs capture details of which transaction failed and why.
- Retry counts can be configured in `Config.xlsx`.

---

## Testing

- `Tests` folder contains test workflows / templates:
  - `GeneralTestCase.xaml`
  - `GetTransactionDataTestCase.xaml`
  - `InitAllSettingsTestCase.xaml`
  - … etc.
- Use these to validate individual components before running full end-to-end.
- Update test data/sample Excel sheets to match your environment.

---

## Limitations & Future Improvements

- Currently assumes Excel + local file system. Could be extended to databases or web APIs.
- Offer letter templating is presumably simplistic; more complex templates or mail-merge logic could be added.
- Better exception categorization (e.g. distinguish transient vs fatal).
- Add notification (e.g. email) when process completes or fails.
- Parallel processing / scaling enhancements.

---

## License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

## Acknowledgements

- Leverages UiPath’s REFramework as the structural foundation.
- Inspired by common HR automation use cases (offer letter generation, queue-based orchestration).

---


