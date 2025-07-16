# Voice Quiz Testing Guide

## 🚀 Quick Start - Local Server

### Option 1: Simple HTTP Server (Recommended)
```bash
npm run dev
```
- Opens at: http://localhost:3000
- Best for: General testing and development
- Features: CORS enabled, no caching

### Option 2: Production-like Server
```bash
npm start
```
- Opens at: http://localhost:8080
- Best for: Production testing
- Features: Standard port, CORS enabled

### Option 3: HTTPS Server (for secure testing)
```bash
npm run dev-https
```
- Opens at: https://localhost:3000
- Best for: Testing secure features (microphone permissions)
- Note: Requires SSL certificates (see setup below)

## 🔧 Manual Server Options

### Using Python (if installed)
```bash
# Python 3
python -m http.server 3000

# Python 2
python -m SimpleHTTPServer 3000
```

### Using Node.js http-server directly
```bash
npx http-server . -p 3000 -c-1 --cors
```

### Using Live Server (VS Code extension)
1. Install "Live Server" extension in VS Code
2. Right-click on `index.html`
3. Select "Open with Live Server"

## 🧪 Testing Scenarios

### 1. Basic Functionality Test
1. Open http://localhost:3000
2. Click "Test Microphone" on landing page
3. Allow microphone permissions
4. Verify microphone status shows "✅ Microphone ready"
5. Click "Start Study"

### 2. Full Study Flow Test
1. Complete registration form
2. Record answers to all 7 questions
3. Verify audio processing between questions
4. Test version comparison (A, B, C, D)
5. Complete the study

### 3. Audio Processing Verification
1. Open browser console (F12)
2. Run: `testAudioProcessing()`
3. Check console for processing differences
4. Verify all 4 versions have different audio characteristics

### 4. Memory Management Test
1. Complete multiple questions
2. Check console for memory cleanup logs
3. Monitor browser memory usage
4. Verify no audio accumulation

## 🔍 Console Testing Commands

### Test Audio Processing
```javascript
// Test the complete processing pipeline
testAudioProcessing()

// Verify processing differences
audioProcessor.verifyProcessingDifferences(audioBlob)

// Check Superpowered initialization
audioProcessor.exploreSuperpoweredAPI()

// Test memory management
audioProcessor.cleanup()
```

### Test Individual Components
```javascript
// Test microphone access
navigator.mediaDevices.getUserMedia({ audio: true })

// Test audio recording
const recorder = new MediaRecorder(stream)
recorder.start()

// Test audio playback
const audio = new Audio()
audio.src = URL.createObjectURL(blob)
audio.play()
```

## 🛠️ Troubleshooting

### Microphone Issues
- **Permission denied**: Check browser settings
- **No audio input**: Test microphone in other apps
- **Chrome issues**: Try `chrome://settings/content/microphone`

### Audio Processing Issues
- **Superpowered not loading**: Check network connection
- **Processing fails**: Check console for errors
- **No differences**: Run `testAudioProcessing()` to verify

### Server Issues
- **Port already in use**: Try different port (3001, 8080, etc.)
- **CORS errors**: Ensure `--cors` flag is used
- **HTTPS required**: Use `npm run dev-https`

## 📱 Device Testing

### Desktop Testing
- Chrome (recommended)
- Firefox
- Safari
- Edge

### Mobile Testing
- iOS Safari
- Android Chrome
- Test responsive design
- Verify touch interactions

### Browser Compatibility
- Modern browsers (ES6+ support)
- Web Audio API support
- MediaRecorder API support
- Superpowered SDK compatibility

## 🔒 Security Testing

### HTTPS Setup (Optional)
```bash
# Generate self-signed certificates
openssl req -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout key.pem -out cert.pem

# Run HTTPS server
npm run dev-https
```

### Privacy Testing
- Verify audio data is not stored
- Check network requests
- Test data deletion
- Verify consent flow

## 📊 Performance Testing

### Memory Usage
- Monitor browser memory
- Check for memory leaks
- Verify cleanup between questions
- Test with multiple recordings

### Audio Quality
- Test different microphone qualities
- Verify processing maintains quality
- Check for audio artifacts
- Test with various voice types

## 🎯 Research Study Testing

### Participant Flow
1. **Landing Page**: Clear instructions
2. **Registration**: Privacy consent
3. **Recording**: Natural voice capture
4. **Processing**: Audio enhancement
5. **Comparison**: Version selection
6. **Completion**: Data collection

### Data Collection
- Verify preference data is captured
- Test participant tracking
- Check for duplicate submissions
- Validate data format

## 🚨 Common Issues & Solutions

### "Superpowered not initialized"
- Check internet connection
- Verify Superpowered SDK loading
- Try refreshing the page

### "Audio processing failed"
- Check browser console for errors
- Verify audio input quality
- Test with different audio length

### "Memory issues"
- Check for audio blob accumulation
- Verify cleanup is working
- Monitor browser memory usage

### "Version differences not audible"
- Run `testAudioProcessing()`
- Check processing configurations
- Verify Superpowered processing

## 📞 Support

If you encounter issues:
1. Check browser console for errors
2. Verify all dependencies are installed
3. Test with different browsers
4. Check network connectivity
5. Review this testing guide

---

**Happy Testing! 🎙️✨** 