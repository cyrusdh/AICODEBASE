Located:  https://script.google.com/u/0/home/projects/1Z6IqyuvG1qmG8suPmFzROzI7OUHD9jL3l7pBhzwEpFndFg5tFMhC3XET/edit

## PROJECT DESCRIPTION

AI LEGAL GOD is an advanced legal document management and analysis system built on Google Apps Script. It automates the processing of legal case documents, generating AI-powered summaries, case briefs, and chronological timelines to help attorneys quickly understand complex cases.

## PROGRAMMING LANGUAGES

- JavaScript (Google Apps Script)
- HTML
- Google APIs (Drive, Sheets, Docs)

## ARCHITECTURE OVERVIEW

The system consists of interconnected components that work together to process, analyze, and organize legal documents:

```
┌─────────────────┐       ┌────────────────────┐       ┌─────────────────┐
│                 │       │                    │       │                 │
│    Index.html   │━━━━━━▶│      Code.gs       │━━━━━━▶│ SheetFunctions  │
│   (Web UI)      │       │  (Main Controller) │       │ (Sheet Mgmt)    │
│                 │       │                    │       │                 │
└─────────────────┘       └────────────────────┘       └─────────────────┘
                                    ┃                           ┃
                                    ▼                           ▼
                          ┌────────────────────┐      ┌─────────────────────┐
                          │                    │      │                     │
                          │ DocumentProcessing │◀━━━━▶│  CaseBriefFunctions │
                          │  (Doc Analysis)    │      │  (Brief Generation) │
                          │                    │      │                     │
                          └────────────────────┘      └─────────────────────┘
                                    ┃                           ┃
                                    ▼                           ▼
                          ┌────────────────────┐      ┌─────────────────────┐
                          │    Google Drive    │      │     Gemini API      │
                          │   (File Storage)   │      │      (AI Model)     │
                          └────────────────────┘      └─────────────────────┘
                                                              ┃
                                                              ▼
                                                      ┌─────────────────────┐
                                                      │      Zoho CRM       │
                                                      │   (Case Metadata)   │
                                                      └─────────────────────┘
```

## COMPONENT INTERACTIONS

1. **User Interface (Index.html)**
    
    - Provides a simple web interface for triggering the document processing and case brief generation
    - Accepts a Google Drive folder ID as input
    - Communicates with Code.gs to execute operations
2. **Main Controller (Code.gs)**
    
    - Acts as the central orchestrator for all system operations
    - Handles HTTP requests from the web interface
    - Manages workflow execution, Zoho token refreshing, and error handling
    - Initiates document processing and case brief generation
3. **Document Processing (DocumentProcessing.gs)**
    
    - Manages the workflow for analyzing legal documents
    - Implements batch processing with state preservation for handling long-running tasks
    - Integrates with SheetFunctions.js to organize documents in spreadsheets
    - Calls Gemini API to generate document summaries
4. **Sheet Management (SheetFunctions.js)**
    
    - Creates and manages the spreadsheet structure for organizing document analyses
    - Implements the tab mapping system for categorizing different document types
    - Handles document metadata extraction and formatting
    - Provides logging functionality for system monitoring
5. **Case Brief Generation (CaseBriefFunctions.js)**
    
    - Generates comprehensive case briefs from document analyses
    - Creates chronological timelines of case events
    - Integrates with Zoho CRM to retrieve case snapshots
    - Uses Gemini AI for generating strategic insights and recommendations

## KEY FEATURES

- Automated document classification and organization
- AI-powered document summarization and analysis
- Chronological case timeline generation
- Integration with Zoho CRM for case metadata
- State persistence for handling large document sets
- Comprehensive logging and error tracking
- Web interface for easy operation

## IMPLEMENTATION HIGHLIGHTS

- Uses Google's Gemini AI for intelligent document analysis
- Implements a sophisticated document categorization system
- Handles batch processing to overcome Apps Script time limitations
- Maintains processing state between executions
- Generates Google Docs with formatted case briefs and timelines

## USAGE EXAMPLES

### Basic Usage

1. Open the web interface (published as a Google Apps Script web app)
2. Enter a Google Drive folder ID containing case documents
3. Click "Process Docs" to analyze and organize documents
4. Click "Generate Case Brief" to create a comprehensive case brief

### Advanced Usage

- Modify tab mapping in SheetFunctions.js to customize document categorization
- Adjust Gemini prompts in DocumentProcessing.gs to tailor AI analysis
- Configure CaseBriefFunctions.js parameters to customize brief generation

## LESSONS LEARNED

- **Successes**:
    
    - Effective integration of AI for legal document analysis
    - Robust state management for processing large document sets
    - Clean separation of concerns between components
- **Challenges**:
    
    - Google Apps Script execution time limitations
    - Complex document categorization requirements
    - Handling diverse document formats
- **Solutions**:
    
    - Implemented batch processing with state persistence
    - Created a flexible tab mapping system for document organization
    - Used multiple AI prompts optimized for different document types
- **Technical Insights**:
    
    - Google Apps Script works well for document automation but requires careful handling of execution time limits
    - Structured prompting strategies significantly improve AI analysis quality
    - Integration between Google Workspace and external APIs (Zoho, Gemini) requires careful token management

---

This overview serves as a guide to understanding how the components of the AI LEGAL GOD system work together. For detailed code and implementation details of each component, refer to the individual file sections in this document. below. 
# ###Code.gs

## Overview
This file acts as the main controller for the "AI LEGAL GOD" web app, managing HTTP requests, document processing, and case brief generation. It includes utility functions for Zoho token management, testing, and state/trigger cleanup.

## Functions

### `doGet(e)`
- **Purpose**: Handles HTTP GET requests to serve the web app UI or test logging.
- **Parameters**: `e` (event object with query parameters).
- **Behavior**:
  - Sets `currentFolderId` from `e.parameter.folderId`.
  - Logs start via `Logger` and `logInfo`.
  - If `action` is "testLogging", returns the result of `testLogging`.
  - Otherwise, serves `Index.html` with the title "AI LEGAL GOD".
- **Returns**: `ContentService.TextOutput` or `HtmlService.HtmlOutput`.

### `processWithFolderId(folderId)`
- **Purpose**: Processes litigation documents for a given folder with locking to prevent overlaps.
- **Parameters**: `folderId` (Google Drive folder ID).
- **Behavior**:
  - Sets `currentFolderId`, logs start, and uses `LockService` to ensure single execution.
  - Clears old processing states except for the current `folderId`.
  - Initializes log sheet, fetches case insights, processes documents, and returns the result.
  - Logs errors and flushes logs on failure or completion.
- **Returns**: Result of `processDocuments` or throws an error.
- **Dependencies**: `LockService`, `PropertiesService`, `initializeLogSheet`, `processAttorneysTab`, `processDocuments`.

### `generateCaseBrief(folderId)`
- **Purpose**: Generates a case brief for a specified folder.
- **Parameters**: `folderId` (Google Drive folder ID).
- **Behavior**:
  - Sets `currentFolderId`, logs start, initializes log sheet, and calls `CaseBriefFunctions.generateCaseBrief`.
  - Logs errors and flushes logs on failure or completion.
- **Returns**: Result of `CaseBriefFunctions.generateCaseBrief` or throws an error.
- **Dependencies**: `CaseBriefFunctions.generateCaseBrief`.

### `saveZohoTokens()`
- **Purpose**: Stores hardcoded Zoho OAuth and refresh tokens in script properties.
- **Behavior**: Sets `ZOHO_OAUTH_TOKEN` and `ZOHO_REFRESH_TOKEN` using `PropertiesService`, logs success.
- **Dependencies**: `PropertiesService`.

### `testGenerateCaseBrief()`
- **Purpose**: Tests `generateCaseBrief` with a hardcoded folder ID.
- **Behavior**: Calls `generateCaseBrief` with "1aXbgh-P3ups7tQjRiMIxa_fNn3UiXumo".
- **Dependencies**: `generateCaseBrief`.

### `updateScriptProperties()`
- **Purpose**: Updates script properties with Zoho credentials and logs success.
- **Behavior**:
  - Sets `ZOHO_OAUTH_TOKEN`, `ZOHO_REFRESH_TOKEN`, `ZOHO_CLIENT_ID`, `ZOHO_CLIENT_SECRET`.
  - Contains a nested `testLogging` function (likely a typo, should be separate).
- **Dependencies**: `PropertiesService`.

### `testLogging(folderId)` (Nested in `updateScriptProperties`)
- **Purpose**: Tests logging functionality.
- **Parameters**: `folderId` (Google Drive folder ID).
- **Behavior**: Logs start via `Logger` and `logInfo`, flushes logs, returns a success message.
- **Returns**: "Logging test completed".
- **Note**: Appears misplaced inside `updateScriptProperties`; should likely be a top-level function.

### `deleteAllResumeTriggers()`
- **Purpose**: Deletes all triggers for `resumeDocumentProcessing`.
- **Behavior**: Iterates through project triggers, removes those with handler `resumeDocumentProcessing`, logs via `console`.
- **Dependencies**: `ScriptApp`.

### `clearAllProcessingStates()`
- **Purpose**: Clears all document processing states from script properties.
- **Behavior**: Filters and deletes keys starting with "docProcessingState_", logs via `console`.
- **Dependencies**: `PropertiesService`.

## Dependencies
- **External Functions**: `logInfo`, `logError`, `flushLogs`, `initializeLogSheet`, `processAttorneysTab`, `processDocuments`, `CaseBriefFunctions.generateCaseBrief` (assumed in other files).
- **Services**: `Logger`, `HtmlService`, `ContentService`, `LockService`, `PropertiesService`, `ScriptApp`, `console`.

## Notes
- `currentFolderId` is a global variable set here, likely used by logging functions.
- `testLogging` nesting in `updateScriptProperties` seems unintentional and may need relocation.
- Functions rely on external implementations not shown here (e.g., `CaseBriefFunctions`).

## CODE

```
function doGet(e) {

  const folderId = e.parameter.folderId;

  currentFolderId = folderId; // Set currentFolderId before logging

  Logger.log("Starting doGet");

  logInfo("Starting doGet");

  flushLogs(folderId); // Pass folderId explicitly to flushLogs

  

  const action = e.parameter.action;

  

  if (action === "testLogging") {

    return ContentService.createTextOutput(testLogging(folderId));

  }

  

  return HtmlService.createHtmlOutputFromFile('Index')

    .setTitle('AI LEGAL GOD');

}

  

function processWithFolderId(folderId) {

  currentFolderId = folderId; // Set currentFolderId for logging

  Logger.log("Starting processWithFolderId with folder ID: " + folderId);

  logInfo("Starting processWithFolderId with folder ID: " + folderId);

  flushLogs(folderId);

  

  // Use LockService to prevent overlapping executions

  const lock = LockService.getScriptLock();

  try {

    if (!lock.tryLock(30000)) { // Wait up to 30 seconds for the lock

      logError(`Could not acquire lock for folder ID: ${folderId}, another process is running`);

      flushLogs(folderId);

      throw new Error("Another process is already running for this folder");

    }

  

    // Clear all existing states except for the current folder

    const scriptProperties = PropertiesService.getScriptProperties();

    const keys = scriptProperties.getKeys();

    const stateKeys = keys.filter(k => k.startsWith("docProcessingState_") && k !== `docProcessingState_${folderId}`);

    for (const key of stateKeys) {

      scriptProperties.deleteProperty(key);

      logInfo(`Cleared old state: ${key}`);

    }

    flushLogs(folderId);

  

    // Initialize the log sheet

    initializeLogSheet(folderId);

    logInfo("Log sheet initialized for folder ID: " + folderId);

    flushLogs(folderId);

  

    // Process the Attorneys tab to get case insights

    const caseInsights = processAttorneysTab(folderId);

    logInfo("Case insights: " + JSON.stringify(caseInsights));

    flushLogs(folderId);

  

    // Process litigation documents

    const result = processDocuments(folderId, caseInsights);

    logInfo("processDocuments completed with result: " + result);

    flushLogs(folderId);

  

    return result;

  } catch (e) {

    logError("Error in processWithFolderId: " + e.message);

    flushLogs(folderId);

    throw new Error("Failed to process documents: " + e.message);

  } finally {

    lock.releaseLock();

    flushLogs(folderId);

  }

}

  

function generateCaseBrief(folderId) {

  Logger.log("Starting generateCaseBrief with folder ID: " + folderId);

  logInfo("Starting generateCaseBrief with folder ID: " + folderId);

  flushLogs(folderId);

  

  try {

    // Initialize the log sheet

    initializeLogSheet(folderId);

    logInfo("Log sheet initialized for folder ID: " + folderId);

    flushLogs(folderId);

  

    return CaseBriefFunctions.generateCaseBrief(folderId);

  } catch (e) {

    logError("Error in generateCaseBrief: " + e.message);

    flushLogs(folderId);

    throw new Error("Failed to generate case brief: " + e.message);

  } finally {

    // Ensure logs are flushed even if an error occurs

    flushLogs(folderId);

  }

}

  

function saveZohoTokens() {

  PropertiesService.getScriptProperties().setProperty("ZOHO_OAUTH_TOKEN", "1000.168f0acefd5b1fd66b7235f2fa07ad8f.b23beee815d0c5d0058680e9dafdfffb");

  PropertiesService.getScriptProperties().setProperty("ZOHO_REFRESH_TOKEN", "1000.253176157c005d88ef20f5f464786d84.06aa7e141e9b0e565b40b6768aec6586");

  Logger.log("Tokens saved successfully");

}

  

function testGenerateCaseBrief() {

  generateCaseBrief("1aXbgh-P3ups7tQjRiMIxa_fNn3UiXumo");

}

  

function updateScriptProperties() {

  PropertiesService.getScriptProperties().setProperty("ZOHO_OAUTH_TOKEN", "1000.e64d4fb34e5991ba327d97d221a33bde.a555ec7ae72ac31beb21d7b7aec4e1cb");

  PropertiesService.getScriptProperties().setProperty("ZOHO_REFRESH_TOKEN", "1000.8679b21138f9900c4fd152bb921c8ddb.3e28c0ab73811525b3e5a3791f7efea1");

  PropertiesService.getScriptProperties().setProperty("ZOHO_CLIENT_ID", "1000.94UYXL4J71ZXK6G6WNBCEP7BA79EQI");

  PropertiesService.getScriptProperties().setProperty("ZOHO_CLIENT_SECRET", "755032e9c2d8cb8329762a37a2ba56b22b8ec90d1b");

  Logger.log("Script properties updated successfully.");

  

  function testLogging(folderId) {

  Logger.log("Starting testLogging with folder ID: " + folderId);

  logInfo("Starting testLogging with folder ID: " + folderId);

  flushLogs(folderId);

  return "Logging test completed";

}

}

function deleteAllResumeTriggers() {

  const triggers = ScriptApp.getProjectTriggers();

  for (const trigger of triggers) {

    if (trigger.getHandlerFunction() === "resumeDocumentProcessing") {

      ScriptApp.deleteTrigger(trigger);

      console.info("Deleted resumeDocumentProcessing trigger");

    }

  }

  console.info("All resumeDocumentProcessing triggers deleted");

}

function clearAllProcessingStates() {

  const scriptProperties = PropertiesService.getScriptProperties();

  const keys = scriptProperties.getKeys();

  const stateKeys = keys.filter(key => key.startsWith("docProcessingState_"));

  for (const key of stateKeys) {

    scriptProperties.deleteProperty(key);

    console.info(`Cleared state: ${key}`);

  }

  console.info("All processing states cleared");

}
```



# #####Constants.gs

## Overview
This file contains shared constants for the "AI LEGAL GOD" project, focusing on configuration values for the Gemini API integration.

## Constants

### `GEMINI_API_KEY`
- **Value**: `"AIzaSyAJ3NATWpKBeEOro8cYutGu994ERpRQOTE"`
- **Purpose**: The API key for authenticating requests to the Gemini API.

### `GEMINI_MODEL`
- **Value**: `"gemini-1.5-pro"`
- **Purpose**: Specifies the Gemini model version used for generating content (e.g., case briefs, insights).

## Dependencies
- None (pure constant definitions).

## Notes
- Used by functions like `callGeminiForCaseBrief` (assumed in another file) for API calls.
- No dynamic logic or functions—just static values.





# #####DocumentProcessing.gs

## Overview
This file manages the processing of litigation documents for the "AI LEGAL GOD" project, generating summaries from Google Drive files and organizing them into a spreadsheet. It supports batch processing, state persistence, and resumption for long-running tasks, integrating with Zoho CRM and the Gemini API.

## Functions

### `processDocuments(folderId, caseInsights)`
- **Purpose**: Processes documents in a folder, generating summaries and appending them to spreadsheet tabs.
- **Parameters**: 
  - `folderId` (string): Google Drive folder ID.
  - `caseInsights` (object): Case context from `processAttorneysTab`.
- **Behavior**:
  - Validates `folderId`, accesses the "Litigation Document Analysis" spreadsheet, and fetches a Zoho snapshot.
  - Uses "Tab Mapping" to identify subfolders, processes files in batches (20), and generates summaries via `generateDocumentSummary`.
  - Resumes from saved state if present, pauses with triggers if time exceeds 300 seconds, and consolidates summaries into a "Root" tab.
  - Logs progress and errors, flushes logs frequently.
- **Returns**: Success message or pause status (e.g., "Processed documents..." or "Processing paused...").
- **Dependencies**: `DriveApp`, `SpreadsheetApp`, `fetchCaseSnapshotFromZoho`, `getProcessingState`, `saveProcessingState`, `getAllFilesInFolder`, `generateDocumentSummary`, `appendSummariesToTab`.

### `resumeDocumentProcessing()`
- **Purpose**: Resumes paused document processing via a time-based trigger.
- **Behavior**:
  - Uses global and folder-specific locks to prevent overlaps, deletes redundant triggers, and processes the first pending state.
  - Calls `processDocuments` with saved `caseInsights`, logs progress, and clears other states.
- **Dependencies**: `LockService`, `ScriptApp`, `PropertiesService`, `processDocuments`, `getProcessingState`, `clearProcessingState`.

### `saveProcessingState(folderId, state)`
- **Purpose**: Saves the current processing state to script properties.
- **Parameters**: 
  - `folderId` (string): Folder ID.
  - `state` (object): Processing state (e.g., `currentSubfolder`, `fileIndex`).
- **Behavior**: Stringifies and stores state, logs success, throws on failure.
- **Dependencies**: `PropertiesService`.

### `getProcessingState(folderId)`
- **Purpose**: Retrieves the saved processing state for a folder.
- **Parameters**: `folderId` (string): Folder ID.
- **Behavior**: Fetches and parses state, returns `null` if missing or corrupt, logs result.
- **Returns**: State object or `null`.
- **Dependencies**: `PropertiesService`.

### `clearProcessingState(folderId)`
- **Purpose**: Deletes the processing state for a folder.
- **Parameters**: `folderId` (string): Folder ID.
- **Behavior**: Removes state from properties, logs success, throws on failure.
- **Dependencies**: `PropertiesService`.

### `getAllFilesInFolder(folder, depth = 0, maxDepth = 10)`
- **Purpose**: Recursively retrieves all files in a folder and its subfolders.
- **Parameters**: 
  - `folder` (Folder): Drive folder object.
  - `depth` (number): Current recursion depth.
  - `maxDepth` (number): Max recursion depth (default 10).
- **Behavior**: Collects files, traverses subfolders up to `maxDepth`, logs errors.
- **Returns**: Array of File objects.
- **Dependencies**: `DriveApp`.

### `generateDocumentSummary(document, tabType, priority, caseInsights, snapshot)`
- **Purpose**: Generates a detailed summary for a document using the Gemini API.
- **Parameters**: 
  - `document` (File): Drive file object.
  - `tabType` (string): Tab type from "Tab Mapping".
  - `priority` (string): Document priority.
  - `caseInsights` (object): Case context.
  - `snapshot` (string): Zoho CRM snapshot.
- **Behavior**:
  - Builds a prompt based on `tabType` (e.g., pleadings, discovery), calls Gemini API with file content (PDF, image, or text).
  - Parses response into a structured summary, adds defaults for missing fields, logs errors with fallback data.
- **Returns**: Summary object with fields like `fileName`, `summary`, `keyInformation`.
- **Dependencies**: `callGeminiWithPDF`, `callGeminiWithImage`, `callGeminiWithText`, `parseGeminiResponse`, `getFolderPath`.

