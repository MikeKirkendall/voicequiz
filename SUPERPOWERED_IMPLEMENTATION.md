# Superpowered SDK Web Integration Guide

This unified guide replaces and corrects the previous three documents, preserving your research context while fixing import patterns, code samples, and integration strategies.

---

## 1. Research Context & Objectives

* **Study:** Bone‐conduction perception differences in self‐voice playback
* **Versions:** A/B/C/D with escalating processing intensity
* **Goals:** Provide research‐grade processing, maintain version consistency, ensure clean memory management

---

## 2. Package & Version Information

* **Package:** `@superpoweredsdk/web`
* **Version:** 2.7.2 (lock in via npm or CDN URL)
* **Documentation:** [https://docs.superpowered.com/getting-started/example-code/?lang=js](https://docs.superpowered.com/getting-started/example-code/?lang=js)
* **GitHub:** [https://github.com/superpoweredSDK/web](https://github.com/superpoweredSDK/web)

**Install (npm):**

```bash
npm install @superpoweredsdk/web@2.7.2
```

**CDN import (browser ESM):**

```js
import { SuperpoweredGlue, SuperpoweredWebAudio }
  from "https://cdn.jsdelivr.net/npm/@superpoweredsdk/web@2.7.2";
```

---

## 3. Main-Thread Audio Processor (`audio-processor.js`)

```javascript
class AudioProcessor {
  constructor() {
    this.audioContext = null;
    this.webaudioManager = null;
    this.workletNode = null;
    this.superpowered = null;
    this.isInitialized = false;
  }

  async initialize() {
    if (this.isInitialized) return;

    try {
      // 1. Load SDK
      const { SuperpoweredGlue, SuperpoweredWebAudio } =
        await import('@superpoweredsdk/web');

      // 2. Instantiate WASM engine
      this.superpowered = await SuperpoweredGlue.Instantiate(
        'YourLicenseKey-ForProduction',
        '/assets/superpowered.wasm'
      );

      // 3. Create AudioContext & Manager
      this.audioContext = new AudioContext();
      this.webaudioManager = new SuperpoweredWebAudio(
        this.audioContext.sampleRate,
        this.superpowered
      );

      // 4. Load and register the Worklet
      const url = '/scripts/voice-processor-worklet.js';
      await this.audioContext.audioWorklet.addModule(url);

      // 5. Create the node via helper
      this.workletNode = await this.webaudioManager.createAudioNodeAsync(
        this.audioContext,
        url,
        'VoiceProcessor',
        (msg) => {
          if (msg.event === 'ready') console.log('Worklet ready');
        }
      );

      this.workletNode.connect(this.audioContext.destination);
      this.isInitialized = true;
      console.log('✅ AudioProcessor initialized');

    } catch (err) {
      console.error('❌ Initialization failed:', err);
      throw err;
    }
  }

  async processRecording(audioBuffer) {
    if (!this.isInitialized) await this.initialize();

    return new Promise((resolve, reject) => {
      const id = Date.now();

      const handler = (event) => {
        if (event.data.requestId === id) {
          this.workletNode.port.removeEventListener('message', handler);
          resolve(event.data.versions);
        }
      };

      this.workletNode.port.addEventListener('message', handler);
      this.workletNode.port.postMessage({
        command: 'processVoice',
        requestId: id,
        audioData: audioBuffer.getChannelData(0)
      });
    });
  }

  cleanup() {
    if (this.workletNode) {
      this.workletNode.port.postMessage({ command: 'cleanup' });
      this.workletNode.disconnect();
    }
    this.audioContext?.close();
    this.superpowered?.destruct();
    this.isInitialized = false;
  }
}

window.AudioProcessor = AudioProcessor;
```

---

## 4. AudioWorklet Processor (`voice-processor-worklet.js`)

```javascript
// ESM import (no importScripts)
import { SuperpoweredWebAudio }
  from "https://cdn.jsdelivr.net/npm/@superpoweredsdk/web@2.7.2";

class VoiceProcessor extends SuperpoweredWebAudio.AudioWorkletProcessor {
  onReady() {
    this._initChain();
    this.sendMessageToMainScope({ event: 'ready' });
  }

  _initChain() {
    const sr = this.samplerate;
    // DSP objects
    this.pitchShifter = new this.Superpowered.PitchShifter(sr);
    this.compressor  = new this.Superpowered.Compressor(sr);
    this.filter      = new this.Superpowered.Filter(sr);
    this.eq          = new this.Superpowered.EQ(sr);

    // Buffers
    this.inputBuf  = new this.Superpowered.Float32Buffer(2048);
    this.workBuf   = new this.Superpowered.Float32Buffer(2048);
  }

  onMessageFromMainScope(msg) {
    if (msg.command === 'processVoice') {
      const versions = this._makeVersions(msg.audioData);
      this.sendMessageToMainScope({
        requestId: msg.requestId,
        versions
      });
    } else if (msg.command === 'cleanup') {
      this._cleanup();
    }
  }

  processAudio(inBuf, outBuf, size) {
    // passthrough if no message-driven processing
    outBuf.getChannelData(0).set(inBuf.getChannelData(0));
    return true;
  }

  _makeVersions(data) {
    return {
      A: this._identity(data),
      B: this._apply(data, {
        pitch: -60, formant: 0.9,
        hp: 80, lp: 12000,
        compR: 2, compT: -24,
        eqL: 2, eqM: 1, eqH: 0
      }),
      C: this._apply(data, {
        pitch: -120, formant: 0.8,
        hp: 200, lp: 8000,
        compR: 3, compT: -20,
        eqL: 0, eqM: 3, eqH: 0
      }),
      D: this._apply(data, {
        pitch: -180, formant: 0.75,
        hp: 250, lp: 6000,
        compR: 4, compT: -18,
        eqL: 1, eqM: 2, eqH: -1
      })
    };
  }

  _identity(d) {
    return new Float32Array(d);
  }

  _apply(data, p) {
    const len = data.length;
    this.inputBuf.copyFromFloat32Array(data);
    this.workBuf.copyFromFloat32Array(data);

    // HPF
    if (p.hp) {
      this.filter.setHighpassFrequency(p.hp);
      this.filter.process(this.inputBuf.pointer, this.workBuf.pointer, len);
      this._swap();
    }
    // LPF
    if (p.lp) {
      this.filter.setLowpassFrequency(p.lp);
      this.filter.process(this.inputBuf.pointer, this.workBuf.pointer, len);
      this._swap();
    }
    // Pitch + formant
    if (p.pitch) {
      this.pitchShifter.setPitchShift(Math.pow(2, p.pitch/1200));
      this.pitchShifter.setFormantCorrection(p.formant);
      this.pitchShifter.process(this.inputBuf.pointer, this.workBuf.pointer, len);
      this._swap();
    }
    // Compression
    if (p.compR > 1) {
      this.compressor.setRatio(p.compR);
      this.compressor.setThreshold(p.compT);
      this.compressor.process(this.inputBuf.pointer, this.workBuf.pointer, len);
      this._swap();
    }
    // EQ
    if (p.eqL||p.eqM||p.eqH) {
      this.eq.setLowGain(p.eqL);
      this.eq.setMidGain(p.eqM);
      this.eq.setHighGain(p.eqH);
      this.eq.process(this.inputBuf.pointer, this.workBuf.pointer, len);
    }

    // Copy out
    const out = new Float32Array(len);
    this.workBuf.copyToFloat32Array(out);
    return out;
  }

  _swap() {
    [this.inputBuf, this.workBuf] = [this.workBuf, this.inputBuf];
  }

  _cleanup() {
    [this.pitchShifter, this.compressor, this.filter, this.eq]
      .forEach(obj => obj?.destruct());
    [this.inputBuf, this.workBuf]
      .forEach(buf => buf?.free());
  }

  onDestruct() { this._cleanup(); }
}

registerProcessor('VoiceProcessor', VoiceProcessor);
```

---

## 5. Version-Specific Processing Parameters

| Version | Pitch (cents) | Formant | HPF (Hz) | LPF (Hz) | Comp Ratio | Comp Thres (dB) | EQ (L/M/H dB) |
| ------- | ------------- | ------- | -------- | -------- | ---------- | --------------- | ------------- |
| A       | 0             | —       | —        | —        | —          | —               | —             |
| B       | -60           | 0.9     | 80       | 12000    | 2:1        | -24             | +2/+1/0       |
| C       | -120          | 0.8     | 200      | 8000     | 3:1        | -20             | 0/+3/0        |
| D       | -180          | 0.75    | 250      | 6000     | 4:1        | -18             | +1/+2/-1      |

---

## 6. Testing & Validation

1. **Module loading:** no import/export errors (console & network)
2. **Worklet registration:** `VoiceProcessor` ready event
3. **Audio outputs:** distinct A–D playback differences
4. **Memory cleanup:** no leaks, `destruct()` and `free()` calls verified
5. **Secure context:** HTTPS or localhost only

---

## 7. Deployment & Best Practices

* **Self-host `.js` & `.wasm`** for production
* **Use valid production license key** with `Glue.Instantiate(...)`
* **Test on mobile Safari & Edge**
* **Monitor memory** on iOS (WASM \~350 MB limit)
* **Implement fallback** to pure Web Audio if initialization fails

---

Ready for your systematic bone‐conduction voice study, with rock‑solid Superpowered integration!
