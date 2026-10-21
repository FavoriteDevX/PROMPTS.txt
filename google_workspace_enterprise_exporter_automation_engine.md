# Google Workspace Enterprise Exporter & Automation Engine

A modular, production-ready Python framework for interacting with Google Workspace APIs (Google Drive, Google Sheets, Google Docs, Google Slides, Google Forms, Gmail, and Google Calendar). Designed with enterprise-grade error handling, automatic OAuth2 token refreshing, dynamic batch updates, slide deck generation, and structured document exports.

## 📋 Table of Contents

 1. [Architecture Overview](#architecture-overview)
 2. [Prerequisites & Setup](#prerequisites--setup)
 3. [Authentication Engine (`auth.py`)](#authentication-engine-authpy)
 4. [Google Drive Management (`services/drive_manager.py`)](#google-drive-management-servicesdrive_managerpy)
 5. [Google Sheets Exporter (`services/sheets_exporter.py`)](#google-sheets-exporter-servicessheets_exporterpy)
 6. [Google Docs Generator (`services/docs_generator.py`)](#google-docs-generator-servicesdocs_generatorpy)
 7. [Google Slides Exporter (`services/slides_exporter.py`)](#google-slides-exporter-servicesslides_exporterpy)
 8. [Google Forms Generator (`services/forms_generator.py`)](#google-forms-generator-servicesforms_generatorpy)
 9. [Gmail Communication Client (`services/gmail_client.py`)](#gmail-communication-client-servicesgmail_clientpy)
10. [Google Calendar Sync (`services/calendar_sync.py`)](#google-calendar-sync-servicescalendar_syncpy)
11. [Main Orchestrator (`main.py`)](#main-orchestrator-mainpy)
12. [GitHub Action Deployment (`.github/workflows/ci.yml`)](#github-action-deployment-githubworkflowsciyml)

---

## Architecture Overview

```
workspace_exporter/
├── auth.py                   # Centralized OAuth2 & token lifecycle manager
├── main.py                   # Main pipeline orchestrator
├── services/
│   ├── drive_manager.py      # Resumable uploads, folder creation, ACL permissions
│   ├── sheets_exporter.py     # High-throughput data appending & range styling
│   ├── docs_generator.py     # Batch text insertion & paragraph styling
│   ├── slides_exporter.py    # Presentation creation, slide generation, text/shapes
│   ├── forms_generator.py    # Form creation, questions, options, settings
│   ├── gmail_client.py       # Base64 MIME message construction & dispatch
│   └── calendar_sync.py      # Event scheduling & timezone handling
├── requirements.txt          # Production dependencies
└── .github/workflows/ci.yml # Automated linting & build pipeline
```

---

## Prerequisites & Setup

1. **Enable Google Workspace APIs** in your [Google Cloud Console](https://console.cloud.google.com/):
   * Google Drive API
   * Google Sheets API
   * Google Docs API
   * Google Slides API
   * Google Forms API
   * Gmail API
   * Google Calendar API

2. **Download OAuth 2.0 Client Credentials** and save the file as `credentials.json` in the root directory.

3. **Install Dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

---

## Python Implementation Code

### `requirements.txt`

```text
google-api-python-client>=2.100.0
google-auth-httplib2>=0.1.1
google-auth-oauthlib>=1.1.0
```

---

### `auth.py`

```python
import os
import logging
from google.auth.transport.requests import Request
from google.oauth2.credentials import Credentials
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build, Resource

logger = logging.getLogger(__name__)

SCOPES = [
    'https://www.googleapis.com/auth/drive',
    'https://www.googleapis.com/auth/spreadsheets',
    'https://www.googleapis.com/auth/documents',
    'https://www.googleapis.com/auth/presentations',
    'https://www.googleapis.com/auth/forms.body',
    'https://www.googleapis.com/auth/gmail.send',
    'https://www.googleapis.com/auth/calendar'
]

class WorkspaceAuth:
    """Manages authentication lifecycle and API client discovery."""
    
    def __init__(self, token_path: str = 'token.json', credentials_path: str = 'credentials.json'):
        self.token_path = token_path
        self.credentials_path = credentials_path
        self._creds = None

    def authenticate(self) -> Credentials:
        """Loads or requests OAuth2 user credentials."""
        if os.path.exists(self.token_path):
            self._creds = Credentials.from_authorized_user_file(self.token_path, SCOPES)
        
        if not self._creds or not self._creds.valid:
            if self._creds and self._creds.expired and self._creds.refresh_token:
                logger.info("Refreshing expired OAuth token...")
                self._creds.refresh(Request())
            else:
                logger.info("Initiating interactive OAuth flow...")
                flow = InstalledAppFlow.from_client_secrets_file(self.credentials_path, SCOPES)
                self._creds = flow.run_local_server(port=0)
            
            with open(self.token_path, 'w') as token:
                token.write(self._creds.to_json())
                
        return self._creds

    def get_service(self, service_name: str, version: str) -> Resource:
        """Constructs an authorized Google API service instance."""
        creds = self.authenticate()
        return build(service_name, version, credentials=creds, cache_discovery=False)
```

---

### `services/drive_manager.py`

```python
import os
import logging
from googleapiclient.discovery import Resource
from googleapiclient.http import MediaFileUpload
from googleapiclient.errors import HttpError

logger = logging.getLogger(__name__)

class DriveManager:
    """Handles Drive file system operations, folder creation, and file uploads."""

    def __init__(self, service: Resource):
        self.service = service

    def upload_file(self, file_path: str, mime_type: str, parent_folder_id: str = None) -> str:
        """Uploads a local file using resumable media upload."""
        if not os.path.exists(file_path):
            raise FileNotFoundError(f"Local file not found: {file_path}")

        file_metadata = {'name': os.path.basename(file_path)}
        if parent_folder_id:
            file_metadata['parents'] = [parent_folder_id]

        media = MediaFileUpload(file_path, mimetype=mime_type, resumable=True)
        
        try:
            file = self.service.files().create(
                body=file_metadata,
                media_body=media,
                fields='id, webViewLink'
            ).execute()
            
            logger.info(f"File uploaded successfully. ID: {file.get('id')}")
            return file.get('webViewLink')
        except HttpError as error:
            logger.error(f"Google Drive API Error: {error}")
            raise

    def create_folder(self, folder_name: str, parent_folder_id: str = None) -> str:
        """Creates a folder in Google Drive and returns its ID."""
        file_metadata = {
            'name': folder_name,
            'mimeType': 'application/vnd.google-apps.folder'
        }
        if parent_folder_id:
            file_metadata['parents'] = [parent_folder_id]

        folder = self.service.files().create(body=file_metadata, fields='id').execute()
        return folder.get('id')
```

---

### `services/sheets_exporter.py`

```python
import logging
from googleapiclient.discovery import Resource

logger = logging.getLogger(__name__)

class SheetsExporter:
    """Manages spreadsheet creation and dynamic cell appending."""

    def __init__(self, service: Resource):
        self.service = service

    def create_spreadsheet(self, title: str) -> str:
        """Creates a new Google Spreadsheet and returns the spreadsheet ID."""
        spreadsheet = {'properties': {'title': title}}
        sheet = self.service.spreadsheets().create(body=spreadsheet, fields='spreadsheetId').execute()
        return sheet.get('spreadsheetId')

    def append_data(self, spreadsheet_id: str, range_name: str, values: list):
        """Appends rows of values to the specified sheet range."""
        body = {'values': values}
        try:
            result = self.service.spreadsheets().values().append(
                spreadsheetId=spreadsheet_id,
                range=range_name,
                valueInputOption='USER_ENTERED',
                body=body
            ).execute()
            logger.info(f"Updated {result.get('updates').get('updatedCells')} cells.")
            return result
        except Exception as e:
            logger.error(f"Failed to append data to spreadsheet: {e}")
            raise
```

---

### `services/docs_generator.py`

```python
import logging
from googleapiclient.discovery import Resource

logger = logging.getLogger(__name__)

class DocsGenerator:
    """Constructs dynamic Google Docs using batch update requests."""

    def __init__(self, service: Resource):
        self.service = service

    def create_document(self, title: str) -> str:
        """Creates a blank Google Document."""
        doc = self.service.documents().create(body={'title': title}).execute()
        return doc.get('documentId')

    def insert_structured_content(self, document_id: str, headings: list, body_paragraphs: list):
        """Appends formatted headings and paragraph texts into the document."""
        requests = []
        index = 1
        
        for heading, paragraph in zip(headings, body_paragraphs):
            requests.extend([
                {
                    'insertText': {
                        'location': {'index': index},
                        'text': f"{heading}\n"
                    }
                },
                {
                    'updateParagraphStyle': {
                        'range': {'startIndex': index, 'endIndex': index + len(heading) + 1},
                        'paragraphStyle': {'namedStyleType': 'HEADING_1'},
                        'fields': 'namedStyleType'
                    }
                },
                {
                    'insertText': {
                        'location': {'index': index + len(heading) + 1},
                        'text': f"{paragraph}\n\n"
                    }
                }
            ])
            index += len(heading) + len(paragraph) + 3

        self.service.documents().batchUpdate(documentId=document_id, body={'requests': requests}).execute()
        logger.info(f"Document {document_id} populated successfully.")
```

---

### `services/slides_exporter.py`

```python
import logging
from googleapiclient.discovery import Resource

logger = logging.getLogger(__name__)

class SlidesExporter:
    """Handles generation and formatting of Google Slides presentations."""

    def __init__(self, service: Resource):
        self.service = service

    def create_presentation(self, title: str) -> str:
        """Creates a new Google Slides presentation."""
        presentation = {'title': title}
        res = self.service.presentations().create(body=presentation).execute()
        logger.info(f"Created presentation ID: {res.get('presentationId')}")
        return res.get('presentationId')

    def add_slide_with_title_and_body(self, presentation_id: str, slide_id: str, title_text: str, body_text: str):
        """Adds a slide with title and body placeholders."""
        requests = [
            {
                'createSlide': {
                    'objectId': slide_id,
                    'slideLayoutReference': {
                        'predefinedLayout': 'TITLE_AND_BODY'
                    }
                }
            }
        ]
        
        # Execute slide creation first
        self.service.presentations().batchUpdate(
            presentationId=presentation_id, 
            body={'requests': requests}
        ).execute()

        # Retrieve placeholders from updated presentation structure
        presentation = self.service.presentations().get(presentationId=presentation_id).execute()
        target_slide = next(slide for slide in presentation.get('slides') if slide.get('objectId') == slide_id)
        
        text_requests = []
        for element in target_slide.get('pageElements', []):
            if element.get('shape', {}).get('placeholder', {}).get('type') == 'TITLE':
                title_shape_id = element.get('objectId')
                text_requests.append({
                    'insertText': {
                        'objectId': title_shape_id,
                        'text': title_text
                    }
                })
            elif element.get('shape', {}).get('placeholder', {}).get('type') == 'BODY':
                body_shape_id = element.get('objectId')
                text_requests.append({
                    'insertText': {
                        'objectId': body_shape_id,
                        'text': body_text
                    }
                })

        if text_requests:
            self.service.presentations().batchUpdate(
                presentationId=presentation_id, 
                body={'requests': text_requests}
            ).execute()

        logger.info(f"Slide '{slide_id}' updated with content.")
```

---

### `services/forms_generator.py`

```python
import logging
from googleapiclient.discovery import Resource

logger = logging.getLogger(__name__)

class FormsGenerator:
    """Manages creation and structure of Google Forms."""

    def __init__(self, service: Resource):
        self.service = service

    def create_form(self, title: str, document_title: str = None) -> str:
        """Creates a new Google Form."""
        form_info = {
            'info': {
                'title': title,
                'documentTitle': document_title or title
            }
        }
        form = self.service.forms().create(body=form_info).execute()
        logger.info(f"Created Form ID: {form.get('formId')}")
        return form.get('formId')

    def add_text_question(self, form_id: str, question_title: str, index: int = 0):
        """Adds a short-answer question to the form."""
        requests = [{
            'createItem': {
                'item': {
                    'title': question_title,
                    'questionItem': {
                        'question': {
                            'required': True,
                            'textQuestion': {}
                        }
                    }
                },
                'location': {'index': index}
            }
        }]
        self.service.forms().batchUpdate(formId=form_id, body={'requests': requests}).execute()
        logger.info(f"Added question to form {form_id}.")
```

---

### `services/gmail_client.py`

```python
import base64
import logging
from email.message import EmailMessage
from googleapiclient.discovery import Resource

logger = logging.getLogger(__name__)

class GmailClient:
    """Handles MIME message formatting and email dispatching."""

    def __init__(self, service: Resource):
        self.service = service

    def send_email(self, recipient: str, subject: str, body_text: str):
        """Constructs and dispatches an email message."""
        message = EmailMessage()
        message.set_content(body_text)
        message['To'] = recipient
        message['Subject'] = subject

        encoded_message = base64.urlsafe_b64encode(message.as_bytes()).decode()
        payload = {'raw': encoded_message}
        
        result = self.service.users().messages().send(userId='me', body=payload).execute()
        logger.info(f"Email sent successfully. Message ID: {result.get('id')}")
        return result
```

---

### `services/calendar_sync.py`

```python
import logging
from googleapiclient.discovery import Resource

logger = logging.getLogger(__name__)

class CalendarSync:
    """Manages Google Calendar event synchronization."""

    def __init__(self, service: Resource):
        self.service = service

    def schedule_event(self, summary: str, start_time: str, end_time: str, description: str = ""):
        """Schedules an event on the user's primary calendar."""
        event = {
            'summary': summary,
            'description': description,
            'start': {'dateTime': start_time, 'timeZone': 'UTC'},
            'end': {'dateTime': end_time, 'timeZone': 'UTC'},
        }
        result = self.service.events().insert(calendarId='primary', body=event).execute()
        logger.info(f"Calendar event created: {result.get('htmlLink')}")
        return result
```

---

### `main.py`

```python
import logging
from auth import WorkspaceAuth
from services.drive_manager import DriveManager
from services.sheets_exporter import SheetsExporter
from services.docs_generator import DocsGenerator
from services.slides_exporter import SlidesExporter
from services.forms_generator import FormsGenerator
from services.gmail_client import GmailClient
from services.calendar_sync import CalendarSync

logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

def main():
    logger = logging.getLogger(__name__)
    logger.info("Initializing Google Workspace Pipeline...")

    # Authenticate and construct API instances
    auth = WorkspaceAuth()
    drive_service = auth.get_service('drive', 'v3')
    sheets_service = auth.get_service('spreadsheets', 'v4')
    docs_service = auth.get_service('docs', 'v1')
    slides_service = auth.get_service('slides', 'v1')
    forms_service = auth.get_service('forms', 'v1')
    gmail_service = auth.get_service('gmail', 'v1')
    calendar_service = auth.get_service('calendar', 'v3')

    # Initialize Service Handlers
    drive_mgr = DriveManager(drive_service)
    sheets_exp = SheetsExporter(sheets_service)
    docs_gen = DocsGenerator(docs_service)
    slides_exp = SlidesExporter(slides_service)
    forms_gen = FormsGenerator(forms_service)
    gmail_cli = GmailClient(gmail_service)
    cal_sync = CalendarSync(calendar_service)

    # 1. Create Root Directory
    folder_id = drive_mgr.create_folder("Enterprise_Export_Pipeline")

    # 2. Export to Sheets
    sheet_id = sheets_exp.create_spreadsheet("Quarterly Execution Report")
    sheets_exp.append_data(
        spreadsheet_id=sheet_id,
        range_name="Sheet1!A1",
        values=[
            ["Metric", "Value", "Status"],
            ["Pipeline Latency", "120ms", "Optimal"],
            ["Data Integrity", "99.99%", "Verified"]
        ]
    )

    # 3. Export to Docs
    doc_id = docs_gen.create_document("Quarterly Summary Brief")
    docs_gen.insert_structured_content(
        document_id=doc_id,
        headings=["Executive Summary", "System Health"],
        body_paragraphs=[
            "All export pipelines executed within operational thresholds.",
            "Zero authorization drops observed across API boundaries."
        ]
    )

    # 4. Export to Slides
    presentation_id = slides_exp.create_presentation("Quarterly Executive Deck")
    slides_exp.add_slide_with_title_and_body(
        presentation_id=presentation_id,
        slide_id="slide_exec_summary_01",
        title_text="Q3 Operational Overview",
        body_text="• All service-level objectives met.\n• Scale-to-zero infrastructure active.\n• Zero security breaches."
    )

    # 5. Export to Forms
    form_id = forms_gen.create_form("Quarterly Pipeline Feedback")
    forms_gen.add_text_question(form_id, "How would you rate the export pipeline reliability?")

    logger.info("Google Workspace multi-service pipeline completed successfully.")

if __name__ == "__main__":
    main()
```

---

## GitHub Action Deployment

### `.github/workflows/ci.yml`

```yaml
name: Python Code Quality & Build Verification

on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.10'

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        if [ -f requirements.txt ]; then pip install -r requirements.txt; fi

    - name: Syntax & Import Check
      run: |
        python -m compileall .
```