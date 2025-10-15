# Quatex-analysis-
My 1-minute OTC predictor with Ai
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Next-Candle AI Predictor</title>
    <!-- Tailwind CSS CDN for styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Custom styles for the file input button */
        .file-input-button {
            cursor: pointer;
            padding: 0.75rem 1.5rem;
            border-radius: 0.5rem;
            background-color: #4A5568; /* gray-700 */
            color: white;
            transition: background-color 0.3s;
        }
        .file-input-button:hover {
            background-color: #2D3748; /* gray-800 */
        }
        /* Custom styles for the loading spinner */
        .loader {
            border: 4px solid rgba(255, 255, 255, 0.3);
            border-radius: 50%;
            border-top: 4px solid #4299e1; /* blue-400 */
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body class="bg-gray-900 text-gray-200 font-sans flex items-center justify-center min-h-screen p-4">

    <!-- Main Card Container -->
    <div class="w-full max-w-2xl bg-gray-800 rounded-xl shadow-2xl p-8 space-y-8">
        
        <!-- Heading -->
        <h1 class="text-4xl font-bold text-center text-white tracking-wider">
            Next-Candle AI Predictor
        </h1>

        <!-- File Upload Section -->
        <div class="flex flex-col items-center space-y-4">
            <label for="screenshot-upload" class="file-input-button text-lg">
                Upload Screenshot
            </label>
            <input id="screenshot-upload" type="file" class="hidden" accept=".jpg, .jpeg, .png">
            <p id="file-name" class="text-gray-400 text-sm">No file selected</p>
        </div>

        <!-- Analyze Button -->
        <div class="text-center">
            <button id="analyze-button" class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-8 rounded-lg text-xl transition duration-300 ease-in-out transform hover:scale-105 shadow-lg">
                Analyze Chart
            </button>
        </div>
        
        <!-- Error Message Area -->
        <div id="error-message" class="text-center text-red-400 font-semibold hidden"></div>

        <!-- Loading Indicator -->
        <div id="loading-indicator" class="hidden flex-col items-center justify-center space-y-4 pt-6">
            <div class="loader"></div>
            <p class="text-blue-300 text-lg animate-pulse">Analyzing with Gemini AI...</p>
        </div>

        <!-- Output Section -->
        <div id="output-container" class="hidden pt-6 space-y-6">
            <!-- Content will be injected here by JavaScript -->
        </div>

        <!-- Strategy Section -->
        <div id="strategy-section" class="hidden pt-6 space-y-4 text-center">
             <button id="strategy-button" class="bg-purple-600 hover:bg-purple-700 text-white font-bold py-3 px-8 rounded-lg text-xl transition duration-300 ease-in-out transform hover:scale-105 shadow-lg">
                ✨ Generate Trading Strategy
            </button>
            <div id="strategy-loading-indicator" class="hidden flex-col items-center justify-center space-y-4 pt-6">
                <div class="loader"></div>
                <p class="text-purple-300 text-lg animate-pulse">Generating strategy...</p>
            </div>
            <div id="strategy-output" class="hidden bg-gray-900 p-4 rounded-lg mt-4 text-left"></div>
        </div>
    </div>

    <script>
        // DOM Elements
        const uploadInput = document.getElementById('screenshot-upload');
        const fileNameDisplay = document.getElementById('file-name');
        const analyzeButton = document.getElementById('analyze-button');
        const loadingIndicator = document.getElementById('loading-indicator');
        const outputContainer = document.getElementById('output-container');
        const errorMessage = document.getElementById('error-message');

        const strategySection = document.getElementById('strategy-section');
        const strategyButton = document.getElementById('strategy-button');
        const strategyLoadingIndicator = document.getElementById('strategy-loading-indicator');
        const strategyOutput = document.getElementById('strategy-output');
        
        let currentAnalysisText = ""; // To store the analysis for strategy generation

        // File input change listener
        uploadInput.addEventListener('change', () => {
            if (uploadInput.files.length > 0) {
                fileNameDisplay.textContent = `File: ${uploadInput.files[0].name}`;
                errorMessage.classList.add('hidden');
                outputContainer.classList.add('hidden');
                strategySection.classList.add('hidden');
                strategyOutput.classList.add('hidden');
            } else {
                fileNameDisplay.textContent = 'No file selected';
            }
        });

        // Analyze button click listener
        analyzeButton.addEventListener('click', async () => {
            if (uploadInput.files.length === 0) {
                showError('Please upload an image first.');
                return;
            }
            
            // Reset UI
            showError(''); // Clear previous errors
            outputContainer.classList.add('hidden');
            strategySection.classList.add('hidden');
            strategyOutput.classList.add('hidden');
            loadingIndicator.classList.remove('hidden');
            loadingIndicator.classList.add('flex');
            
            try {
                // Call Gemini API for analysis
                await predictCandleWithGemini();
            } catch (error) {
                console.error("Error during analysis:", error);
                showError("Failed to analyze the image. Please try again.");
            } finally {
                // Hide loading indicator
                loadingIndicator.classList.add('hidden');
                loadingIndicator.classList.remove('flex');
            }
        });

        // Strategy button click listener
        strategyButton.addEventListener('click', async () => {
            if (!currentAnalysisText) {
                showError("No analysis available to generate a strategy.");
                return;
            }

            strategyLoadingIndicator.classList.remove('hidden');
            strategyLoadingIndicator.classList.add('flex');
            strategyOutput.classList.add('hidden');
            
            try {
                const prompt = `You are a trading mentor. Based on the following market analysis, suggest a simple, actionable trading strategy for a beginner. Mention entry points, stop-loss, and take-profit targets in a clear, easy-to-understand format. The analysis is: ${currentAnalysisText}`;
                const strategyText = await callGeminiTextOnly(prompt);
                
                strategyOutput.innerHTML = `<h3 class="text-xl font-semibold text-purple-300 mb-3">Trading Strategy</h3><p class="text-gray-300 whitespace-pre-wrap">${strategyText}</p>`;
                strategyOutput.classList.remove('hidden');

            } catch (error) {
                console.error("Error generating strategy:", error);
                showError("Failed to generate a trading strategy.");
            } finally {
                strategyLoadingIndicator.classList.add('hidden');
                strategyLoadingIndicator.classList.remove('flex');
            }
        });

        // Function to convert file to base64
        function fileToBase64(file) {
            return new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.readAsDataURL(file);
                reader.onload = () => resolve(reader.result.split(',')[1]);
                reader.onerror = error => reject(error);
            });
        }

        /**
         * Calls Gemini API with image for analysis
         */
        async function predictCandleWithGemini() {
            const file = uploadInput.files[0];
            const base64ImageData = await fileToBase64(file);
            
            const apiKey = ""; // API key is handled by the environment
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;

            const systemPrompt = `You are a professional trading analyst. Analyze the provided trading chart screenshot. Based on the candlestick patterns, support/resistance levels, and momentum indicators you can see, provide a prediction for the next candle. Your response MUST be a valid JSON object with no markdown formatting. The JSON object should have two main keys: 'mainSignal' and 'analysis'. The 'mainSignal' object should contain 'greenClose' (with 'next', 'expiry', 'confidence'), 'redClose' (with 'next', 'expiry', 'confidence'), 'backup', and 'contingency'. The 'analysis' object should contain 'structure', 'srAndSpace', 'psychology', 'momentum', and 'setup'.`;

            const payload = {
                contents: [{
                    parts: [
                        { text: systemPrompt },
                        { inlineData: { mimeType: file.type, data: base64ImageData } }
                    ]
                }]
            };

            const response = await fetch(apiUrl, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });

            if (!response.ok) {
                throw new Error(`API call failed with status: ${response.status}`);
            }

            const result = await response.json();
            const textResponse = result.candidates?.[0]?.content?.parts?.[0]?.text;
            
            if (!textResponse) {
                throw new Error("Invalid response from Gemini API.");
            }

            try {
                const analysisData = JSON.parse(textResponse);
                displayAnalysis(analysisData);
                outputContainer.classList.remove('hidden');
                strategySection.classList.remove('hidden'); // Show strategy button
            } catch (e) {
                console.error("Failed to parse JSON response:", textResponse);
                throw new Error("Model returned an invalid format. Please try again.");
            }
        }
        
        /**
         * Generic function to call Gemini with a text-only prompt
         */
        async function callGeminiTextOnly(prompt) {
            const apiKey = "";
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;
            const payload = { contents: [{ parts: [{ text: prompt }] }] };

            const response = await fetch(apiUrl, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });

             if (!response.ok) {
                throw new Error(`API call failed with status: ${response.status}`);
            }

            const result = await response.json();
            const textResponse = result.candidates?.[0]?.content?.parts?.[0]?.text;

            if (!textResponse) {
                throw new Error("Invalid response from Gemini API for strategy generation.");
            }
            return textResponse;
        }

        /**
         * Displays the analysis data in the UI
         */
        function displayAnalysis(data) {
            currentAnalysisText = `MAIN SIGNAL: If close GREEN: Next=${data.mainSignal.greenClose.next}, Expiry=${data.mainSignal.greenClose.expiry}. If close RED: Next=${data.mainSignal.redClose.next}, Expiry=${data.mainSignal.redClose.expiry}. ANALYSIS: Structure: ${data.analysis.structure}. S/R and Space: ${data.analysis.srAndSpace}. Psychology: ${data.analysis.psychology}. Momentum: ${data.analysis.momentum}. Setup: ${data.analysis.setup}.`;
            
            const analysisHTML = `
                <h2 class="text-2xl font-bold text-center text-blue-300 border-b-2 border-gray-700 pb-2">Analysis Result</h2>
                <div class="bg-gray-900 p-4 rounded-lg">
                    <h3 class="text-xl font-semibold text-gray-100 mb-3">MAIN SIGNAL</h3>
                    <ul class="list-disc list-inside space-y-2 text-gray-300">
                        <li><span class="font-semibold text-green-400">If close GREEN:</span> Next = ${data.mainSignal.greenClose.next}, Expiry = ${data.mainSignal.greenClose.expiry}, Conf = ${data.mainSignal.greenClose.confidence}%</li>
                        <li><span class="font-semibold text-red-400">If close RED:</span> Next = ${data.mainSignal.redClose.next}, Expiry = ${data.mainSignal.redClose.expiry}, Conf = ${data.mainSignal.redClose.confidence}%</li>
                        <li><span class="font-semibold text-yellow-400">Backup (at close):</span> ${data.mainSignal.backup}</li>
                        <li><span class="font-semibold text-orange-400">Loss-Contingency:</span> ${data.mainSignal.contingency}</li>
                    </ul>
                </div>
                <div class="bg-gray-900 p-4 rounded-lg">
                    <h3 class="text-xl font-semibold text-gray-100 mb-3">ANALYSIS</h3>
                    <ul class="list-disc list-inside space-y-2 text-gray-300">
                        <li><span class="font-semibold">Structure:</span> ${data.analysis.structure}</li>
                        <li><span class="font-semibold">S/R and Space:</span> ${data.analysis.srAndSpace}</li>
                        <li><span class="font-semibold">Psychology & Wicks:</span> ${data.analysis.psychology}</li>
                        <li><span class="font-semibold">Momentum:</span> ${data.analysis.momentum}</li>
                        <li><span class="font-semibold">Setup & Final Bias:</span> ${data.analysis.setup}</li>
                    </ul>
                </div>
            `;
            outputContainer.innerHTML = analysisHTML;
        }

        /**
         * Shows an error message in the UI
         */
        function showError(message) {
            if (message) {
                errorMessage.textContent = message;
                errorMessage.classList.remove('hidden');
            } else {
                errorMessage.classList.add('hidden');
            }
        }
    </script>
</body>
</html>

