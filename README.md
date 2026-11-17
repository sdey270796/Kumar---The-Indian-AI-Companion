# Kumar --- The Indian AI Companion

Kumar is a multimodal AI assistant built in Python with **Gradio** and
the **DeepSeek API**. The project is implemented as a Google Colab
notebook and brings text chat, speech interaction, image understanding,
document extraction, web tools, and conversation export together in one
interface.

> **Project author:** Subhadeep Dey\
> **Project name:** Kumar --- The Indian AI Companion\
> **Implementation:** Python · Gradio 6 · DeepSeek API\
> **Primary environment:** Google Colab

------------------------------------------------------------------------

## Table of Contents

-   [Features](#features)
-   [Architecture](#architecture)
-   [Models and voices](#models-and-voices)
-   [Technology stack](#technology-stack)
-   [Getting started](#getting-started)
-   [DeepSeek API configuration](#deepseek-api-configuration)
-   [Running the application](#running-the-application)
-   [Using Kumar](#using-kumar)
-   [Conversation export](#conversation-export)
-   [Project structure](#project-structure)
-   [Configuration](#configuration)
-   [Known limitations](#known-limitations)
-   [Security considerations](#security-considerations)
-   [Planned improvements](#planned-improvements)
-   [Acknowledgements](#acknowledgements)
-   [License](#license)

------------------------------------------------------------------------

## Features

### 1. Text-based AI conversation

-   Interactive chat interface powered by DeepSeek.
-   Streaming responses, so generated text appears progressively.
-   Customizable system instructions to adjust Kumar's behaviour and
    persona.
-   Selectable model configurations.

### 2. Multimodal input

Users can submit messages through the Gradio interface using: - Text. -
Microphone recordings. - Uploaded audio files. - Images. - PDF
documents. - DOCX documents. - Other text-readable files.

Image inputs are encoded as base64 data URLs and included in the model
request when the selected model configuration supports image input.

### 3. Speech-to-text (ASR)

-   Audio is submitted to DeepSeek's ASR endpoint.
-   The notebook attempts to convert uploaded or recorded audio into 16
    kHz, mono, 16-bit PCM WAV before transcription.
-   The recognized transcript is incorporated into the user's chat
    message.

### 4. Text-to-speech (TTS)

-   Optional spoken responses.
-   Two voice choices are configured in the notebook:
    -   **Tide**
    -   **Stella**
-   Generated WAV audio is returned to the Gradio audio component for
    playback.

### 5. Web search and URL reading

The application can request DeepSeek's web-search and URL-reading tools
when the web-search option is enabled.

### 6. Document extraction

The notebook includes local text extraction for: - PDF files, using
`pypdf`. - DOCX files, using `python-docx`. - Other files treated as
UTF-8 text.

Extracted text is added to the prompt sent to the model.

### 7. Conversation export

The current conversation can be exported in either: - Markdown (`.md`) -
PDF (`.pdf`)

The PDF export uses FPDF2 and attempts to use DejaVu Sans fonts for
Unicode text where those fonts are available.

### 8. Customizable interface

The Gradio interface includes: - Light, dark, and flagship themes. -
Model selection. - Voice selection. - Web-search toggle. - Spoken-reply
toggle. - Editable system instructions. - Conversation reset. - Export
controls.

------------------------------------------------------------------------

## Architecture

At a high level, the application follows this flow:

``` text
                 User
                  |
                  v
          Gradio 6 Interface
                  |
                  v
          chat_with_kumar()
                  |
        +---------+----------+
        |         |          |
        v         v          v
    Text Input  Audio     Attachments
                  |       /    |    \
                  v      v     v     v
                 ASR   Images  PDF  DOCX/Text
                  |      |     |     |
                  +------+-----+-----+
                         |
                         v
                 Conversation Builder
                         |
                         v
                    DeepSeek API
                  /      |       \
                 v       v        v
             Streaming Web Tools  Thinking
              Answer
                 |
          +------+------+
          |             |
          v             v
      Gradio Chat      DeepSeek TTS
          |             |
          v             v
      Chat Display   Audio Playback
          |
          v
     Markdown / PDF Export
```

The notebook is organized into seven main sections:

1.  Dependency installation and imports.
2.  DeepSeek API key and client setup.
3.  Conversation export helpers.
4.  ASR and TTS helpers.
5.  Document and image processing helpers.
6.  Core chat orchestration.
7.  Custom CSS and Gradio interface.

------------------------------------------------------------------------

## Models and voices

The notebook currently defines these model configurations:

  -----------------------------------------------------------------------
  Interface option        API model ID            Thinking
  ----------------------- ----------------------- -----------------------
  DeepSeek-V4.1-Flash     `deepseek-flash`        Disabled
  (Fast + Vision + Web)                           

  DeepSeek-V4.1-Flash     `deepseek-flash`        Enabled
  Thinking (Deep Search +                         
  Vision)                                         

  DeepSeek-V4-Pro (Deep   `deepseek-v4-pro`       Enabled
  Text & Code)                                    
  -----------------------------------------------------------------------

The application contains automatic model routing: when an image or
web-search capability is requested while a non-Flash model is selected,
the current implementation switches the request to `deepseek-flash`.

Configured TTS voices:

  Display name   Voice ID
  -------------- ----------
  Tide           `tide`
  Stella         `stella`

**Compatibility note:** Model IDs, feature availability, API request
formats, and voice support are controlled by DeepSeek and may change.
Confirm that the configured identifiers and endpoints are enabled for
your API account before running the notebook.

------------------------------------------------------------------------

## Technology stack

  Technology / library   Purpose
  ---------------------- --------------------------------------------
  Python                 Application logic
  Gradio                 Interactive web interface
  OpenAI Python SDK      DeepSeek-compatible chat completion client
  DeepSeek API           Language model, ASR, TTS, and web tools
  Requests               HTTP requests to DeepSeek speech endpoints
  PyPDF                  PDF text extraction
  python-docx            DOCX text extraction
  FPDF2                  PDF conversation export
  FFmpeg                 Audio conversion
  Base64                 Image encoding for model requests

------------------------------------------------------------------------

## Getting started

### Requirements

-   A Google account to run the notebook in Google Colab.
-   A DeepSeek API key with access to the required models and tools.
-   An internet connection.
-   FFmpeg available in the runtime for audio conversion.
-   A modern browser for the Gradio interface.

### 1. Open the notebook

Upload `My_Chatbot.ipynb` to Google Colab, or open it from your Google
Drive.

### 2. Install dependencies

The notebook installs the Python packages:

``` python
!pip install -q gradio openai pypdf python-docx fpdf2
```

FFmpeg is also required for the audio conversion helper. If it is not
already available in the Colab runtime, install it:

``` python
!apt-get update -qq
!apt-get install -y ffmpeg
```

### 3. Configure the API key

The notebook attempts to retrieve the key from Google Colab Secrets:

``` python
from google.colab import userdata

DEEPSEEK_API_KEY = userdata.get("DEEPSEEK_API_KEY")
```

In Colab:

1.  Open the **Secrets** panel.
2.  Create a secret named `DEEPSEEK_API_KEY`.
3.  Set its value to your DeepSeek API key.
4.  Enable notebook access to the secret.

Do not publish your API key, commit it to a repository, or share it in
screenshots or notebook outputs.

### 4. Initialize the client

The notebook uses the OpenAI-compatible Python client with DeepSeek's
base URL:

``` python
from openai import OpenAI

DEEPSEEK_BASE_URL = "https://api.deepseek.com"

client = OpenAI(
    api_key=DEEPSEEK_API_KEY,
    base_url=DEEPSEEK_BASE_URL
)
```

------------------------------------------------------------------------

## Running the application

Run the notebook cells **from top to bottom**:

1.  Install and import dependencies.
2.  Configure the DeepSeek API key and client.
3.  Define model and voice options.
4.  Define Markdown and PDF export functions.
5.  Define ASR and TTS functions.
6.  Define document and image processing functions.
7.  Define the core chat function.
8.  Define the custom CSS.
9.  Build and launch the Gradio interface.

The final notebook cell creates the Gradio Blocks interface.

The current launch configuration is:

``` python
demo.launch(
    share=True,
    debug=True,
    css=custom_css
)
```

This creates a public Gradio sharing link. See [Security
considerations](#security-considerations) before using this setting.

------------------------------------------------------------------------

## Using Kumar

### Send a text message

Enter a message in the chat input and submit it. Kumar streams the
response into the conversation window.

### Talk to Kumar

Use the microphone input. The application transcribes the recording and
sends the resulting text as part of the conversation.

### Upload an image

Attach an image and ask a question about it. The application encodes
supported image files and sends them as image content when using the
compatible model configuration.

### Upload a document

Attach a PDF, DOCX, or text-readable file. The application extracts its
text and appends it to the prompt.

### Enable web search

Turn on **Web Search & URL Reader** to allow the application to request
the configured DeepSeek web tools.

### Enable spoken replies

Turn on **Spoken Replies (TTS)** and select a voice. Kumar will request
speech synthesis after generating the text response.

### Customize Kumar

Open **Customize Kumar's Persona** and edit the system instructions.
These instructions are sent as the system message for the conversation.

### Change the appearance

Choose one of the three interface themes: - Light Theme - Dark Theme -
Flagship Theme

### Reset or export a conversation

Use **Reset Conversation** to clear the current chat. Choose Markdown or
PDF and select **Export Conversation** to generate a transcript.

------------------------------------------------------------------------

## Project structure

The project is currently delivered as a single Google Colab notebook:

``` text
Kumar/
│
├── My_Chatbot.ipynb
└── README.md
```

The notebook contains the application code, helper functions, CSS, and
Gradio interface in separate cells.

------------------------------------------------------------------------

## Configuration

The principal settings are defined near the beginning of the notebook.

  -----------------------------------------------------------------------
  Setting                 Variable                Description
  ----------------------- ----------------------- -----------------------
  DeepSeek API URL        `DEEPSEEK_BASE_URL`     API base URL

  Model configurations    `MODEL_OPTIONS`         UI labels, model IDs,
                                                  and thinking settings

  Voice configurations    `VOICE_OPTIONS`         Available TTS voices

  Image extensions        `IMAGE_EXTENSIONS`      Recognized image file
                                                  extensions

  Audio extensions        `AUDIO_EXTENSIONS`      Recognized audio file
                                                  extensions

  Initial greeting        `INITIAL_GREETING`      First assistant message
  -----------------------------------------------------------------------

The application also accepts custom system instructions from the
interface.

------------------------------------------------------------------------

## Known limitations

The following limitations apply to the current notebook implementation
and should be considered before production deployment:

1.  **Document formats:** PDF and DOCX have dedicated extraction logic.
    Other files are read as UTF-8 text, so binary formats such as XLSX
    and PPTX are not properly parsed.
2.  **Scanned PDFs:** The current PDF extraction method does not include
    OCR. Image-only or scanned PDFs may produce little or no extracted
    text.
3.  **Audio conversion fallback:** If FFmpeg conversion fails, the
    current helper returns the original file path. The subsequent ASR
    request may therefore receive audio that is not in the expected WAV
    format.
4.  **Conversation attachments:** Historical attachment handling is tied
    to the Gradio conversation history and current model capability.
    Previous image context may not be preserved correctly after model
    switching.
5.  **Context growth:** Conversation history is rebuilt and submitted on
    subsequent requests. Long conversations and repeated attachments can
    increase request size and latency.
6.  **Speech generation:** TTS is requested after the text response has
    completed; it is not currently streamed alongside text generation.
7.  **Temporary files:** Generated audio and export files use temporary
    storage. They are not a persistent conversation archive.
8.  **Language support:** ASR and TTS language support depends on the
    capabilities of the configured DeepSeek endpoints and voices.

These are known areas for improvement, not claims that every
corresponding workflow will fail.

------------------------------------------------------------------------

## Security considerations

-   Keep the DeepSeek API key in Colab Secrets or a secure server-side
    environment variable.
-   Never expose API keys in client-side code, public repositories,
    notebook output, or shared screenshots.
-   Do not use `share=True` for an unrestricted public application
    without appropriate access controls.
-   For public deployment, add authentication, rate limiting,
    request-size limits, and usage monitoring.
-   Disable Gradio debug mode in production.
-   Validate uploaded file types and sizes before processing.
-   Treat uploaded documents and extracted content as untrusted input.
-   Review API provider policies, privacy requirements, and
    data-retention implications before processing sensitive information.

For a local or private development session, consider:

``` python
demo.launch(
    share=False,
    debug=False,
    css=custom_css
)
```

------------------------------------------------------------------------

## Planned improvements

Potential directions for future versions include:

-   Modularizing the core chat function into independent application
    services.
-   More robust conversation-state and attachment management.
-   OCR support for scanned PDFs.
-   Dedicated parsers for CSV, Excel, PowerPoint, and additional
    document formats.
-   Improved exception handling and retry logic.
-   Persistent conversation storage and conversation search.
-   Better context-window management and attachment reuse.
-   Streaming speech generation.
-   Authentication, rate limiting, and deployment hardening.
-   Retrieval-Augmented Generation (RAG) over user-provided knowledge
    bases.
-   Controlled Python execution and additional tool integrations.
-   Agentic research and task-planning workflows.

------------------------------------------------------------------------

## Acknowledgements

Kumar is built using Python, Gradio, and the DeepSeek API, with
supporting open-source libraries for document processing, audio
conversion, and PDF generation.

**Created by Subhadeep Dey.**

------------------------------------------------------------------------

## License

No license is specified in the current notebook.

------------------------------------------------------------------------

Any type of contributions will be welcome.
