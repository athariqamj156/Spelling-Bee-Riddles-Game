# SpellingBeeGame
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Spelling Bee + Riddles</title>
    <style>
        /* Gaya CSS */
        body {
            font-family: 'Arial', sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            background-color: #f4f4f9;
            color: #333;
            margin: 0;
            padding: 20px;
        }

        .container {
            background-color: #fff;
            padding: 30px 40px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
            width: 90%;
            max-width: 600px;
            text-align: center;
        }

        h1 {
            color: #4CAF50;
            margin-bottom: 20px;
        }

        button {
            background-color: #007bff;
            color: white;
            padding: 10px 20px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
            margin: 5px;
            transition: background-color 0.3s;
        }

        button:hover {
            background-color: #0056b3;
        }

        #level-selection button {
            background-color: #28a745;
        }

        #level-selection button:hover {
            background-color: #1e7e34;
        }

        input[type="text"] {
            padding: 10px;
            margin: 15px 0;
            width: 80%;
            border: 2px solid #ccc;
            border-radius: 5px;
            font-size: 18px;
            text-align: center;
        }

        #timer {
            font-size: 2em;
            font-weight: bold;
            color: #dc3545;
            margin: 10px 0;
        }

        #word-display {
            font-size: 1.5em;
            font-style: italic;
            margin-bottom: 20px;
            color: #6c757d;
        }

        #message {
            margin-top: 20px;
            font-weight: bold;
            min-height: 25px; /* Menjaga tata letak */
        }

        .correct {
            color: #28a745;
        }

        .wrong {
            color: #dc3545;
        }

        .score-box {
            margin-top: 15px;
            font-size: 1.2em;
            color: #007bff;
        }
    </style>
