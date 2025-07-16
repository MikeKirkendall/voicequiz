# Voice Quiz App - Memory Management Guide

## Overview
This document outlines the comprehensive memory management system implemented for the voice quiz research study to ensure no audio accumulation between questions and proper resource cleanup.

## Research Study Flow
```
Per Question (7 total):
1. Record 1 answer (user's voice responding to question)
2. Create 4 versions from that same recording:
   - Version A: Raw recording (baseline)
   - Version B: Raw + Superpowered processing (-65¢)
   - Version C: Raw + Superpowered processing (-125¢) 
   - Version D: Raw + Superpowered processing (-185¢)
3. User compares all 4 versions of their same answer
4. User selects preferred version + adds feedback tags/comments
5. Save only the selection data (version choice + feedback)
6. DELETE all audio (original + all 4 processed versions)
7. Move to next question with clean slate
```

## Memory Management System

### 1. AudioContext Management
- **Single AudioContext**: One AudioContext created at app initialization and reused for entire study
- **No Recreation**: AudioContext is never recreated between questions
- **Resource Efficiency**: AudioContext suspended when tab is hidden, resumed when visible
- **Cleanup**: AudioContext only closed when user leaves the page

### 2. Audio URL Tracking
- **Tracked URLs**: All `URL.createObjectURL()` calls tracked in `audioUrls` Set
- **Automatic Cleanup**: URLs automatically revoked when audio ends or is stopped
- **Memory Leak Prevention**: Comprehensive URL cleanup between questions

### 3. Audio Blob Tracking
- **Tracked Blobs**: All audio blobs tracked in `audioBlobs` Set
- **Garbage Collection**: Blobs marked for cleanup when no longer needed
- **Reference Management**: Clear all blob references between questions

### 4. Question Transition Cleanup
```javascript
async completeQuestionCleanup() {
    // 1. Stop all audio playback
    // 2. Delete original recording
    // 3. Delete processed versions
    // 4. Clean up all tracked audio URLs
    // 5. Clean up all tracked audio blobs
    // 6. Reset audio state
    // 7. Force garbage collection
}
```

### 5. Memory Monitoring
- **Real-time Tracking**: Memory status logged at key points
- **Verification**: Cleanup verification after each question
- **Performance Monitoring**: Browser memory usage tracking when available

## Key Methods

### Memory Management Methods
- `completeQuestionCleanup()`: Comprehensive cleanup between questions
- `cleanupAllAudioUrls()`: Revoke all tracked audio URLs
- `cleanupAllAudioBlobs()`: Clear all tracked audio blobs
- `resetAudioState()`: Reset all audio-related properties
- `forceGarbageCollection()`: Force garbage collection if available

### URL Tracking Methods
- `createTrackedAudioUrl(blob)`: Create and track audio URL
- `revokeTrackedAudioUrl(url)`: Revoke and untrack audio URL

### Monitoring Methods
- `logMemoryStatus(location)`: Log current memory state
- `verifyMemoryCleanup()`: Verify cleanup was successful

## Expected Resource Usage

### Per Question
- **1 AudioContext**: Reused across all questions
- **1 Original Recording**: Deleted after processing
- **4 Processed Versions**: Deleted after selection
- **0 Audio Files**: Stored between questions
- **Only Preference Data**: Accumulated (7 selections + feedback)

### Memory Reset Between Questions
- ✅ All audio URLs revoked
- ✅ All audio blobs cleared
- ✅ Original recording deleted
- ✅ Processed versions deleted
- ✅ Audio state reset
- ✅ Garbage collection forced

## Implementation Details

### Constructor Initialization
```javascript
// Memory management tracking
this.audioUrls = new Set(); // Track all created URLs for cleanup
this.audioBlobs = new Set(); // Track all audio blobs for cleanup
```

### Audio Playback with Tracking
```javascript
// Create audio with tracked URL management
const audioUrl = this.createTrackedAudioUrl(this.processedVersions[version]);
const audio = new Audio(audioUrl);

// Clean up when audio ends
audio.addEventListener('ended', () => {
    this.revokeTrackedAudioUrl(audioUrl);
    this.currentlyPlayingAudio = null;
});
```

### Question Transition
```javascript
async nextQuestion() {
    // Record response data only
    await window.userManager.recordResponse(...);
    
    // COMPREHENSIVE MEMORY CLEANUP: Delete ALL audio data
    await this.completeQuestionCleanup();
    
    // Move to next question with clean slate
    this.currentQuestion++;
}
```

## Verification Points

### Console Logs to Monitor
- `🧹 Starting complete question cleanup...`
- `📊 Memory Status [Before Cleanup]:`
- `📊 Memory Status [After Cleanup]:`
- `✅ Memory cleanup verified - all audio resources released`
- `✅ Complete question cleanup finished`

### Memory Status Logs
```
📊 Memory Status [Location]:
   - Tracked URLs: 0
   - Tracked Blobs: 0
   - Current Recording: No
   - Processed Versions: 0
   - Currently Playing: No
   - Current Audio URL: No
   - Memory Usage: XMB / YMB
   - Question: N/7
```

## Testing Checklist

### Before Each Question
- [ ] Memory status shows 0 tracked URLs and blobs
- [ ] No current recording or processed versions
- [ ] No audio currently playing
- [ ] Memory usage stable (not growing)

### After Each Question
- [ ] Complete cleanup executed
- [ ] Memory verification passed
- [ ] All audio resources released
- [ ] Ready for next question with clean slate

### End of Study
- [ ] All 7 questions completed
- [ ] Only preference data accumulated
- [ ] No audio files remaining
- [ ] Memory usage returned to baseline

## Troubleshooting

### Memory Leaks Detected
If `verifyMemoryCleanup()` reports issues:
1. Check for audio elements not properly stopped
2. Verify all event listeners are removed
3. Ensure all URLs are properly revoked
4. Check for circular references

### AudioContext Issues
If AudioContext becomes unusable:
1. Check browser console for errors
2. Verify AudioContext state is 'running'
3. Resume AudioContext if suspended
4. Only recreate AudioContext as last resort

### Performance Issues
If memory usage grows unexpectedly:
1. Monitor memory status logs
2. Check for untracked audio URLs
3. Verify garbage collection is working
4. Check browser memory profiler

## Best Practices

1. **Always use tracked URL methods**: `createTrackedAudioUrl()` and `revokeTrackedAudioUrl()`
2. **Call cleanup after each question**: `completeQuestionCleanup()`
3. **Monitor memory status**: Use `logMemoryStatus()` at key points
4. **Verify cleanup**: Use `verifyMemoryCleanup()` after transitions
5. **Reuse AudioContext**: Never create new AudioContext between questions
6. **Handle page visibility**: Suspend/resume AudioContext appropriately

This memory management system ensures the voice quiz app maintains optimal performance and privacy compliance throughout the research study. 