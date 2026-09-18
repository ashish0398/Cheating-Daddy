# Cheating Daddy - Self-Deployment Guide with Updated Gemini Engine

This guide will help you deploy your own version of Cheating Daddy with the latest Google Gemini engine.

---

## 📋 Prerequisites

### 1. System Requirements

| Requirement | Minimum | Recommended |
|-------------|---------|-------------|
| **Node.js** | v18+ | v20+ |
| **npm** | v9+ | v10+ |
| **OS** | Windows 10+, macOS 12+, Linux | Latest |
| **RAM** | 4GB | 8GB+ |
| **Disk** | 2GB | 10GB+ (for local AI models) |

### 2. Google Gemini API Key

You need a **Google Gemini API key**:
- Go to: [Google AI Studio - API Keys](https://aistudio.google.com/apikey)
- Create a new API key
- **Important**: Enable billing for your Google Cloud project (free tier has limits)
- Recommended models: `gemini-3-flash-live` (latest)

### 3. Git (Optional)

If you want to track changes:
```bash
# Clone your fork (optional)
git clone https://github.com/your-username/cheating-daddy.git
cd cheating-daddy
```

---

## 🚀 Step-by-Step Deployment

### Step 1: Update Package Dependencies

The package.json has been updated to use `@google/genai@^2.23.0`. Install dependencies:

```bash
cd /path/to/cheating-daddy

# Install all dependencies (including updated @google/genai)
npm install
```

**What was updated:**
- `@google/genai`: `^1.2.0` → `^2.23.0` (latest)
- Default model: `gemini-3.1-flash-live-preview` → `gemini-3-flash-live`
- Rate limiting: Updated to track new model names

### Step 2: Configure Your API Key

After installing, you need to configure your API key. You can do this in two ways:

#### Option A: Via the UI
1. Run the app: `npm start`
2. Open the app
3. Go to Settings
4. Enter your Google Gemini API key
5. Save

#### Option B: Directly in Storage

The app stores configuration in OS-specific directories. You can manually edit the config:

**Config File Location:**
- **Windows**: `%APPDATA%\cheating-daddy-config\credentials.json`
- **macOS**: `~/Library/Application Support/cheating-daddy-config/credentials.json`
- **Linux**: `~/.config/cheating-daddy-config/credentials.json`

**Edit credentials.json:**
```json
{
  "apiKey": "YOUR_GOOGLE_GEMINI_API_KEY",
  "groqApiKey": ""  // Optional: Add if using Groq
}
```

**Edit config.json:**
```json
{
  "configVersion": 1,
  "onboarded": true,
  "layout": "normal",
  "geminiLiveModel": "gemini-3-flash-live",
  "groqModel": "qwen/qwen3.6-27b",
  "groqImageModel": "qwen/qwen3.6-27b",
  "disableGroqThinking": true
}
```

### Step 3: Update Model Configuration (Optional)

You can use different Gemini models. Update in `src/storage.js`:

```javascript
// Line 12: Change the default model
 geminiLiveModel: 'gemini-3-flash-live',  // Latest model

// Other available models (as of Sept 2026):
// - gemini-3-flash-live
// - gemini-3-flash
// - gemini-3-pro
// - gemini-2.5-flash
// - gemini-2.5-pro
```

### Step 4: Test the Application

```bash
# Run in development mode
npm start
```

**Test Checklist:**
- [ ] Application starts without errors
- [ ] Window appears correctly
- [ ] Can enter API key
- [ ] Can select profile (Interview, Sales, etc.)
- [ ] Can start a session
- [ ] Audio capture works (check microphone permissions)
- [ ] AI responds to voice input
- [ ] Responses appear in the UI

### Step 5: Build for Production

Once testing is successful, build the application:

```bash
# Package the application (creates unpacked app in out/ directory)
npm run package

# OR create distributables for your platform
npm run make
```

**Platform-Specific Outputs:**
- **Windows**: `out/cheating-daddy-win32-x64/` (Squirrel installer)
- **macOS**: `out/Cheating Daddy-darwin-x64/` (DMG)
- **Linux**: `out/Cheating Daddy-linux-x64/` (AppImage)

---

## 🔧 Configuration Options

### Available Gemini Models

| Model | Speed | Cost | Use Case |
|-------|-------|------|----------|
| `gemini-3-flash-live` | ⚡⚡⚡ | $ | Real-time audio, fastest |
| `gemini-3-flash` | ⚡⚡ | $ | General purpose, fast |
| `gemini-3-pro` | ⚡ | $$ | More capable, slower |
| `gemini-2.5-flash` | ⚡⚡⚡ | $ | Legacy, still good |
| `gemini-2.5-pro` | ⚡ | $$ | Legacy, more capable |

### Update Model in Code

**File: `src/storage.js`** (Line 12)
```javascript
geminiLiveModel: 'gemini-3-flash-live',  // Change this
```

**File: `src/utils/gemini.js`** (Line 666)
```javascript
model: getConfig().geminiLiveModel,  // Uses the configured model
```

### Provider Modes

The app supports 4 AI provider modes:

1. **BYOK (Bring Your Own Key)** - Default
   - Uses your Google Gemini API key
   - Real-time audio streaming
   - Best for most users

2. **Cloud API**
   - Uses hosted service at `wss://api.cheatingdaddy.com/ws`
   - Requires token from the service
   - No API key needed

3. **Groq**
   - Uses Groq API
   - Faster inference
   - Different models available

4. **Local AI**
   - Offline processing
   - Requires model downloads
   - Uses Llama.cpp + Whisper.cpp

**Configure provider mode in preferences.json:**
```json
{
  "providerMode": "byok"  // "byok", "cloud", "groq", or "local"
}
```

---

## 🎛️ Customization Options

### 1. Profiles

6 specialized profiles are available:
- **interview** - Job interview assistance
- **sales** - Sales call support
- **meeting** - Meeting assistance
- **presentation** - Presentation coaching
- **negotiation** - Negotiation help
- **exam** - Exam/test assistance

**Customize prompts:**
- Go to AI Customization view in the app
- Edit the custom prompt for each profile
- Save changes

### 2. Keyboard Shortcuts

**Default Shortcuts:**

| Action | macOS | Windows/Linux |
|--------|-------|---------------|
| Move Window | Alt+Arrow | Ctrl+Arrow |
| Toggle Visibility | Cmd+\ | Ctrl+\ |
| Toggle Click-Through | Cmd+M | Ctrl+M |
| Start Session | Cmd+Enter | Ctrl+Enter |
| Emergency Erase | Cmd+Shift+E | Ctrl+Shift+E |

**Customize shortcuts in keybinds.json:**
```json
{
  "moveUp": "Alt+Up",
  "moveDown": "Alt+Down",
  "toggleVisibility": "Cmd+\\",
  "toggleClickThrough": "Cmd+M"
}
```

### 3. Audio Settings

**VAD (Voice Activity Detection) Modes:**
- NORMAL
- LOW_BITRATE
- AGGRESSIVE
- VERY_AGGRESSIVE (default)

**Screenshot Settings:**
- Interval: 5 seconds (default)
- Quality: low, medium, high

---

## 🐛 Troubleshooting

### Common Issues

#### 1. "Cannot find module '@google/genai'"

**Solution:** Run `npm install` to install dependencies.

```bash
npm install
```

#### 2. "Invalid API key"

**Solution:** Verify your Google Gemini API key:
- Go to [Google AI Studio](https://aistudio.google.com/apikey)
- Check the key is valid
- Ensure billing is enabled
- Wait a few minutes after creating the key

#### 3. "Microphone permission denied"

**macOS:**
- Go to System Settings → Privacy & Security → Microphone
- Enable for Electron or the app

**Windows:**
- Check microphone privacy settings
- Allow app to use microphone

**Browser-based:** Not applicable (Electron desktop app)

#### 4. "Screen recording permission denied" (macOS only)

**Solution:**
- The app requests this on startup
- Click "Allow" when prompted
- Or manually enable in:
  - System Settings → Privacy & Security → Screen Recording
  - Enable for Electron

#### 5. "Audio not capturing"

**Check:**
- Microphone is working (test with other apps)
- Correct input device selected
- Permissions granted
- VAD settings (try different modes)

#### 6. "Model not found"

**Solution:** Update the model name in `src/storage.js`:
```javascript
geminiLiveModel: 'gemini-3-flash-live',  // Use exact model name
```

Check latest models at: [Google AI Model Garden](https://ai.google.com/gemini-api/docs/models)

### Debug Mode

Enable debug logging:

```bash
# Run with debug output
DEBUG=cheating-daddy:* npm start
```

**Debug files:**
- Transport logs: `{configDir}/logs/{sessionId}.json`
- Debug audio: `~/cheating-daddy-debug/`

---

## 📦 Building for Distribution

### 1. Install Build Tools

```bash
# Install Electron Forge (if not already installed)
npm install --global electron-forge
```

### 2. Build Commands

```bash
# Development mode
npm start

# Package (unpacked)
npm run package

# Make distributables (installers)
npm run make

# Publish to GitHub releases (if configured)
npm run publish
```

### 3. Platform-Specific Notes

#### Windows
- Output: Squirrel-based installer
- Creates desktop shortcut by default
- May need code signing for distribution

#### macOS
- Output: DMG file
- Requires code signing for Gatekeeper
- Notarization recommended for distribution

**Enable signing in forge.config.js:**
```javascript
packagerConfig: {
    osxSign: {
        identity: 'Your Apple Developer ID',
        optionsForFile: (filePath) => ({
            entitlements: 'entitlements.plist'
        })
    },
    osxNotarize: {
        appleId: 'your@apple.id',
        appleIdPassword: 'app-specific-password',
        teamId: 'your-team-id'
    }
}
```

#### Linux
- Output: AppImage
- Portable, no installation needed
- Works on most modern distributions

---

## 🔒 Security Best Practices

### 1. Protect Your API Key
- Never commit API keys to version control
- Use environment variables for development
- The app stores keys locally in encrypted form

### 2. Code Signing
- **macOS**: Required for distribution without warnings
- **Windows**: Recommended for better user experience
- **Linux**: Optional but recommended

### 3. Enable Context Isolation (Recommended)

**File: `src/utils/window.js`** (Line 26)
```javascript
webPreferences: {
    nodeIntegration: true,
    contextIsolation: true,  // Change from false to true
    backgroundThrottling: false,
    enableBlinkFeatures: 'GetDisplayMedia',
    webSecurity: true,
    allowRunningInsecureContent: false,
}
```

### 4. Update Securely
- Always verify package checksums
- Review dependency changes
- Test thoroughly before deployment

---

## 📊 Performance Optimization

### For Local AI Mode

| Component | Recommended |
|-----------|-------------|
| RAM | 8GB+ |
| CPU | Quad-core |
| Disk | 10GB+ for models |

**Model Size Estimates:**
- Whisper tiny.en: 75MB
- Whisper base.en: 142MB
- Whisper small.en: 469MB
- LLM models: 2-10GB

### For Cloud/BYOK Mode

| Requirement | Recommended |
|-------------|-------------|
| Internet | Stable, low latency |
| Bandwidth | 1Mbps+ |
| API Key | Paid tier for production |

---

## 🎯 Deployment Checklist

- [ ] Node.js v18+ installed
- [ ] npm installed
- [ ] Google Gemini API key created
- [ ] Billing enabled for Google Cloud
- [ ] Dependencies installed (`npm install`)
- [ ] API key configured
- [ ] Model selected (gemini-3-flash-live)
- [ ] App tested in development mode
- [ ] Microphone permissions granted
- [ ] Screen recording permissions granted (macOS)
- [ ] Application starts without errors
- [ ] AI responds correctly
- [ ] Audio capture working
- [ ] UI displays properly
- [ ] Built for production (`npm run make`)
- [ ] Distributable tested

---

## 📚 Additional Resources

### Google Gemini
- [API Keys](https://aistudio.google.com/apikey)
- [Documentation](https://ai.google.com/gemini-api/docs)
- [Models](https://ai.google.com/gemini-api/docs/models)
- [Pricing](https://ai.google.com/pricing)

### Electron
- [Electron Documentation](https://www.electronjs.org/docs)
- [Electron Forge](https://www.electronforge.io/)

### Troubleshooting
- [GitHub Issues](https://github.com/sohzm/cheating-daddy/issues)
- [Discussions](https://github.com/sohzm/cheating-daddy/discussions)

---

## 🏁 Quick Start (Summary)

```bash
# 1. Install dependencies
cd cheating-daddy
npm install

# 2. Get API key from https://aistudio.google.com/apikey
# 3. Run in development
npm start

# 4. Configure API key in the app UI
# 5. Start a session and test

# 6. Build for production
npm run make

# 7. Distribute the output from out/ directory
```

---

## 📞 Support

If you encounter issues:

1. **Check this guide** - Most common issues are covered
2. **Check the console** - Look for error messages
3. **Check debug logs** - In config directory
4. **Update dependencies** - Run `npm install` again
5. **Verify API key** - Test with curl or Postman
6. **Ask for help** - Open an issue on GitHub

---

**Good luck with your deployment!** 🚀

*Generated for personal deployment*
*Last updated: 2026-09-19*