### `getFolderPath(folder)`
- **Purpose**: Constructs the full path of a folder.
- **Parameters**: `folder` (Folder): Drive folder object.
- **Behavior**: Traverses parent folders, builds path string (e.g., "folder1/folder2").
- **Returns**: Path string.
- **Dependencies**: `DriveApp`.

### `testProcessDocuments()`
- **Purpose**: Tests `processDocuments` with a hardcoded folder ID.
- **Behavior**: Runs `processDocuments` with a test `folderId` and empty `caseInsights`, logs result.
- **Returns**: Result of `processDocuments` or throws error.
- **Dependencies**: `processDocuments`.

## Dependencies
- **External Functions**: `logInfo`, `logError`, `logWarn`, `flushLogs`, `initializeLogSheet`, `fetchCaseSnapshotFromZoho`, `callGeminiWithPDF`, `callGeminiWithImage`, `callGeminiWithText`, `parseGeminiResponse`, `appendSummariesToTab` (assumed in other files).
- **Services**: `DriveApp`, `SpreadsheetApp`, `LockService`, `PropertiesService`, `ScriptApp`, `Utilities`.

## Notes
- **State Management**: Uses `PropertiesService` for resumable processing, critical for long-running tasks.
- **Tab Mapping**: Relies on a "Tab Mapping" sheet with columns: Subfolder Name, Mapped Tab Type, Priority, Process Documents.
- **Error Handling**: Robust with retries for trigger deletion and fallbacks for file access/summary generation.
- **Test Folder**: Hardcoded test ID (`1TD6_fodftXP8cEVa33G34nbZZYvzApeu`) differs from your usual test (`1aXbgh-P3ups7tQjRiMIxa_fNn3UiXumo`).

## CODE
```
function processDocuments(folderId, caseInsights) {

  logInfo("Starting processDocuments with folder ID: " + folderId);

  

  // Validate folderId

  if (typeof folderId !== 'string' || !folderId || folderId.trim() === "") {

    logError("Invalid folder ID: " + folderId);

    throw new Error("Folder ID cannot be empty or invalid in processDocuments");

  }

  

  try {

    // Get the spreadsheet

    const folder = DriveApp.getFolderById(folderId);

    const files = folder.getFilesByName("Litigation Document Analysis");

    if (!files.hasNext()) {

      logWarn("Spreadsheet 'Litigation Document Analysis' not found in folder");

      throw new Error("Spreadsheet 'Litigation Document Analysis' not found in folder");

    }

    const spreadsheet = SpreadsheetApp.open(files.next());

    flushLogs(folderId);

  

    // Fetch the snapshot from Zoho CRM (this also saves it to the Attorneys tab)

    const snapshot = fetchCaseSnapshotFromZoho(folderId);

    if (!snapshot) {

      logWarn(`No snapshot available for folder ${folderId}. Proceeding without snapshot.`);

    } else {

      logInfo(`Successfully fetched snapshot: ${snapshot.substring(0, 100)}...`);

    }

    flushLogs(folderId);

  

    // Get the Tab Mapping tab to determine which subfolders to process

    const tabMappingSheet = spreadsheet.getSheetByName("Tab Mapping");

    if (!tabMappingSheet) {

      logWarn("Tab Mapping tab not found in spreadsheet");

      throw new Error("Tab Mapping tab not found in spreadsheet");

    }

  

    // Read the Tab Mapping tab to get subfolder mappings

    const tabMappingData = tabMappingSheet.getDataRange().getValues();

    const subfolderMap = new Map();

    for (let i = 1; i < tabMappingData.length; i++) { // Skip header row

      const subfolderName = tabMappingData[i][0]; // Column A: Subfolder Name

      const tabType = tabMappingData[i][2]; // Column C: Mapped Tab Type

      const priority = tabMappingData[i][3]; // Column D: Priority

      const processDocs = tabMappingData[i][4]; // Column E: Process Documents

      subfolderMap.set(subfolderName, { tabType, priority, processDocs });

    }

    flushLogs(folderId);

  

    // Check for existing state to resume processing

    let state = getProcessingState(folderId);

    let remainingSubfolders = [];

    let currentSubfolder = null;

    let fileIndex = 0;

    let subfolderFiles = [];

    const startTime = new Date().getTime();

  

    if (state) {

      logInfo("Resuming document processing from saved state");

      currentSubfolder = state.currentSubfolder;

      fileIndex = state.fileIndex;

      subfolderFiles = state.subfolderFiles;

      remainingSubfolders = state.remainingSubfolders;

  

      if (currentSubfolder) {

        let targetTab;

        if (currentSubfolder === "(Root)") {

          targetTab = spreadsheet.getSheetByName("Root");

        } else {

          const tabName = currentSubfolder.replace(/[^a-zA-Z0-9]/g, '_');

          targetTab = spreadsheet.getSheetByName(tabName);

        }

  

        if (targetTab) {

          let existingSummaries = [];

          let filenameColumnIndex = -1;

          let startRow = 1;

          const maxRowsToCheck = Math.min(targetTab.getLastRow(), 20);

          for (let row = 1; row <= maxRowsToCheck; row++) {

            const firstCell = targetTab.getRange(row, 1).getValue();

            if (firstCell === "[Document Summaries]") {

              startRow = row + 1;

              break;

            }

          }

          const headers = targetTab.getRange(startRow, 1, 1, targetTab.getLastColumn()).getValues()[0];

          headers.forEach((header, index) => {

            if (header === "Filename") filenameColumnIndex = index;

          });

  

          if (filenameColumnIndex !== -1) {

            const numRows = targetTab.getLastRow() - startRow;

            if (numRows > 0) {

              existingSummaries = targetTab.getRange(startRow + 1, 1, numRows, targetTab.getLastColumn()).getValues();

            }

          }

  

          const expectedFiles = new Set();

          subfolderFiles.forEach(fileId => {

            try {

              const fileName = DriveApp.getFileById(fileId).getName();

              expectedFiles.add(fileName);

            } catch (e) {

              logError(`Error accessing file with ID ${fileId}: ${e.message}`);

            }

          });

          const existingFileNames = new Set(existingSummaries.map(summary => summary[filenameColumnIndex]).filter(name => name));

  

          if (existingFileNames.size < expectedFiles.size) {

            logInfo(`Not all expected summaries found in tab ${targetTab.getName()}, resetting fileIndex to reprocess`);

            fileIndex = 0;

          }

        }

      }

    } else {

      logInfo("Starting new document processing session");

      remainingSubfolders = Array.from(subfolderMap.keys());

      saveProcessingState(folderId, {

        currentSubfolder: null,

        fileIndex: 0,

        subfolderFiles: [],

        remainingSubfolders: remainingSubfolders,

        lastBatchTime: new Date().toISOString(),

        caseInsights: caseInsights

      });

    }

    flushLogs(folderId);

  

    const BATCH_SIZE = 20;

    const TIME_LIMIT_SECONDS = 360;

    const CHECKPOINT_SECONDS = 300;

  

    while (remainingSubfolders.length > 0 || currentSubfolder) {

      const elapsedTime = (new Date().getTime() - startTime) / 1000;

      if (elapsedTime > CHECKPOINT_SECONDS) {

        logInfo(`Elapsed time (${elapsedTime} seconds) exceeds checkpoint, pausing processing`);

        saveProcessingState(folderId, {

          currentSubfolder: currentSubfolder,

          fileIndex: fileIndex,

          subfolderFiles: subfolderFiles,

          remainingSubfolders: remainingSubfolders,

          lastBatchTime: new Date().toISOString(),

          caseInsights: caseInsights

        });

  

        // Check if a resumeDocumentProcessing trigger already exists

        let triggerExists = false;

        const triggers = ScriptApp.getProjectTriggers();

        for (const trigger of triggers) {

          if (trigger.getHandlerFunction() === "resumeDocumentProcessing") {

            triggerExists = true;

            logInfo("Resume trigger already exists, skipping creation");

            break;

          }

        }

  

        if (!triggerExists) {

          ScriptApp.newTrigger("resumeDocumentProcessing")

            .timeBased()

            .after(1 * 60 * 1000) // 1 minute delay

            .create();

          logInfo(`Created new resumeDocumentProcessing trigger for folder ID: ${folderId}`);

        }

  

        if (summaries.length > 0) {

          logInfo(`Appending ${summaries.length} document summaries to tab: ${targetTab.getName()} before pausing`);

          appendSummariesToTab(targetTab, summaries);

        }

        flushLogs(folderId);

        return `Processing paused due to time limit after ${elapsedTime.toFixed(2)} seconds`;

      }

  

      if (!currentSubfolder) {

        currentSubfolder = remainingSubfolders.shift();

        fileIndex = 0;

        subfolderFiles = [];

  

        if (!subfolderMap.has(currentSubfolder)) {

          logWarn(`Subfolder ${currentSubfolder} not found in subfolderMap`);

          currentSubfolder = null;

          continue;

        }

  

        const value = subfolderMap.get(currentSubfolder);

        if (value.processDocs.toLowerCase() !== "yes") {

          logInfo(`Skipping subfolder ${currentSubfolder}`);

          currentSubfolder = null;

          continue;

        }

  

        let targetTab;

        if (currentSubfolder === "(Root)") {

          targetTab = spreadsheet.getSheetByName("Root");

          const rootFiles = [];

          const filesIterator = folder.getFiles();

          while (filesIterator.hasNext()) {

            rootFiles.push(filesIterator.next());

          }

          subfolderFiles = rootFiles.map(file => file.getId());

        } else {

          const tabName = currentSubfolder.replace(/[^a-zA-Z0-9]/g, '_');

          targetTab = spreadsheet.getSheetByName(tabName);

          const subfolders = folder.getFoldersByName(currentSubfolder);

          if (subfolders.hasNext()) {

            const subfolder = subfolders.next();

            subfolderFiles = getAllFilesInFolder(subfolder).map(file => file.getId());

          } else {

            logWarn(`Subfolder ${currentSubfolder} not found`);

            currentSubfolder = null;

            continue;

          }

        }

  

        saveProcessingState(folderId, {

          currentSubfolder: currentSubfolder,

          fileIndex: fileIndex,

          subfolderFiles: subfolderFiles,

          remainingSubfolders: remainingSubfolders,

          lastBatchTime: new Date().toISOString(),

          caseInsights: caseInsights

        });

      }

  

      let targetTab;

      if (currentSubfolder === "(Root)") {

        targetTab = spreadsheet.getSheetByName("Root");

      } else {

        const tabName = currentSubfolder.replace(/[^a-zA-Z0-9]/g, '_');

        targetTab = spreadsheet.getSheetByName(tabName);

      }

      if (!targetTab) {

        logWarn(`Target tab for subfolder ${currentSubfolder} not found`);

        currentSubfolder = null;

        continue;

      }

  

      const value = subfolderMap.get(currentSubfolder);

      const totalFiles = subfolderFiles.length;

      logInfo(`Total files to process in subfolder ${currentSubfolder}: ${totalFiles}`);

  

      let existingSummaries = [];

      let filenameColumnIndex = -1;

      let processedColumnIndex = -1;

      let startRow = 1;

      const maxRowsToCheck = Math.min(targetTab.getLastRow(), 20);

      for (let row = 1; row <= maxRowsToCheck; row++) {

        const firstCell = targetTab.getRange(row, 1).getValue();

        if (firstCell === "[Document Summaries]") {

          startRow = row + 1;

          break;

        }

      }

      const headers = targetTab.getRange(startRow, 1, 1, targetTab.getLastColumn()).getValues()[0];

      headers.forEach((header, index) => {

        if (header === "Filename") filenameColumnIndex = index;

        if (header === "Processed") processedColumnIndex = index;

      });

  

      if (filenameColumnIndex !== -1 && processedColumnIndex !== -1) {

        const numRows = targetTab.getLastRow() - startRow;

        if (numRows > 0) {

          existingSummaries = targetTab.getRange(startRow + 1, 1, numRows, targetTab.getLastColumn()).getValues();

        }

      }

  

      const summaries = [];

      const existingFileNames = new Set(existingSummaries.map(summary => summary[filenameColumnIndex]).filter(name => name));

      while (fileIndex < totalFiles) {

        const elapsedTime = (new Date().getTime() - startTime) / 1000;

        if (elapsedTime > CHECKPOINT_SECONDS) {

          logInfo(`Elapsed time (${elapsedTime} seconds) exceeds checkpoint, pausing processing`);

          saveProcessingState(folderId, {

            currentSubfolder: currentSubfolder,

            fileIndex: fileIndex,

            subfolderFiles: subfolderFiles,

            remainingSubfolders: remainingSubfolders,

            lastBatchTime: new Date().toISOString(),

            caseInsights: caseInsights

          });

  

          // Check if a resumeDocumentProcessing trigger already exists

          let triggerExists = false;

          const triggers = ScriptApp.getProjectTriggers();

          for (const trigger of triggers) {

            if (trigger.getHandlerFunction() === "resumeDocumentProcessing") {

              triggerExists = true;

              logInfo("Resume trigger already exists, skipping creation");

              break;

            }

          }

  

          if (!triggerExists) {

            ScriptApp.newTrigger("resumeDocumentProcessing")

              .timeBased()

              .after(1 * 60 * 1000) // 1 minute delay

              .create();

            logInfo(`Created new resumeDocumentProcessing trigger for folder ID: ${folderId}`);

          }

  

          if (summaries.length > 0) {

            logInfo(`Appending ${summaries.length} document summaries to tab: ${targetTab.getName()} before pausing`);

            appendSummariesToTab(targetTab, summaries);

          }

          flushLogs(folderId);

          return `Processing paused due to time limit after ${elapsedTime.toFixed(2)} seconds`;

        }

  

        const batchEnd = Math.min(fileIndex + BATCH_SIZE, totalFiles);

        logInfo(`Processing batch: files ${fileIndex + 1} to ${batchEnd} in subfolder ${currentSubfolder}`);

  

        for (let i = fileIndex; i < batchEnd; i++) {

          const currentElapsedTime = (new Date().getTime() - startTime) / 1000;

          if (currentElapsedTime > CHECKPOINT_SECONDS) {

            logInfo(`Elapsed time (${currentElapsedTime} seconds) exceeds checkpoint at file ${i + 1}, pausing processing`);

            saveProcessingState(folderId, {

              currentSubfolder: currentSubfolder,

              fileIndex: i,

              subfolderFiles: subfolderFiles,

              remainingSubfolders: remainingSubfolders,

              lastBatchTime: new Date().toISOString(),

              caseInsights: caseInsights

            });

  

            // Check if a resumeDocumentProcessing trigger already exists

            let triggerExists = false;

            const triggers = ScriptApp.getProjectTriggers();

            for (const trigger of triggers) {

              if (trigger.getHandlerFunction() === "resumeDocumentProcessing") {

                triggerExists = true;

                logInfo("Resume trigger already exists, skipping creation");

                break;

              }

            }

  

            if (!triggerExists) {

              ScriptApp.newTrigger("resumeDocumentProcessing")

                .timeBased()

                .after(1 * 60 * 1000) // 1 minute delay

                .create();

              logInfo(`Created new resumeDocumentProcessing trigger for folder ID: ${folderId}`);

            }

  

            if (summaries.length > 0) {

              logInfo(`Appending ${summaries.length} document summaries to tab: ${targetTab.getName()} before pausing`);

              appendSummariesToTab(targetTab, summaries);

            }

            flushLogs(folderId);

            return `Processing paused due to time limit after ${currentElapsedTime.toFixed(2)} seconds`;

          }

  

          const fileId = subfolderFiles[i];

          let file;

          try {

            file = DriveApp.getFileById(fileId);

          } catch (e) {

            logError(`Error accessing file with ID ${fileId}: ${e.message}`);

            continue;

          }

  

          const fileName = file.getName();

          logInfo(`Processing file ${i + 1} of ${totalFiles} in subfolder ${currentSubfolder}: ${fileName}`);

  

          if (existingFileNames.has(fileName)) {

            logInfo(`Skipping file ${fileName}: already processed`);

            continue;

          }

  

          try {

            const summary = generateDocumentSummary(file, value.tabType, value.priority, caseInsights, snapshot);

            summaries.push(summary);

            existingFileNames.add(fileName);

          } catch (e) {

            logError(`Error processing file ${fileName}: ${e.message}`);

          }

        }

  

        if (summaries.length > 0) {

          logInfo(`Appending ${summaries.length} document summaries to tab: ${targetTab.getName()}`);

          appendSummariesToTab(targetTab, summaries);

          summaries.length = 0;

          flushLogs(folderId);

        }

  

        fileIndex = batchEnd;

        saveProcessingState(folderId, {

          currentSubfolder: currentSubfolder,

          fileIndex: fileIndex,

          subfolderFiles: subfolderFiles,

          remainingSubfolders: remainingSubfolders,

          lastBatchTime: new Date().toISOString(),

          caseInsights: caseInsights

        });

      }

  

      logInfo(`Completed processing subfolder ${currentSubfolder}, ${fileIndex} files processed`);

      currentSubfolder = null;

      fileIndex = 0;

      subfolderFiles = [];

      saveProcessingState(folderId, {

        currentSubfolder: null,

        fileIndex: 0,

        subfolderFiles: [],

        remainingSubfolders: remainingSubfolders,

        lastBatchTime: new Date().toISOString(),

        caseInsights: caseInsights

      });

      flushLogs(folderId);

    }

  

    // After processing all subfolders, copy summaries to the Root tab

    const rootTab = spreadsheet.getSheetByName("Root");

    if (rootTab) {

      logInfo("Copying summaries from subfolder tabs to Root tab");

      const allSummaries = [];

      const subfolderNames = Array.from(subfolderMap.keys()).filter(name => name !== "(Root)");

      for (const subfolderName of subfolderNames) {

        const tabName = subfolderName.replace(/[^a-zA-Z0-9]/g, '_');

        const subfolderTab = spreadsheet.getSheetByName(tabName);

        if (!subfolderTab) {

          logWarn(`Tab for subfolder ${subfolderName} not found, skipping`);

          continue;

        }

  

        let subfolderSummaries = [];

        let subfolderFilenameColumnIndex = -1;

        let subfolderHeaders = [];

        let subfolderStartRow = 1;

        const subfolderMaxRowsToCheck = Math.min(subfolderTab.getLastRow(), 20);

        for (let row = 1; row <= subfolderMaxRowsToCheck; row++) {

          const firstCell = subfolderTab.getRange(row, 1).getValue();

          if (firstCell === "[Document Summaries]") {

            subfolderStartRow = row + 1;

            break;

          }

        }

        subfolderHeaders = subfolderTab.getRange(subfolderStartRow, 1, 1, subfolderTab.getLastColumn()).getValues()[0];

        subfolderHeaders.forEach((header, index) => {

          if (header === "Filename") subfolderFilenameColumnIndex = index;

        });

  

        if (subfolderFilenameColumnIndex !== -1) {

          const numRows = Math.max(1, subfolderTab.getLastRow() - subfolderStartRow);

          if (numRows > 0) {

            subfolderSummaries = subfolderTab.getRange(subfolderStartRow + 1, 1, numRows, subfolderTab.getLastColumn()).getValues();

            subfolderSummaries.forEach(summary => {

              allSummaries.push({ summary, headers: subfolderHeaders });

            });

          }

        }

      }

  

      if (allSummaries.length > 0) {

        let rootStartRow = 1;

        const rootMaxRowsToCheck = Math.min(rootTab.getLastRow(), 20);

        for (let row = 1; row <= rootMaxRowsToCheck; row++) {

          const firstCell = rootTab.getRange(row, 1).getValue();

          if (firstCell === "[Document Summaries]") {

            rootStartRow = row + 1;

            break;

          }

        }

        const rootHeaders = rootTab.getRange(rootStartRow, 1, 1, rootTab.getLastColumn()).getValues()[0];

        let rootFilenameColumnIndex = -1;

        rootHeaders.forEach((header, index) => {

          if (header === "Filename") rootFilenameColumnIndex = index;

        });

  

        const existingSummaries = [];

        const numRows = Math.max(1, rootTab.getLastRow() - rootStartRow);

        if (numRows > 0 && rootFilenameColumnIndex !== -1) {

          const summaries = rootTab.getRange(rootStartRow + 1, 1, numRows, rootTab.getLastColumn()).getValues();

          existingSummaries.push(...summaries);

        }

        const existingFileNames = new Set(existingSummaries.map(summary => summary[rootFilenameColumnIndex]).filter(name => name));

  

        const formattedSummaries = allSummaries

          .filter(item => !existingFileNames.has(item.summary[item.headers.indexOf("Filename")]))

          .map(item => {

            const summary = item.summary;

            const subfolderHeaders = item.headers;

            const summaryObj = {};

            rootHeaders.forEach((header, index) => {

              const fieldKey = header.toLowerCase().replace(/[^a-z0-9]/g, '');

              const subfolderIndex = subfolderHeaders.indexOf(header);

              summaryObj[fieldKey] = subfolderIndex !== -1 ? summary[subfolderIndex] || "" : "";

            });

            summaryObj.fileName = summary[subfolderHeaders.indexOf("Filename")];

            summaryObj.link = summary[subfolderHeaders.indexOf("Link")];

            return summaryObj;

          });

  

        if (formattedSummaries.length > 0) {

          appendSummariesToTab(rootTab, formattedSummaries);

        }

      }

      flushLogs(folderId);

    }

  

    clearProcessingState(folderId);

    logInfo("All subfolders processed, state cleared");

  } catch (e) {

    logError(`Error in processDocuments: ${e.message} (Stack: ${e.stack})`);

    throw e; // Re-throw to ensure the error is visible in the execution log

  } finally {

    flushLogs(folderId);

  }

  

  return "Processed documents and appended summaries to respective tabs";

}

  

// Function to resume document processing via a trigger

function resumeDocumentProcessing() {

  // Use a global lock to prevent any overlapping executions

  const globalLock = LockService.getScriptLock();

  try {

    if (!globalLock.tryLock(30000)) { // Wait up to 30 seconds for the global lock

      logError("Could not acquire global lock, another process is running");

      flushLogs(null);

      return;

    }

  

    // Delete all existing resumeDocumentProcessing triggers to prevent overlaps

    let retryCount = 0;

    const maxRetries = 3;

    while (retryCount < maxRetries) {

      try {

        const triggers = ScriptApp.getProjectTriggers();

        for (const trigger of triggers) {

          if (trigger.getHandlerFunction() === "resumeDocumentProcessing") {

            ScriptApp.deleteTrigger(trigger);

            logInfo("Deleted existing resumeDocumentProcessing trigger at start");

          }

        }

        break; // Success, exit the retry loop

      } catch (e) {

        retryCount++;

        logWarn(`Failed to delete triggers (attempt ${retryCount}/${maxRetries}): ${e.message}`);

        if (retryCount === maxRetries) {

          logError(`Failed to delete triggers after ${maxRetries} attempts: ${e.message}`);

        }

        Utilities.sleep(1000); // Wait 1 second before retrying

      }

    }

    flushLogs(null);

  

    // Find all states in PropertiesService

    const scriptProperties = PropertiesService.getScriptProperties();

    const keys = scriptProperties.getKeys();

    const stateKeys = keys.filter(key => key.startsWith("docProcessingState_"));

  

    if (stateKeys.length === 0) {

      logInfo("No processing states found to resume");

      flushLogs(null);

      return;

    }

  

    // Process only the first state to avoid timeouts

    const key = stateKeys[0]; // Process the first state only

    const folderId = key.replace("docProcessingState_", "");

    logInfo(`Resuming document processing for folder ID: ${folderId}`);

    flushLogs(folderId);

  

    // Clear all other states except the current one

    const otherStateKeys = stateKeys.filter(k => k !== key);

    for (const otherKey of otherStateKeys) {

      scriptProperties.deleteProperty(otherKey);

      logInfo(`Cleared old state: ${otherKey}`);

    }

    flushLogs(folderId);

  

    // Use a folder-specific lock to prevent overlapping executions for the same folder

    const folderLock = LockService.getScriptLock();

    try {

      if (!folderLock.tryLock(30000)) { // Wait up to 30 seconds for the folder-specific lock

        logError(`Could not acquire folder lock for folder ID: ${folderId}, another process is running`);

        flushLogs(folderId);

        return;

      }

  

      const state = getProcessingState(folderId);

      if (!state) {

        logWarn(`No state found for folder ID: ${folderId}, clearing state`);

        clearProcessingState(folderId);

        flushLogs(folderId);

        return;

      }

  

      const caseInsights = state.caseInsights || {};

      try {

        const result = processDocuments(folderId, caseInsights);

        logInfo(`Resume result for folder ID ${folderId}: ${result}`);

        flushLogs(folderId);

      } catch (e) {

        logError(`Error resuming document processing for folder ID ${folderId}: ${e.message}`);

        flushLogs(folderId);

        throw e; // Re-throw to ensure the error is visible in the execution log

      }

    } finally {

      folderLock.releaseLock();

      flushLogs(folderId);

    }

  } finally {

    globalLock.releaseLock();

    flushLogs(null);

  }

}

  

// Helper function to save processing state

function saveProcessingState(folderId, state) {

  const scriptProperties = PropertiesService.getScriptProperties();

  const key = `docProcessingState_${folderId}`;

  try {

    const stateString = JSON.stringify(state);

    scriptProperties.setProperty(key, stateString);

    logInfo(`Saved processing state for folder ID ${folderId}`);

  } catch (e) {

    logError(`Failed to save processing state for folder ID ${folderId}: ${e.message}`);

    throw new Error(`Failed to save processing state: ${e.message}`);

  }

  flushLogs(folderId);

}

  

// Helper function to get processing state

function getProcessingState(folderId) {

  const scriptProperties = PropertiesService.getScriptProperties();

  const key = `docProcessingState_${folderId}`;

  const stateString = scriptProperties.getProperty(key);

  if (!stateString) {

    logInfo(`No processing state found for folder ID ${folderId}`);

    return null;

  }

  

  try {

    const state = JSON.parse(stateString);

    logInfo(`Retrieved processing state for folder ID ${folderId}`);

    return state;

  } catch (e) {

    logError(`Failed to parse processing state for folder ID ${folderId}: ${e.message}`);

    return null; // Return null if parsing fails to allow the process to continue

  }

}

  

// Helper function to clear processing state

function clearProcessingState(folderId) {

  const scriptProperties = PropertiesService.getScriptProperties();

  const key = `docProcessingState_${folderId}`;

  try {

    scriptProperties.deleteProperty(key);

    logInfo(`Cleared state for folder ID ${folderId}`);

  } catch (e) {

    logError(`Failed to clear processing state for folder ID ${folderId}: ${e.message}`);

    throw new Error(`Failed to clear processing state: ${e.message}`);

  }

  flushLogs(folderId);

}

  

// Helper function to recursively get all files in a folder and its subfolders

function getAllFilesInFolder(folder, depth = 0, maxDepth = 10) {

  if (depth > maxDepth) {

    logWarn(`Maximum recursion depth (${maxDepth}) reached, stopping traversal`);

    flushLogs(null);

    return [];

  }

  

  const allFiles = [];

  try {

    const files = folder.getFiles();

    while (files.hasNext()) {

      allFiles.push(files.next());

    }

  } catch (e) {

    logError(`Error accessing files in folder ${folder.getName()}: ${e.message}`);

    flushLogs(null);

    return allFiles;

  }

  

  try {

    const subfolders = folder.getFolders();

    while (subfolders.hasNext()) {

      const subfolder = subfolders.next();

      logInfo(`Traversing nested subfolder: ${subfolder.getName()}`);

      flushLogs(null);

      const nestedFiles = getAllFilesInFolder(subfolder, depth + 1, maxDepth);

      allFiles.push(...nestedFiles);

    }

  } catch (e) {

    logError(`Error accessing subfolders in folder ${folder.getName()}: ${e.message}`);

    flushLogs(null);

  }

  

  return allFiles;

}

  

function generateDocumentSummary(document, tabType, priority, caseInsights, snapshot) {

  logInfo(`Generating summary for document: ${document.getName()} in tab: ${tabType}`);

  

  const fileName = document.getName();

  const fileId = document.getId();

  const filePath = getFolderPath(document.getParents().next()) + "/" + fileName;

  const fileUrl = document.getUrl();

  

  let prompt = `

