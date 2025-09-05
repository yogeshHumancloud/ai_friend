# Flutter Caring AI App - Architecture Documentation

## Table of Contents
1. [System Overview](#system-overview)
2. [Application Architecture](#application-architecture)
3. [Voice Processing Pipeline](#voice-processing-pipeline)
4. [3D Avatar System](#3d-avatar-system)
5. [State Management](#state-management)
6. [Background Processing](#background-processing)
7. [Technology Stack](#technology-stack)
8. [Performance Considerations](#performance-considerations)

## System Overview

The Flutter Caring AI App is a voice-enabled mobile application featuring an always-listening wake word detection system and an interactive 3D avatar. The app provides seamless voice interaction with AI while maintaining efficient battery usage and responsive UI performance.

### Key Features
- **Always-listening wake word detection** ("Hey Caring")
- **Interactive 3D animated avatar** with emotional expressions
- **Real-time speech-to-text** and text-to-speech processing
- **Background voice processing** with minimal battery impact
- **Seamless conversation flow** with natural interactions

## Application Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                  FLUTTER APP                                    │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐            │
│  │   Main Screen   │    │  Avatar Screen  │    │ Settings Screen │            │
│  │                 │    │                 │    │                 │            │
│  │ • Chat Interface│◄──►│ • 3D Avatar     │    │ • Voice Config  │            │
│  │ • Voice Buttons │    │ • Animations    │    │ • Avatar Config │            │
│  │ • Status Icons  │    │ • Emotions      │    │ • Permissions   │            │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘            │
│           │                       │                       │                    │
│           └───────────────────────┼───────────────────────┘                    │
│                                   │                                            │
├───────────────────────────────────┼────────────────────────────────────────────┤
│                         PRESENTATION LAYER                                     │
├───────────────────────────────────┼────────────────────────────────────────────┤
│                                   │                                            │
│  ┌─────────────────┐    ┌─────────▼───────┐    ┌─────────────────┐            │
│  │  Voice Manager  │    │ Avatar Manager  │    │  Chat Manager   │            │
│  │                 │    │                 │    │                 │            │
│  │ • State Control │◄──►│ • Animation Ctrl│◄──►│ • Message Queue │            │
│  │ • UI Updates    │    │ • Emotion Ctrl  │    │ • Response Mgmt │            │
│  │ • Status Events │    │ • Visual State  │    │ • History Cache │            │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘            │
│           │                       │                       │                    │
├───────────┼───────────────────────┼───────────────────────┼────────────────────┤
│           │              BUSINESS LOGIC LAYER             │                    │
├───────────┼───────────────────────┼───────────────────────┼────────────────────┤
│           │                       │                       │                    │
│  ┌────────▼──────────┐   ┌────────▼─────────┐   ┌────────▼──────────┐         │
│  │ Voice Service      │   │ Avatar Service   │   │ Chat Service      │         │
│  │                    │   │                  │   │                   │         │
│  │ • Wake Word Detect │   │ • 3D Model Load  │   │ • API Communication│        │
│  │ • STT Processing   │   │ • Animation Mgmt │   │ • Message Processing│       │
│  │ • TTS Synthesis    │   │ • Emotion Mapping│   │ • Context Management│       │
│  │ • Audio Processing │   │ • Visual Updates │   │ • Response Handling│        │
│  │ • Continuous Mode  │   │ • WebView Bridge │   │ • Error Handling  │         │
│  └────────┬──────────┘   └────────┬─────────┘   └────────┬──────────┘         │
│           │                       │                       │                    │
├───────────┼───────────────────────┼───────────────────────┼────────────────────┤
│           │                    DATA LAYER                 │                    │
├───────────┼───────────────────────┼───────────────────────┼────────────────────┤
│           │                       │                       │                    │
│  ┌────────▼──────────┐   ┌────────▼─────────┐   ┌────────▼──────────┐         │
│  │ Audio Repository   │   │ Avatar Repository│   │ Message Repository│         │
│  │                    │   │                  │   │                   │         │
│  │ • Permission Mgmt  │   │ • Model Storage  │   │ • Local Storage   │         │
│  │ • Audio Config     │   │ • Asset Loading  │   │ • Cache Management│         │
│  │ • Stream Handling  │   │ • WebView Data   │   │ • Conversation Log│         │
│  │ • Background Mode  │   │ • Animation Data │   │ • User Preferences│         │
│  └────────┬──────────┘   └────────┬─────────┘   └────────┬──────────┘         │
│           │                       │                       │                    │
└───────────┼───────────────────────┼───────────────────────┼────────────────────┘
            │                       │                       │
┌───────────┼───────────────────────┼───────────────────────┼────────────────────┐
│           │                NATIVE PLUGINS LAYER          │                    │
├───────────┼───────────────────────┼───────────────────────┼────────────────────┤
│           │                       │                       │                    │
│  ┌────────▼──────────┐   ┌────────▼─────────┐   ┌────────▼──────────┐         │
│  │ Speech Plugins     │   │ 3D/WebView       │   │ Network Plugins   │         │
│  │                    │   │ Plugins          │   │                   │         │
│  │ • picovoice_flutter│   │ • flutter_inapp  │   │ • dio/http        │         │
│  │ • speech_to_text   │   │   _webview       │   │ • connectivity    │         │
│  │ • flutter_tts      │   │ • webview_flutter│   │ • shared_prefs    │         │
│  │ • permission_handler│  │                  │   │ • sqflite         │         │
│  │ • background_fetch │   │                  │   │                   │         │
│  └────────┬──────────┘   └────────┬─────────┘   └────────┬──────────┘         │
│           │                       │                       │                    │
└───────────┼───────────────────────┼───────────────────────┼────────────────────┘
            │                       │                       │
┌───────────┼───────────────────────┼───────────────────────┼────────────────────┐
│           │              EXTERNAL DEPENDENCIES           │                    │
├───────────┼───────────────────────┼───────────────────────┼────────────────────┤
│           │                       │                       │                    │
│  ┌────────▼──────────┐   ┌────────▼─────────┐   ┌────────▼──────────┐         │
│  │ Device Hardware    │   │ 3D Assets &      │   │ Backend Services  │         │
│  │                    │   │ Web Technologies │   │                   │         │
│  │ • Microphone       │   │ • Three.js       │   │ • FastAPI Server  │         │
│  │ • Speakers         │   │ • Ready Player Me│   │ • Chat API        │         │
│  │ • Audio Processing │   │ • GLTF Models    │   │ • Authentication  │         │
│  │ • Background Tasks │   │ • WebGL Rendering│   │ • Message Queue   │         │
│  │ • Push Notifications│  │ • CSS3 Animations│   │ • Database        │         │
│  └───────────────────┘   └──────────────────┘   └───────────────────┘         │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

## Voice Processing Pipeline

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                            VOICE PROCESSING PIPELINE                            │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│    ┌───────────────┐        ┌───────────────┐        ┌───────────────┐        │
│    │ Always Listen │        │ Wake Word     │        │ Full Speech   │        │
│    │ Mode          │───────►│ Detection     │───────►│ Recognition   │        │
│    │               │        │               │        │               │        │
│    │ • Low Power   │        │ • "Hey Caring"│        │ • Full STT    │        │
│    │ • Background  │        │ • Picovoice   │        │ • Context     │        │
│    │ • Continuous  │        │ • Local Only  │        │ • Intent      │        │
│    └───────────────┘        └───────────────┘        └───────────────┘        │
│            │                         │                         │               │
│            │                         │                         │               │
│            ▼                         ▼                         ▼               │
│    ┌───────────────┐        ┌───────────────┐        ┌───────────────┐        │
│    │ Microphone    │        │ Audio Buffer  │        │ Text Processing│        │
│    │ Monitoring    │        │ Analysis      │        │               │        │
│    │               │        │               │        │ • NLP         │        │
│    │ • 16kHz       │        │ • Pattern     │        │ • Sentiment   │        │
│    │ • Noise Filter│        │   Matching    │        │ • Intent      │        │
│    │ • VAD         │        │ • Confidence  │        │ • Context     │        │
│    └───────────────┘        └───────────────┘        └───────────────┘        │
│                                                               │               │
│                                                               ▼               │
│                              ┌─────────────────────────────────────────┐      │
│                              │         CONVERSATION ENGINE             │      │
│                              │                                         │      │
│                              │ • Message to Backend API                │      │
│                              │ • Context + History                     │      │
│                              │ • Response Generation                   │      │
│                              │ • Emotion Analysis                      │      │
│                              └─────────────────────────────────────────┘      │
│                                                               │               │
│                                                               ▼               │
│    ┌───────────────┐        ┌───────────────┐        ┌───────────────┐        │
│    │ Avatar Update │◄───────│ TTS Synthesis │◄───────│ Response      │        │
│    │               │        │               │        │ Processing    │        │
│    │ • Lip Sync    │        │ • Neural TTS  │        │               │        │
│    │ • Emotions    │        │ • Natural     │        │ • Text Clean  │        │
│    │ • Gestures    │        │ • SSML        │        │ • Emotion Tag │        │
│    │ • Eye Contact │        │ • Prosody     │        │ • Audio Cues  │        │
│    └───────────────┘        └───────────────┘        └───────────────┘        │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Voice Processing States

| State | Description | Power Usage | Features |
|-------|-------------|-------------|----------|
| **Always Listen** | Background wake word detection | Low | Continuous monitoring, pattern matching |
| **Wake Detected** | Transition to full recognition | Medium | Audio feedback, STT initialization |
| **Active Conversation** | Full voice interaction | High | Complete STT/TTS, context processing |
| **Response Generation** | AI processing and TTS | Medium | Backend communication, avatar animation |
| **Conversation End** | Return to wake mode | Low | Cleanup, state reset |

## 3D Avatar System

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                               3D AVATAR SYSTEM                                  │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐            │
│  │ Flutter Widget  │    │ WebView Bridge  │    │ Three.js Engine │            │
│  │                 │    │                 │    │                 │            │
│  │ • Avatar State  │◄──►│ • JS ↔ Dart     │◄──►│ • Scene Manager │            │
│  │ • Emotion API   │    │ • Method Calls  │    │ • Renderer      │            │
│  │ • Animation Ctrl│    │ • Event Passing │    │ • Asset Loader  │            │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘            │
│           │                       │                       │                    │
│           ▼                       ▼                       ▼                    │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐            │
│  │ State Manager   │    │ Communication   │    │ 3D Assets       │            │
│  │                 │    │ Layer           │    │                 │            │
│  │ • Speaking      │    │                 │    │ • GLTF Models   │            │
│  │ • Listening     │    │ • evaluateJS()  │    │ • Textures      │            │
│  │ • Emotions      │    │ • postMessage() │    │ • Animations    │            │
│  │ • Eye Contact   │    │ • onMessage()   │    │ • Morph Targets │            │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘            │
│                                                                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│                           AVATAR ANIMATION PIPELINE                             │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  Voice Input ──► ┌─────────────┐ ──► ┌─────────────┐ ──► ┌─────────────┐      │
│                  │ Emotion     │     │ Animation   │     │ Visual      │      │
│                  │ Detection   │     │ Selection   │     │ Rendering   │      │
│                  │             │     │             │     │             │      │
│                  │ • Sentiment │     │ • Lip Sync  │     │ • Mouth     │      │
│                  │ • Intent    │     │ • Gestures  │     │ • Eyes      │      │
│                  │ • Energy    │     │ • Poses     │     │ • Body      │      │
│                  └─────────────┘     └─────────────┘     └─────────────┘      │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                        ANIMATION STATES                                │  │
│  │                                                                         │  │
│  │  Idle ──► Listening ──► Speaking ──► Thinking ──► Responding ──► Idle  │  │
│  │   │         │            │           │             │            │      │  │
│  │   │         ▼            ▼           ▼             ▼            │      │  │
│  │   └─► Blinking ──► Eye Contact ──► Lip Sync ──► Gestures ───────┘      │  │
│  │                                                                         │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Avatar Animation System

#### Animation States
- **Idle**: Subtle breathing, blinking, occasional head movements
- **Listening**: Attentive pose, focused eye contact, responsive micro-expressions
- **Speaking**: Lip-sync animation, natural gestures, emotional expressions
- **Thinking**: Contemplative pose, slower blinking, slight head tilts
- **Responding**: Transition animations, preparation for speech

#### Emotion Mapping
| Detected Emotion | Avatar Response | Visual Cues |
|------------------|----------------|-------------|
| **Happy** | Smile, bright eyes | Mouth curve, eye crinkle |
| **Sad** | Downturned mouth, soft eyes | Slower movements, head tilt |
| **Excited** | Wide smile, energetic | Quick gestures, upright posture |
| **Concerned** | Furrowed brow, gentle | Slower speech, forward lean |
| **Neutral** | Natural expression | Balanced features, calm pose |

## State Management

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              APP STATE FLOW                                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────┐         ┌─────────────────┐         ┌─────────────────┐   │
│  │ App Launch      │────────►│ Permissions     │────────►│ Voice Init      │   │
│  │                 │         │                 │         │                 │   │
│  │ • Splash Screen │         │ • Microphone    │         │ • Wake Word     │   │
│  │ • Dependencies  │         │ • Audio Output  │         │ • STT Setup     │   │
│  │ • Asset Loading │         │ • Background    │         │ • TTS Setup     │   │
│  └─────────────────┘         └─────────────────┘         └─────────────────┘   │
│                                                                   │             │
│                                                                   ▼             │
│  ┌─────────────────┐         ┌─────────────────┐         ┌─────────────────┐   │
│  │ Avatar Loading  │◄────────│ Always Listen   │◄────────│ Ready State     │   │
│  │                 │         │                 │         │                 │   │
│  │ • 3D Model      │         │ • Background    │         │ • All Services  │   │
│  │ • Animations    │         │ • Wake Word     │         │ • Avatar Ready  │   │
│  │ • WebView Setup │         │ • Low Power     │         │ • UI Active     │   │
│  └─────────────────┘         └─────────────────┘         └─────────────────┘   │
│           │                           │                           │             │
│           ▼                           ▼                           ▼             │
│  ┌─────────────────┐         ┌─────────────────┐         ┌─────────────────┐   │
│  │ Wake Detected   │────────►│ Conversation    │────────►│ Response        │   │
│  │                 │         │                 │         │                 │   │
│  │ • Stop Wake     │         │ • Speech-to-Text│         │ • TTS Output    │   │
│  │ • Start STT     │         │ • Send to API   │         │ • Avatar Speak  │   │
│  │ • Avatar Alert  │         │ • Show Thinking │         │ • Lip Sync      │   │
│  └─────────────────┘         └─────────────────┘         └─────────────────┘   │
│                                       │                           │             │
│                                       ▼                           ▼             │
│                               ┌─────────────────┐         ┌─────────────────┐   │
│                               │ End Conversation│◄────────│ Continue or End │   │
│                               │                 │         │                 │   │
│                               │ • Timeout       │         │ • User Choice   │   │
│                               │ • Goodbye       │         │ • Auto Timeout  │   │
│                               │ • Return Wake   │         │ • Context Check │   │
│                               └─────────────────┘         └─────────────────┘   │
│                                       │                                         │
│                                       ▼                                         │
│                               ┌─────────────────┐                               │
│                               │ Back to Always  │                               │
│                               │ Listen Mode     │                               │
│                               │                 │                               │
│                               │ • Reset State   │                               │
│                               │ • Avatar Idle   │                               │
│                               │ • Wake Detection│                               │
│                               └─────────────────┘                               │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### State Management Pattern

The app uses a combination of:
- **Provider/Riverpod** for global state management
- **BLoC** for complex voice processing logic
- **StatefulWidget** for local UI state
- **Stream Controllers** for real-time audio processing

## Background Processing

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          BACKGROUND PROCESSING                                  │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐            │
│  │ App Foreground  │    │ App Background  │    │ App Terminated  │            │
│  │                 │    │                 │    │                 │            │
│  │ • Full Features │    │ • Wake Word Only│    │ • Push Notifications│        │
│  │ • 3D Avatar     │    │ • No 3D Render  │    │ • Schedule Wake │            │
│  │ • All Services  │    │ • Audio Only    │    │ • Minimal State│            │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘            │
│           │                       │                       │                    │
│           ▼                       ▼                       ▼                    │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐            │
│  │ Full Processing │    │ Limited Processing│   │ Push Notifications│          │
│  │                 │    │                 │    │                 │            │
│  │ • 60fps Avatar  │    │ • Audio Only    │    │ • Remote Wake   │            │
│  │ • Real-time STT │    │ • Simple Voice  │    │ • App Resume    │            │
│  │ • Rich UI       │    │ • Background STT│    │ • State Restore │            │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘            │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Background Processing Strategy

#### Foreground Mode
- Full 3D avatar rendering at 60fps
- Complete voice processing pipeline
- Rich UI interactions and animations
- Real-time response to all voice commands

#### Background Mode
- Wake word detection only
- Minimal CPU usage for battery preservation
- Audio-only processing (no visual rendering)
- Quick transition to foreground when activated

#### Terminated State
- Push notification system for remote wake
- Scheduled background tasks for state preservation
- Quick app resume with state restoration

## Technology Stack

### Flutter Dependencies

```yaml
dependencies:
  # Core Flutter
  flutter:
    sdk: flutter
  
  # Voice Processing
  picovoice_flutter: ^3.0.1          # Wake word detection
  speech_to_text: ^6.6.0             # Speech recognition
  flutter_tts: ^3.8.3                # Text-to-speech
  permission_handler: ^11.0.1         # Audio permissions
  
  # 3D Avatar & WebView
  flutter_inappwebview: ^6.0.0        # WebView for 3D rendering
  webview_flutter: ^4.4.2             # Alternative WebView
  
  # State Management
  provider: ^6.1.1                    # State management
  riverpod: ^2.4.9                    # Advanced state management
  flutter_bloc: ^8.1.3                # BLoC pattern
  
  # Networking & Storage
  dio: ^5.3.4                         # HTTP client
  shared_preferences: ^2.2.2          # Local storage
  sqflite: ^2.3.0                     # Local database
  hive: ^2.2.3                        # NoSQL database
  
  # Background Processing
  background_fetch: ^1.3.4            # Background tasks
  workmanager: ^0.5.2                 # Background work
  
  # UI & Animations
  animations: ^2.0.11                 # Flutter animations
  lottie: ^2.7.0                      # Lottie animations
  audio_waveforms: ^1.0.5             # Audio visualization
  
  # Utilities
  connectivity_plus: ^5.0.2           # Network connectivity
  device_info_plus: ^9.1.1            # Device information
  package_info_plus: ^4.2.0           # App information
```

### Web Technologies (for 3D Avatar)

```javascript
// Core 3D Libraries
"three": "^0.158.0"                   // 3D rendering engine
"@types/three": "^0.158.0"            // TypeScript definitions

// Model Loading
"three/examples/jsm/loaders/GLTFLoader.js"  // GLTF model loader
"three/examples/jsm/loaders/DRACOLoader.js" // Compressed models

// Animation & Controls
"three/examples/jsm/controls/OrbitControls.js"  // Camera controls
"three/examples/jsm/utils/SkeletonUtils.js"     // Skeleton utilities

// Avatar Services
"@readyplayerme/core": "^1.4.0"       // Ready Player Me integration
"@tensorflow/tfjs": "^4.15.0"         // ML for emotion detection
```

### Backend Integration

```yaml
# API Endpoints
base_url: "https://your-api-domain.com/api/v1"

endpoints:
  chat: "/chat/message"
  auth: "/auth/login"
  user: "/user/profile"
  avatar: "/avatar/config"
  voice: "/voice/process"
```

## Performance Considerations

### Memory Management
- **Avatar Models**: Load optimized GLTF models (< 5MB)
- **Audio Buffers**: Efficient circular buffer management
- **State Cleanup**: Proper disposal of streams and controllers
- **Texture Optimization**: Compressed textures for mobile GPUs

### Battery Optimization
- **Wake Word Detection**: Use hardware-accelerated DSP when available
- **Background Processing**: Minimal CPU usage in background mode
- **Screen Updates**: Reduce animation frame rate when not active
- **Network Requests**: Batch API calls and implement smart caching

### Latency Optimization
- **Audio Processing**: < 100ms for wake word detection
- **Speech Recognition**: < 200ms for STT initialization
- **Avatar Response**: < 50ms for animation triggers
- **Network Requests**: Connection pooling and request prioritization

### Cross-Platform Considerations
- **iOS**: Background audio permissions and App Store guidelines
- **Android**: Background service limitations and battery optimization
- **Hardware Variations**: Adaptive quality based on device capabilities
- **Permissions**: Platform-specific permission handling

## Security & Privacy

### Data Protection
- **Local Processing**: Wake word detection happens locally
- **Encrypted Storage**: Secure storage for user preferences
- **Network Security**: TLS 1.3 for all API communications
- **Audio Privacy**: No audio stored without explicit consent

### Permissions Management
- **Microphone Access**: Clear explanation and granular control
- **Background Processing**: User-configurable background behavior
- **Data Collection**: Transparent data usage policies
- **User Control**: Easy opt-out mechanisms

## Deployment Architecture

### Development Environment
```yaml
# Local development setup
- Flutter SDK: >= 3.16.0
- Dart SDK: >= 3.2.0
- Android Studio / VS Code
- iOS Simulator / Android Emulator
- Local backend server for testing
```

### Production Environment
```yaml
# App distribution
- iOS App Store
- Google Play Store
- Flutter web (optional)

# Backend services
- Cloud hosting (AWS/GCP/Azure)
- CDN for 3D assets
- Load balancing for API endpoints
- Monitoring and analytics
```

## Future Enhancements

### Planned Features
- **Multi-language Support**: Wake words in multiple languages
- **Custom Avatar Creation**: User-generated avatar models
- **Emotion Recognition**: Real-time facial emotion detection
- **Gesture Control**: Hand gesture recognition for avatar control
- **AR Integration**: Augmented reality avatar placement

### Technical Improvements
- **Performance Optimization**: GPU acceleration for audio processing
- **Model Compression**: Smaller, faster 3D models
- **Offline Mode**: Fully offline voice processing
- **Cloud Sync**: Multi-device conversation synchronization

---

## Getting Started

1. **Clone the repository**
2. **Install Flutter dependencies**: `flutter pub get`
3. **Configure environment variables** for API endpoints
4. **Set up device permissions** for microphone and background processing
5. **Run the app**: `flutter run`

For detailed setup instructions, see the [Installation Guide](./docs/installation.md).

For API documentation, see the [API Reference](./docs/api-reference.md).
