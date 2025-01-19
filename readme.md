This code defines a `youtube_summarizer` class that extracts a YouTube video's transcript and summarizes its content using AI models like OpenAI's GPT-4 and Ollama's models. Let’s break the code down step by step:

* * *

### **1\. Preliminaries**

```python
# !pip install youtube-transcript-api
yt_url = 'https://www.youtube.com/watch?v=_C8kWso4ne4&t=309s'
import os
from openai import OpenAI
import ollama
from dotenv import load_dotenv
from IPython.display import Markdown, display
```

-   The **`!pip install youtube-transcript-api`** command installs the `youtube-transcript-api` library (a tool for retrieving YouTube video transcripts). This line is commented out but can be run in Jupyter Notebook if needed.
-   The `yt_url` variable holds the link to a specific YouTube video.
-   Libraries used:
    -   `os`: For accessing environment variables (e.g., API keys).
    -   `openai` and `ollama`: APIs for interfacing with OpenAI and Ollama models.
    -   `dotenv`: For loading environment variables from a `.env` file.
    -   `IPython.display`: For rendering Markdown content (e.g., the summary).
* * *

### **2\. Loading Environment Variables**

```python
load_dotenv(override=True)
openai_api_key = os.getenv('OPENAI_API_KEY')
ollama_api_key = "http://localhost:11434/api/chat"
ollama_via_openai = OpenAI(base_url='http://localhost:11434/v1', api_key='ollama')

open_ai = OpenAI()
```

-   **`load_dotenv(override=True)`**: Loads environment variables from a `.env` file.
-   `openai_api_key`: Retrieves the OpenAI API key from environment variables.
-   `ollama_api_key`: Sets the base URL for accessing the local Ollama API.
-   `ollama_via_openai`: Configures an instance of OpenAI’s library to work with Ollama via OpenAI’s interface.
-   `open_ai`: Creates an instance of OpenAI’s API client.
* * *

### **3\. Class Definition: `youtube_summarizer`**

This class encapsulates all the logic for retrieving, processing, and summarizing a YouTube transcript.

#### **Attributes**

```python
openai_model = "gpt-4o-mini"
ollama_model = 'llama3.2'

system_prompt = "You are an expert in summarizing the content in the youtube transcript \
                make the summary presice and cover all the topics in the transcript \
                do not deviate from the transcript and stick only to the content in the transcript"
```

-   **`openai_model`**: Specifies the OpenAI model to use for summarization (`gpt-4o-mini`).
-   **`ollama_model`**: Specifies the Ollama model (`llama3.2`).
-   **`system_prompt`**: A system message that instructs the AI to summarize the transcript concisely and precisely, adhering strictly to the transcript content.
* * *

### **4\. Constructor**

```python
def __init__(self, url):
    self.url = url
    text = url.split('?v=')[-1]
    self.video_id = text
```

-   The `__init__` method initializes the class with the YouTube video URL.
-   The `video_id` is extracted from the URL (everything after `?v=`).

For example, if the `url` is `https://www.youtube.com/watch?v=_C8kWso4ne4&t=309s`, `self.video_id` becomes `_C8kWso4ne4`.

* * *

### **5\. Transcript Retrieval**

```python
def get_transcript(self):
    transcript = YouTubeTranscriptApi.get_transcript(video_id=self.video_id)
    return transcript
```

-   Uses `YouTubeTranscriptApi.get_transcript()` to fetch the transcript of the video based on its `video_id`.
-   Returns a list of dictionaries, where each dictionary contains a segment of the transcript.

Example of the transcript format:

```python
[
    {'text': 'Hello world', 'start': 0.0, 'duration': 2.0},
    {'text': 'Welcome to the video', 'start': 2.0, 'duration': 3.0},
    ...
]
```

* * *

### **6\. Combining the Transcript**

```python
def full_transcript(self):
    full_transcripts = ""
    transcript = self.get_transcript()
    for i in range(len(transcript)):
        full_transcripts += (transcript[i]['text'] + " ")
    return full_transcripts
```

-   Concatenates all the text segments from the transcript into a single string.
-   Returns the complete transcript as plain text.
* * *

### **7\. Preparing the AI Messages**

```python
def messages(self):
    messages = [
        {'role': 'system', 'content': self.system_prompt},
        {'role': 'user', 'content': self.full_transcript()}
    ]
    return messages
```

-   Prepares a list of messages to be passed to the AI model.
-   Includes:
    -   A **system message** with summarization instructions.
    -   A **user message** containing the full transcript.
* * *

### **8\. Summarizing with OpenAI**

```python
def openai_summarizer(self):
    response = open_ai.chat.completions.create(
        model=self.openai_model,
        messages=self.messages()
    )
    summary = response.choices[0].message.content
    result = display(Markdown(summary))
    return result
```

-   Sends the prepared messages to the OpenAI API for summarization using the `gpt-4o-mini` model.
-   Extracts the summary from the response and displays it in Markdown format.
* * *

### **9\. Summarizing with Ollama**

```python
def ollama_summarizer(self):
    response = ollama.chat(model=self.ollama_model, messages=self.messages())
    summary = response['message']['content']
    result = display(Markdown(summary))
    return result
```

-   Uses the Ollama API to summarize the transcript.
-   Extracts the summary from the response and displays it in Markdown format.
* * *

### **10\. Summarizing with Ollama via OpenAI Interface**

```python
def ollama_summarizer_through_openai(self):
    response = ollama_via_openai.chat.completions.create(
        model=self.ollama_model,
        messages=self.messages()
    )
    summary = response.choices[0].message.content
    result = display(Markdown(summary))
    return result
```

-   Uses OpenAI’s interface to call the Ollama model.
-   Displays the summary in Markdown format.
* * *

### **How It Works**

1.  Initialize the `youtube_summarizer` with a YouTube URL:

    ```python
    yt_summary = youtube_summarizer(yt_url)
    ```

2.  Retrieve the transcript:

    ```python
    full_transcript = yt_summary.full_transcript()
    ```

3.  Summarize using one of the summarization methods:
    -   OpenAI:

        ```python
        yt_summary.openai_summarizer()
        ```

    -   Ollama:

        ```python
        yt_summary.ollama_summarizer()
        ```

    -   Ollama via OpenAI:

        ```python
        yt_summary.ollama_summarizer_through_openai()
        ```

* * *

### **Summary of the Workflow**

1.  Extract the `video_id` from the YouTube URL.
2.  Fetch the video transcript using `YouTubeTranscriptApi`.
3.  Prepare a system and user prompt with the transcript text.
4.  Send the prepared messages to AI models (OpenAI or Ollama) for summarization.
5.  Display the summary in Markdown format.

This structure allows easy retrieval and summarization of YouTube video transcripts, using multiple AI models for flexibility.