You are a legal document analyst specialized in litigation.

Analyze this legal document with filename: ${fileName}

Located in folder: ${filePath}

  

Provide context based on the following case insights (use if available):

- Case Overview: ${caseInsights.caseOverview || "N/A"}

- Strategy: ${caseInsights.strategy || "N/A"}

- Claims: ${caseInsights.claims || "N/A"}

- Key Issues: ${caseInsights.keyIssues || "N/A"}

- Summary Guidance: ${caseInsights.summaryGuidance || "N/A"}

- Helps Us Guidance: ${caseInsights.helpsUsGuidance || "N/A"}

- Hurts Us Guidance: ${caseInsights.hurtsUsGuidance || "N/A"}

  

Additionally, use the following Case Action Plan snapshot for further context (use if available):

${snapshot || "N/A"}

  

This document is from the ${tabType} tab, which has a ${priority} priority. Extract the following information:

  

**Common Information (for all documents):**

- Filename: The name of the document.

- File Path: The full path of the document in Google Drive.

- Document Type: The specific type of legal document (e.g., Pleading, Invoice, Discovery).

- Summary: An overall summary of the document’s content, focusing on its significance in the case. Adjust the level of detail based on the priority: High priority = 3-5 sentences, Medium priority = 2-3 sentences, Low priority = 1-2 sentences. Incorporate relevant details from the Case Action Plan snapshot to provide context (e.g., relate the document to upcoming deadlines, open tasks, or timeline events).

- Key Information: A concise highlight of the most critical detail in the document (e.g., a key fact, figure, or statement). Limit to 1-2 sentences. For invoices, this should include the total amount billed or claimed (e.g., "The invoice amount is $75,090.37."). For other document types, focus on the most significant detail relevant to the case, considering the snapshot context.

- Priority: The document’s priority, set to ${priority}.

- Relevant Dates: All key dates associated with the document (e.g., filing date, hearing date, deadlines). Format as a comma-separated list (e.g., "01/01/2023 (Filing Date), 02/01/2023 (Due Date)"). If none, return "N/A".

- Key Parties: All parties mentioned in the document (e.g., requesting party, responding party). If none, return "N/A".

- Link: The Google Drive URL to the document.

- Processed: Yes, followed by the current timestamp (e.g., "Yes, ${new Date().toISOString()}").

- Helps Us: How this document supports our case, based on the case insights, Helps Us Guidance, and snapshot. If none, return "N/A".

- Hurts Us: How this document may harm our case, based on the case insights, Hurts Us Guidance, and snapshot. If none, return "N/A".

- Next Actions: Recommend 1-2 specific legal actions only if the document is less than 1 year old (based on Relevant Dates compared to today, ${new Date().toISOString().split('T')[0]}) and a clear, urgent action is required based on the document’s content and snapshot (e.g., align with open tasks or upcoming deadlines). Otherwise, return 'N/A'.

  

**Tab-Specific Information:**

`;

  

  switch (tabType.toLowerCase()) {

    case "pleadings":

      prompt += `

- Legal References: Any legal citations or references mentioned in the document (e.g., "Section 303(f)", "Texas Estates Code").

- Filing Attorney: The name and contact information of the attorney who filed this, if specified.

- Filing Party: The party on whose behalf this was filed.

- Legal Strategies: Any legal strategies or arguments presented.

- Claims/Defenses: Specific claims or defenses raised in the document.

- Filing Date: The date the document was filed, if available.

- Case Number: The court case number associated with the document, if present.

- Court: The court and judge information (e.g., court name, county, division, judge name), if available.

`;

      break;

    case "legal fees & expenses":

    case "expenses":

      prompt += `

- Amount: The total amount billed or claimed in the document.

- Billing Period: The period covered by the invoice or expense.

- Service Description: Description of the services or expenses.

- Payment Status: Whether the invoice has been paid, unpaid, or disputed.

- Financial Impact: How this expense impacts the case financially.

`;

      break;

    case "hearing preparation":

      prompt += `

- Hearing Type: The type of hearing (e.g., motion to dismiss, status hearing).

- Exhibits: Any exhibits or evidence mentioned for the hearing.

- Court: The court and judge information (e.g., court name, county, division, judge name), if available.

- Case Number: The court case number associated with the document, if present.

- Preparation Notes: Any notes or instructions for preparing for the hearing.

`;

      break;

    case "discovery":

      prompt += `

- Request/Response IDs: Specific request numbers or identifiers.

- Discovery Owner: The party that owns or produced the discovery. For Requests, this is the requesting party (e.g., "Our Side" if we requested it, "Opposing Side" if they did). For Production/Responses, this is the producing party (e.g., "Our Side" if we produced it, "Opposing Side" if they did). Identify the document as a Request or Response to determine ownership.

- Discovery Type: The type of discovery document (e.g., interrogatories, request for production).

- Production Details: Details about what was produced or requested.

- Bates/Exhibit Labels: Any Bates numbers or exhibit labels associated with the document.

`;

      break;

    case "mediation":

      prompt += `

- Mediation Type: The type of mediation (e.g., court-ordered, voluntary).

- Mediation Role: The role of the parties in mediation (e.g., plaintiff, defendant).

- Settlement Impact: How this document impacts potential settlement.

- Mediation Notes: Any notes or outcomes from the mediation.

`;

      break;

    case "client files":

      prompt += `

- Evidence Owner: The party producing the evidence. Specify "Our Side" if we are producing it, "Opposing Side" if they are producing it.

- Evidence Type: The type of evidence (e.g., contract, email).

- Supporting Claim: The claim or issue this evidence supports.

- Admissibility: Whether the evidence is likely admissible in court.

- Client Context: Context provided by the client regarding this document.

- Bates/Exhibit Labels: Any Bates numbers or exhibit labels associated with the document.

`;

      break;

    case "correspondence":

      prompt += `

- Correspondence Type: The type of correspondence (e.g., email, letter).

- Recipient/Sender: The recipient and sender of the correspondence.

- Purpose: The purpose or intent of the correspondence.

`;

      break;

    case "attorneys":

    case "attorney & paralegal notes":

    case "root":

    default:

      prompt += `

(No additional tab-specific information required.)

`;

      break;

  }

  

  prompt += `

Be specific and direct. Avoid phrases like "appears to be" or "likely contains." Structure your response with these exact headings to make parsing easier:

  

Filename:

File Path:

Document Type:

Summary:

Key Information:

Priority:

Relevant Dates:

Key Parties:

Link:

Processed:

`;

  

  if (tabType.toLowerCase() === "pleadings") {

    prompt += `

Legal References:

Filing Attorney:

Filing Party:

Legal Strategies:

Claims/Defenses:

Filing Date:

Case Number:

Court:

`;

  } else if (tabType.toLowerCase() === "legal fees & expenses" || tabType.toLowerCase() === "expenses") {

    prompt += `

Amount:

Billing Period:

Service Description:

Payment Status:

Financial Impact:

`;

  } else if (tabType.toLowerCase() === "hearing preparation") {

    prompt += `

Hearing Type:

Exhibits:

Court:

Case Number:

Preparation Notes:

`;

  } else if (tabType.toLowerCase() === "discovery") {

    prompt += `

Request/Response IDs:

Discovery Owner:

Discovery Type:

Production Details:

Bates/Exhibit Labels:

`;

  } else if (tabType.toLowerCase() === "mediation") {

    prompt += `

Mediation Type:

Mediation Role:

Settlement Impact:

Mediation Notes:

`;

  } else if (tabType.toLowerCase() === "client files") {

    prompt += `

Evidence Owner:

Evidence Type:

Supporting Claim:

Admissibility:

Client Context:

Bates/Exhibit Labels:

`;

  } else if (tabType.toLowerCase() === "correspondence") {

    prompt += `

Correspondence Type:

Recipient/Sender:

Purpose:

`;

  }

  

  prompt += `

Helps Us:

Hurts Us:

Next Actions:

`;

  

  // Call Gemini API and parse the response, with error handling

  const fileBlob = document.getBlob();

  const mimeType = document.getMimeType();

  let analysis;

  try {

    if (mimeType === 'application/pdf') {

      logInfo(`Calling Gemini API with PDF for file: ${fileName}`);

      analysis = callGeminiWithPDF(fileBlob, fileName, prompt);

    } else if (mimeType.includes('image/')) {

      logInfo(`Calling Gemini API with image for file: ${fileName}`);

      analysis = callGeminiWithImage(fileBlob, fileName, prompt);

    } else {

      try {

        const content = fileBlob.getDataAsString();

        logInfo(`Calling Gemini API with text for file: ${fileName}`);

        analysis = callGeminiWithText(content, fileName, prompt);

      } catch (textError) {

        logError(`Error extracting text from ${fileName}: ${textError}`);

        logInfo(`Falling back to PDF processing for file: ${fileName}`);

        analysis = callGeminiWithPDF(fileBlob, fileName, prompt);

      }

    }

  

    // Parse the response

    logInfo(`Parsing Gemini response for file: ${fileName}`);

    const summary = parseGeminiResponse(analysis, fileName);

    summary.fileName = fileName;

    summary.filePath = filePath;

    summary.link = fileUrl;

    summary.processed = `Yes, ${new Date().toISOString()}`;

    summary.priority = priority;

  

    // Ensure tab-specific fields have default values if not provided

    summary.legalReferences = summary.legalReferences || "N/A";

    summary.filingAttorney = summary.filingAttorney || "N/A";

    summary.filingParty = summary.filingParty || "N/A";

    summary.legalStrategies = summary.legalStrategies || "N/A";

    summary.claimsDefenses = summary.claimsDefenses || "N/A";

    summary.filingDate = summary.filingDate || "N/A";

    summary.hearingType = summary.hearingType || "N/A";

    summary.exhibits = summary.exhibits || "N/A";

    summary.court = summary.court || "N/A";

    summary.caseNumber = summary.caseNumber || "N/A";

    summary.amount = summary.amount || "N/A";

    summary.billingPeriod = summary.billingPeriod || "N/A";

    summary.serviceDescription = summary.serviceDescription || "N/A";

    summary.paymentStatus = summary.paymentStatus || "N/A";

    summary.financialImpact = summary.financialImpact || "N/A";

    summary.requestResponseIDs = summary.requestResponseIDs || "N/A";

    summary.discoveryOwner = summary.discoveryOwner || "N/A";

    summary.discoveryType = summary.discoveryType || "N/A";

    summary.productionDetails = summary.productionDetails || "N/A";

    summary.batesExhibitLabels = summary.batesExhibitLabels || "None";

    summary.exhibitNumber = summary.exhibitNumber || "N/A";

    summary.batesRange = summary.batesRange || "None";

    summary.producedBy = summary.producedBy || "N/A";

    summary.nextActions = summary.nextActions || "N/A";

    summary.mediationType = summary.mediationType || "N/A";

    summary.mediationRole = summary.mediationRole || "N/A";

    summary.settlementImpact = summary.settlementImpact || "N/A";

    summary.mediationNotes = summary.mediationNotes || "N/A";

    summary.evidenceOwner = summary.evidenceOwner || "N/A";

    summary.evidenceType = summary.evidenceType || "N/A";

    summary.supportingClaim = summary.supportingClaim || "N/A";

    summary.admissibility = summary.admissibility || "N/A";

    summary.clientContext = summary.clientContext || "N/A";

    summary.correspondenceType = summary.correspondenceType || "N/A";

    summary.recipientSender = summary.recipientSender || "N/A";

    summary.purpose = summary.purpose || "N/A";

  

    return summary;

  } catch (e) {

    logError(`Failed to analyze document ${fileName}: ${e.message}`);

    const summary = {

      fileName: fileName,

      filePath: filePath,

      documentType: "Error - Failed to analyze",

      summary: "Failed to analyze document due to an error.",

      keyInformation: `Error: ${e.message}`,

      priority: priority,

      relevantDates: "N/A",

      keyParties: "N/A",

      link: fileUrl,

      processed: `Yes, ${new Date().toISOString()}`,

      helpsUs: "N/A",

      hurtsUs: "N/A",

      nextActions: "Manual review required due to analysis error",

      evidenceOwner: "N/A",

      evidenceType: "N/A",

      supportingClaim: "N/A",

      admissibility: "N/A",

      clientContext: "N/A",

      batesExhibitLabels: "None"

    };

    return summary;

  }

}

  

function getFolderPath(folder) {

  const path = [];

  let currentFolder = folder;

  while (currentFolder) {

    path.unshift(currentFolder.getName());

    const parents = currentFolder.getParents();

    currentFolder = parents.hasNext() ? parents.next() : null;

  }

  return path.join('/');

}

  

// Test wrapper function for manual execution

function testProcessDocuments() {

  logInfo("Starting testProcessDocuments");

  

  // Replace with your test folder ID

  const testFolderId = "1TD6_fodftXP8cEVa33G34nbZZYvzApeu"; // e.g., "1lRgmavra1DTP8Un4X1pDEiJtCAJRTl5i"

  // Initialize the log sheet

  initializeLogSheet(testFolderId);

  

  // Use empty case insights for testing (or populate with test data)

  const caseInsights = {

    caseOverview: "",

    strategy: "",

    claims: "",

    keyIssues: "",

    summaryGuidance: "",

    helpsUsGuidance: "",

    hurtsUsGuidance: ""

  };

  

  try {

    const result = processDocuments(testFolderId, caseInsights);

    logInfo("Test result: " + result);

    // Flush logs after the test

    flushLogs(testFolderId);

    return result;

  } catch (e) {

    logError("Test failed: " + e.message);

    // Flush logs even if there's an error

    flushLogs(testFolderId);

    throw new Error("Test failed: " + e.message);

  }

}
```



