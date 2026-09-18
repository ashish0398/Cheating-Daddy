# Cheating Daddy - Complete Technical Documentation

> **A real-time AI assistant for interviews, meetings, presentations, and more**

---

## 📌 Table of Contents

1. [🚀 Overview](#-overview)
2. [🏗️ Architecture](#-architecture)
3. [📁 Project Structure](#-project-structure)
4. [🔧 Core Components](#-core-components)
5. [🤖 AI Integration](#-ai-integration)
6. [🎤 Audio Capture](#-audio-capture)
7. [💾 Storage System](#-storage-system)
8. [⌨️ Keyboard Shortcuts](#-keyboard-shortcuts)
9. [📦 Build & Packaging](#-build--packaging)
10. [🔒 Security](#-security)

---

## 🚀 Overview

**Cheating Daddy** is a real-time AI assistant built with Electron that provides contextual help during:
- 💼 Job Interviews
- 📞 Sales Calls
- 👥 Meetings
- 🎤 Presentations
- 🤝 Negotiations
- 📝 Exams

### ✨ Key Features

| Feature | Description |
|---------|-------------|
| **Real-time AI** | Powered by Google Gemini, Groq, or local LLMs |
| **Multi-modal Input** | Combines screen + audio for context |
| **6 Profiles** | Specialized modes for different scenarios |
| **Transparent Overlay** | Always-on-top window, positionable anywhere |
| **Click-through Mode** | Make window transparent to mouse clicks |
| **Cross-platform** | macOS, Windows, Linux |
| **Offline Mode** | Local AI with downloaded models |
| **Session History** | Saves conversation history |

---

## 🏗️ Architecture

### Process Model

```
MAIN PROCESS (index.js)
├── Window Management
├── IPC Handlers (Storage, AI, General)
└── Lifecycle Management
     
     ▼
     
RENDERER PROCESS (CheatingDaddyApp.js)
├── UI Components (LitElement)
├── Event Handlers
└── State Management
```

### Data Flow

```
Audio Capture → VAD → Transcription → AI Processing → Response Generation → Display
                                  ▲
Screen Capture ──────────────────┘
```

### Supported AI Providers

| Provider | Type | Models |
|----------|------|--------|
| Google Gemini | BYOK | gemini-3.1-flash-live, gemini-2.5-flash |
| Groq | API | qwen3.6-27b, gpt-oss-120b, gpt-oss-20b |
| Cloud API | Hosted | wss://api.cheatingdaddy.com/ws |
| Local AI | Offline | Llama.cpp + Whisper.cpp |

---

## 📁 Project Structure

```
Cheating-Daddy/
├── src/
│   ├── assets/              # Icons, libraries, binaries
│   │   ├── logo.*           # Application icons
│   │   ├── SystemAudioDump  # macOS audio capture binary
│   │   └── libs/            # Minified JS libraries
│   │
│   ├── components/         # LitElement UI components
│   │   ├── app/
│   │   │   └── CheatingDaddyApp.js   # Main app component
│   │   └── views/         # Individual views
│   │       ├── MainView.js
│   │       ├── AssistantView.js
│   │       ├── CustomizeView.js
│   │       └── ... (8 views total)
│   │
│   ├── index.html         # Main HTML entry
│   ├── index.js           # Electron main process
│   ├── preload.js         # Preload script
│   ├── storage.js         # Local storage management
│   └── audioUtils.js      # Audio processing
│
│   └── utils/             # Utility modules
│       ├── cloud.js        # Cloud API (WebSocket)
│       ├── gemini.js       # Google Gemini integration
│       ├── localai.js      # Local AI processing
│       ├── native-ai-runtime.js  # Native binary management
│       ├── prompts.js      # AI prompt templates
│       ├── window.js       # Window management
│       └── transportLogger.js  # Event logging
│
├── package.json
├── forge.config.js
├── entitlements.plist
└── README.md
```

---

## 🔧 Core Components

### 1. Main Process (src/index.js)

**Responsibilities:**
- Application lifecycle (startup, shutdown, activation)
- Window creation and management
- IPC handler registration
- Storage initialization
- Permission requests

**Key Functions:**
```javascript
app.whenReady().then(async () => {
    storage.initializeStorage();
    createMainWindow();
    setupGeminiIpcHandlers(geminiSessionRef);
    setupStorageIpcHandlers();
    setupGeneralIpcHandlers();
});
```

### 2. Storage System (src/storage.js)

**Config Directory Locations:**
- Windows: `%APPDATA%\cheating-daddy-config`
- macOS: `~/Library/Application Support/cheating-daddy-config`
- Linux: `~/.config/cheating-daddy-config`

**File Structure:**
```
config/
├── config.json          # App configuration
├── credentials.json     # API keys
├── preferences.json     # User preferences
├── keybinds.json        # Keyboard shortcuts
├── limits.json          # Rate limiting
└── history/             # Session history
    └── {timestamp}.json
```

**Configuration API:**
```javascript
// Config
getConfig() / setConfig() / updateConfig(key, value)

// Credentials
getCredentials() / setCredentials() / getApiKey() / setApiKey()
getGroqApiKey() / setGroqApiKey()

// Preferences
getPreferences() / setPreferences() / updatePreference(key, value)

// History
saveSession() / getSession() / getAllSessions() / deleteSession()

// Rate Limiting
getTodayLimits() / incrementLimitCount() / incrementCharUsage()
```

**Default Preferences:**
```javascript
{
    providerMode: 'byok',
    selectedProfile: 'interview',
    selectedLanguage: 'en-US',
    selectedScreenshotInterval: '5',
    selectedImageQuality: 'medium',
    localLlmModel: 'unsloth/Qwen3.5-4B-GGUF:Q4_K_M',
    whisperModel: 'tiny.en',
    googleSearchEnabled: false
}
```

### 3. AI Integration (src/utils/gemini.js)

**State Management:**
```javascript
let currentProviderMode = 'byok';    // 'byok' | 'cloud' | 'local'
let currentSessionId = null;
let currentTranscription = '';
let conversationHistory = [];
let screenAnalysisHistory = [];
let currentProfile = null;
let currentSystemPrompt = null;
```

**Main Functions:**
```javascript
initializeNewSession(profile, customPrompt)  // Start new session
saveConversationTurn(transcription, aiResponse)  // Save conversation
sendToGroq(transcription)                    // Send to Groq API
sendImageToGroq(base64Data, prompt)        // Send image to Groq
processAudioChunk(pcmBuffer)               // Process audio
stopMacOSAudioCapture()                   // Stop audio capture
```

### 4. Cloud Integration (src/utils/cloud.js)

**WebSocket Connection:**
```javascript
wss://api.cheatingdaddy.com/ws?token={token}

// Message Types:
// - connected: Connection established
// - transcription: Audio transcription result
// - response_start/chunk/end: Response streaming
// - session_end: Session terminated
// - error: Error occurred
```

### 5. Local AI (src/utils/localai.js + native-ai-runtime.js)

**Components:**
- **Whisper Server**: Audio transcription (whisper.cpp)
- **Llama Server**: LLM inference (llama.cpp)

**VAD Configuration:**
```javascript
const VAD_MODES = {
    NORMAL: { energyThreshold: 0.01, speechFramesRequired: 3, silenceFramesRequired: 30 },
    VERY_AGGRESSIVE: { energyThreshold: 0.02, speechFramesRequired: 2, silenceFramesRequired: 15 }
};
```

**Native Binaries:**
- macOS (arm64, x64): ~10MB each
- Windows (x64): ~10MB each
- Downloaded from GitHub releases (v0.7.0)
- SHA256 verified

**Supported Models:**
- Whisper: tiny.en (75MB), base.en (142MB), small.en (469MB)
- LLM: GGUF format from Hugging Face

---

## 🤖 AI Integration

### Profile System (src/utils/prompts.js)

**6 Specialized Profiles:**

| Profile | Focus | Style |
|---------|-------|-------|
| interview | Job interviews | Direct, ready-to-speak |
| sales | Sales calls | Persuasive, professional |
| meeting | Meetings | Clear, action-oriented |
| presentation | Presentations | Confident, engaging |
| negotiation | Negotiations | Strategic, professional |
| exam | Exams | Direct, accurate |

**Prompt Structure:**
```javascript
{
    intro: "Profile introduction...",
    formatRequirements: "Response format...",
    searchUsage: "Google search usage...",
    content: "Examples and guidelines...",
    outputInstructions: "Output formatting..."
}
```

**System Prompt Construction:**
```javascript
function buildSystemPrompt(promptParts, customPrompt, googleSearchEnabled) {
    return [
        promptParts.intro,
        promptParts.formatRequirements,
        googleSearchEnabled ? promptParts.searchUsage : '',
        promptParts.content,
        'User-provided context\n-----\n',
        customPrompt,
        '\n-----\n\n',
        promptParts.outputInstructions
    ].join('\n\n');
}
```

---

## 🎤 Audio Capture

### Platform-Specific Implementations

| Platform | Method | Status |
|----------|--------|--------|
| macOS | SystemAudioDump binary | ✅ Full |
| Windows | getDisplayMedia with loopback | ✅ Full |
| Linux | Microphone input | ⚠️ Limited |

### Audio Processing Flow

```
1. Audio Capture (PCM format)
   │
2. Buffer Accumulation
   │
3. VAD Detection (RMS-based)
   │
4. Speech Detected → Accumulate buffers
   │
5. Silence Detected → Process audio
   │
6. Transcription (Whisper/Gemini/Groq)
   │
7. Response Generation (AI)
   │
8. Streaming to UI
```

**VAD Implementation:**
```javascript
function processVad(pcm16kBuffer) {
    const rms = calculateRms(pcm16kBuffer);
    const isVoice = rms > vadConfig.energyThreshold;
    
    if (isVoice) {
        speechFrameCount++;
        silenceFrameCount = 0;
        if (!isSpeaking && speechFrameCount >= vadConfig.speechFramesRequired) {
            isSpeaking = true;
            speechBuffers = [];
        }
        if (isSpeaking) speechBuffers.push(Buffer.from(pcm16kBuffer));
    } else {
        silenceFrameCount++;
        speechFrameCount = 0;
        if (isSpeaking && silenceFrameCount >= vadConfig.silenceFramesRequired) {
            isSpeaking = false;
            handleSpeechEnd(Buffer.concat(speechBuffers));
        }
    }
}
```

### Audio Format Conversion

```javascript
// src/utils/audioUtils.js
function pcmToWav(pcmBuffer, sampleRate = 24000, channels = 1, bitDepth = 16) {
    const byteRate = sampleRate * channels * (bitDepth / 8);
    const blockAlign = channels * (bitDepth / 8);
    const dataSize = pcmBuffer.length;
    
    // Create 44-byte WAV header
    const header = Buffer.alloc(44);
    header.write('RIFF', 0);
    header.writeUInt32LE(dataSize + 36, 4);
    header.write('WAVE', 8);
    // ... (write fmt and data chunks)
    
    return Buffer.concat([header, pcmBuffer]);
}
```

---

## 💾 Storage System

### Session Data Structure

```javascript
{
    "sessionId": "1699999999999",
    "createdAt": 1699999999999,
    "lastUpdated": 1699999999999,
    "profile": "interview",
    "customPrompt": "User's custom context",
    
    "conversationHistory": [
        {
            "timestamp": 1699999999999,
            "transcription": "Interviewer: Tell me about yourself",
            "ai_response": "I'm a software engineer with..."
        }
    ],
    
    "screenAnalysisHistory": [
        {
            "timestamp": 1700000000500,
            "prompt": "Describe screen",
            "response": "The screen shows a code editor...",
            "model": "gemini-2.5-flash"
        }
    ]
}
```

### Rate Limiting

**Google Gemini:**
- 20 requests/day for gemini-2.5-flash
- 20 requests/day for gemini-2.5-flash-lite
- Tracked in `limits.json`

**Groq:**
- Character-based limits per model
- Tracked per provider and model
- Example: qwen3-32b has 1,500,000 char limit

---

## ⌨️ Keyboard Shortcuts

| Action | macOS | Windows/Linux |
|--------|-------|---------------|
| Move Up | Alt+↑ | Ctrl+↑ |
| Move Down | Alt+↓ | Ctrl+↓ |
| Move Left | Alt+← | Ctrl+← |
| Move Right | Alt+→ | Ctrl+→ |
| Toggle Visibility | Cmd+\ | Ctrl+\ |
| Toggle Click-Through | Cmd+M | Ctrl+M |
| Next Step | Cmd+Enter | Ctrl+Enter |
| Previous Response | Cmd+[ | Ctrl+[ |
| Next Response | Cmd+] | Ctrl+] |
| Scroll Up | Cmd+Shift+↑ | Ctrl+Shift+↑ |
| Scroll Down | Cmd+Shift+↓ | Ctrl+Shift+↓ |
| **Emergency Erase** | **Cmd+Shift+E** | **Ctrl+Shift+E** |

**Emergency Erase:** Immediately hides window, closes session, clears data, quits app.

### Shortcut Implementation

```javascript
// src/utils/window.js
function updateGlobalShortcuts(keybinds, mainWindow, sendToRenderer, geminiSessionRef) {
    globalShortcut.unregisterAll();
    
    const moveIncrement = Math.floor(Math.min(width, height) * 0.1);
    
    // Movement
    globalShortcut.register(keybinds.moveUp, () => {
        const [x, y] = mainWindow.getPosition();
        mainWindow.setPosition(x, y - moveIncrement);
    });
    
    // Toggle visibility
    globalShortcut.register(keybinds.toggleVisibility, () => {
        if (mainWindow.isVisible()) mainWindow.hide();
        else mainWindow.showInactive();
    });
    
    // Click-through
    globalShortcut.register(keybinds.toggleClickThrough, () => {
        mouseEventsIgnored = !mouseEventsIgnored;
        mainWindow.setIgnoreMouseEvents(mouseEventsIgnored, { forward: true });
    });
    
    // Emergency erase
    globalShortcut.register(keybinds.emergencyErase, () => {
        mainWindow.hide();
        if (geminiSessionRef.current) {
            geminiSessionRef.current.close();
            geminiSessionRef.current = null;
        }
        sendToRenderer('clear-sensitive-data');
        setTimeout(() => app.quit(), 300);
    });
}
```

---

## 🌐 Network Communication

### IPC (Inter-Process Communication)

**Main → Renderer Events:**
```javascript
// Via ipcRenderer.send()
'new-response'           // New AI response
'update-response'        // Streaming response update
'update-status'          // Status text update
'click-through-toggled'  // Click-through mode changed
'reconnect-failed'       // Reconnection failed
'whisper-downloading'    // Whisper download status
'local-ai-download-progress' // Local AI download progress
'save-session-context'   // Save session metadata
'save-conversation-turn' // Save conversation turn
'save-screen-analysis'  // Save screen analysis
'view-changed'           // View changed
```

**Renderer → Main Requests:**
```javascript
// Via ipcRenderer.invoke()
'get-app-version'       // Get app version
'quit-application'      // Quit app
'open-external'         // Open URL
'toggle-window-visibility' // Toggle window
'close-session'         // Close session
'window-minimize'       // Minimize window
'update-keybinds'       // Update shortcuts

// Storage operations (20+ methods)
'storage:get-config'
'storage:set-config'
'storage:get-api-key'
'storage:set-api-key'
'storage:get-groq-api-key'
'storage:get-preferences'
'storage:set-preferences'
'storage:update-preference'
'storage:get-all-sessions'
'storage:save-session'
'storage:delete-session'
'storage:delete-all-sessions'
'storage:clear-all'
```

### HTTP APIs

**Google Gemini:**
```
POST https://generativelanguage.googleapis.com/v1beta/models/{model}:generateContent
Headers: { Authorization: Bearer {apiKey}, Content-Type: application/json }
Body: { contents: [{parts: [{text: "prompt"}]}], generationConfig: {temperature: 0.7} }
```

**Groq:**
```
POST https://api.groq.com/openai/v1/chat/completions
Headers: { Authorization: Bearer {apiKey}, Content-Type: application/json }
Body: {
    model: "qwen/qwen3.6-27b",
    messages: [{role: "user", content: "prompt"}],
    stream: true,
    temperature: 0.7
}
```

### WebSocket (Cloud API)

```javascript
const url = `wss://api.cheatingdaddy.com/ws?token=${encodeURIComponent(token)}`;
cloudWs = new WebSocket(url);

// Messages:
// Binary: Audio chunks (PCM)
// JSON: { type: 'set_config', profile: 'interview', user_context: '' }
// JSON: { type: 'test_text', text: 'message' }
// JSON: { type: 'image', image: 'base64...' }
// JSON: { type: 'end_connection' }
```

---

## 📦 Build & Packaging

### Electron Forge Configuration

```javascript
// forge.config.js
module.exports = {
    packagerConfig: {
        asar: true,
        extraResource: ['./src/assets/SystemAudioDump'],
        name: 'Cheating Daddy',
        icon: 'src/assets/logo'
    },
    makers: [
        { name: '@electron-forge/maker-squirrel', config: { name: 'cheating-daddy' } }, // Windows
        { name: '@electron-forge/maker-dmg', platforms: ['darwin'] }, // macOS
        { name: '@reforged/maker-appimage', platforms: ['linux'] } // Linux
    ],
    plugins: [
        '@electron-forge/plugin-auto-unpack-natives',
        new FusesPlugin({ version: FuseVersion.V1, ... })
    ]
};
```

### Build Commands

```bash
npm install           # Install dependencies
npm start             # Run development app
npm run package       # Package application
npm run make          # Create distributables
npm run publish       # Publish releases
```

### Dependencies

**Runtime:**
- Electron ^30.0.5
- @google/genai ^1.2.0
- ws ^8.19.0
- electron-squirrel-startup ^1.0.1

**Development:**
- @electron-forge/cli ^7.8.1
- @electron-forge/maker-* ^7.8.1
- @electron/fuses ^1.8.0
- @reforged/maker-appimage ^5.0.0

**Frontend (via minified bundles):**
- LitElement 2.7.4
- Marked 4.3.0
- Highlight.js 11.9.0

---

## 🔒 Security

### Context Isolation
- **Current**: `contextIsolation: false` (for development)
- **Recommended**: Enable for production

### Secure IPC
- Validate and sanitize all IPC parameters
- Don't trust data from renderer process

### Data Protection
- Local processing when possible
- User control over data storage
- Clear disclosure of API key usage
- Emergency erase feature

### Permissions
- Screen recording (macOS)
- Microphone access
- User must explicitly configure API keys

### Code Signing
- macOS signing disabled in config
- Notarization disabled due to long processing times
- For distribution: Enable and configure osxSign and osxNotarize

---

## 🎨 UI Components

### Component Hierarchy

```
CheatingDaddyApp (Root)
├── Top Drag Bar
│   ├── Traffic Lights (Close/Minimize/Maximize)
│   └── Drag Region
├── Sidebar (Hidden in live mode)
│   ├── Brand
│   ├── Navigation (8 items)
│   └── Footer (Update button / Version)
└── Content Area
    ├── Live Bar (Assistant view only)
    │   ├── Back Button
    │   ├── Profile Label
    │   ├── Status Text
    │   ├── Elapsed Time
    │   └── Hide Button
    └── Current View
        └── MainView | AssistantView | CustomizeView | ...
```

### View Components

1. **MainView.js** - Start/stop controls, profile selection, API key config
2. **AssistantView.js** - Response display, markdown rendering, navigation
3. **CustomizeView.js** - Settings, preferences, configuration
4. **AICustomizeView.js** - AI customization, custom prompts
5. **HistoryView.js** - Session history, management
6. **HelpView.js** - Documentation
7. **FeedbackView.js** - User feedback
8. **OnboardingView.js** - First-time setup

---

## 🎯 Usage Patterns

### Starting a Session

1. User opens app → Loads preferences
2. Selects profile (default: interview)
3. Clicks "Start Session" (or Ctrl/Cmd + Enter)
4. App:
   - Initializes AI with profile
   - Starts audio capture
   - Creates new session
   - Switches to AssistantView
5. User speaks → Audio captured → Transcribed → Sent to AI
6. Response streams in real-time with markdown formatting

### Emergency Situation

1. User presses **Cmd/Ctrl + Shift + E**
2. App:
   - Hides window immediately
   - Closes active AI session
   - Clears sensitive data
   - Quits after 300ms

---

## 🐞 Debugging

### Log Files
- **Transport Logs**: `configDir/logs/{sessionId}.json`
- **Debug Audio**: `~/cheating-daddy-debug/` (PCM/WAV files)

### Console Output
- Main process: Node.js console
- Renderer process: Browser console
- Native processes: Piped to main process

### Tips
- Check IPC communication between processes
- Enable `saveDebugAudio()` for audio issues
- Verify API keys and network connectivity
- Test with different models

---

## 📊 Technical Specifications

### Platform Support

| Platform | Arch | Status | Notes |
|----------|------|--------|-------|
| macOS | arm64 | ✅ | Requires screen recording permission |
| macOS | x64 | ✅ | Requires screen recording permission |
| Windows | x64 | ✅ | Uses loopback audio |
| Linux | x64 | ⚠️ | Microphone only |

### Performance

| Resource | Minimum | Recommended (Local AI) |
|----------|---------|------------------------|
| RAM | 2GB | 8GB+ |
| CPU | Dual-core | Quad-core |
| Disk | 1GB | 10GB+ |

### Binary Sizes

| Component | Size |
|-----------|------|
| SystemAudioDump | ~5MB |
| llama-server | ~10MB |
| whisper-server | ~5MB |
| Whisper models | 75-469MB |
| LLM models | 2-10GB |

---

## 📜 License

**GPL-3.0** - GNU General Public License v3.0

---

## 🏁 Conclusion

Cheating Daddy is a **sophisticated Electron application** combining:
- ✅ Real-time audio and screen capture
- ✅ AI-powered contextual assistance
- ✅ Cross-platform support
- ✅ Multiple AI provider support
- ✅ Transparent overlay with click-through
- ✅ Session history and conversation tracking
- ✅ Comprehensive keyboard shortcuts

**Architecture:** Clean separation of concerns with modular utilities

**Future:** TypeScript migration, React adoption, improved local AI, better testing

---

## 📞 Links

- GitHub: [sohzm/cheating-daddy](https://github.com/sohzm/cheating-daddy)
- Website: [cheatingdaddy.com](https://cheatingdaddy.com)
- Google API Keys: [aistudio.google.com/apikey](https://aistudio.google.com/apikey)
- Groq API Keys: [console.groq.com/keys](https://console.groq.com/keys)

---

*Generated by Mistral Vibe*
*Last updated: 2026-09-19*
