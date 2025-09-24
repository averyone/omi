# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

Omi is an open-source AI wearable platform that captures conversations, provides summaries, action items, and executes actions. The repository contains multiple components working together to create a complete wearable AI solution.

## Architecture

### Core Components

1. **Mobile App** (`app/`) - Flutter-based companion app for iOS/Android/macOS
2. **Backend API** (`backend/`) - Python FastAPI server handling transcription, memory processing, and AI interactions
3. **Firmware** (`omi/firmware/`) - Zephyr RTOS-based firmware for the wearable device
4. **MCP Server** (`mcp/`) - Model Context Protocol server for memory and conversation management
5. **Documentation** (`docs/`) - Mintlify-based documentation site
6. **Hardware Designs** - Physical device specifications and assembly guides

### Data Flow

- Audio capture via wearable device → Bluetooth streaming to mobile app → Backend API for transcription (Deepgram) → Memory processing (OpenAI/Groq) → Vector storage (Pinecone) → Real-time sync with mobile app

### Technology Stack

- **Frontend**: Flutter (cross-platform mobile)
- **Backend**: Python FastAPI, Firebase, Redis, Pinecone
- **Firmware**: Zephyr RTOS (Nordic nRF)
- **AI**: OpenAI GPT, Deepgram STT, vector embeddings
- **Infrastructure**: Modal, Google Cloud, ngrok for development

## Development Commands

### Mobile App (`app/`)

```bash
# Setup for specific platform
bash setup.sh ios
bash setup.sh android
bash setup.sh macos

# Run in development mode
flutter run --flavor dev

# Build for release
flutter build ios --flavor dev --release
flutter build apk --flavor dev --release

# Install dependencies
flutter pub get

# Generate code (for serialization, etc.)
dart run build_runner build

# Install iOS dependencies
cd ios && pod install --repo-update
```

### Backend (`backend/`)

```bash
# Setup virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run development server
uvicorn main:app --reload --env-file .env

# Start ngrok tunnel for development
ngrok http --domain=your-domain.ngrok-free.app 8000

# Load testing
cd testing && python locustfile.py
```

### Firmware (`omi/firmware/`)

```bash
# Build using Docker (recommended)
./scripts/build-docker.sh

# Build outputs will be in build/docker_build/
```

### Documentation (`docs/`)

```bash
# Install Mintlify
npm i -g mintlify

# Run development server
mintlify dev

# Check for broken links
mintlify broken-links
```

### MCP Server (`mcp/`)

```bash
# Run tests
python -m pytest tests/

# Debug MCP server
npx @modelcontextprotocol/inspector uv run mcp-server-omi
```

## Testing

### Mobile App Testing

```bash
# Run unit tests
flutter test

# Run integration tests
flutter test integration_test/

# Run specific test file
flutter test test/models/memory_test.dart
```

### Backend Testing

```bash
# Load testing
cd backend/testing
python load_test.py

# API testing with locust
python locustfile.py
```

## Project Structure

### Mobile App (`app/lib/`)

- `pages/` - Screen components organized by feature
- `providers/` - State management (Provider pattern)
- `services/` - API clients and external service integrations
- `models/` - Data models and serialization
- `utils/` - Helper functions and utilities
- `widgets/` - Reusable UI components
- `ui/` - Atomic design system (atoms, molecules)

### Backend (`backend/`)

- `routers/` - FastAPI route handlers organized by domain
- `utils/` - Shared utilities (STT, LLM, storage, etc.)
- `models/` - Pydantic data models
- `testing/` - Load testing and performance tools

### Key Configuration Files

- `app/pubspec.yaml` - Flutter dependencies and build configuration
- `backend/requirements.txt` - Python dependencies
- `backend/.env` - Environment variables (API keys, database URLs)
- `app/.dev.env` - Mobile app environment configuration
- `omi/firmware/CMakePresets.json` - Firmware build configurations

## Development Environment Setup

### Prerequisites

**For Mobile Development:**
- Flutter SDK v3.35.3+
- For iOS: Xcode v16.4+, CocoaPods v1.16.2+
- For Android: Android Studio, API 35, JDK 21, Gradle v8.10, NDK 28.2.13676358
- Opus Codec library

**For Backend Development:**
- Python 3.11+
- Google Cloud SDK
- Firebase project with enabled APIs (Cloud Resource Manager, Firebase Management, Cloud Firestore)
- Redis instance (Upstash recommended)
- ngrok account for local development tunneling

**For Firmware Development:**
- nRF Connect for VS Code, or
- Docker for cross-platform building

### API Keys Required

- OpenAI API key
- Deepgram API key
- Pinecone API key and index
- Firebase service account credentials
- Redis connection credentials

## Key Development Notes

### Mobile App Architecture

The Flutter app follows a Provider-based state management pattern with atomic design principles. The app supports multiple "flavors" (dev/prod) and includes extensive Bluetooth integration for device connectivity.

### Backend API Design

The FastAPI backend is modular with domain-specific routers. It processes real-time audio streams, performs speech-to-text conversion, generates structured memories using LLMs, and manages vector embeddings for similarity search.

### Firmware Integration

The Zephyr-based firmware handles audio capture, Bluetooth streaming, and local storage. It automatically switches between streaming mode (when connected to app) and storage mode (when disconnected).

### Memory and Conversation Processing

The system creates structured memories from conversations using:
1. Real-time transcription with speaker diarization
2. LLM-based summarization and action item extraction
3. Vector embedding generation for semantic search
4. Local similarity search for contextual retrieval

### Development Workflow

1. Backend changes require restarting the uvicorn server
2. Mobile app uses hot reload for UI changes
3. Firmware changes require full rebuild and flashing
4. Use ngrok for mobile app to connect to local backend during development
5. Firebase configuration is automatically set up via setup scripts

### Testing Strategy

- Mobile: Integration tests for device connectivity, unit tests for business logic
- Backend: Load testing with Locust, API endpoint validation
- End-to-end: Real device testing with audio processing pipeline

This architecture enables rapid development of AI-powered wearable experiences while maintaining separation of concerns across hardware, mobile, and cloud components.