# #####resumeDocumentProcessing.gs`
## Overview

This file contains a single function designed to resume document processing for a specific folder within the "AI LEGAL GOD" project. It integrates with a broader document processing workflow, likely focused on legal document analysis utilizing the Gemini API.

## Functions

### `resumeDocumentProcessing()`

- **Purpose**: Resumes the processing of documents located in a predefined Google Drive folder.
- **Process**:
    - Sets a hardcoded folder ID (`"1aXbgh-P3ups7tQjRiMIxa_fNn3UiXumo"`) as the `currentFolderId` for logging purposes.
    - Logs the resumption of the processing task.
    - Calls `processDocuments(folderId, true)` to handle the documents within that folder, explicitly indicating this is a resumed operation.
- **Parameters**: None (utilizes a hardcoded folder ID).
- **Returns**: None (triggers processing via a separate function).

## Dependencies

- **Google Apps Script Services**:
    - Implied use of the Google Drive API (via the `processDocuments` function, assumed to be defined elsewhere) to access the specified folder.
- **Global Variables**:
    - `currentFolderId`: A global variable used for tracking the folder currently being processed.
- **Functions**:
    - `logInfo`: A logging function (assumed to be defined elsewhere) used to record the resumption event.
    - `processDocuments(folderId, isResume)`: A function (assumed to be defined elsewhere) that processes documents in the specified folder. It includes an `isResume` flag to differentiate between resumed and new processing tasks.

## Notes

- The folder ID is hardcoded, indicating this function is specifically tailored for a particular use case or folder.
- It relies significantly on external functions (`processDocuments`, `logInfo`), positioning it as a small but essential component of a larger system.
- It is designed for a Google Apps Script environment, presumably interacting with Google Drive for document storage and retrieval.


## CODE
```
function resumeDocumentProcessing() {

  const folderId = "1aXbgh-P3ups7tQjRiMIxa_fNn3UiXumo";

  currentFolderId = folderId; // Set currentFolderId for logging

  logInfo(`Resuming document processing for folder ID: ${folderId}`);

  processDocuments(folderId, true); // Pass true to indicate this is a resume

}
```


# ######CaseBriefFunctions.js 

## Overview

This JavaScript file contains functions for generating comprehensive case briefs for legal matters. It's designed to work within Google Apps Script, integrating with Google Drive, Spreadsheets, and Documents to create organized analyses of legal cases and related documents.

## Core Functionality

### Primary Functions

1. **`generateCaseBrief(folderId, reprocessDocuments)`**
    
    - The main function that orchestrates the case brief generation process
    - Processes documents, extracts case insights, and creates a structured case brief with timeline
    - Parameters:
        - `folderId`: Google Drive folder ID containing case documents
        - `reprocessDocuments`: Boolean flag to control whether documents should be reanalyzed
2. **`generateTimelineWithGemini(allAnalyses, caseInsights, caseSnapshot)`**
    
    - Generates a chronological timeline of case events using Google's Gemini AI
    - Filters, prioritizes, and batches document analyses for processing
    - Creates multiple sections including case summary, supporting facts, challenges, and strategy
3. **`fetchCaseSnapshotFromZoho(folderId)`**
    
    - Retrieves case action plan snapshot from Zoho CRM
    - Includes authentication with the Zoho API via OAuth token
4. **`processMultipleFolders()`**
    
    - Processes multiple case folders in sequence
    - Contains a configurable list of folder IDs to process

### Supporting Functions

- **`generateProcessDocument(folderId, documentAnalyses, snapshot, caseBriefSections)`**: Creates a Google Doc with the case brief, snapshot, and AI research prompt
- **`updateCaseBriefWorksheet(spreadsheet, caseBrief)`**: Updates a spreadsheet with structured case brief content
- **`parseCaseBriefSections(caseBrief)`**: Parses raw case brief text into structured sections
- **`generateCaseInsightsWithGemini(allAnalyses, existingInsights, caseSnapshot)`**: Generates case insights using Gemini AI
- **`generateAIResearchPrompt(caseBriefSections, snapshot, documentAnalyses)`**: Creates a dynamic prompt for additional legal research
- **`refreshZohoToken()`**: Handles Zoho API authentication token refresh

## Integration Points

1. **Google Services**
    
    - Google Drive: File and folder management
    - Google Sheets: Document analysis storage and case brief presentation
    - Google Docs: Process document generation
2. **External APIs**
    
    - Zoho CRM: Retrieves case action plan snapshots
    - Gemini AI: AI-powered text generation for case analysis
3. **File Organization**
    
    - Works with a specific folder structure and spreadsheet format
    - Recognizes a "Tab Mapping" system for organizing documents by subfolder

## Technical Details

- **Error Handling**: Comprehensive try-catch blocks with detailed logging
- **Logging System**: Custom logging functions (logInfo, logWarn, logError)
- **Batching Logic**: Processes documents in manageable batches to avoid timeouts
- **Document Prioritization**: Sophisticated algorithm for prioritizing documents based on type and content
- **Test Functions**: Includes testing functions for verifying functionality

## API Credentials

- Contains Zoho API credentials (client ID, client secret)
- Uses Google Apps Script Properties Service for secure token storage

## Usage

The code appears to be part of a larger legal document processing system called "AI LEGAL GOD" that assists attorneys with case analysis and organization.
## CODE
```
/**

 * Case Brief Functions

 * Integrates Zoho CRM snapshot using GOOGLE_DRIVE field in Deals module.

 */

  

// Configuration

const CONFIG = { BATCH_SIZE: 20 };

  

// Function to call Gemini API for case brief (assumed to be defined elsewhere)

function callGeminiForCaseBrief(prompt) {

  logInfo(`Calling Gemini API for case brief`);

  const payload = {

    contents: [{ parts: [{ text: prompt }] }]

  };

  const url = `https://generativelanguage.googleapis.com/v1beta/models/${GEMINI_MODEL}:generateContent?key=${GEMINI_API_KEY}`;

  const options = {

    method: "post",

    contentType: "application/json",

    payload: JSON.stringify(payload),

    muteHttpExceptions: true

  };

  let response = UrlFetchApp.fetch(url, options);

  let responseCode = response.getResponseCode();

  let responseText = response.getContentText();

  if (responseCode !== 200) {

    logError(`Gemini API error: ${responseCode} - ${responseText}`);

    throw new Error(`Gemini API error: ${responseCode} - ${responseText}`);

  }

  let jsonResponse = JSON.parse(responseText);

  if (!jsonResponse.candidates || jsonResponse.candidates.length === 0) {

    logError(`No candidates in Gemini API response: ${responseText}`);

    throw new Error("No candidates in Gemini API response");

  }

  logInfo(`Gemini API response: ${jsonResponse.candidates[0].content.parts[0].text}`);

  return jsonResponse.candidates[0].content.parts[0].text;

}

  

function fetchCaseSnapshotFromZoho(folderId) {

  try {

    // Step 1: Search for the Deal to get the dealId

    const zohoSearchUrl = "https://www.zohoapis.com/crm/v7/Deals/search";

    let oauthToken = PropertiesService.getScriptProperties().getProperty("ZOHO_OAUTH_TOKEN");

    const folderLink = `https://drive.google.com/drive/folders/${folderId}`;

    const searchParams = {

      criteria: `(GOOGLE_DRIVE:equals:${encodeURIComponent(folderLink)})`

    };

    const searchQueryString = Object.keys(searchParams).map(k => `${k}=${searchParams[k]}`).join('&');

    const searchOptions = {

      method: "get",

      headers: { "Authorization": `Zoho-oauthtoken ${oauthToken}` },

      muteHttpExceptions: true

    };

    logInfo(`Searching for Deal with URL: ${zohoSearchUrl}?${searchQueryString}`);

    let searchResponse = UrlFetchApp.fetch(`${zohoSearchUrl}?${searchQueryString}`, searchOptions);

    if (searchResponse.getResponseCode() === 401) {

      logInfo("Access token expired, refreshing...");

      oauthToken = refreshZohoToken();

      searchOptions.headers.Authorization = `Zoho-oauthtoken ${oauthToken}`;

      searchResponse = UrlFetchApp.fetch(`${zohoSearchUrl}?${searchQueryString}`, searchOptions);

    }

    if (searchResponse.getResponseCode() !== 200) {

      logError(`Zoho API search error: ${searchResponse.getContentText()}`);

      throw new Error(`Zoho API search error: ${searchResponse.getContentText()}`);

    }

    const searchResult = JSON.parse(searchResponse.getContentText());

    if (!searchResult.data || searchResult.data.length === 0) {

      logWarn(`No Deal found for folder ID: ${folderId}`);

      throw new Error(`No Deal found for folder ID: ${folderId}`);

    }

    const dealId = searchResult.data[0].id;

    logInfo(`Found dealId ${dealId} for folderId ${folderId}`);

  

    // Step 2: Update the Trigger_Snapshot_Update field to trigger the workflow rule

    const updateUrl = `https://www.zohoapis.com/crm/v7/Deals/${dealId}`;

    const updateBody = {

      data: [{

        Trigger_Snapshot_Update: "Trigger Snapshot Update - " + new Date().toISOString()

      }]

    };

    const updateOptions = {

      method: "put",

      headers: { "Authorization": `Zoho-oauthtoken ${oauthToken}` },

      contentType: "application/json",

      payload: JSON.stringify(updateBody),

      muteHttpExceptions: true

    };

    logInfo(`Updating Deal to trigger snapshot update: URL: ${updateUrl}, Body: ${JSON.stringify(updateBody)}`);

    let updateResponse = UrlFetchApp.fetch(updateUrl, updateOptions);

    if (updateResponse.getResponseCode() === 401) {

      logInfo("Access token expired, refreshing...");

      oauthToken = refreshZohoToken();

      updateOptions.headers.Authorization = `Zoho-oauthtoken ${oauthToken}`;

      updateResponse = UrlFetchApp.fetch(updateUrl, updateOptions);

    }

    if (updateResponse.getResponseCode() !== 200) {

      logError(`Failed to update Deal to trigger snapshot: ${updateResponse.getContentText()}`);

      throw new Error(`Failed to update Deal to trigger snapshot: ${updateResponse.getContentText()}`);

    }

    logInfo(`Successfully updated Deal to trigger snapshot: ${updateResponse.getContentText()}`);

  

    // Wait 30 seconds for the workflow rule to execute and the snapshot to be generated

    logInfo("Waiting 30 seconds for the snapshot to be generated...");

    Utilities.sleep(30000);

  

    // Step 3: Fetch the full Case_Action_Plan content using the fetch_full_data endpoint

    const zohoFetchFullDataUrl = `https://www.zohoapis.com/crm/v7/Deals/${dealId}/actions/fetch_full_data`;

    const getOptions = {

      method: "get",

      headers: { "Authorization": `Zoho-oauthtoken ${oauthToken}` },

      muteHttpExceptions: true

    };

    const getParams = {

      fields: "Case_Action_Plan,Trigger_Snapshot_Update"

    };

    const getQueryString = Object.keys(getParams).map(k => `${k}=${encodeURIComponent(getParams[k])}`).join('&');

    logInfo(`Fetching updated snapshot with URL: ${zohoFetchFullDataUrl}?${getQueryString}`);

    let getResponse = UrlFetchApp.fetch(`${zohoFetchFullDataUrl}?${getQueryString}`, getOptions);

    if (getResponse.getResponseCode() === 401) {

      logInfo("Access token expired, refreshing...");

      oauthToken = refreshZohoToken();

      getOptions.headers.Authorization = `Zoho-oauthtoken ${oauthToken}`;

      getResponse = UrlFetchApp.fetch(`${zohoFetchFullDataUrl}?${getQueryString}`, getOptions);

    }

    if (getResponse.getResponseCode() !== 200) {

      logError(`Zoho API fetch_full_data error: ${getResponse.getContentText()}`);

      throw new Error(`Zoho API fetch_full_data error: ${getResponse.getContentText()}`);

    }

    const getResult = JSON.parse(getResponse.getContentText());

    if (!getResult.data || getResult.data.length === 0) {

      logWarn(`No Deal found for dealId: ${dealId}`);

      throw new Error(`No Deal found for dealId: ${dealId}`);

    }

    logInfo(`Deal record (fetch_full_data): ${JSON.stringify(getResult.data[0], null, 2)}`);

    let snapshot = getResult.data[0].Case_Action_Plan || "";

  

    // Step 4: Save the snapshot to the Attorneys tab

    const folder = DriveApp.getFolderById(folderId);

    const files = folder.getFilesByName("Litigation Document Analysis");

    if (!files.hasNext()) {

      logWarn("Spreadsheet 'Litigation Document Analysis' not found in folder");

      throw new Error("Spreadsheet 'Litigation Document Analysis' not found in folder");

    }

    const spreadsheet = SpreadsheetApp.open(files.next());

    // Use the Tab Mapping tab to find the correct tab for "Attorney & Paralegal Notes"

    const tabMappingSheet = spreadsheet.getSheetByName("Tab Mapping");

    let attorneysTab = null;

    if (tabMappingSheet) {

      const tabMappingData = tabMappingSheet.getDataRange().getValues();

      for (let i = 1; i < tabMappingData.length; i++) {

        const mappedTabType = tabMappingData[i][2]; // Column C: Mapped Tab Type

        if (mappedTabType === "Attorney & Paralegal Notes") {

          const subfolderName = tabMappingData[i][0]; // Column A: Subfolder Name

          const tabName = subfolderName === "(Root)" ? "Root" : subfolderName.replace(/[^a-zA-Z0-9]/g, '_');

          attorneysTab = spreadsheet.getSheetByName(tabName);

          break;

        }

      }

    }

  

    if (!attorneysTab) {

      logWarn("Attorneys tab (for 'Attorney & Paralegal Notes') not found in spreadsheet");

    } else {

      // Find the "Case Action Plan Snapshot" row

      let snapshotRowIndex = -1;

      for (let i = 1; i <= attorneysTab.getLastRow(); i++) {

        const key = attorneysTab.getRange(i, 1).getValue();

        if (key === "Case Action Plan Snapshot") {

          snapshotRowIndex = i;

          break;

        }

      }

  

      if (snapshotRowIndex === -1) {

        logWarn("Case Action Plan Snapshot field not found in Attorneys tab");

      } else {

        attorneysTab.getRange(snapshotRowIndex, 2).setValue(snapshot);

        logInfo("Saved Case Action Plan Snapshot to Attorneys tab");

      }

    }

  

    // Step 5: Verify the snapshot content and freshness

    const currentDate = new Date();

    const todayStr = Utilities.formatDate(currentDate, "GMT", "MM/dd/yyyy");

    const snapshotDateMatch = snapshot.match(/Updated as of: (\d{2}\/\d{2}\/\d{4} \d{1,2}:\d{2} [AP]M)/);

    if (snapshotDateMatch) {

      const snapshotDateStr = snapshotDateMatch[1];

      const snapshotDate = new Date(snapshotDateStr);

      const snapshotDateOnly = Utilities.formatDate(snapshotDate, "GMT", "MM/dd/yyyy");

      if (snapshotDateOnly !== todayStr) {

        logWarn(`Snapshot is not fresh (Updated as of: ${snapshotDateStr}). It should be from today (${todayStr}). Retrying field update...`);

        updateResponse = UrlFetchApp.fetch(updateUrl, updateOptions);

        if (updateResponse.getResponseCode() === 401) {

          logInfo("Access token expired, refreshing...");

          oauthToken = refreshZohoToken();

          updateOptions.headers.Authorization = `Zoho-oauthtoken ${oauthToken}`;

          updateResponse = UrlFetchApp.fetch(updateUrl, updateOptions);

        }

        if (updateResponse.getResponseCode() !== 200) {

          logError(`Failed to retrigger snapshot update: ${updateResponse.getContentText()}`);

          throw new Error(`Failed to retrigger snapshot update: ${updateResponse.getContentText()}`);

        }

        logInfo(`Successfully retriggered snapshot update: ${updateResponse.getContentText()}`);

        logInfo("Waiting 30 seconds for the snapshot to be regenerated...");

        Utilities.sleep(30000);

        getResponse = UrlFetchApp.fetch(`${zohoFetchFullDataUrl}?${getQueryString}`, getOptions);

        if (getResponse.getResponseCode() === 401) {

          logInfo("Access token expired, refreshing...");

          oauthToken = refreshZohoToken();

          getOptions.headers.Authorization = `Zoho-oauthtoken ${oauthToken}`;

          getResponse = UrlFetchApp.fetch(`${zohoFetchFullDataUrl}?${getQueryString}`, getOptions);

        }

        if (getResponse.getResponseCode() !== 200) {

          logError(`Zoho API fetch_full_data error on retry: ${getResponse.getContentText()}`);

          throw new Error(`Zoho API fetch_full_data error on retry: ${getResponse.getContentText()}`);

        }

        const retryResult = JSON.parse(getResponse.getContentText());

        if (!retryResult.data || retryResult.data.length === 0) {

          logWarn(`No Deal found for dealId: ${dealId} on retry`);

          throw new Error(`No Deal found for dealId: ${dealId} on retry`);

        }

        logInfo(`Deal record (fetch_full_data after retry): ${JSON.stringify(retryResult.data[0], null, 2)}`);

        snapshot = retryResult.data[0].Case_Action_Plan || "";

  

        // Save the retried snapshot to the Attorneys tab

        const filesRetry = folder.getFilesByName("Litigation Document Analysis");

        if (filesRetry.hasNext()) {

          const spreadsheetRetry = SpreadsheetApp.open(filesRetry.next());

          const tabMappingSheetRetry = spreadsheetRetry.getSheetByName("Tab Mapping");

          let attorneysTabRetry = null;

          if (tabMappingSheetRetry) {

            const tabMappingDataRetry = tabMappingSheetRetry.getDataRange().getValues();

            for (let i = 1; i < tabMappingDataRetry.length; i++) {

              const mappedTabType = tabMappingDataRetry[i][2];

              if (mappedTabType === "Attorney & Paralegal Notes") {

                const subfolderName = tabMappingDataRetry[i][0];

                const tabName = subfolderName === "(Root)" ? "Root" : subfolderName.replace(/[^a-zA-Z0-9]/g, '_');

                attorneysTabRetry = spreadsheetRetry.getSheetByName(tabName);

                break;

              }

            }

          }

          if (attorneysTabRetry) {

            let snapshotRowIndexRetry = -1;

            for (let i = 1; i <= attorneysTabRetry.getLastRow(); i++) {

              const key = attorneysTabRetry.getRange(i, 1).getValue();

              if (key === "Case Action Plan Snapshot") {

                snapshotRowIndexRetry = i;

                break;

              }

            }

            if (snapshotRowIndexRetry !== -1) {

              attorneysTabRetry.getRange(snapshotRowIndexRetry, 2).setValue(snapshot);

              logInfo("Saved retried Case Action Plan Snapshot to Attorneys tab");

            }

          }

        }

      }

    } else {

      logWarn("Could not parse 'Updated as of' date from snapshot. Proceeding with available content.");

    }

  

    logInfo(`Successfully fetched snapshot for dealId ${dealId}: ${snapshot.substring(0, 100)}...`);

    return snapshot;

  } catch (e) {

    logError(`Error in fetchCaseSnapshotFromZoho for folderId ${folderId}: ${e.toString()}`);

    throw new Error(`Failed to fetch Case Action Plan Snapshot for folderId ${folderId}: ${e.message}`);

  }

}

  

// Function to process multiple folders

