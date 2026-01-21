# CAP60 Voice Agent

A real-time voice agent backend that bridges Twilio telephony with Deepgram's conversational AI platform. This enables voice-based customer service interactions over phone calls.

## Architecture

```
Phone Call → Twilio → WebSocket → CAP60 Backend → Deepgram Agent API
                                       ↓
                              Audio Orchestration
                              (STT → LLM → TTS)
```

The system handles bidirectional audio streaming between Twilio and Deepgram's agent API, supporting features like barge-in (user interruption).

## Features

- Real-time voice conversation via phone calls
- Speech-to-Text using Deepgram Nova-3
- Text-to-Speech using Deepgram Aura-2
- LLM responses powered by OpenAI GPT-4o-mini
- Barge-in support (interruption handling)
- mu-law audio encoding at 8kHz (telephony standard)

## Prerequisites

- Python 3.11+
- Deepgram API key
- Twilio account (for phone integration)
- ngrok or similar tunneling service (for local development)

## Installation

1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd CAP60-voice-agent
   ```

2. Create and activate a virtual environment:
   ```bash
   python -m venv cap60-voice-agent
   # Windows
   cap60-voice-agent\Scripts\activate
   # Linux/macOS
   source cap60-voice-agent/bin/activate
   ```

3. Install dependencies:
   ```bash
   pip install -r requirements.txt
   # Or if using uv:
   uv sync
   ```

4. Create a `.env` file with your API key:
   ```
   DEEPGRAM_API_KEY=your_deepgram_api_key_here
   ```

## Configuration

Edit `settings.json` to customize the agent behavior:

```json
{
    "type": "Settings",
    "audio": {
        "input": { "encoding": "mulaw", "sample_rate": 8000 },
        "output": { "encoding": "mulaw", "sample_rate": 8000, "container": "none" }
    },
    "agent": {
        "language": "en",
        "listen": {
            "provider": { "type": "deepgram", "model": "nova-3" }
        },
        "think": {
            "provider": { "type": "open_ai", "model": "gpt-4o-mini" },
            "prompt": "You are a helpful AI assistant focused on customer service."
        },
        "speak": {
            "provider": { "type": "deepgram", "model": "aura-2-thalia-en" }
        },
        "greeting": "Hello! How can I help you today?"
    }
}
```

### Configuration Options

| Section | Option | Description |
|---------|--------|-------------|
| `agent.listen` | `model` | Deepgram STT model (e.g., `nova-3`) |
| `agent.think` | `model` | OpenAI model for responses |
| `agent.think` | `prompt` | System prompt for the AI |
| `agent.speak` | `model` | Deepgram TTS voice model |
| `agent.greeting` | - | Initial greeting message |

## Usage

1. Start the server:
   ```bash
   python main.py
   ```
   The server runs on `localhost:5050`.

2. Expose the local server (for Twilio integration):
   ```bash
   ngrok http 5050
   ```

3. Configure your Twilio phone number to use the WebSocket URL:
   - Set the webhook to your ngrok URL
   - Configure TwiML to stream audio to your WebSocket endpoint

## Project Structure

```
CAP60-voice-agent/
├── main.py           # Main application entry point
├── settings.json     # Agent configuration
├── pyproject.toml    # Project dependencies
├── .env              # Environment variables (not committed)
└── README.md         # This file
```

## How It Works

1. **Twilio Connection**: Incoming calls connect via WebSocket to the server on port 5050
2. **Audio Buffering**: Inbound audio is buffered (20 frames × 160 bytes) before processing
3. **Deepgram Processing**: Audio is sent to Deepgram's agent API for STT → LLM → TTS pipeline
4. **Response Streaming**: Generated audio responses are streamed back to the caller
5. **Barge-in Handling**: When the user speaks, any ongoing agent audio is cleared

## Dependencies

- `websockets` - WebSocket client/server implementation
- `python-dotenv` - Environment variable management

## License

MIT