</head>
<body>

    <div class="container">
        <h1>Spelling Bee Challenge 🐝</h1>

        <div id="level-selection">
            <h2>Pilih Level</h2>
            <button onclick="startGame('easy')">EASY</button>
            <button onclick="startGame('medium')">MEDIUM</button>
            <button onclick="startGame('hard')">HARD</button>
        </div>

        <div id="game-area" style="display: none;">
            <p id="current-level"></p>
            <p>Please type the letters in the box:</p>
            <div id="timer">10</div>
            <p id="word-display"></p>
            <input type="text" id="user-input" placeholder="Type the spellings here!" disabled>
            <button id="submit-button" onclick="checkSpelling()" disabled>Submit</button>
            <p id="message"></p>
            <div class="score-box">Skor: <span id="score">0</span> / <span id="total-words">0</span></div>
            <button id="next-word-button" onclick="nextWord()" style="display: none;">Kata Selanjutnya</button>
        </div>

        <div id="result-screen" style="display: none;">
            <h2>Game Done!</h2>
            <p>Your last score: <span id="final-score"></span></p>
            <button onclick="resetGame()">Main Lagi</button>
        </div>
    </div>

    <script>
        // Data kata untuk setiap level
        const WORDS = {
            easy: [
                { word: "apple", hint: "A fruit that has red and green color." },
                { word: "book", hint: "A group of papers combined in one thing." },
                { word: "cat", hint: "A pet that loves to meow." },
                { word: "dog", hint: "A pet that loves to bark." },
                { word: "sun", hint: "The star at the center of our solar system." },
                { word: "car", hint: "A vehicle with four wheels that you drive." },
                { word: "tree", hint: "A tall plant with a trunk and branches." }
            ],
            medium: [
                { word: "banana", hint: "Long and curvy fruit colored yellow." },
                { word: "computer", hint: "Electronic machine that used to prosessing data, searching, and many more!." },
                { word: "family", hint: "A group of peoples that has a close connection." },
                { word: "travel", hint: "Something you do when you go to another city or country." },
                { word: "science", hint: "A subject that tells you about life, energy, and others." },
                { word: "guitar", hint: "A musical instrument with strings that you strum or pluck." },
                { word: "picture", hint: "A visual representation of something, like a photo or drawing." }
            ],
            hard: [
                { word: "sunkist", hint: "A type of orange that can be a juice or eaten." },
                { word: "empathy", hint: "The ability to understand and share the feelings of another." },
                { word: "consequences", hint: "A result of an action" },
                { word: "relevant", hint: "closely connected or appropriate to what is being done or considered." },
                { word: "millennium", hint: "A thousand years period." },
                { word: "rhythm", hint: "A regular, repetitive pattern of movement or sound." },
                { word: "phenomenon", hint: "An observed fact or situation, like a solar eclipse." }
            ]
        };

        let currentLevelWords = [];
        let currentWordIndex = 0;
        let score = 0;
        let timerInterval;
        const TIME_LIMIT = 10; // Batas waktu 10 detik per kata

        // Elemen DOM
        const levelSelection = document.getElementById('level-selection');
        const gameArea = document.getElementById('game-area');
        const resultScreen = document.getElementById('result-screen');
        const wordDisplay = document.getElementById('word-display');
        const userInput = document.getElementById('user-input');
        const timerElement = document.getElementById('timer');
        const messageElement = document.getElementById('message');
        const scoreElement = document.getElementById('score');
        const totalWordsElement = document.getElementById('total-words');
        const submitButton = document.getElementById('submit-button');
        const nextWordButton = document.getElementById('next-word-button');
        const currentLevelElement = document.getElementById('current-level');

        /**
         * Mulai permainan untuk level tertentu.
         * @param {string} level - 'easy', 'medium', atau 'hard'.
         */
        function startGame(level) {
            currentLevelWords = WORDS[level];
            currentWordIndex = 0;
            score = 0;
            
            // Atur tampilan
            levelSelection.style.display = 'none';
            gameArea.style.display = 'block';
            resultScreen.style.display = 'none';

            // Reset UI game area
            scoreElement.textContent = score;
            totalWordsElement.textContent = currentLevelWords.length;
            currentLevelElement.textContent = `Level: ${level.toUpperCase()}`;
            userInput.value = '';
            userInput.disabled = false;
            submitButton.disabled = false;
            messageElement.textContent = '';
            nextWordButton.style.display = 'none';

            loadWord();
        }

        /**
         * Muat kata saat ini dan mulai timer.
         */
        function loadWord() {
            if (currentWordIndex >= currentLevelWords.length) {
                endGame();
                return;
            }

            const currentWordData = currentLevelWords[currentWordIndex];
            
            // Reset UI untuk kata baru
            userInput.value = '';
            userInput.disabled = false;
            submitButton.disabled = false;
            messageElement.textContent = 'Ketik ejaan kata di bawah.';
            messageElement.className = '';
            nextWordButton.style.display = 'none';
            
            // "Disdiktek" kata
            wordDisplay.textContent = `[Kata ke-${currentWordIndex + 1}] Petunjuk: ${currentWordData.hint}`;

            startTimer();
        }

        /**
         * Mulai hitungan mundur 10 detik.
         */
        function startTimer() {
            let timeLeft = TIME_LIMIT;
            timerElement.textContent = timeLeft;
            clearInterval(timerInterval);

            timerInterval = setInterval(() => {
                timeLeft--;
                timerElement.textContent = timeLeft;

                if (timeLeft <= 0) {
                    clearInterval(timerInterval);
                    timerElement.textContent = '0';
                    handleTimeout();
                }
            }, 1000);
        }

        /**
         * Tangani saat waktu habis.
         */
        function handleTimeout() {
            const currentWord = currentLevelWords[currentWordIndex].word;
            messageElement.textContent = `Waktu habis! Jawaban benar adalah: "${currentWord}"`;
            messageElement.classList.add('wrong');
            
            // Nonaktifkan input dan tombol submit setelah waktu habis
            userInput.disabled = true;
            submitButton.disabled = true;

            // Tampilkan tombol "Kata Selanjutnya"
            nextWordButton.style.display = 'block';
        }

        /**
         * Periksa ejaan yang dimasukkan pengguna.
         */
        function checkSpelling() {
            clearInterval(timerInterval); // Hentikan timer

            const currentWord = currentLevelWords[currentWordIndex].word;
            const userSpelling = userInput.value.trim().toLowerCase();
            
            // Nonaktifkan input dan tombol submit
            userInput.disabled = true;
            submitButton.disabled = true;

            if (userSpelling === currentWord) {
                score++;
                scoreElement.textContent = score;
                messageElement.textContent = 'BENAR! 🎉 Ejaan Anda benar.';
                messageElement.classList.add('correct');
            } else {
                messageElement.textContent = `SALAH. Jawaban benar adalah: "${currentWord}"`;
                messageElement.classList.add('wrong');
            }

            // Tampilkan tombol "Kata Selanjutnya"
            nextWordButton.style.display = 'block';
        }

        /**
         * Pindah ke kata berikutnya.
         */
        function nextWord() {
            currentWordIndex++;
            loadWord();
        }

        /**
         * Akhiri permainan dan tampilkan skor akhir.
         */
        function endGame() {
            clearInterval(timerInterval);

            gameArea.style.display = 'none';
            resultScreen.style.display = 'block';
            document.getElementById('final-score').textContent = `${score} / ${currentLevelWords.length}`;
        }

        /**
         * Reset permainan kembali ke layar pemilihan level.
         */
        function resetGame() {
            levelSelection.style.display = 'block';
            gameArea.style.display = 'none';
            resultScreen.style.display = 'none';
            currentWordIndex = 0;
            score = 0;
        }

        // Tambahkan event listener untuk menanggapi tombol Enter
        userInput.addEventListener('keypress', function(event) {
            if (event.key === 'Enter' && !userInput.disabled) {
                checkSpelling();
            }
        });
    </script>
</body>
</html>