function processMultipleFolders() {

  // List of folder IDs to process (replace with your folder IDs)

  const folderIds = [

    "1aXbgh-P3ups7tQjRiMIxa_fNn3UiXumo", // Mary Hunt's folder

    // Add more folder IDs here

    "another-folder-id-1",

    "another-folder-id-2"

  ];

  

  for (const folderId of folderIds) {

    logInfo(`Processing folder: ${folderId}`);

    try {

      CaseBriefFunctions.generateCaseBrief(folderId);

      logInfo(`Successfully processed folder: ${folderId}`);

    } catch (e) {

      logError(`Failed to process folder ${folderId}: ${e.toString()}`);

    }

  }

}

  

function refreshZohoToken() {

  const clientId = "1000.94UYXL4J71ZXK6G6WNBCEP7BA79EQI";

  const clientSecret = "755032e9c2d8cb8329762a37a2ba56b22b8ec90d1b";

  const refreshToken = PropertiesService.getScriptProperties().getProperty("ZOHO_REFRESH_TOKEN");

  const url = "https://accounts.zoho.com/oauth/v2/token";

  const payload = {

    grant_type: "refresh_token",

    client_id: clientId,

    client_secret: clientSecret,

    refresh_token: refreshToken

  };

  const options = {

    method: "post",

    payload: payload,

    muteHttpExceptions: true

  };

  const response = UrlFetchApp.fetch(url, options);

  const result = JSON.parse(response.getContentText());

  if (response.getResponseCode() === 200) {

    const newAccessToken = result.access_token;

    PropertiesService.getScriptProperties().setProperty("ZOHO_OAUTH_TOKEN", newAccessToken);

    logInfo("Refreshed access token: " + newAccessToken);

    return newAccessToken;

  } else {

    logError("Token refresh failed: " + JSON.stringify(result));

    throw new Error("Failed to refresh Zoho token");

  }

}

  

function generateCaseBrief(folderId, reprocessDocuments = false) {

  try {

    logInfo(`Starting generateCaseBrief with folder ID: ${folderId}, reprocessDocuments: ${reprocessDocuments}`);

    initializeLogSheet(folderId);

  

    if (!folderId || typeof folderId !== 'string' || folderId.trim() === "") {

      logError(`Invalid folder ID: ${folderId}`);

      throw new Error("Folder ID cannot be empty or invalid in generateCaseBrief");

    }

  

    logInfo(`Accessing folder with ID: ${folderId}`);

    const folder = DriveApp.getFolderById(folderId);

    logInfo(`Folder accessed successfully: ${folder.getName()}`);

  

    // Fetch case insights from the Attorneys tab

    logInfo("Fetching case insights from Attorneys tab");

    const caseInsights = processAttorneysTab(folderId);

    logInfo("Case insights retrieved: " + JSON.stringify(caseInsights));

  

    // Optionally reprocess documents to update summaries with the latest snapshot

    if (reprocessDocuments) {

      logInfo("Reprocessing documents");

      const result = processDocuments(folderId, caseInsights);

      logInfo(`Document processing result: ${result}`);

    } else {

      logInfo("Skipping document reprocessing; using existing summaries");

    }

  

    // Fetch the snapshot (already saved in the Attorneys tab if reprocessDocuments was true)

    logInfo("Fetching Case Action Plan Snapshot from Zoho CRM");

    const caseSnapshot = fetchCaseSnapshotFromZoho(folderId);

    if (!caseSnapshot) {

      logWarn("No case snapshot retrieved; proceeding with documents only");

    } else {

      logInfo(`Fetched snapshot: ${caseSnapshot.substring(0, 100)}...`);

    }

  

    // Collect all summaries from the spreadsheet

    logInfo(`Searching for spreadsheet 'Litigation Document Analysis' in folder`);

    const files = folder.getFilesByName("Litigation Document Analysis");

    if (!files.hasNext()) {

      logError("Spreadsheet 'Litigation Document Analysis' not found");

      throw new Error("Spreadsheet 'Litigation Document Analysis' not found");

    }

    const file = files.next();

    const spreadsheetId = file.getId();

    logInfo(`Found spreadsheet with ID: ${spreadsheetId}`);

    const spreadsheet = SpreadsheetApp.open(file);

    logInfo(`Spreadsheet opened: ${spreadsheet.getName()}`);

  

    const tabMappingSheet = spreadsheet.getSheetByName("Tab Mapping");

    if (!tabMappingSheet) {

      logError("Tab Mapping tab not found");

      throw new Error("Tab Mapping tab not found");

    }

  

    const tabMappingData = tabMappingSheet.getDataRange().getValues();

    const subfolderMap = new Map();

    for (let i = 1; i < tabMappingData.length; i++) {

      const subfolderName = tabMappingData[i][0];

      const tabType = tabMappingData[i][2];

      const priority = tabMappingData[i][3];

      subfolderMap.set(subfolderName, { tabType, priority });

    }

    logInfo(`Loaded ${subfolderMap.size} subfolders from Tab Mapping`);

  

    const allAnalyses = [];

    const subfolderNames = Array.from(subfolderMap.keys());

    for (const subfolderName of subfolderNames) {

      let tabName = subfolderName === "(Root)" ? "Root" : subfolderName.replace(/[^a-zA-Z0-9]/g, '_');

      const tab = spreadsheet.getSheetByName(tabName);

      if (!tab) {

        logWarn(`Tab for subfolder ${subfolderName} not found, skipping`);

        continue;

      }

  

      let summaries = [];

      let columnIndices = {};

      let startRow = 1;

      const maxRowsToCheck = Math.min(tab.getLastRow(), 20);

      for (let row = 1; row <= maxRowsToCheck; row++) {

        if (tab.getRange(row, 1).getValue() === "[Document Summaries]") {

          startRow = row + 1;

          break;

        }

      }

      const headers = tab.getRange(startRow, 1, 1, tab.getLastColumn()).getValues()[0];

      headers.forEach((header, index) => columnIndices[header] = index);

  

      const numRows = tab.getLastRow() - startRow;

      if (numRows > 0) {

        summaries = tab.getRange(startRow + 1, 1, numRows, tab.getLastColumn()).getValues();

      }

      logInfo(`Found ${summaries.length} summaries in tab ${tabName}`);

  

      summaries.forEach(summary => {

        allAnalyses.push({

          summary,

          columnIndices,

          sourceTab: tabName,

          subfolderName,

          priority: subfolderMap.get(subfolderName).priority

        });

      });

    }

    logInfo(`Total document analyses collected: ${allAnalyses.length}`);

  

    if (allAnalyses.length === 0) {

      logWarn("No document analyses found");

      return "No document analyses found to generate case brief";

    }

  

    const uniqueAnalyses = [];

    const seenFilenames = new Set();

    for (const entry of allAnalyses) {

      const filename = entry.summary[entry.columnIndices["Filename"]];

      if (!seenFilenames.has(filename)) {

        seenFilenames.add(filename);

        uniqueAnalyses.push(entry);

      } else {

        const existingEntry = uniqueAnalyses.find(e => e.summary[e.columnIndices["Filename"]] === filename);

        if (existingEntry.sourceTab === "Root" && entry.sourceTab !== "Root") {

          uniqueAnalyses.splice(uniqueAnalyses.indexOf(existingEntry), 1, entry);

        }

      }

    }

    logInfo(`Deduplicated analyses, final count: ${uniqueAnalyses.length}`);

  

    // Generate the case brief

    logInfo("Checking if generateTimelineWithGemini is defined");

    if (typeof generateTimelineWithGemini === "function") {

      logInfo("generateTimelineWithGemini is defined, proceeding with call");

    } else {

      logError("generateTimelineWithGemini is NOT defined, aborting");

      throw new Error("generateTimelineWithGemini is not defined");

    }

    const caseBrief = generateTimelineWithGemini(uniqueAnalyses, caseInsights, caseSnapshot);

logInfo("Case brief generated with Gemini");

  

logInfo("Parsing case brief sections");

const sections = parseCaseBriefSections(caseBrief);

logInfo(`Extracted ${sections.length} sections from case brief`);

  

const timelineSections = sections.filter(s => {

  const match = s.title.match(/^(january|february|march|april|may|june|july|august|september|october|november|december)\s+\d{4}(\s*\([^)]+\))?/i);

  return match;

});

logInfo(`Extracted ${timelineSections.length} timeline sections from case brief`);

  

// Sort timeline sections chronologically

const monthOrder = {

  january: 1, february: 2, march: 3, april: 4, may: 5, june: 6,

  july: 7, august: 8, september: 9, october: 10, november: 11, december: 12

};

timelineSections.sort((a, b) => {

  const [monthA, yearA] = a.title.split(' ');

  const [monthB, yearB] = b.title.split(' ');

  const yearDiff = parseInt(yearA) - parseInt(yearB);

  if (yearDiff !== 0) return yearDiff;

  return monthOrder[monthA.toLowerCase()] - monthOrder[monthB.toLowerCase()];

});

logInfo("Sorted timeline sections chronologically");

  

logInfo("Generating process document");

const processDocUrl = generateProcessDocument(folderId, uniqueAnalyses, caseSnapshot, sections);

    logInfo(`Process document generated: ${processDocUrl}`);

  

    // Update the Case Brief tab

    logInfo("Updating Case Brief tab");

    updateCaseBriefWorksheet(spreadsheet, caseBrief);

    logInfo("Case Brief tab updated");

  

    // Generate case insights (for logging only, not saving to Attorneys tab)

    logInfo("Generating case insights with Gemini");

    const generatedInsights = generateCaseInsightsWithGemini(uniqueAnalyses, caseInsights, caseSnapshot);

    logInfo("Generated Case Insights:\n" + JSON.stringify(generatedInsights, null, 2));

  

    flushLogs(folderId);

    logInfo("Case brief, process document, and insights generated successfully");

    return "Case brief, process document, and insights generated successfully";

  } catch (e) {

    logError(`Error in generateCaseBrief: ${e.message}`);

    flushLogs(folderId);

    throw e;

  }

}

function generateCaseInsightsWithGemini(allAnalyses, existingInsights, caseSnapshot) {

  const today = new Date().toISOString().split('T')[0];

  logInfo(`Generating case insights as of ${today}`);

  

  const analysesData = allAnalyses.map(doc => {

    const summaryObj = {};

    for (const header in doc.columnIndices) {

      summaryObj[header] = doc.summary[doc.columnIndices[header]] || "N/A";

    }

    return `

DOCUMENT: ${summaryObj["Filename"] || "Unknown"}

TYPE: ${summaryObj["Document Type"] || "Unknown"}

FOLDER PATH: ${summaryObj["File Path"] || "Unknown"}

SOURCE TAB: ${doc.sourceTab}

TAB PRIORITY: ${doc.priority}

SUMMARY: ${summaryObj["Summary"] || ""}

KEY INFORMATION: ${summaryObj["Key Information"] || ""}

DATES: ${summaryObj["Relevant Dates"] || ""}

PARTIES: ${summaryObj["Key Parties"] || ""}

LEGAL REFERENCES: ${summaryObj["Legal References"] || "None"}

BATES/EXHIBIT: ${summaryObj["Bates/Exhibit Labels"] || "None"}

BATES RANGE: ${summaryObj["Bates Range"] || "None"}

${Object.keys(summaryObj)

  .filter(key => !["Filename", "File Path", "Document Type", "Summary", "Key Information", "Relevant Dates", "Key Parties", "Link", "Processed", "Helps Us", "Hurts Us", "Next Actions", "Legal References", "Bates/Exhibit Labels", "Bates Range"].includes(key))

  .map(key => `${key.toUpperCase()}: ${summaryObj[key]}`)

  .join("\n")}

`;

  }).join('\n---\n');

  

  const insightsPrompt = `

As of ${today}, generate case insights based on the document summaries and CRM snapshot below, incorporating any existing insights:

  

**Existing Case Insights:**

- Case Overview: ${existingInsights.caseOverview || "N/A"}

- Strategy: ${existingInsights.strategy || "N/A"}

- Claims: ${existingInsights.claims || "N/A"}

- Key Issues: ${existingInsights.keyIssues || "N/A"}

- Summary Guidance: ${existingInsights.summaryGuidance || "N/A"}

- Helps Us Guidance: ${existingInsights.helpsUsGuidance || "N/A"}

- Hurts Us Guidance: ${existingInsights.hurtsUsGuidance || "N/A"}

  

**CRM Snapshot:**

${caseSnapshot || "No CRM snapshot available."}

  

Generate the following insights based on the provided document summaries and CRM snapshot, refining or expanding on existing insights where applicable:

1. Case Overview: A concise overview of the case (3-5 sentences) summarizing what it’s about, the main dispute, and the stakes.

2. Strategy: A suggested strategy to approach the case (3-5 sentences), focusing on leveraging strengths and mitigating weaknesses.

3. Claims: Key claims being made by either party (3-5 sentences), derived from pleadings, contracts, or correspondence.

4. Key Issues: Core issues that will determine the case outcome (3-5 sentences), based on patterns in the summaries.

5. Summary Guidance: High-level guidance for summarizing the case to stakeholders (3-5 sentences).

6. Helps Us Guidance: Specific actions or arguments we can use to strengthen our position (3-5 sentences), tied to helpful documents.

7. Hurts Us Guidance: Specific risks or counterarguments we need to address (3-5 sentences), tied to harmful documents.

  

Output in a structured format:

- Case Overview: [Your text here]

- Strategy: [Your text here]

- Claims: [Your text here]

- Key Issues: [Your text here]

- Summary Guidance: [Your text here]

- Helps Us Guidance: [Your text here]

- Hurts Us Guidance: [Your text here]

  

Use only the provided data.

  

SUMMARIES:

${analysesData}

`;

  

  try {

    const insightsResult = callGeminiForCaseBrief(insightsPrompt);

    logInfo("Case insights generated with Gemini");

  

    const insights = {};

    const lines = insightsResult.split('\n');

    let currentKey = null;

    for (const line of lines) {

      const match = line.match(/^-\s*([^:]+):\s*(.*)$/);

      if (match) {

        currentKey = match[1].trim();

        insights[currentKey] = match[2].trim();

      } else if (currentKey && line.trim()) {

        insights[currentKey] += "\n" + line.trim();

      }

    }

    return insights;

  } catch (error) {

    logError(`Error generating case insights: ${error.message}`);

    return {

      CaseOverview: "Error generating insight",

      Strategy: "Error generating insight",

      Claims: "Error generating insight",

      KeyIssues: "Error generating insight",

      SummaryGuidance: "Error generating insight",

      HelpsUsGuidance: "Error generating insight",

      HurtsUsGuidance: "Error generating insight"

    };

  }

}

function generateProcessDocument(folderId, documentAnalyses, snapshot, caseBriefSections) {

  logInfo(`Generating process document for folder: ${folderId}`);

  const folder = DriveApp.getFolderById(folderId);

  const doc = DocumentApp.create(`Process Document - ${folder.getName()} - ${new Date().toISOString().split('T')[0]}`);

  const docFile = DriveApp.getFileById(doc.getId());

  docFile.moveTo(folder);

  const body = doc.getBody();

  

  const teamEmails = ["daniel.herrin@herrinlaw.com", "nick.shahbazi@herrinlaw.com", "lua.faicol@herrinlaw.com"];

  teamEmails.forEach(email => docFile.addEditor(email));

  logInfo(`Shared process document with team: ${teamEmails.join(", ")}`);

  

  body.appendParagraph("Case Brief").setHeading(DocumentApp.ParagraphHeading.HEADING1);

  for (const section of caseBriefSections) {

    body.appendParagraph(section.title).setHeading(DocumentApp.ParagraphHeading.HEADING2);

    const paragraphs = section.content.split('\n').filter(line => line.trim());

    paragraphs.forEach(paragraph => body.appendParagraph(paragraph).setHeading(DocumentApp.ParagraphHeading.NORMAL));

    body.appendParagraph("");

  }

  

  body.appendParagraph("Case Action Plan Snapshot").setHeading(DocumentApp.ParagraphHeading.HEADING1);

  body.appendParagraph(snapshot || "N/A");

  body.appendParagraph("");

  

  body.appendParagraph("AI Research Prompt").setHeading(DocumentApp.ParagraphHeading.HEADING1);

  const aiPrompt = generateAIResearchPrompt(caseBriefSections, snapshot, documentAnalyses);

  const promptParagraphs = aiPrompt.split('\n').filter(line => line.trim());

  promptParagraphs.forEach(paragraph => body.appendParagraph(paragraph).setHeading(DocumentApp.ParagraphHeading.NORMAL));

  body.appendParagraph("");

  

  body.appendParagraph("Comments and Notes").setHeading(DocumentApp.ParagraphHeading.HEADING1);

  body.appendParagraph("Please add your comments, notes, or feedback on the case to collaborate with the team.").setHeading(DocumentApp.ParagraphHeading.NORMAL);

  body.appendParagraph("Last Updated: " + new Date().toISOString() + " | View Version History: " + doc.getUrl());

  body.appendParagraph("");

  

  logInfo(`Generated process document: ${doc.getUrl()}`);

  return doc.getUrl();

}

  

// Helper function to generate a dynamic AI research prompt based on the Case Brief, Case Action Plan Snapshot, and Document Summaries

