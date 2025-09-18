# .github.io
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Interval Timer</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #1a1a1a;
            color: #e0e0e0;
        }
        .container {
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            padding: 1rem;
        }
        .card {
            background-color: #2a2a2a;
            border: 1px solid #444;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            max-width: 400px;
            width: 100%;
        }
        .btn {
            transition: all 0.2s;
        }
        .btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
        }
        .input-group input {
            background-color: #333;
            border: 1px solid #555;
            color: #e0e0e0;
        }
        #message-box {
            animation: fadeIn 0.5s ease-in-out;
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: scale(0.95); }
            to { opacity: 1; transform: scale(1); }
        }
    </style>
</head>
<body class="bg-gray-900 flex items-center justify-center min-h-screen">
    <div class="container">
        <!-- Message Box -->
        <div id="message-box" class="fixed top-5 left-1/2 -translate-x-1/2 z-50 p-4 rounded-lg text-center font-semibold text-white bg-blue-600 shadow-lg hidden">
        </div>

        <div class="card p-6 rounded-2xl">
            <h1 class="text-3xl font-bold text-center mb-6 text-indigo-400">Interval Timer</h1>
            <div class="flex space-x-4 mb-6">
                <button id="mode1-btn" class="flex-1 py-3 px-6 text-lg font-semibold rounded-xl bg-indigo-600 text-white btn">Leave Timer</button>
                <button id="mode2-btn" class="flex-1 py-3 px-6 text-lg font-semibold rounded-xl bg-gray-700 text-gray-300 btn">Activity Timer</button>
            </div>

            <div id="mode1-content" class="space-y-4">
                <p class="text-gray-400 text-center">Get callouts before you need to leave.</p>
                <div class="input-group">
                    <label for="leaveTime" class="block text-sm font-medium text-gray-400 mb-1">Total time until you leave (in minutes):</label>
                    <input type="number" id="leaveTime" class="w-full rounded-lg p-3" value="60">
                </div>
                <button id="start-leave-timer" class="w-full py-3 px-6 text-lg font-semibold rounded-xl bg-green-500 text-white btn">Start</button>
                <button id="stop-timer" class="w-full py-3 px-6 text-lg font-semibold rounded-xl bg-red-500 text-white btn hidden">Stop</button>
                <p id="timer-display" class="text-3xl text-center font-bold mt-4 text-white">00:00</p>
            </div>

            <div id="mode2-content" class="space-y-4 hidden">
                <p class="text-gray-400 text-center">Get callouts at regular activity intervals.</p>
                <div class="input-group">
                    <label for="activityInterval" class="block text-sm font-medium text-gray-400 mb-1">Callout interval (in minutes):</label>
                    <input type="number" id="activityInterval" class="w-full rounded-lg p-3" value="15">
                </div>
                <button id="start-activity-timer" class="w-full py-3 px-6 text-lg font-semibold rounded-xl bg-green-500 text-white btn">Start</button>
                <button id="stop-timer-2" class="w-full py-3 px-6 text-lg font-semibold rounded-xl bg-red-500 text-white btn hidden">Stop</button>
                <p id="timer-display-2" class="text-3xl text-center font-bold mt-4 text-white">00:00</p>
            </div>
        </div>
    </div>
    <script>
        // Global variables for timer state
        let timerInterval;
        let timeRemainingInSeconds; // For the leave timer
        let elapsedTimeInSeconds; // For the activity timer
        let mode;
        let isTimerRunning = false;

        // DOM Elements
        const mode1Btn = document.getElementById('mode1-btn');
        const mode2Btn = document.getElementById('mode2-btn');
        const mode1Content = document.getElementById('mode1-content');
        const mode2Content = document.getElementById('mode2-content');
        const startLeaveTimerBtn = document.getElementById('start-leave-timer');
        const startActivityTimerBtn = document.getElementById('start-activity-timer');
        const stopTimerBtn = document.getElementById('stop-timer');
        const stopTimerBtn2 = document.getElementById('stop-timer-2');
        const timerDisplay = document.getElementById('timer-display');
        const timerDisplay2 = document.getElementById('timer-display-2');
        const leaveTimeInput = document.getElementById('leaveTime');
        const activityIntervalInput = document.getElementById('activityInterval');
        const messageBox = document.getElementById('message-box');

        // Function to show a message box instead of alert
        function showMessage(message, duration = 3000) {
            messageBox.textContent = message;
            messageBox.classList.remove('hidden');
            setTimeout(() => {
                messageBox.classList.add('hidden');
            }, duration);
        }

        // Helper function to convert base64 to ArrayBuffer
        function base64ToArrayBuffer(base64) {
            const binaryString = atob(base64);
            const len = binaryString.length;
            const bytes = new Uint8Array(len);
            for (let i = 0; i < len; i++) {
                bytes[i] = binaryString.charCodeAt(i);
            }
            return bytes.buffer;
        }

        // Helper function to convert PCM data to WAV blob
        function pcmToWav(pcm16, sampleRate) {
            const numChannels = 1;
            const bytesPerSample = 2;
            const byteRate = sampleRate * numChannels * bytesPerSample;
            const blockAlign = numChannels * bytesPerSample;
            const buffer = new ArrayBuffer(44 + pcm16.length * bytesPerSample);
            const view = new DataView(buffer);

            // WAV header
            view.setUint32(0, 0x46464952, true); // "RIFF"
            view.setUint32(4, 36 + pcm16.length * bytesPerSample, true); // file size
            view.setUint32(8, 0x45564157, true); // "WAVE"
            view.setUint32(12, 0x20746d66, true); // "fmt "
            view.setUint32(16, 16, true); // fmt chunk size
            view.setUint16(20, 1, true); // audio format (PCM)
            view.setUint16(22, numChannels, true); // number of channels
            view.setUint32(24, sampleRate, true); // sample rate
            view.setUint32(28, byteRate, true); // byte rate
            view.setUint16(32, blockAlign, true); // block align
            view.setUint16(34, bytesPerSample * 8, true); // bits per sample
            view.setUint32(36, 0x61746164, true); // "data"
            view.setUint32(40, pcm16.length * bytesPerSample, true); // data size

            let offset = 44;
            for (let i = 0; i < pcm16.length; i++) {
                view.setInt16(offset, pcm16[i], true);
                offset += bytesPerSample;
            }

            return new Blob([view], { type: 'audio/wav' });
        }

        // Function to speak a message using the Gemini API
        async function speak(text) {
            const payload = {
                contents: [{
                    parts: [{ text: text }]
                }],
                generationConfig: {
                    responseModalities: ["AUDIO"],
                    speechConfig: {
                        voiceConfig: {
                            prebuiltVoiceConfig: { voiceName: "Kore" }
                        }
                    }
                },
                model: "gemini-2.5-flash-preview-tts"
            };
            
            const apiKey = ""; 
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-tts:generateContent?key=${apiKey}`;

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });
                
                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }

                const result = await response.json();
                const audioData = result?.candidates?.[0]?.content?.parts?.[0]?.inlineData?.data;
                const mimeType = result?.candidates?.[0]?.content?.parts?.[0]?.inlineData?.mimeType;

                if (audioData && mimeType && mimeType.startsWith("audio/")) {
                    const sampleRate = parseInt(mimeType.match(/rate=(\d+)/)[1], 10);
                    const pcmData = base64ToArrayBuffer(audioData);
                    const pcm16 = new Int16Array(pcmData);
                    const wavBlob = pcmToWav(pcm16, sampleRate);
                    const audioUrl = URL.createObjectURL(wavBlob);
                    
                    const audio = new Audio(audioUrl);
                    audio.play();

                    audio.onended = () => {
                        URL.revokeObjectURL(audioUrl);
                    };
                } else {
                    showMessage("Could not generate audio. Please try again.");
                }
            } catch (error) {
                console.error("Error generating audio:", error);
                showMessage("Failed to generate audio. Please check your connection.");
            }
        }

        // Function to switch modes
        function switchMode(newMode) {
            mode = newMode;
            if (newMode === 'leave') {
                mode1Btn.classList.remove('bg-gray-700', 'text-gray-300');
                mode1Btn.classList.add('bg-indigo-600', 'text-white');
                mode2Btn.classList.remove('bg-indigo-600', 'text-white');
                mode2Btn.classList.add('bg-gray-700', 'text-gray-300');
                mode1Content.classList.remove('hidden');
                mode2Content.classList.add('hidden');
            } else {
                mode2Btn.classList.remove('bg-gray-700', 'text-gray-300');
                mode2Btn.classList.add('bg-indigo-600', 'text-white');
                mode1Btn.classList.remove('bg-indigo-600', 'text-white');
                mode1Btn.classList.add('bg-gray-700', 'text-gray-300');
                mode2Content.classList.remove('hidden');
                mode1Content.classList.add('hidden');
            }
        }

        // Function to start the timer
        function startTimer() {
            if (isTimerRunning) return;
            isTimerRunning = true;
            
            // Show and hide buttons based on mode
            if (mode === 'leave') {
                startLeaveTimerBtn.classList.add('hidden');
                stopTimerBtn.classList.remove('hidden');
                timeRemainingInSeconds = parseInt(leaveTimeInput.value, 10) * 60;
            } else { // mode === 'activity'
                startActivityTimerBtn.classList.add('hidden');
                stopTimerBtn2.classList.remove('hidden');
                elapsedTimeInSeconds = 0;
            }

            timerInterval = setInterval(() => {
                if (mode === 'leave') {
                    // This is the countdown timer
                    const minutes = Math.floor(timeRemainingInSeconds / 60);
                    const seconds = timeRemainingInSeconds % 60;
                    const display = `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
                    timerDisplay.textContent = display;
                    
                    // Mode 1 callout logic
                    if (timeRemainingInSeconds > 0) {
                        if (minutes % 15 === 0 && seconds === 0) {
                            if (minutes === 0) {
                                speak("You need to leave now!");
                            } else {
                                speak(`You need to leave in ${minutes} minutes.`);
                            }
                        }
                    }

                    timeRemainingInSeconds--;
                    if (timeRemainingInSeconds < 0) {
                        stopTimer();
                        showMessage("Timer finished!");
                    }
                } else { // mode === 'activity'
                    // This is the count-up timer
                    const minutes = Math.floor(elapsedTimeInSeconds / 60);
                    const seconds = elapsedTimeInSeconds % 60;
                    const display = `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
                    timerDisplay2.textContent = display;

                    // Mode 2 callout logic
                    const interval = parseInt(activityIntervalInput.value, 10);
                    if (elapsedTimeInSeconds > 0 && minutes > 0 && minutes % interval === 0 && seconds === 0) {
                        speak(`You have been active for ${minutes} minutes.`);
                    }

                    elapsedTimeInSeconds++;
                }
            }, 1000);
        }

        // Function to stop the timer
        function stopTimer() {
            clearInterval(timerInterval);
            isTimerRunning = false;
            startLeaveTimerBtn.classList.remove('hidden');
            stopTimerBtn.classList.add('hidden');
            startActivityTimerBtn.classList.remove('hidden');
            stopTimerBtn2.classList.add('hidden');
        }

        // Event Listeners
        mode1Btn.addEventListener('click', () => {
            switchMode('leave');
            stopTimer(); // Stop any running timer when switching modes
            timerDisplay.textContent = "00:00";
        });

        mode2Btn.addEventListener('click', () => {
            switchMode('activity');
            stopTimer(); // Stop any running timer when switching modes
            timerDisplay2.textContent = "00:00";
        });

        startLeaveTimerBtn.addEventListener('click', () => {
            const leaveTime = parseInt(leaveTimeInput.value, 10);
            if (leaveTime > 0) {
                startTimer();
                showMessage("Leave timer started!");
            } else {
                showMessage("Please enter a valid time.");
            }
        });

        startActivityTimerBtn.addEventListener('click', () => {
            const interval = parseInt(activityIntervalInput.value, 10);
            if (interval > 0) {
                startTimer();
                showMessage("Activity timer started!");
            } else {
                showMessage("Please enter a valid interval.");
            }
        });

        stopTimerBtn.addEventListener('click', stopTimer);
        stopTimerBtn2.addEventListener('click', stopTimer);

        // Initial mode setup
        switchMode('leave');
    </script>
</body>
</html>
