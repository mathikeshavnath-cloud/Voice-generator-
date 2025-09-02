# Voice-generator-
Creating a voice generator 

// VoiceGenForCoding - Single-file React app (Tailwind CSS) // Purpose: Internship project — a simple, production-ready voice generator app focused on converting code/comments to speech. // Features: // - Paste code or text (supports syntax-highlighting preview via <pre>) // - Choose from available browser TTS voices (Web Speech API) // - Configure rate, pitch // - Play / Stop speech in-browser // - Copy to clipboard, clear, and basic UI // - Notes and instructions included below for adding a server-side MP3 generation using services like ElevenLabs or Google Cloud TTS.

import React, { useEffect, useState, useRef } from "react";

export default function VoiceGenForCoding() { const [text, setText] = useState(// Paste your code or write a comment here\nfunction helloWorld() {\n  console.log('Hello, internship!')\n}); const [voices, setVoices] = useState([]); const [selectedVoice, setSelectedVoice] = useState(null); const [rate, setRate] = useState(1); const [pitch, setPitch] = useState(1); const [speaking, setSpeaking] = useState(false); const utterRef = useRef(null);

useEffect(() => { // Load voices. Some browsers load voices asynchronously. function loadVoices() { const v = window.speechSynthesis.getVoices(); setVoices(v); if (v.length && !selectedVoice) setSelectedVoice(v[0]?.name || null); }

loadVoices();
window.speechSynthesis.onvoiceschanged = loadVoices;

return () => {
  window.speechSynthesis.onvoiceschanged = null;
};

}, []);

function speak() { if (!('speechSynthesis' in window)) { alert('Web Speech API not supported in this browser. Use Chrome/Edge/Firefox (newer).'); return; }

if (speaking) {
  stop();
  return;
}

window.speechSynthesis.cancel();
const utter = new SpeechSynthesisUtterance(text);
utter.rate = rate;
utter.pitch = pitch;

const voiceObj = voices.find(v => v.name === selectedVoice) || voices[0];
if (voiceObj) utter.voice = voiceObj;

utter.onstart = () => setSpeaking(true);
utter.onend = () => setSpeaking(false);
utter.onerror = () => setSpeaking(false);

utterRef.current = utter;
window.speechSynthesis.speak(utter);

}

function stop() { window.speechSynthesis.cancel(); setSpeaking(false); }

function copyToClipboard() { navigator.clipboard.writeText(text).then(() => { alert('Copied to clipboard'); }).catch(() => alert('Clipboard failed')); }

function clearText() { setText(''); }

// Fallback: download using server-side TTS (example provided in README below)

return ( <div className="min-h-screen bg-slate-50 flex items-start justify-center p-6"> <div className="w-full max-w-4xl bg-white rounded-2xl shadow-md p-6"> <header className="flex items-center justify-between mb-4"> <h1 className="text-2xl font-semibold">VoiceGen for Coding — Internship Project</h1> <div className="text-sm text-slate-500">Uses your browser's TTS (Web Speech API)</div> </header>

<main className="grid grid-cols-1 md:grid-cols-2 gap-6">
      <section>
        <label className="block text-sm font-medium text-slate-700 mb-2">Code / Text</label>
        <textarea
          value={text}
          onChange={e => setText(e.target.value)}
          rows={16}
          className="w-full p-3 border rounded-lg font-mono text-sm bg-slate-50"
        />

        <div className="flex gap-2 mt-3">
          <button onClick={speak} className="px-4 py-2 rounded-md shadow-sm bg-indigo-600 text-white">
            {speaking ? 'Stop' : 'Play'}
          </button>
          <button onClick={copyToClipboard} className="px-4 py-2 rounded-md border">Copy</button>
          <button onClick={clearText} className="px-4 py-2 rounded-md border">Clear</button>
        </div>

        <div className="mt-4 text-xs text-slate-500">
          Tip: Add short comments or a clean narration line above code to make the spoken output clearer for listeners.
        </div>
      </section>

      <aside>
        <label className="block text-sm font-medium text-slate-700 mb-2">Voice</label>
        <select
          value={selectedVoice || ''}
          onChange={e => setSelectedVoice(e.target.value)}
          className="w-full p-2 border rounded-md mb-3"
        >
          {voices.length === 0 && <option>Loading voices...</option>}
          {voices.map((v, i) => (
            <option key={i} value={v.name}>{v.name} — {v.lang}{v.default ? ' (default)' : ''}</option>
          ))}
        </select>

        <div className="mt-2">
          <label className="block text-sm">Rate: {rate.toFixed(1)}</label>
          <input type="range" min="0.5" max="2" step="0.1" value={rate} onChange={e => setRate(parseFloat(e.target.value))} />
        </div>

        <div className="mt-2">
          <label className="block text-sm">Pitch: {pitch.toFixed(1)}</label>
          <input type="range" min="0.5" max="2" step="0.1" value={pitch} onChange={e => setPitch(parseFloat(e.target.value))} />
        </div>

        <div className="mt-4 p-3 border rounded-md bg-slate-50 text-sm">
          <strong>Export / Download</strong>
          <p className="mt-2 text-xs text-slate-600">Browser TTS (used here) plays audio but cannot always save an MP3 reliably. For downloadable files, set up a small server that uses a TTS provider (ElevenLabs, Google Cloud TTS, AWS Polly) — example server code is in the README below.</p>
        </div>
      </aside>
    </main>

    <hr className="my-6" />

    <section>
      <h2 className="text-lg font-medium mb-2">Live Preview</h2>
      <div className="p-4 rounded-md bg-black text-white font-mono text-sm overflow-auto max-h-48">
        <pre>{text}</pre>
      </div>
    </section>

    <footer className="mt-6 text-xs text-slate-500">
      <strong>Internship notes:</strong> include this app in your portfolio as a small React project. Add a server-side TTS to support MP3 downloads and to show your backend skills. See the example Node/Express snippet below.

      <details className="mt-3">
        <summary className="cursor-pointer">Server-side example (Node.js + Express + ElevenLabs / Google TTS)</summary>
        <div className="mt-2 text-xs text-slate-600">

{`// Example: server.js (Node/Express) - pseudocode, replace with your API key and provider

const express = require('express'); const fetch = require('node-fetch'); const fs = require('fs'); const app = express(); app.use(express.json());

app.post('/synthesize', async (req, res) => { const { text, voice } = req.body; // Example for ElevenLabs (replace URL and headers with correct provider docs) const response = await fetch('https://api.elevenlabs.io/v1/text-to-speech/' + (voice || 'alloy'), { method: 'POST', headers: { 'Content-Type': 'application/json', 'xi-api-key': process.env.ELEVENLABS_KEY }, body: JSON.stringify({ text }) });

const arrayBuffer = await response.arrayBuffer(); res.set('Content-Type', 'audio/mpeg'); res.send(Buffer.from(arrayBuffer)); });

app.listen(3001, () => console.log('TTS server listening on 3001'));

// On the frontend you would call POST /synthesize, then download the returned audio and save as .mp3 `} </div> </details> </footer> </div> </div> ); }

/* README / How to run (for your internship submission)

1. Frontend (this React component)



Create a React app (Vite or Create React App).

Add Tailwind CSS (recommended) for styling.

Drop this component in src/App.jsx and import it in main.jsx.

Run: npm install && npm run dev


2. Optional Backend for downloadable MP3s



Node.js + Express server (example above).

Use a TTS provider (ElevenLabs, Google Cloud, AWS Polly) — requires API key and following provider docs.

Expose an endpoint that converts text to audio and returns an audio file.


3. Demo & Testing



Show how the app reads a code snippet.

Add a short README describing tech choices and API considerations.


Good luck — if you want, I can also generate the Node/Express server file and a sample README ZIP you can submit for your internship. */