function generateAIResearchPrompt(caseBriefSections, snapshot, documentAnalyses) {

  const sectionMap = {};

  caseBriefSections.forEach(section => sectionMap[section.title] = section.content);

  

  const caseSummary = sectionMap["Case Summary"] || "N/A";

  const factsSupportingUs = sectionMap["Facts/Evidence Supporting Us"] || "N/A";

  const challenges = sectionMap["Challenges"] || "N/A";

  const howWeWin = sectionMap["How We Win"] || "N/A";

  const helpfulDocs = sectionMap["Helpful Docs"] || "N/A";

  const harmfulDocs = sectionMap["Harmful Docs"] || "N/A";

  

  const upcomingDeadlinesMatch = snapshot.match(/UPCOMING DEADLINES & MEETINGS\s*([\s\S]*?)(TASKS|$)/);

  const upcomingDeadlines = upcomingDeadlinesMatch ? upcomingDeadlinesMatch[1].trim() : "N/A";

  const tasksMatch = snapshot.match(/TASKS\s*OPEN TASKS\s*([\s\S]*?)(COMPLETED TASKS|$)/);

  const openTasks = tasksMatch ? tasksMatch[1].trim() : "N/A";

  const actionPlanMatch = snapshot.match(/ACTION PLAN\s*([\s\S]*?)$/);

  const actionPlan = actionPlanMatch ? actionPlanMatch[1].trim() : "N/A";

  

  const caseNameMatch = caseSummary.match(/(Estate of [A-Za-z\s]+|Case of [A-Za-z\s]+)/i) || snapshot.match(/(Estate of [A-Za-z\s]+|Case of [A-Za-z\s]+)/i);

  const caseName = caseNameMatch ? caseNameMatch[0] : "the case";

  const jurisdictionMatch = caseSummary.match(/(in|at|of)\s+([A-Za-z\s]+(?:County|City)?,\s+[A-Za-z\s]+)/i) ||

                          snapshot.match(/(in|at|of)\s+([A-Za-z\s]+(?:County|City)?,\s+[A-Za-z\s]+)/i);

  const jurisdiction = jurisdictionMatch ? jurisdictionMatch[2].trim() : "the relevant jurisdiction";

  const firmNameMatch = caseSummary.match(/(represented by|attorney at)\s([A-Za-z\s,]+(LLP|PLLC|Law))/i) ||

                       snapshot.match(/(represented by|attorney at)\s([A-Za-z\s,]+(LLP|PLLC|Law))/i);

  const firmName = firmNameMatch ? firmNameMatch[2] : "the firm";

  

  const docReferences = [];

  const helpfulDocLines = helpfulDocs.split('\n').filter(line => line.trim() && line.match(/Doc \d+:/i));

  const harmfulDocLines = harmfulDocs.split('\n').filter(line => line.trim() && line.match(/Doc \d+:/i));

  helpfulDocLines.concat(harmfulDocLines).forEach(line => {

    const docMatch = line.match(/Doc \d+:\s*([^\(]+)\s*(\(Bates Range:[^\)]+\))?/i);

    if (docMatch) {

      const filename = docMatch[1].trim();

      const analysis = documentAnalyses.find(doc =>

        (doc.summary[doc.columnIndices["Filename"]] || "").toLowerCase() === filename.toLowerCase()

      );

      if (analysis && !docReferences.some(ref => ref.summary[ref.columnIndices["Filename"]] === filename)) {

        docReferences.push(analysis);

      }

    }

  });

  

  const criticalDocs = docReferences.slice(0, 5);

  const documentSummaries = criticalDocs.length > 0 ? criticalDocs.map((doc, index) => {

    const summaryObj = {};

    for (const header in doc.columnIndices) summaryObj[header] = doc.summary[doc.columnIndices[header]] || "N/A";

    return `

DOCUMENT ${index + 1}:

  DOCUMENT: ${summaryObj["Filename"] || "Unknown"}

  SUMMARY: ${summaryObj["Summary"] || "N/A"}

  KEY INFORMATION: ${summaryObj["Key Information"] || "N/A"}

  HELPS US: ${summaryObj["Helps Us"] || "N/A"}

  HURTS US: ${summaryObj["Hurts Us"] || "N/A"}

`;

  }).join('\n---\n') : "No critical document summaries included.";

  

  const legalIssues = [];

  const keywords = /procedural|legal|court|attorney-client|fiduciary|balance|payment|withdrawal|contract|breach|evidence|discovery|motion|settlement|jurisdiction|negligence/i;

  const sources = [challenges, howWeWin, actionPlan];

  sources.forEach(source => {

    source.split('\n').filter(line => line.trim()).forEach(line => {

      if (line.match(keywords)) legalIssues.push(line);

    });

  });

  

  const openLegalQuestions = [];

  const uniqueIssues = [...new Set(legalIssues)];

  const priorityIssues = uniqueIssues.filter(issue =>

    issue.match(/withdrawal|procedural|payment|attorney-client/i)

  ).slice(0, 5);

  

  priorityIssues.forEach(issue => {

    const q = `What are the legal implications in ${jurisdiction} of "${issue}" as it relates to ${caseName}'s current status and action plan?`;

    if (!openLegalQuestions.includes(q)) openLegalQuestions.push(q);

  });

  

  if (openLegalQuestions.length === 0) {

    openLegalQuestions.push(`What key legal issues in ${jurisdiction} could impact ${caseName}'s outcome based on the brief?`);

  }

  

  const prompt = `

**AI Research Prompt for ${caseName}**

  

**Case Context and Current Status:**  

${caseSummary}  

- **Upcoming Deadlines & Meetings:**  

${upcomingDeadlines}  

- **Open Tasks:**  

${openTasks}  

- **Action Plan:**  

${actionPlan}

  

**Key Facts and Evidence Supporting Us:**  

${factsSupportingUs}

  

**Challenges and Risks:**  

${challenges}

  

**Critical Documents (Referenced in Brief):**  

${documentSummaries}

  

**Research Objectives:**  

Support ${firmName} in ${caseName} with deep legal research, focusing on:  

1. **What We Need to Know:** Identify gaps in our understanding (e.g., implications of challenges, court perspectives, missing info) that could affect the case outcome, based on the context and status.  

2. **Open Legal Questions:**  

${openLegalQuestions.map((q, i) => `   - ${q}`).join('\n')}  

3. **Case Law Supporting Us:**  

   - Find ${jurisdiction} case law supporting our position on issues from the brief.  

   - Identify precedents aligning with our facts/evidence and action plan.  

   - Locate cases addressing challenges with mitigation guidance.  

4. **Additional Insights:**  

   - Offer strategic recommendations to address challenges and meet action plan goals.  

   - Highlight risks to ${firmName}, the client, or case status, with mitigation strategies.  

   - Suggest arguments/evidence to strengthen our position using the provided data.

  

**Output Format:**  

- **Open Legal Questions:** Detailed analysis per question, citing statutes/principles.  

- **Relevant Case Law:** 3-5 cases per question (name, citation, relevance summary).  

- **Strategic Recommendations:** Actionable steps forward.  

- **Potential Risks:** Risks and mitigation strategies.

`;

  

  return prompt;

}

  

// Update the Case Brief worksheet with a structured format

function updateCaseBriefWorksheet(spreadsheet, caseBrief) {

  logInfo("Updating Case Brief with content: " + caseBrief.substring(0, 200) + "...");

  try {

    let sheet = spreadsheet.getSheetByName("Case Brief");

    if (!sheet) {

      logInfo("Case Brief sheet not found, creating new...");

      sheet = spreadsheet.insertSheet("Case Brief");

    } else {

      sheet.clear();

    }

  

    sheet.getRange(1, 1, 1, 2).merge();

    sheet.getRange(1, 1).setValue("CASE BRIEF");

    sheet.getRange(1, 1).setFontWeight("bold").setFontSize(14).setBackground("#4285F4").setFontColor("white");

  

    sheet.getRange(2, 1).setValue("SECTION");

    sheet.getRange(2, 2).setValue("CONTENT");

    sheet.getRange(2, 1, 1, 2).setBackground("#D9D9D9").setFontWeight("bold").setBorder(true, true, true, true, true, true);

  

    sheet.setColumnWidth(1, 200);

    sheet.setColumnWidth(2, 600);

  

    const sectionMap = {

      "Case Summary": 3,

      "Facts/Evidence Supporting Us": 4,

      "Challenges": 5,

      "How We Win": 6,

      "Helpful Docs": 7,

      "Harmful Docs": 8,

      "Consolidated Timeline": 9

    };

  

    Object.keys(sectionMap).forEach(sectionTitle => {

      const row = sectionMap[sectionTitle];

      sheet.getRange(row, 1).setValue(sectionTitle);

      sheet.getRange(row, 1).setFontWeight("bold").setBackground("#E8F0FE");

      sheet.getRange(row, 1, 1, 2).setBorder(true, true, true, true, true, true);

      sheet.getRange(row, 2).setWrapStrategy(SpreadsheetApp.WrapStrategy.WRAP);

      sheet.setRowHeight(row, 30);

    });

  

    const sections = parseCaseBriefSections(caseBrief);

    logInfo("Parsed sections: " + JSON.stringify(sections.map(s => s.title)));

  

    sections.forEach(section => {

      const cleanTitle = section.title

        .replace(/^\d+\.\s*/, "")

        .replace(/\s*\(Consolidated\)/, "")

        .replace(/\s*\(Detailed Descriptions\)/, "")

        .replace(/\s*\(See below\)/, "")

        .trim();

      const row = sectionMap[cleanTitle];

      if (row) {

        logInfo(`Writing section: ${cleanTitle} to row ${row}`);

        let content = section.content;

  

        if (cleanTitle === "Helpful Docs" || cleanTitle === "Harmful Docs") {

          const docEntries = content.split('\n').filter(entry => entry.trim());

          content = docEntries.join('\n\n');

        }

  

        const nestedSections = sections.filter(s => s.title.startsWith(cleanTitle + " ") && !s.title.includes("Docs"));

        if (nestedSections.length > 0) {

          content += "\n\n";

          nestedSections.forEach(nested => {

            const nestedCleanTitle = nested.title.replace(cleanTitle + " ", "").trim();

            content += `${nestedCleanTitle}:\n${nested.content}\n\n`;

          });

        }

  

        sheet.getRange(row, 2).setValue(content);

        sheet.getRange(row, 2).setWrapStrategy(SpreadsheetApp.WrapStrategy.WRAP);

  

        const contentLength = content.length;

        const estimatedLines = Math.ceil(contentLength / 100);

        const estimatedHeight = Math.max(30, Math.min(500, estimatedLines * 20));

        sheet.setRowHeight(row, estimatedHeight);

  

        sheet.getRange(row, 1, 1, 2).setBorder(true, true, true, true, true, true);

      } else {

        logInfo(`Section not mapped: ${cleanTitle}`);

      }

    });

  

    let timelineRow = 10;

    const months = sections.filter(s => {

      const match = s.title.match(/^(january|february|march|april|may|june|july|august|september|october|november|december)\s+\d{4}(\s*\([^)]+\))?/i);

      if (match) {

        logInfo(`Found timeline month: ${s.title}`);

      } else {

        logInfo(`Not a timeline month: ${s.title}`);

      }

      return match;

    });

    logInfo(`Found ${months.length} timeline months: ${months.map(m => m.title).join(", ")}`);

    months.forEach(month => {

      logInfo(`Writing timeline month: ${month.title} at row ${timelineRow}`);

      sheet.getRange(timelineRow, 1).setValue(month.title);

      sheet.getRange(timelineRow, 1).setFontWeight("bold").setBackground("#E8F0FE");

      let content = month.content;

  

      const docSections = sections.filter(s => s.title.startsWith(month.title + ": Doc"));

      if (docSections.length > 0) {

        content += "\n\nDocuments:\n";

        docSections.forEach(doc => {

          content += `${doc.title.replace(month.title + ": ", "")}: ${doc.content}\n`;

        });

      }

  

      sheet.getRange(timelineRow, 2).setValue(content);

      sheet.getRange(timelineRow, 2).setWrapStrategy(SpreadsheetApp.WrapStrategy.WRAP);

  

      const monthLength = content.length;

      const monthLines = Math.ceil(monthLength / 100);

      const monthHeight = Math.max(30, Math.min(500, monthLines * 20));

      sheet.setRowHeight(timelineRow, monthHeight);

  

      sheet.getRange(timelineRow, 1, 1, 2).setBorder(true, true, true, true, true, true);

      timelineRow++;

    });

  

    logInfo("Case brief updated successfully");

    return sheet;

  } catch (error) {

    logError("Error updating case brief worksheet: " + error.message);

    return null;

  }

}

  

// Helper function to parse case brief into sections

function parseCaseBriefSections(caseBrief) {

  logInfo("Parsing case brief (first 500 chars): " + caseBrief.substring(0, 500) + "...");

  logInfo("Full case brief length: " + caseBrief.length);

  

  const sections = [];

  const lines = caseBrief.split('\n');

  let i = 0;

  

  while (i < lines.length) {

    const line = lines[i].trim();

    logInfo("Line " + i + ": " + line.substring(0, 100) + (line.length > 100 ? "..." : ""));

  

    const sectionMatch = line.match(/^(\d+\.\s)?\*\*([^\*]+?)(?::\*\*|\*\*|\s*\*\*)(.*)/);

    if (sectionMatch) {

      let title = sectionMatch[2].trim();

      title = title.replace(/^\d+\.\s*/, "").trim();

      let content = sectionMatch[3] ? sectionMatch[3].trim() : "";

      logInfo("Found section: " + title + " with initial content: " + (content ? content.substring(0, 50) + "..." : "none"));

      i++;

  

      while (i < lines.length) {

        const nextLine = lines[i].trim();

        logInfo("Checking line " + i + " for " + title + ": " + nextLine.substring(0, 100) + (nextLine.length > 100 ? "..." : ""));

  

        if (nextLine.match(/^(\d+\.\s)?\*\*([^\*]+?)(?::\*\*|\*\*|\s*\*\*)/)) {

          break;

        }

  

        if (nextLine) {

          content += (content ? "\n" : "") + nextLine;

          logInfo("Added to " + title + ": " + nextLine.substring(0, 50) + "...");

        }

        i++;

      }

  

      sections.push({ title: title, content: content.trim() });

      logInfo("Found section: " + title + " with final content length: " + content.length);

    } else {

      if (line) {

        logInfo("Line " + i + " ignored (not a section header): " + line.substring(0, 50) + "...");

      }

      i++;

    }

  }

  

  logInfo("Parsed sections: " + JSON.stringify(sections.map(s => ({

    title: s.title,

    content: s.content.substring(0, 50) + (s.content.length > 50 ? "..." : "")

  }))));

  

  return sections;

}

  

// Generate timeline brief with Gemini, incorporating case insights and CRM snapshot

function generateTimelineWithGemini(allAnalyses, caseInsights, caseSnapshot) {

  const today = new Date();

  const todayStr = today.toISOString().split('T')[0];

  logInfo(`Creating timeline brief with Gemini from document analyses as of ${todayStr}`);

  

  const filteredAnalyses = allAnalyses.filter(analysis => {

    const filePath = analysis.summary[analysis.columnIndices["File Path"]] || "";

    const summary = analysis.summary[analysis.columnIndices["Summary"]] || "";

    const keyInfo = analysis.summary[analysis.columnIndices["Key Information"]] || "";

    const isDraft = filePath.toLowerCase().includes("drafts") || summary.toLowerCase().includes("draft") || keyInfo.toLowerCase().includes("draft");

    return !isDraft;

  });

  logInfo(`Filtered ${allAnalyses.length - filteredAnalyses.length} draft documents`);

  

  const prioritizedAnalyses = filteredAnalyses.map((analysis, index) => {

    let priority = analysis.priority || "Low";

    const filePath = analysis.summary[analysis.columnIndices["File Path"]] || "";

    const fileName = analysis.summary[analysis.columnIndices["Filename"]] || "";

    const keyInfo = analysis.summary[analysis.columnIndices["Key Information"]] || "";

    const docType = analysis.summary[analysis.columnIndices["Document Type"]] || "";

    const summary = analysis.summary[analysis.columnIndices["Summary"]] || "";

  

    if (priority === "Low") {

      if (filePath.toLowerCase().includes("hearing preparation")) priority = "Highest";

      else if (docType.toLowerCase() === "pleading" || docType.toLowerCase().includes("court order")) priority = "High";

      else if (fileName.toLowerCase().includes("financial") || fileName.toLowerCase().includes("invoice") || fileName.toLowerCase().includes("affidavit") || fileName.toLowerCase().includes("records") || fileName.toLowerCase().includes("statement") || fileName.toLowerCase().includes("fees") || fileName.toLowerCase().includes("payment") || keyInfo.toLowerCase().includes("financial") || summary.toLowerCase().includes("financial")) priority = "High";

      else if (fileName.toLowerCase().includes("contract") || fileName.toLowerCase().includes("agreement") || fileName.toLowerCase().includes("subcontractor")) priority = "Medium-High";

      else if (docType.toLowerCase() === "discovery" || filePath.toLowerCase().includes("state case")) priority = "Medium";

    }

  

    return { ...analysis, priority, docId: index + 1 };

  });

  

  const batchSize = CONFIG.BATCH_SIZE;

  const batches = [];

  for (let i = 0; i < prioritizedAnalyses.length; i += batchSize) {

    batches.push(prioritizedAnalyses.slice(i, i + batchSize));

  }

  logInfo(`Split into ${batches.length} batches of ~${batchSize} documents each`);

  

  const startTime = new Date().getTime();

  const partialTimelines = [];

  for (let i = 0; i < batches.length; i++) {

    const elapsedTime = (new Date().getTime() - startTime) / 1000;

    if (elapsedTime > 300) {

      logInfo(`Time limit exceeded after ${i} batches`);

      break;

    }

  

    const batch = batches[i];

    const batchData = batch.map(doc => {

      const summaryObj = {};

      for (const header in doc.columnIndices) summaryObj[header] = doc.summary[doc.columnIndices[header]] || "N/A";

      return `

DOCUMENT: ${summaryObj["Filename"] || "Unknown"}

TYPE: ${summaryObj["Document Type"] || "Unknown"}

FOLDER PATH: ${summaryObj["File Path"] || "Unknown"}

SOURCE TAB: ${doc.sourceTab}

TAB PRIORITY: ${doc.priority}

SUMMARY: ${summaryObj["Summary"] || ""}

KEY INFORMATION: ${summaryObj["Key Information"] || ""}

DATES: ${summaryObj["Relevant Dates"] || ""}

PARTIES: ${summaryObj["Key Parties"] || ""}

LEGAL REFERENCES: ${summaryObj["Legal References"] || "None"}

BATES/EXHIBIT: ${summaryObj["Bates/Exhibit Labels"] || "None"}

BATES RANGE: ${summaryObj["Bates Range"] || "None"}

LINK: ${summaryObj["Link"] || "N/A"}

`;

    }).join('\n---\n');

  

    const batchPrompt = `

As of ${todayStr}, create a partial timeline brief for the case, batch ${i + 1} of ${batches.length}. Use the case insights, CRM snapshot, and summaries below:

  

**Case Insights:**

- Case Overview: ${caseInsights.caseOverview || "N/A"}

- Strategy: ${caseInsights.strategy || "N/A"}

- Claims: ${caseInsights.claims || "N/A"}

- Key Issues: ${caseInsights.keyIssues || "N/A"}

- Summary Guidance: ${caseInsights.summaryGuidance || "N/A"}

- Helps Us Guidance: ${caseInsights.helpsUsGuidance || "N/A"}

- Hurts Us Guidance: ${caseInsights.hurtsUsGuidance || "N/A"}

  

**CRM Snapshot:**

${caseSnapshot || "No CRM snapshot available."}

  

1. Categorize documents by type, folder path, source tab, and tab priority:

   - "Hearing Preparation": Likely exhibits/evidence, highest priority.

   - "Pleadings, Orders": Pleadings and court orders, high priority.

   - Financial documents: High priority (keywords: "Financial", "Invoice", "Records", "Fees", "Payment", "Affidavit", "Statement").

   - Contracts/Subcontracts: Medium-high priority (keywords: "Contract", "Agreement", "Subcontractor").

   - Discovery: Medium priority (completed phase).

   - Prior Lawsuits: Medium priority (folder: "State Case").

   - Others: Low priority.

   - Use TAB PRIORITY as the primary priority.

2. Identify patterns within categories:

   - Pleadings: Look for trends in legal strategies (e.g., motions to dismiss, compel discovery).

   - Financials: Highlight financial ranges and dates from SUMMARY and DATES.

   - Evidence/Exhibits: Note recurring claims or defenses.

3. Extract key events as concise narrative sentences, grouped by month:

   - Use exact dates from DATES, details from SUMMARY, filenames—e.g., "In March 2025, an emergency motion was filed alleging fraud [Doc X, Link: LINK]."

   - Include parties from PARTIES, evidence from BATES/EXHIBIT and BATES RANGE.

   - Allow up to 20 sentences per month, focusing on high-priority documents.

   - Deduplicate events.

   - Ensure all months between earliest and latest events are included (e.g., "February 2025: No significant events recorded.").

4. Flag critical documents:

   - Helpful: Supports our defense—provide a detailed description (2-3 paragraphs).

   - Harmful: Supports opposing claims—provide a detailed description (2-3 paragraphs).

   - Include 2-5 Helpful and 2-5 Harmful per batch.

5. Output:

   - Partial Timeline: Narrative by month—e.g., "In October 2024, the petition was filed [Doc X, Link: LINK]."

   - Helpful Docs: ~2-5—filenames, detailed descriptions.

   - Harmful Docs: ~2-5—filenames, detailed descriptions.

   - Use shorthand references in-text with links (e.g., "[Doc 1, Link: LINK]").

  

Use only provided data.

  

SUMMARIES:

${batchData}

`;

  

    try {

      const partialResult = callGeminiForCaseBrief(batchPrompt);

      partialTimelines.push(partialResult);

      logInfo(`Generated partial timeline for batch ${i + 1}`);

      Utilities.sleep(10000); // Avoid rate limits

    } catch (error) {

      logError(`Error in batch ${i + 1}: ${error.message}`);

      return `# Error in Batch ${i + 1}\n\n${error.message}`;

    }

  }

  

  const combinePromptPart1 = `

As of ${todayStr}, create a cohesive brief for the case:

  

**Case Insights:**

- Case Overview: ${caseInsights.caseOverview || "N/A"}

- Strategy: ${caseInsights.strategy || "N/A"}

- Claims: ${caseInsights.claims || "N/A"}

- Key Issues: ${caseInsights.keyIssues || "N/A"}

- Summary Guidance: ${caseInsights.summaryGuidance || "N/A"}

- Helps Us Guidance: ${caseInsights.helpsUsGuidance || "N/A"}

- Hurts Us Guidance: ${caseInsights.hurtsUsGuidance || "N/A"}

  

**CRM Snapshot:**

${caseSnapshot || "No CRM snapshot available."}

  

1. Case Summary: Core issues, stakes, ~5-15 sentences.

2. Facts/Evidence Supporting Us: Strengths, ~5-10 sentences, prioritize financial documents.

3. Challenges: Weaknesses or threats, ~5-10 sentences.

4. How We Win: Strategy to win, ~5-10 sentences.

5. Helpful Docs: ~2-5—filenames, descriptions (2-3 paragraphs).

6. Harmful Docs: ~2-5—filenames, descriptions (2-3 paragraphs).

   - Deduplicate across batches.

7. Reference Style: Use "[Doc X, Link: LINK]" in-text.

  

Use only provided data.

  

PARTIAL TIMELINES:

${partialTimelines.join('\n---\n')}

`;

  

  let part1Result;

  try {

    part1Result = callGeminiForCaseBrief(combinePromptPart1);

    logInfo("Part 1 of timeline brief generated with Gemini");

  } catch (error) {

    logError("Error generating part 1: " + error.message);

    return `# Error Generating Part 1\n\n${error.message}`;

  }

  

  const combinePromptPart2 = `

As of ${todayStr}, create the timeline section for a cohesive brief:

  

**CRM Snapshot:**

${caseSnapshot || "No CRM snapshot available."}

  

1. Consolidated Timeline: Merge partial timelines into one chronological narrative, grouped by month:

   - Keep natural language, deduplicate events.

   - Prioritize recent months (15-20 sentences), older months (5-10 sentences).

   - Summarize discovery as completed, focus on outcomes.

   - Include all months between earliest and latest events (e.g., "February 2025: No significant events recorded.").

   - Total ~80-120 sentences.

   - Use "[Doc X, Link: LINK]" for each event with a document reference.

2. Reference Style: Inline "[Doc X, Link: LINK]".

  

Use only provided data.

  

PARTIAL TIMELINES:

${partialTimelines.join('\n---\n')}

`;

  

  let part2Result;

  try {

    part2Result = callGeminiForCaseBrief(combinePromptPart2);

    logInfo("Part 2 of timeline brief generated with Gemini");

  } catch (error) {

    logError("Error generating part 2: " + error.message);

    return `# Error Generating Part 2\n\n${error.message}`;

  }

  

  const finalResult = `${part1Result}\n\n${part2Result}`;

  logInfo("Combined timeline brief fully generated");

  return finalResult;

}

  

