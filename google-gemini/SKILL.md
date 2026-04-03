---
name: google-gemini
description: Google Gemini API and Vertex AI — text generation, multimodal, streaming, Live API, embeddings, function calling, and Gemini Workspace CLI. Auto-activates for Gemini or Google AI projects.
origin: VoltAgent/awesome-agent-skills (google-gemini)
tools: Read, Write, Edit, Bash
---

# Google Gemini API / Vertex AI Skills

Official Google Gemini team skills for building AI-powered applications.

## Setup

```bash
pip install google-genai
# or for Vertex AI
pip install google-cloud-aiplatform
```

```python
import google.generativeai as genai
# or new SDK:
from google import genai
from google.genai import types

# API Key (Gemini Developer API)
client = genai.Client(api_key=os.environ['GEMINI_API_KEY'])

# Vertex AI
client = genai.Client(
    vertexai=True,
    project=os.environ['GOOGLE_CLOUD_PROJECT'],
    location='us-central1',
)
```

## Text Generation

```python
response = client.models.generate_content(
    model='gemini-2.0-flash',
    contents='Explain quantum computing in simple terms',
)
print(response.text)

# With system instruction
response = client.models.generate_content(
    model='gemini-2.0-flash',
    config=types.GenerateContentConfig(
        system_instruction='You are a helpful financial analyst.',
        temperature=0.3,
        max_output_tokens=1024,
    ),
    contents='Analyze this balance sheet: ...',
)
```

## Multi-turn Chat

```python
chat = client.chats.create(model='gemini-2.0-flash')

response = chat.send_message('What is the capital of France?')
print(response.text)  # Paris

response = chat.send_message('What is its population?')
print(response.text)  # Context preserved — knows "its" = Paris
```

## Streaming

```python
# Stream text
for chunk in client.models.generate_content_stream(
    model='gemini-2.0-flash',
    contents='Write a poem about the ocean',
):
    print(chunk.text, end='', flush=True)

# Stream chat
for chunk in chat.send_message_stream('Explain AI in detail'):
    print(chunk.text, end='', flush=True)
```

## Multimodal (Vision)

```python
import PIL.Image

# Image + text
image = PIL.Image.open('chart.png')
response = client.models.generate_content(
    model='gemini-2.0-flash',
    contents=[image, 'What trends do you see in this chart?'],
)

# URL reference
response = client.models.generate_content(
    model='gemini-2.0-flash',
    contents=[
        types.Part.from_uri('https://example.com/image.jpg', mime_type='image/jpeg'),
        'Describe this image',
    ],
)

# PDF analysis
with open('report.pdf', 'rb') as f:
    pdf_data = f.read()

response = client.models.generate_content(
    model='gemini-2.0-flash',
    contents=[
        types.Part.from_bytes(data=pdf_data, mime_type='application/pdf'),
        'Summarize the key findings',
    ],
)
```

## Function Calling

```python
from google.genai import types

# Define tools
tools = [
    types.Tool(
        function_declarations=[
            types.FunctionDeclaration(
                name='get_stock_price',
                description='Get the current price of a stock',
                parameters={
                    'type': 'object',
                    'properties': {
                        'symbol': {'type': 'string', 'description': 'Stock ticker symbol'},
                    },
                    'required': ['symbol'],
                },
            )
        ]
    )
]

response = client.models.generate_content(
    model='gemini-2.0-flash',
    contents="What's the current price of AAPL?",
    config=types.GenerateContentConfig(tools=tools),
)

# Handle function call
if response.candidates[0].content.parts[0].function_call:
    fc = response.candidates[0].content.parts[0].function_call
    result = get_stock_price(fc.args['symbol'])

    # Send result back
    response = client.models.generate_content(
        model='gemini-2.0-flash',
        contents=[
            {'role': 'user', 'parts': [{'text': "What's AAPL price?"}]},
            {'role': 'model', 'parts': [response.candidates[0].content.parts[0]]},
            {'role': 'user', 'parts': [{'function_response': {'name': fc.name, 'response': result}}]},
        ],
    )
```

## Embeddings

```python
result = client.models.embed_content(
    model='text-embedding-004',
    contents='Machine learning is a subset of AI',
)
embedding = result.embeddings[0].values  # 768-dim vector

# Batch embeddings
result = client.models.embed_content(
    model='text-embedding-004',
    contents=['text 1', 'text 2', 'text 3'],
)
embeddings = [e.values for e in result.embeddings]
```

## Live API (Real-time Bidirectional)

```python
import asyncio
from google import genai

async def live_session():
    async with client.aio.live.connect(
        model='gemini-2.0-flash-live',
        config=types.LiveConnectConfig(
            response_modalities=['TEXT'],
            system_instruction='You are a helpful assistant.',
        ),
    ) as session:
        # Send audio or text in real-time
        await session.send(input='Hello!', end_of_turn=True)

        async for response in session.receive():
            if response.text:
                print(response.text, end='', flush=True)

asyncio.run(live_session())
```

## Vertex AI (Google Cloud)

```python
import vertexai
from vertexai.generative_models import GenerativeModel, Part

vertexai.init(project='my-project', location='us-central1')
model = GenerativeModel('gemini-2.0-flash')

# Generate with grounding (real-time web search)
from vertexai.generative_models import Tool, grounding
google_search_tool = Tool.from_google_search_retrieval(grounding.GoogleSearchRetrieval())

response = model.generate_content(
    'What are the latest AI developments?',
    tools=[google_search_tool],
)

# Batch prediction (large scale)
from vertexai.batch_prediction import BatchPredictionJob
job = BatchPredictionJob.submit(
    source_model='gemini-2.0-flash',
    input_dataset='gs://my-bucket/inputs.jsonl',
    output_uri_prefix='gs://my-bucket/outputs/',
)
```

## Structured Output (JSON)

```python
from pydantic import BaseModel

class Recipe(BaseModel):
    name: str
    ingredients: list[str]
    steps: list[str]
    prep_time_minutes: int

response = client.models.generate_content(
    model='gemini-2.0-flash',
    contents='Give me a pasta carbonara recipe',
    config=types.GenerateContentConfig(
        response_mime_type='application/json',
        response_schema=Recipe,
    ),
)

recipe = Recipe.model_validate_json(response.text)
```

## Google Workspace CLI (Gemini)

```bash
# Use Gemini for Workspace automation
gemini workspace docs summarize document.gdoc
gemini workspace sheets analyze spreadsheet.gsheets --question "What are the top 5 products by revenue?"
gemini workspace slides create --topic "Q4 Results" --slides 10
gemini workspace gmail draft --to user@example.com --subject "Meeting" --tone professional
```

## Sources
- google-gemini/gemini-api-dev
- google-gemini/vertex-ai-api-dev
- google-gemini/gemini-live-api-dev
- google-gemini/gemini-interactions-api
