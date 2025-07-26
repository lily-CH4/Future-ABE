<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Google Sheets Quiz with Timer</title>
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gradient-to-br from-blue-50 to-indigo-100 min-h-screen">
    <div class="container mx-auto px-4 py-8">
        <!-- Header -->
        <div class="text-center mb-8">
            <h1 class="text-4xl font-bold text-gray-800 mb-2">Google Sheets Quiz</h1>
            <p class="text-gray-600">Questions loaded directly from your Google Sheet</p>
        </div>

        <!-- Configuration Panel -->
        <div class="bg-white rounded-lg shadow-lg p-6 mb-6">
            <h3 class="text-lg font-semibold text-gray-800 mb-4">Quiz Configuration</h3>
            <div class="flex flex-col md:flex-row gap-4">
                <div class="flex-1">
                    <label class="block text-sm font-medium text-gray-700 mb-2">Google Apps Script URL:</label>
                    <input type="url" id="sheetUrl" placeholder="https://script.google.com/macros/s/YOUR_SCRIPT_ID/exec" 
                           class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500">
                </div>
                <div class="flex items-end">
                    <button id="loadQuestionsBtn" class="bg-green-600 hover:bg-green-700 text-white font-bold py-2 px-6 rounded-lg transition duration-300">
                        Load Questions
                    </button>
                </div>
            </div>
            <div id="loadStatus" class="mt-2 text-sm"></div>
        </div>

        <!-- Timer Display -->
        <div class="bg-white rounded-lg shadow-lg p-6 mb-6 text-center">
            <div class="flex items-center justify-center space-x-4">
                <div class="text-2xl font-bold text-indigo-600">
                    Time: <span id="timer">00:00</span>
                </div>
                <div class="text-lg text-gray-600">
                    Question <span id="questionNumber">1</span> of <span id="totalQuestions">0</span>
                </div>
            </div>
        </div>

        <!-- Quiz Container -->
        <div id="quizContainer" class="bg-white rounded-lg shadow-lg p-8">
            <!-- Start Screen -->
            <div id="startScreen" class="text-center">
                <div class="mb-6">
                    <svg class="w-24 h-24 mx-auto text-indigo-500 mb-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.663 17h4.673M12 3v1m6.364 1.636l-.707.707M21 12h-1M4 12H3m3.343-5.657l-.707-.707m2.828 9.9a5 5 0 117.072 0l-.548.547A3.374 3.374 0 0014 18.469V19a2 2 0 11-4 0v-.531c0-.895-.356-1.754-.988-2.386l-.548-.547z"></path>
                    </svg>
                    <h2 class="text-2xl font-bold text-gray-800 mb-2">Ready to Start?</h2>
                    <p class="text-gray-600 mb-6">Load questions from your Google Sheet first, then start the quiz!</p>
                    <div id="questionsLoaded" class="hidden mb-4 p-3 bg-green-100 text-green-800 rounded-lg">
                        <span id="questionsCount">0</span> questions loaded successfully!
                    </div>
                </div>
                <button id="startBtn" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 px-8 rounded-lg transition duration-300 transform hover:scale-105 disabled:opacity-50 disabled:cursor-not-allowed" disabled>
                    Start Quiz
                </button>
            </div>

            <!-- Question Screen -->
            <div id="questionScreen" class="hidden">
                <div class="mb-6">
                    <h2 id="questionText" class="text-xl font-semibold text-gray-800 mb-6"></h2>
                    <div id="answersContainer" class="space-y-3"></div>
                </div>
                <div class="flex justify-between items-center">
                    <div class="text-sm text-gray-500">
                        Select an answer to continue
                    </div>
                    <button id="nextBtn" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-6 rounded-lg transition duration-300 disabled:opacity-50 disabled:cursor-not-allowed" disabled>
                        Next Question
                    </button>
                </div>
            </div>

            <!-- Results Screen -->
            <div id="resultsScreen" class="hidden text-center">
                <div class="mb-6">
                    <svg class="w-24 h-24 mx-auto text-green-500 mb-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"></path>
                    </svg>
                    <h2 class="text-2xl font-bold text-gray-800 mb-4">Quiz Complete!</h2>
                    <div class="bg-gray-50 rounded-lg p-6 mb-6">
                        <div class="grid grid-cols-1 md:grid-cols-3 gap-4 text-center">
                            <div>
                                <div class="text-2xl font-bold text-indigo-600" id="finalScore">0/0</div>
                                <div class="text-sm text-gray-600">Score</div>
                            </div>
                            <div>
                                <div class="text-2xl font-bold text-green-600" id="finalTime">00:00</div>
                                <div class="text-sm text-gray-600">Time Taken</div>
                            </div>
                            <div>
                                <div class="text-2xl font-bold text-purple-600" id="accuracy">0%</div>
                                <div class="text-sm text-gray-600">Accuracy</div>
                            </div>
                        </div>
                    </div>
                </div>
                <button id="restartBtn" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 px-8 rounded-lg transition duration-300 transform hover:scale-105">
                    Take Quiz Again
                </button>
            </div>
        </div>

        <!-- Instructions -->
        <div class="mt-8 bg-white rounded-lg shadow-lg p-6">
            <h3 class="text-lg font-semibold text-gray-800 mb-3">Setup Instructions:</h3>
            <div class="text-sm text-gray-600 space-y-2">
                <p><strong>Step 1:</strong> Create a Google Sheet with columns: Question, Option A, Option B, Option C, Option D, Correct Answer (0-3)</p>
                <p><strong>Step 2:</strong> Go to Extensions > Apps Script and paste the provided code</p>
                <p><strong>Step 3:</strong> Deploy as web app with "Anyone" access</p>
                <p><strong>Step 4:</strong> Copy the web app URL and paste it above</p>
                <p><strong>Step 5:</strong> Click "Load Questions" then start your quiz!</p>
            </div>
        </div>
    </div>

    <script>
        // Quiz state
        let quizData = [];
        let currentQuestions = [];
        let currentQuestionIndex = 0;
        let score = 0;
        let startTime = null;
        let timerInterval = null;
        let selectedAnswer = null;

        // DOM elements
        const sheetUrl = document.getElementById('sheetUrl');
        const loadQuestionsBtn = document.getElementById('loadQuestionsBtn');
        const loadStatus = document.getElementById('loadStatus');
        const questionsLoaded = document.getElementById('questionsLoaded');
        const questionsCount = document.getElementById('questionsCount');
        const startScreen = document.getElementById('startScreen');
        const questionScreen = document.getElementById('questionScreen');
        const resultsScreen = document.getElementById('resultsScreen');
        const startBtn = document.getElementById('startBtn');
        const nextBtn = document.getElementById('nextBtn');
        const restartBtn = document.getElementById('restartBtn');
        const timer = document.getElementById('timer');
        const questionNumber = document.getElementById('questionNumber');
        const totalQuestions = document.getElementById('totalQuestions');
        const questionText = document.getElementById('questionText');
        const answersContainer = document.getElementById('answersContainer');

        // Load questions from Google Sheets
        async function loadQuestions() {
            const url = sheetUrl.value.trim();
            if (!url) {
                showStatus('Please enter your Google Apps Script URL', 'error');
                return;
            }

            loadQuestionsBtn.disabled = true;
            loadQuestionsBtn.textContent = 'Loading...';
            showStatus('Loading questions from Google Sheets...', 'loading');

            try {
                const response = await fetch(url);
                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }
                
                const data = await response.json();
                
                if (!Array.isArray(data) || data.length === 0) {
                    throw new Error('No questions found in the sheet');
                }

                // Validate question format
                const validQuestions = data.filter(q => 
                    q.question && 
                    Array.isArray(q.options) && 
                    q.options.length === 4 && 
                    typeof q.correct === 'number' && 
                    q.correct >= 0 && 
                    q.correct <= 3
                );

                if (validQuestions.length === 0) {
                    throw new Error('No valid questions found. Check your sheet format.');
                }

                quizData = validQuestions;
                questionsCount.textContent = quizData.length;
                questionsLoaded.classList.remove('hidden');
                startBtn.disabled = false;
                
                showStatus(`Successfully loaded ${quizData.length} questions!`, 'success');
                
            } catch (error) {
                console.error('Error loading questions:', error);
                showStatus(`Error: ${error.message}`, 'error');
            } finally {
                loadQuestionsBtn.disabled = false;
                loadQuestionsBtn.textContent = 'Load Questions';
            }
        }

        // Show status message
        function showStatus(message, type) {
            loadStatus.textContent = message;
            loadStatus.className = `mt-2 text-sm ${
                type === 'error' ? 'text-red-600' : 
                type === 'success' ? 'text-green-600' : 
                'text-blue-600'
            }`;
        }

        // Shuffle array function
        function shuffleArray(array) {
            const shuffled = [...array];
            for (let i = shuffled.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
            }
            return shuffled;
        }

        // Format time function
        function formatTime(seconds) {
            const mins = Math.floor(seconds / 60);
            const secs = seconds % 60;
            return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
        }

        // Start timer
        function startTimer() {
            startTime = Date.now();
            timerInterval = setInterval(() => {
                const elapsed = Math.floor((Date.now() - startTime) / 1000);
                timer.textContent = formatTime(elapsed);
            }, 1000);
        }

        // Stop timer
        function stopTimer() {
            if (timerInterval) {
                clearInterval(timerInterval);
                timerInterval = null;
            }
        }

        // Initialize quiz
        function initQuiz() {
            if (quizData.length === 0) {
                alert('Please load questions first!');
                return;
            }

            currentQuestions = shuffleArray(quizData);
            currentQuestionIndex = 0;
            score = 0;
            selectedAnswer = null;
            totalQuestions.textContent = currentQuestions.length;
            
            startScreen.classList.add('hidden');
            questionScreen.classList.remove('hidden');
            resultsScreen.classList.add('hidden');
            
            startTimer();
            showQuestion();
        }

        // Show current question
        function showQuestion() {
            const question = currentQuestions[currentQuestionIndex];
            questionNumber.textContent = currentQuestionIndex + 1;
            questionText.textContent = question.question;
            
            answersContainer.innerHTML = '';
            selectedAnswer = null;
            nextBtn.disabled = true;
            
            question.options.forEach((option, index) => {
                const button = document.createElement('button');
                button.className = 'w-full text-left p-4 rounded-lg border-2 border-gray-200 hover:border-indigo-300 hover:bg-indigo-50 transition duration-300';
                button.textContent = `${String.fromCharCode(65 + index)}. ${option}`;
                button.onclick = () => selectAnswer(index, button);
                answersContainer.appendChild(button);
            });
        }

        // Select answer
        function selectAnswer(answerIndex, buttonElement) {
            // Remove previous selection
            const buttons = answersContainer.querySelectorAll('button');
            buttons.forEach(btn => {
                btn.classList.remove('border-indigo-500', 'bg-indigo-100');
                btn.classList.add('border-gray-200');
            });
            
            // Highlight selected answer
            buttonElement.classList.remove('border-gray-200');
            buttonElement.classList.add('border-indigo-500', 'bg-indigo-100');
            
            selectedAnswer = answerIndex;
            nextBtn.disabled = false;
        }

        // Next question
        function nextQuestion() {
            if (selectedAnswer === null) return;
            
            // Check if answer is correct
            if (selectedAnswer === currentQuestions[currentQuestionIndex].correct) {
                score++;
            }
            
            currentQuestionIndex++;
            
            if (currentQuestionIndex < currentQuestions.length) {
                showQuestion();
            } else {
                showResults();
            }
        }

        // Show results
        function showResults() {
            stopTimer();
            
            const totalTime = Math.floor((Date.now() - startTime) / 1000);
            const accuracy = Math.round((score / currentQuestions.length) * 100);
            
            questionScreen.classList.add('hidden');
            resultsScreen.classList.remove('hidden');
            
            document.getElementById('finalScore').textContent = `${score}/${currentQuestions.length}`;
            document.getElementById('finalTime').textContent = formatTime(totalTime);
            document.getElementById('accuracy').textContent = `${accuracy}%`;
        }

        // Restart quiz
        function restartQuiz() {
            stopTimer();
            timer.textContent = '00:00';
            
            resultsScreen.classList.add('hidden');
            startScreen.classList.remove('hidden');
        }

        // Event listeners
        loadQuestionsBtn.addEventListener('click', loadQuestions);
        startBtn.addEventListener('click', initQuiz);
        nextBtn.addEventListener('click', nextQuestion);
        restartBtn.addEventListener('click', restartQuiz);

        // Keyboard navigation
        document.addEventListener('keydown', (e) => {
            if (questionScreen.classList.contains('hidden')) return;
            
            if (e.key >= '1' && e.key <= '4') {
                const index = parseInt(e.key) - 1;
                const buttons = answersContainer.querySelectorAll('button');
                if (buttons[index]) {
                    selectAnswer(index, buttons[index]);
                }
            } else if (e.key === 'Enter' && !nextBtn.disabled) {
                nextQuestion();
            }
        });

        // Load URL from localStorage if available
        window.addEventListener('load', () => {
            const savedUrl = localStorage.getItem('quizSheetUrl');
            if (savedUrl) {
                sheetUrl.value = savedUrl;
            }
        });

        // Save URL to localStorage
        sheetUrl.addEventListener('change', () => {
            localStorage.setItem('quizSheetUrl', sheetUrl.value);
        });
    </script>
<script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'96575af15296bc3d',t:'MTc1MzU2OTE3MC4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