// Placeholder for processAttorneysTab (assumed to be defined elsewhere)

function processAttorneysTab(folderId) {

  logInfo(`Processing Attorneys tab for folder ID: ${folderId}`);

  const spreadsheet = SpreadsheetApp.openById("1_iJPuy3yK5O-bt6Uibp9rGcGNMqu6KLoWXbVi198PhM"); // Replace with actual spreadsheet ID if needed

  const sheets = spreadsheet.getSheets();

  let attorneysSheet = null;

  for (let i = 0; i < sheets.length; i++) {

    const sheetName = sheets[i].getName().toLowerCase();

    if (sheetName.includes("attorney") || sheetName.includes("atty")) {

      attorneysSheet = sheets[i];

      break;

    }

  }

  if (!attorneysSheet) {

    logWarn("Attorneys tab not found");

    return {

      caseOverview: "",

      strategy: "",

      claims: "",

      keyIssues: "",

      summaryGuidance: "",

      helpsUsGuidance: "",

      hurtsUsGuidance: ""

    };

  }

  // Simplified placeholder logic; replace with actual parsing if needed

  return {

    caseOverview: "",

    strategy: "",

    claims: "",

    keyIssues: "",

    summaryGuidance: "",

    helpsUsGuidance: "",

    hurtsUsGuidance: ""

  };

}

  

// Placeholder logging functions (assumed to be defined elsewhere)

function logInfo(message) {

  Logger.log(`INFO: ${message}`);

}

  

function logWarn(message) {

  Logger.log(`WARN: ${message}`);

}

  

function logError(message) {

  Logger.log(`ERROR: ${message}`);

}

  

function initializeLogSheet(folderId) {

  // Placeholder for logging setup

  logInfo("Initialized log sheet for folder: " + folderId);

}

  

function flushLogs(folderId) {

  // Placeholder for flushing logs

  logInfo("Flushed logs for folder: " + folderId);

}

  

function testGenerateTimelineWithGemini() {

  logInfo("Testing if generateTimelineWithGemini is defined");

  if (typeof generateTimelineWithGemini === "function") {

    logInfo("generateTimelineWithGemini is defined");

  } else {

    logError("generateTimelineWithGemini is NOT defined");

  }

}

function testGenerateAIResearchPrompt() {

  // Folder ID to test

  const folderId = "1aXbgh-P3ups7tQjRiMIxa_fNn3UiXumo";

  

  // Fetch the snapshot (or use a static one from your last run)

  const snapshot = fetchCaseSnapshotFromZoho(folderId);

  // If you don’t want to hit Zoho, uncomment and paste your last snapshot here:

  // const snapshot = "Updated as of: 04/06/2025 05:13 PM\nCURRENT CASE SNAPSHOT\n...";

  

  // Fetch document analyses from the spreadsheet

  const spreadsheet = SpreadsheetApp.open(DriveApp.getFolderById(folderId).getFilesByName("Litigation Document Analysis").next());

  const tabMappingSheet = spreadsheet.getSheetByName("Tab Mapping");

  const tabMappingData = tabMappingSheet.getDataRange().getValues();

  const subfolderMap = new Map();

  for (let i = 1; i < tabMappingData.length; i++) {

    subfolderMap.set(tabMappingData[i][0], { tabType: tabMappingData[i][2], priority: tabMappingData[i][3] });

  }

  

  const allAnalyses = [];

  const subfolderNames = Array.from(subfolderMap.keys());

  for (const subfolderName of subfolderNames) {

    let tabName = subfolderName === "(Root)" ? "Root" : subfolderName.replace(/[^a-zA-Z0-9]/g, '_');

    const tab = spreadsheet.getSheetByName(tabName);

    if (!tab) continue;

  

    let summaries = [];

    let columnIndices = {};

    let startRow = 1;

    const maxRowsToCheck = Math.min(tab.getLastRow(), 20);

    for (let row = 1; row <= maxRowsToCheck; row++) {

      if (tab.getRange(row, 1).getValue() === "[Document Summaries]") {

        startRow = row + 1;

        break;

      }

    }

    const headers = tab.getRange(startRow, 1, 1, tab.getLastColumn()).getValues()[0];

    headers.forEach((header, index) => columnIndices[header] = index);

  

    const numRows = tab.getLastRow() - startRow;

    if (numRows > 0) {

      summaries = tab.getRange(startRow + 1, 1, numRows, tab.getLastColumn()).getValues();

    }

  

    summaries.forEach(summary => {

      allAnalyses.push({

        summary,

        columnIndices,

        sourceTab: tabName,

        subfolderName,

        priority: subfolderMap.get(subfolderName).priority

      });

    });

  }

  

  // Fetch Case Brief sections from the Case Brief tab (or mock it)

  const caseBriefSheet = spreadsheet.getSheetByName("Case Brief");

  let caseBriefSections = [];

  if (caseBriefSheet) {

    const data = caseBriefSheet.getDataRange().getValues();

    let currentSection = null;

    for (let i = 1; i < data.length; i++) { // Skip header

      if (data[i][0] && !data[i][1]) { // New section title

        if (currentSection) caseBriefSections.push(currentSection);

        currentSection = { title: data[i][0], content: "" };

      } else if (currentSection && data[i][1]) { // Content for current section

        currentSection.content += (currentSection.content ? "\n" : "") + data[i][1];

      }

    }

    if (currentSection) caseBriefSections.push(currentSection);

  } else {

    // Mock data from your last output if Case Brief tab isn’t populated

    caseBriefSections = [

      { title: "Case Summary", content: "This probate case centers around the Estate of Ben Butler (2023-PR03517-2), with Mary Hunt and her sisters as beneficiaries. The case has devolved into a contentious situation due to Mary's unilateral and procedurally flawed motion to remove the estate administrator. This action has fractured the attorney-client relationship with Herrin Law, PLLC, leading to their impending withdrawal as counsel. Outstanding legal fees further complicate the matter." },

      { title: "Facts/Evidence Supporting Us", content: "The signed engagement agreement [Doc 1] outlines the terms of representation, including the agreed-upon $2,000 retainer and hourly billing. Records indicate an initial payment of $2,000 in September 2024 and a subsequent payment of $1,000, leaving an outstanding balance of $1,000 as of March 8, 2025 [CRM Snapshot]. This outstanding balance, coupled with Mary's breach of the agreement by acting against counsel's advice, justifies the firm's decision to withdraw.\n*Doc 1: Signed - Engagement Agreement _ Mary Hunt _ Contested Probate _ Matter-10514.pdf*" },

      { title: "Challenges", content: "Mary Hunt's unilateral motion to remove the estate administrator, filed without consulting her attorney, has created procedural issues and jeopardized her standing in the case [CRM Snapshot]. The breakdown in communication and trust between Mary and her attorney further complicates the situation, making effective representation difficult. The outstanding balance owed to the firm adds another layer of complexity to the withdrawal process." },

      { title: "How We Win", content: "We must formally withdraw as counsel while minimizing further damage to Mary's case. This requires clear communication with Mary regarding the reasons for withdrawal, the outstanding balance, and the steps she needs to take to secure new representation. We must also communicate with opposing counsel to ensure a smooth transition and minimize any negative impact on the ongoing probate proceedings. Finally, we need to pursue collection of the outstanding $1,000." },

      { title: "Helpful Docs", content: "Signed - Engagement Agreement _ Mary Hunt _ Contested Probate _ Matter-10514.pdf (CRM Snapshot)\nThis agreement details the terms of representation between Herrin Law and Mary Hunt.\n\n11.30_FM Will (executed 01.16.2023).pdf (Partial Timeline)\nThis document outlines the distribution of Ben Butler's estate and names Mary Hunt and her sisters as beneficiaries.\n*Doc 1: Signed - Engagement Agreement _ Mary Hunt _ Contested Probate _ Matter-10514.pdf*\n*Doc 2: 2023.11.30_FM Will (executed 01.16.2023).pdf*" },

      { title: "Harmful Docs", content: "There are no documents within this batch directly harmful to Herrin Law's position regarding the withdrawal." }

    ];

  }

  

  // Call the updated function

  const prompt = generateAIResearchPrompt(caseBriefSections, snapshot, allAnalyses);

  

  // Log the result

  Logger.log("Generated AI Research Prompt:\n" + prompt);

}
```






# ######SheetFunctions.js Overview

## Purpose

This script manages Google Sheets operations for the AI LEGAL GOD system, handling spreadsheet creation, tab management, document organization, and data manipulation for legal case analysis.

## Core Functionality

### Spreadsheet Management

- **`setupLitigationAnalysisSheet(folderId, tabMapping)`**: Creates and configures the main analysis spreadsheet with tabs based on document categories
- **`setupTabsInSpreadsheet(spreadsheet, tabMapping)`**: Initializes tabs with appropriate headers and formatting
- **`getAnalysisSpreadsheet(folderId)`**: Retrieves or creates the analysis spreadsheet for a specific case folder

### Document Processing

- **`processDocumentForAnalysis(file, sheet, rowData, geminiPrompt)`**: Analyzes legal documents using AI and updates the spreadsheet with summaries
- **`extractDocumentMetadata(file, documentType)`**: Extracts key information like date, type, and parties from document filenames
- **`addDocumentToSheet(sheet, file, summary, documentType, docDate, parties)`**: Adds document entries to the appropriate tab with formatting

### Data Organization

- **`getStructuredTabMapping()`**: Returns the document categorization system with priorities and processing rules
- **`getTabForDocumentType(tabMapping, documentType)`**: Determines which tab a document belongs in based on its type
- **`createRowData(file, metadata)`**: Creates a structured data object for each document row

### Log Management

- **`setupLogSheet(spreadsheet)`**: Creates and configures a log sheet for tracking script execution
- **`logToSheet(spreadsheet, message, level)`**: Records detailed logs with timestamps and severity levels

### Utility Functions

- **`createAdminSheet(spreadsheet)`**: Creates an administration tab with system settings
- **`formatDateForSheet(date)`**: Standardizes date formatting for consistency
- **`getOrCreateFolder(parentFolderId, folderName)`**: Manages folder creation/retrieval in Google Drive
- **`checkIfFileAlreadyProcessed(sheet, fileId)`**: Prevents duplicate processing of documents

## Integration Points

- Interfaces with Google Drive for file access and management
- Uses Google Sheets for data organization and visualization
- Connects with the Gemini AI system via `processDocumentForAnalysis()`
- Works with CaseBriefFunctions.js to form the complete AI LEGAL GOD system

## Technical Details

- Implements batch processing to handle large document sets
- Contains comprehensive error handling with try-catch blocks
- Uses custom formatting for different document types
- Manages document metadata extraction with regular expressions
- Includes detailed logging system for troubleshooting

## Key Data Structures

- **Tab Mapping**: Defines document categorization rules and processing priorities
- **Row Data**: Standardized format for document information in spreadsheets
- **Document Metadata**: Extracted information about legal documents

## CODE
```
function processLitigationDocuments(folderId) {

  // Validate folderId

  if (typeof folderId !== 'string' || !folderId || folderId.trim() === "") {

    logError("Invalid folder ID: " + folderId);

    flushLogs(folderId);

    throw new Error("Folder ID cannot be empty or invalid in processLitigationDocuments");

  }

  

  const folder = DriveApp.getFolderById(folderId);

  const files = folder.getFilesByName("Litigation Document Analysis");

  let spreadsheet;

  

  // Create or open the Litigation Document Analysis spreadsheet

  if (files.hasNext()) {

    spreadsheet = SpreadsheetApp.open(files.next());

  } else {

    spreadsheet = SpreadsheetApp.create("Litigation Document Analysis");

    const spreadsheetFile = DriveApp.getFileById(spreadsheet.getId());

    Utilities.sleep(2000); // Add delay to ensure the spreadsheet is available

    folder.addFile(spreadsheetFile);

    DriveApp.getRootFolder().removeFile(spreadsheetFile);

  }

  

  // Create or update tabs for each subfolder

  const subfolders = folder.getFolders();

  const tabMapping = [];

  const existingTabs = spreadsheet.getSheets().map(sheet => sheet.getName().toLowerCase());

  

  // Define category mappings based on keywords

  const categoryMappings = [

    { keywords: ["pleadings"], category: "Pleadings" },

    { keywords: ["hearing preparation"], category: "Hearing Preparation" },

    { keywords: ["contract", "bills"], category: "Contracts & Bills" },

    { keywords: ["discovery"], category: "Discovery" },

    { keywords: ["client files"], category: "Client Files" },

    { keywords: ["correspondence"], category: "Correspondence" },

    { keywords: ["mediation"], category: "Mediation" },

    { keywords: ["attorney", "atty"], category: "Attorney & Paralegal Notes" },

    { keywords: ["legal fees", "expenses"], category: "Expenses" },

    { keywords: ["root"], category: "Root" }

  ];

  

  while (subfolders.hasNext()) {

    const subfolder = subfolders.next();

    const subfolderName = subfolder.getName();

    const tabName = subfolderName.replace(/[^a-zA-Z0-9]/g, '_');

  

    // Determine the tab type based on keywords

    let tabType = subfolderName; // Default to subfolder name

    const subfolderLower = subfolderName.toLowerCase();

    for (const mapping of categoryMappings) {

      if (mapping.keywords.some(keyword => subfolderLower.includes(keyword))) {

        tabType = mapping.category;

        break;

      }

    }

  

    if (!existingTabs.includes(tabName.toLowerCase())) {

      const sheet = spreadsheet.insertSheet(tabName);

      setupTab(sheet, tabType);

    } else {

      // Ensure existing tab has correct headers

      const sheet = spreadsheet.getSheetByName(tabName);

      setupTab(sheet, tabType);

    }

  

    tabMapping.push({ subfolder: subfolderName, tabType: tabType });

  }

  

  // Add a tab for root folder documents

  const rootTabName = "Root";

  if (!existingTabs.includes(rootTabName.toLowerCase())) {

    const rootSheet = spreadsheet.insertSheet(rootTabName);

    setupTab(rootSheet, "Root");

  } else {

    const rootSheet = spreadsheet.getSheetByName(rootTabName);

    setupTab(rootSheet, "Root");

  }

  tabMapping.push({ subfolder: "(Root)", tabType: "Root" });

  

  // Wait for sheets to be fully committed

  Utilities.sleep(2000);

  

  // Pre-populate the Attorneys tab if it exists

  const attorneysTab = findAttorneysTab(spreadsheet);

  if (attorneysTab) {

    prePopulateAttorneysTab(attorneysTab);

  }

  

  // Populate the Tab Mapping tab

  populateTabMappingTab(spreadsheet, tabMapping);

  

  // Remove the default Sheet1 if it exists and is empty

  const sheet1 = spreadsheet.getSheetByName("Sheet1");

  if (sheet1 && sheet1.getLastRow() === 0) {

    spreadsheet.deleteSheet(sheet1);

  }

  

  const createdTabs = spreadsheet.getSheets().map(sheet => sheet.getName()).join(", ");

  logInfo(`Spreadsheet created with tabs: ${createdTabs}`);

  flushLogs(folderId);

  return `Spreadsheet created with tabs: ${createdTabs}`;

}

  

// The rest of the functions (setupTab, findAttorneysTab, prePopulateAttorneysTab, populateTabMappingTab, appendSummariesToTab) remain unchanged

  

function setupTab(sheet, tabType) {

  // Clear existing content to ensure proper setup

  sheet.clear();

  

  // Define headers based on tab type

  let headers = [

    "Filename", "File Path", "Document Type", "Summary", "Key Information", "Priority",

    "Relevant Dates", "Key Parties", "Link", "Processed"

  ];

  

  switch (tabType.toLowerCase()) {

    case "pleadings":

      headers = headers.concat([

        "Legal References", "Filing Attorney", "Filing Party", "Legal Strategies",

        "Claims/Defenses", "Filing Date", "Case Number", "Court",

        "Helps Us", "Hurts Us", "Next Actions"

      ]);

      break;

    case "legal fees & expenses":

    case "expenses":

      headers = headers.concat([

        "Amount", "Billing Period", "Service Description", "Payment Status", "Financial Impact",

        "Helps Us", "Hurts Us", "Next Actions"

      ]);

      break;

    case "hearing preparation":

      headers = headers.concat([

        "Hearing Type", "Exhibits", "Court", "Case Number", "Preparation Notes",

        "Helps Us", "Hurts Us", "Next Actions"

      ]);

      break;

    case "discovery":

      headers = headers.concat([

        "Request/Response IDs", "Discovery Owner", "Discovery Type", "Production Details",

        "Bates/Exhibit Labels", "Helps Us", "Hurts Us", "Next Actions"

      ]);

      break;

    case "mediation":

      headers = headers.concat([

        "Mediation Type", "Mediation Role", "Settlement Impact", "Mediation Notes",

        "Helps Us", "Hurts Us", "Next Actions"

      ]);

      break;

    case "client files":

      headers = headers.concat([

        "Evidence Owner", "Evidence Type", "Supporting Claim", "Admissibility",

        "Client Context", "Bates/Exhibit Labels", "Helps Us", "Hurts Us", "Next Actions"

      ]);

      break;

    case "correspondence":

      headers = headers.concat([

        "Correspondence Type", "Recipient/Sender", "Purpose", "Helps Us", "Hurts Us", "Next Actions"

      ]);

      break;

    case "attorneys":

    case "attorney & paralegal notes":

    case "root":

    default:

      headers = headers.concat([

        "Helps Us", "Hurts Us", "Next Actions"

      ]);

      break;

  }

  

  // Set headers

  sheet.getRange(1, 1).setValue("[Document Summaries]");

  sheet.getRange(1, 1).setFontWeight("bold").setFontSize(12);

  sheet.getRange(2, 1, 1, headers.length).setValues([headers]);

  sheet.getRange(2, 1, 1, headers.length).setFontWeight("bold").setBackground("#4285F4").setFontColor("white");

  

  // Set column widths

  sheet.setColumnWidth(1, 200); // Filename

  sheet.setColumnWidth(2, 300); // File Path

  sheet.setColumnWidth(4, 400); // Summary

  sheet.setColumnWidth(5, 300); // Key Information

  for (let i = 3; i <= headers.length; i++) {

    if (i !== 1 && i !== 2 && i !== 4 && i !== 5) {

      sheet.setColumnWidth(i, 150);

    }

  }

  

  // Add borders and formatting

  const lastRow = sheet.getLastRow();

  const lastCol = sheet.getLastColumn();

  if (lastRow >= 2) {

    sheet.getRange(1, 1, lastRow, lastCol).setBorder(true, true, true, true, true, true);

  }

  

  // Add data validation for Priority column

  const priorityCol = headers.indexOf("Priority") + 1;

  if (priorityCol > 0) {

    const priorityRule = SpreadsheetApp.newDataValidation()

      .requireValueInList(['High', 'Medium', 'Low'], true)

      .build();

    sheet.getRange(3, priorityCol, sheet.getMaxRows() - 2, 1).setDataValidation(priorityRule);

  }

  

  // Enable text wrapping for the entire sheet

  sheet.getRange(1, 1, sheet.getMaxRows(), sheet.getMaxColumns()).setWrap(true);

  

  // Freeze the header row (row 2, since row 1 is "[Document Summaries]")

  sheet.setFrozenRows(2);

}

  

function findAttorneysTab(spreadsheet) {

  const sheets = spreadsheet.getSheets();

  for (let i = 0; i < sheets.length; i++) {

    const sheetName = sheets[i].getName();

    const lowercaseName = sheetName.toLowerCase();

    if (lowercaseName.includes("attorney") || lowercaseName.includes("atty")) {

      return sheets[i];

    }

  }

  return null;

}

  

function prePopulateAttorneysTab(attorneysTab) {

  // Clear existing content

  attorneysTab.clear();

  

  // Set up Case Insights section

  attorneysTab.getRange(1, 1).setValue("[Case Insights and Variables]");

  attorneysTab.getRange(1, 1).setFontWeight("bold").setFontSize(12);

  attorneysTab.getRange(2, 1, 1, 3).setValues([["Key", "Value", "Notes"]]);

  attorneysTab.getRange(2, 1, 1, 3).setFontWeight("bold").setBackground("#4285F4").setFontColor("white");

  

  const insightFields = [

    "Case Overview", "Strategy", "Claims", "Key Issues",

    "Summary Guidance", "Helps Us Guidance", "Hurts Us Guidance",

    "Case Action Plan Snapshot"

  ];

  

  for (let i = 0; i < insightFields.length; i++) {

    const row = 3 + i;

    attorneysTab.getRange(row, 1).setValue(insightFields[i]);

  }

  

  attorneysTab.getRange(1, 1, insightFields.length + 2, 3).setBorder(true, true, true, true, true, true);

  attorneysTab.setColumnWidth(1, 150);

  attorneysTab.setColumnWidth(2, 400);

  attorneysTab.setColumnWidth(3, 400);

  

  // Set up Document Summaries section

  const startRow = insightFields.length + 5;

  attorneysTab.getRange(startRow, 1).setValue("[Document Summaries]");

  attorneysTab.getRange(startRow, 1).setFontWeight("bold").setFontSize(12);

  

  const summaryHeaders = [

    "Filename", "File Path", "Document Type", "Summary", "Key Information", "Priority",

    "Relevant Dates", "Key Parties", "Link", "Processed",

    "Helps Us", "Hurts Us", "Next Actions"

  ];

  attorneysTab.getRange(startRow + 1, 1, 1, summaryHeaders.length).setValues([summaryHeaders]);

  attorneysTab.getRange(startRow + 1, 1, 1, summaryHeaders.length)

    .setFontWeight("bold").setBackground("#4285F4").setFontColor("white");

  

  attorneysTab.getRange(startRow, 1, 2, summaryHeaders.length).setBorder(true, true, true, true, true, true);

  attorneysTab.setColumnWidth(1, 200);

  attorneysTab.setColumnWidth(2, 300);

  attorneysTab.setColumnWidth(4, 400);

  attorneysTab.setColumnWidth(5, 300);

  for (let i = 3; i <= summaryHeaders.length; i++) {

    if (i !== 1 && i !== 2 && i !== 4 && i !== 5) {

      attorneysTab.setColumnWidth(i, 150);

    }

  }

  

  // Enable text wrapping for the entire sheet

  attorneysTab.getRange(1, 1, attorneysTab.getMaxRows(), attorneysTab.getMaxColumns()).setWrap(true);

  

  // Freeze the header rows: row 2 for Case Insights, and startRow + 1 for Document Summaries

  attorneysTab.setFrozenRows(startRow + 1);

}

  

function populateTabMappingTab(spreadsheet, tabMapping) {

  let tabMappingSheet = spreadsheet.getSheetByName("Tab Mapping");

  if (!tabMappingSheet) {

    tabMappingSheet = spreadsheet.insertSheet("Tab Mapping");

  } else {

    tabMappingSheet.clear();

  }

  

  const headers = ["Subfolder Name", "Suggested Tab Type", "Mapped Tab Type", "Priority", "Process Documents", "Notes"];

  tabMappingSheet.getRange(1, 1, 1, headers.length).setValues([headers]);

  tabMappingSheet.getRange(1, 1, 1, headers.length).setFontWeight("bold").setBackground("#4285F4").setFontColor("white");

  

  // Define priority mapping based on tab type categories

  const priorityMapping = {

    "Pleadings": "High",

    "Hearing Preparation": "High",

    "Contracts & Bills": "Medium",

    "Discovery": "Medium",

    "Client Files": "High",

    "Correspondence": "Low",

    "Mediation": "Low",

    "Attorney & Paralegal Notes": "High",

    "Expenses": "Medium",

    "Root": "Low"

  };

  

  const data = tabMapping.map(mapping => {

    const suggestedTabType = mapping.tabType;

    const priority = priorityMapping[suggestedTabType] || "Medium";

    return [

      mapping.subfolder,

      suggestedTabType,

      suggestedTabType,

      priority,

      "Yes",

      ""

    ];

  });

  

  if (data.length > 0) {

    tabMappingSheet.getRange(2, 1, data.length, headers.length).setValues(data);

  }

  

  // Add data validation for Priority and Process Documents columns

  const priorityCol = headers.indexOf("Priority") + 1;

  const processDocsCol = headers.indexOf("Process Documents") + 1;

  

  if (priorityCol > 0) {

    const priorityRule = SpreadsheetApp.newDataValidation()

      .requireValueInList(['High', 'Medium', 'Low'], true)

      .build();

    tabMappingSheet.getRange(2, priorityCol, tabMappingSheet.getMaxRows() - 1, 1).setDataValidation(priorityRule);

  }

  

  if (processDocsCol > 0) {

    const processDocsRule = SpreadsheetApp.newDataValidation()

      .requireValueInList(['Yes', 'No'], true)

      .build();

    tabMappingSheet.getRange(2, processDocsCol, tabMappingSheet.getMaxRows() - 1, 1).setDataValidation(processDocsRule);

  }

  

  tabMappingSheet.getRange(1, 1, tabMappingSheet.getLastRow(), headers.length).setBorder(true, true, true, true, true, true);

  tabMappingSheet.setColumnWidth(1, 200);

  tabMappingSheet.setColumnWidth(2, 150);

  tabMappingSheet.setColumnWidth(3, 150);

  tabMappingSheet.setColumnWidth(4, 100);

  tabMappingSheet.setColumnWidth(5, 150);

  tabMappingSheet.setColumnWidth(6, 300);

  

  // Enable text wrapping for the entire sheet

  tabMappingSheet.getRange(1, 1, tabMappingSheet.getMaxRows(), tabMappingSheet.getMaxColumns()).setWrap(true);

  

  // Freeze the header row (row 1)

  tabMappingSheet.setFrozenRows(1);

}

  

function appendSummariesToTab(sheet, summaries) {

  // Determine the header row based on the presence of "[Document Summaries]"

  let headerRow = 1;

  let startRow = 1;

  const maxRowsToCheck = Math.min(sheet.getLastRow(), 20);

  for (let row = 1; row <= maxRowsToCheck; row++) {

    const firstCell = sheet.getRange(row, 1).getValue();

    if (firstCell === "[Document Summaries]") {

      headerRow = row + 1;

      startRow = row + 1;

      break;

    }

  }

  

  // Read the headers from the correct row

  const headers = sheet.getRange(headerRow, 1, 1, sheet.getLastColumn()).getValues()[0];

  

  // Define a mapping of headers to camelCase field keys

  const headerToFieldKeyMap = {

    "Filename": "fileName",

    "File Path": "filePath",

    "Document Type": "documentType",

    "Summary": "summary",

    "Key Information": "keyInformation",

    "Priority": "priority",

    "Relevant Dates": "relevantDates",

    "Key Parties": "keyParties",

    "Link": "link",

    "Processed": "processed",

    "Legal References": "legalReferences",

    "Filing Attorney": "filingAttorney",

    "Filing Party": "filingParty",

    "Legal Strategies": "legalStrategies",

    "Claims/Defenses": "claimsDefenses",

    "Filing Date": "filingDate",

    "Case Number": "caseNumber",

    "Court": "court",

    "Amount": "amount",

    "Billing Period": "billingPeriod",

    "Service Description": "serviceDescription",

    "Payment Status": "paymentStatus",

    "Financial Impact": "financialImpact",

    "Hearing Type": "hearingType",

    "Exhibits": "exhibits",

    "Preparation Notes": "preparationNotes",

    "Request/Response IDs": "requestResponseIDs",

    "Discovery Owner": "discoveryOwner",

    "Discovery Type": "discoveryType",

    "Production Details": "productionDetails",

    "Bates/Exhibit Labels": "batesExhibitLabels",

    "Mediation Type": "mediationType",

    "Mediation Role": "mediationRole",

    "Settlement Impact": "settlementImpact",

    "Mediation Notes": "mediationNotes",

    "Evidence Owner": "evidenceOwner",

    "Evidence Type": "evidenceType",

    "Supporting Claim": "supportingClaim",

    "Admissibility": "admissibility",

    "Client Context": "clientContext",

    "Correspondence Type": "correspondenceType",

    "Recipient/Sender": "recipientSender",

    "Purpose": "purpose",

    "Helps Us": "helpsUs",

    "Hurts Us": "hurtsUs",

    "Next Actions": "nextActions"

  };

  

  // Validate headers

  const expectedHeaders = [

    "Filename", "File Path", "Document Type", "Summary", "Key Information", "Priority",

    "Relevant Dates", "Key Parties", "Link", "Processed"

  ];

  const missingHeaders = expectedHeaders.filter(header => !headers.includes(header));

  if (missingHeaders.length > 0) {

    logWarn(`Missing expected headers in tab ${sheet.getName()}: ${missingHeaders.join(", ")}`);

    flushLogs(currentFolderId);

  }

  

  // Calculate the next row to append data

  const existingData = sheet.getRange(startRow + 1, 1, Math.max(1, sheet.getLastRow() - startRow), sheet.getLastColumn()).getValues();

  let nextRow = startRow + 1 + existingData.length;

  

  // Construct the rows to append

  const rows = summaries.map((summary, summaryIndex) => {

    try {

      if (!summary || typeof summary !== 'object') {

        logWarn(`Summary ${summaryIndex + 1} is invalid: ${JSON.stringify(summary)}`);

        flushLogs(currentFolderId);

        return null;

      }

      const row = new Array(headers.length).fill("");

      headers.forEach((header, colIndex) => {

        const fieldKey = headerToFieldKeyMap[header] || header.toLowerCase().replace(/[^a-z0-9]/g, '');

        if (header === "Link" && summary.link) {

          row[colIndex] = `=HYPERLINK("${summary.link}", "Open")`;

        } else if (fieldKey in summary) {

          row[colIndex] = summary[fieldKey] || "";

        }

      });

      return row;

    } catch (e) {

      logError(`Error constructing row for summary ${summaryIndex + 1}: ${e.message}`);

      flushLogs(currentFolderId);

      return null;

    }

  }).filter(row => row !== null);

  

  // Write the rows to the spreadsheet and add borders

  if (rows.length > 0) {

    sheet.getRange(nextRow, 1, rows.length, headers.length).setValues(rows);

    sheet.getRange(nextRow, 1, rows.length, headers.length).setBorder(true, true, true, true, true, true);

  

    // Reapply text wrapping to the entire sheet to ensure new cells are wrapped

    sheet.getRange(1, 1, sheet.getMaxRows(), sheet.getMaxColumns()).setWrap(true);

  }

}
```


# #####Index.html Overview

## Purpose

This is a simple HTML interface for the AI LEGAL GOD system that provides user interaction capabilities to trigger key functions of the document processing system.

## Core Components

### UI Elements

- **Title**: Displays "AI LEGAL GOD" as the system name
- **Input Field**: Accepts a Google Drive Folder ID as the target for processing
- **Action Buttons**: Three primary function buttons:
    - **Create Sheet**: Presumably triggers the spreadsheet creation process
    - **Process Docs**: Initiates the document processing workflow
    - **Generate Case Brief**: Triggers the case brief generation functionality
- **Status Message**: Shows "Processing, please wait..." when operations are running

## Structure

- Minimalist design with basic form elements
- No complex styling or advanced UI components
- Simple input-and-action interface pattern
- Status indicator for user feedback

## Integration Points

- Serves as the frontend interface for the AI LEGAL GOD system
- Connects to the Google Apps Script backend (CaseBriefFunctions.js and SheetFunctions.js)
- Provides a way to specify which Google Drive folder to process
- Offers direct access to the three main system functions

## Technical Details

- Basic HTML structure without advanced frameworks
- Likely intended to be served as a web app from Google Apps Script
- Appears to be optimized for simplicity rather than extensive features
- Focuses on functional access rather than complex UI interactions

## CODE
```
<!DOCTYPE html>

<html>

<head>

  <base target="_top">

  <link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;700;900&display=swap" rel="stylesheet">

  <style>

    body {

      font-family: 'Montserrat', sans-serif;

      margin: 0;

      padding: 0;

      min-height: 100vh;

      display: flex;

      justify-content: center;

      align-items: center;

      position: relative;

      overflow: auto;

      background: url('https://drive.google.com/thumbnail?id=1-0FRCyObPINjZlJzypjN_pZ9xo-HJALO&sz=w1000') no-repeat center center fixed;

      background-size: cover;

      color: #1B263B;

    }

    body::before {

      content: '';

      position: absolute;

      top: 0;

      left: 0;

      width: 100%;

      height: 100%;

      background: rgba(0, 0, 0, 0.5); /* Dark overlay for readability */

      backdrop-filter: blur(3px); /* Slight blur for background */

      z-index: 0;

    }

    .container {

      max-width: 800px;

      background: rgba(255, 255, 255, 0.9);

      padding: 40px;

      border-radius: 15px;

      box-shadow: 0 0 30px rgba(0, 0, 0, 0.3);

      border: 2px solid #D4AF37;

      position: relative;

      z-index: 1;

    }

    h1 {

      color: #1B263B;

      text-align: center;

      margin-bottom: 30px;

      font-size: 2.5em;

      font-weight: 900;

      text-transform: uppercase;

      letter-spacing: 3px;

    }

    label {

      display: block;

      margin: 15px 0 5px;

      font-weight: 700;

      color: #1B263B;

      font-size: 1.2em;

    }

    input[type="text"] {

      width: 100%;

      padding: 15px;

      margin-bottom: 20px;

      border: 2px solid #1B263B;

      border-radius: 8px;

      background: #FFFFFF;

      color: #1B263B;

      font-family: 'Montserrat', sans-serif;

      font-size: 1.2em;

      box-shadow: inset 0 2px 5px rgba(0, 0, 0, 0.1);

      box-sizing: border-box;

      transition: all 0.3s ease;

    }

    input[type="text"]:focus {

      outline: none;

      border-color: #D4AF37;

      box-shadow: 0 0 10px rgba(212, 175, 55, 0.5);

    }

    .button-container {

      display: flex;

      justify-content: center;

      gap: 20px;

      margin: 30px 0;

    }

    button {

      background: #1B263B;

      color: #FFFFFF;

      padding: 15px 40px;

      border: none;

      border-radius: 8px;

      cursor: pointer;

      font-family: 'Montserrat', sans-serif;

      font-size: 1.2em;

      font-weight: 700;

      text-transform: uppercase;

      box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);

      transition: all 0.3s ease;

    }

    button:hover {

      background: #D4AF37;

      color: #1B263B;

      box-shadow: 0 0 15px rgba(212, 175, 55, 0.7);

      transform: scale(1.05);

    }

    #progress {

      margin-top: 20px;

      text-align: center;

      display: none;

    }

    .spinner {

      border: 6px solid #FFFFFF;

      border-top: 6px solid #D4AF37;

      border-radius: 50%;

      width: 40px;

      height: 40px;

      animation: spin 1s linear infinite;

      margin: 0 auto;

      box-shadow: 0 0 10px rgba(212, 175, 55, 0.5);

    }

    @keyframes spin {

      0% { transform: rotate(0deg); }

      100% { transform: rotate(360deg); }

    }

    #progress p {

      color: #1B263B;

      margin-top: 10px;

      font-weight: 700;

    }

    #result {

      margin-top: 20px;

      padding: 15px;

      border-radius: 8px;

      display: none;

      font-size: 1em;

      font-weight: 700;

    }

    .success {

      background: rgba(212, 175, 55, 0.2);

      color: #1B263B;

      border: 2px solid #D4AF37;

      box-shadow: 0 0 10px rgba(212, 175, 55, 0.5);

    }

    .error {

      background: rgba(255, 0, 0, 0.2);

      color: #FF0000;

      border: 2px solid #FF0000;

      box-shadow: 0 0 10px rgba(255, 0, 0, 0.5);

    }

    button:nth-child(3) {

  background: #28a745; /* Green color */

}

  

button:nth-child(3):hover {

  background: #218838; /* Darker green on hover */

  box-shadow: 0 0 15px rgba(40, 167, 69, 0.7);

}

  

  </style>

</head>

<body>

  <div class="container">

    <h1>AI LEGAL GOD</h1>

    <label for="folderId">Google Drive Folder ID:</label>

    <input type="text" id="folderId" placeholder="Enter Folder ID">

    <div class="button-container">

      <button onclick="createSheet()">Create Sheet</button>

      <button onclick="processDocs()">Process Docs</button>

      <button onclick="generateCaseBrief()">Generate Case Brief</button>

    </div>

    <div id="progress">

      <div class="spinner"></div>

      <p>Processing, please wait...</p>

    </div>

    <div id="result"></div>

  </div>

  

  <script>

    function createSheet() {

      const folderId = document.getElementById('folderId').value.trim();

      if (!folderId) {

        alert('Please enter a valid Folder ID.');

        return;

      }

  

      // Show progress indicator

      document.getElementById('progress').style.display = 'block';

      document.getElementById('result').style.display = 'none';

  

      google.script.run

        .withSuccessHandler(function(result) {

          document.getElementById('progress').style.display = 'none';

          const resultDiv = document.getElementById('result');

          resultDiv.style.display = 'block';

          resultDiv.className = 'success';

          resultDiv.innerHTML = '<strong>Success:</strong> ' + result;

        })

        .withFailureHandler(function(error) {

          document.getElementById('progress').style.display = 'none';

          const resultDiv = document.getElementById('result');

          resultDiv.style.display = 'block';

          resultDiv.className = 'error';

          resultDiv.innerHTML = '<strong>Error:</strong> ' + error.message;

        })

        .processLitigationDocuments(folderId);

    }

  

    function processDocs() {

      const folderId = document.getElementById('folderId').value.trim();

      if (!folderId) {

        alert('Please enter a valid Folder ID.');

        return;

      }

  

      // Show progress indicator

      document.getElementById('progress').style.display = 'block';

      document.getElementById('result').style.display = 'none';

  

      google.script.run

        .withSuccessHandler(function(result) {

          document.getElementById('progress').style.display = 'none';

          const resultDiv = document.getElementById('result');

          resultDiv.style.display = 'block';

          resultDiv.className = 'success';

          resultDiv.innerHTML = '<strong>Success:</strong> ' + result;

        })

        .withFailureHandler(function(error) {

          document.getElementById('progress').style.display = 'none';

          const resultDiv = document.getElementById('result');

          resultDiv.style.display = 'block';

          resultDiv.className = 'error';

          resultDiv.innerHTML = '<strong>Error:</strong> ' + error.message;

        })

        .processWithFolderId(folderId);

    }

  

    function generateCaseBrief() {

      const folderId = document.getElementById('folderId').value.trim();

      if (!folderId) {

        alert('Please enter a valid Folder ID.');

        return;

      }

  

      // Show progress indicator

      document.getElementById('progress').style.display = 'block';

      document.getElementById('result').style.display = 'none';

  

      google.script.run

        .withSuccessHandler(function(result) {

          document.getElementById('progress').style.display = 'none';

          const resultDiv = document.getElementById('result');

          resultDiv.style.display = 'block';

          resultDiv.className = 'success';

          resultDiv.innerHTML = '<strong>Success:</strong> ' + result;

        })

        .withFailureHandler(function(error) {

          document.getElementById('progress').style.display = 'none';

          const resultDiv = document.getElementById('result');

          resultDiv.style.display = 'block';

          resultDiv.className = 'error';

          resultDiv.innerHTML = '<strong>Error:</strong> ' + error.message;

        })

        .generateCaseBrief(folderId);

    }

  </script>

</body>

</html>
```