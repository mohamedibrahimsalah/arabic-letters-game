<!DOCTYPE html>
<html lang="en" dir="ltr"> <!-- Changed language to English and direction to LTR -->
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Arabic Letters Game</title> <!-- Changed title to English -->
    <!-- Load Tailwind CSS for flexible and responsive design -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Load Tone.js library for audio effects -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.min.js"></script>
    <!-- Load Google Fonts: Inter and Amiri Quran for better Arabic and English text rendering -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700&family=Amiri+Quran&display=swap" rel="stylesheet">
    <style>
        /* Custom CSS for attractive and responsive appearance */
        body {
            font-family: 'Amiri Quran', 'Inter', sans-serif; /* Using Arabic and English fonts */
            background-color: #f0f4f8; /* Light background color */
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh; /* Full screen height */
            margin: 0;
            padding: 20px;
            box-sizing: border-box;
        }
        .container {
            background-color: #ffffff; /* Container background color */
            border-radius: 20px; /* Rounded corners */
            box-shadow: 0 10px 20px rgba(0, 0, 0, 0.1); /* Soft shadow */
            padding: 30px;
            max-width: 600px; /* Max container width */
            width: 100%;
            text-align: center;
        }
        .letter-display {
            font-size: 8rem; /* Large font size for displayed letter */
            font-weight: bold;
            color: #2c3e50; /* Dark letter color */
            margin-bottom: 30px;
            direction: rtl; /* Ensure Arabic text displays correctly */
        }
        .options-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(120px, 1fr)); /* Responsive grid for options */
            gap: 20px; /* Space between options */
            margin-bottom: 30px;
        }
        .option-button {
            background-color: #e0f7fa; /* Option button background color */
            border: 3px solid #00bcd4; /* Button border */
            border-radius: 15px; /* Rounded corners for button */
            padding: 20px;
            font-size: 4rem; /* Large font size for option text */
            font-weight: bold;
            color: #00796b; /* Option text color */
            cursor: pointer; /* Hand pointer on hover */
            transition: all 0.3s ease; /* Smooth transition on interaction */
            display: flex;
            justify-content: center;
            align-items: center;
            height: 120px;
            user-select: none; /* Prevent text selection */
        }
        .option-button:hover {
            background-color: #b2ebf2; /* Change background color on hover */
            transform: translateY(-5px); /* Lift button slightly */
            box-shadow: 0 5px 15px rgba(0, 188, 212, 0.3); /* Shadow on hover */
        }
        /* Styles for correct and incorrect buttons */
        .option-button.correct {
            background-color: #c8e6c9; /* Green color for correct option */
            border-color: #4caf50;
            color: #2e7d32;
        }
        .option-button.incorrect {
            background-color: #ffcdd2; /* Red color for incorrect option */
            border-color: #f44336;
            color: #d32f2f;
        }
        .feedback {
            font-size: 1.8rem; /* Feedback text size */
            font-weight: bold;
            margin-top: 20px;
            min-height: 40px;
            color: #333;
        }
        .next-button {
            background-color: #4CAF50; /* "Next Letter" button color */
            color: white;
            padding: 15px 30px;
            border: none;
            border-radius: 10px;
            font-size: 1.5rem;
            cursor: pointer;
            transition: background-color 0.3s ease;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }
        .next-button:hover {
            background-color: #45a049; /* Change button color on hover */
        }
        .score-display {
            font-size: 1.5rem;
            color: #555;
            margin-bottom: 20px;
        }
        .instruction-text {
            font-size: 1.8rem;
            color: #333;
            margin-bottom: 20px;
            font-weight: bold;
        }
        .progress-display {
            font-size: 1.2rem;
            color: #777;
            margin-bottom: 10px;
        }

        /* Responsive adjustments for small screens */
        @media (max-width: 640px) {
            .letter-display {
                font-size: 6rem;
            }
            .option-button {
                font-size: 3rem;
                height: 100px;
            }
            .feedback, .instruction-text {
                font-size: 1.5rem;
            }
            .next-button {
                font-size: 1.2rem;
                padding: 12px 25px;
            }
            .options-grid {
                grid-template-columns: 1fr; /* Stack options vertically on small screens */
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1 class="text-4xl font-bold mb-6 text-gray-800">Arabic Letters Game</h1> <!-- Changed title to English -->
        <div class="score-display">Score: <span id="score">0</span></div> <!-- Changed score text to English -->
        <div class="progress-display">Progress: <span id="progress">0/0</span></div> <!-- Added progress display -->
        <div class="instruction-text" id="instruction"></div>
        <div class="letter-display" id="currentLetter"></div>
        <div class="options-grid" id="optionsContainer">
            <!-- Letter options will be dynamically inserted here by JavaScript -->
        </div>
        <div class="feedback" id="feedback"></div>
        <button class="next-button" id="nextButton" style="display: none;">Next Letter</button> <!-- Changed button text to English -->
    </div>

    <script>
        // Global variables for Firebase configuration (provided by Canvas environment)
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        // Tone.js setup for audio effects
        // Two synths are created: one for correct sound, one for incorrect sound.
        const correctSynth = new Tone.Synth().toDestination();
        const incorrectSynth = new Tone.Synth().toDestination();

        // Arabic letter data, including isolated, initial, medial, and final forms.
        // Letters whose shapes change significantly when connected are chosen.
        const arabicLetters = [
            { isolated: 'ب', initial: 'بـ', medial: 'ـبـ', final: 'ـب', name: 'Baa' },
            { isolated: 'ت', initial: 'تـ', medial: 'ـتـ', final: 'ـت', name: 'Taa' },
            { isolated: 'ث', initial: 'ثـ', medial: 'ـثـ', final: 'ـث', name: 'Thaa' },
            { isolated: 'ج', initial: 'جـ', medial: 'ـجـ', final: 'ـج', name: 'Jeem' },
            { isolated: 'ح', initial: 'حـ', medial: 'ـحـ', final: 'ـح', name: 'Haa' },
            { isolated: 'خ', initial: 'خـ', medial: 'ـخـ', final: 'ـخ', name: 'Khaa' },
            { isolated: 'س', initial: 'سـ', medial: 'ـسـ', final: 'ـس', name: 'Seen' },
            { isolated: 'ش', initial: 'شـ', medial: 'ـشـ', final: 'ـش', name: 'Sheen' },
            { isolated: 'ص', initial: 'صـ', medial: 'ـصـ', final: 'ـص', name: 'Saad' },
            { isolated: 'ض', initial: 'ضـ', medial: 'ـضـ', final: 'ـض', name: 'Daad' },
            { isolated: 'ط', initial: 'طـ', medial: 'ـطـ', final: 'ـط', name: 'Taa' },
            { isolated: 'ظ', initial: 'ظـ', medial: 'ـظـ', final: 'ـظ', name: 'Zhaa' },
            { isolated: 'ع', initial: 'عـ', medial: 'ـعـ', final: 'ـع', name: 'Ain' },
            { isolated: 'غ', initial: 'غـ', medial: 'ـغـ', final: 'ـغ', name: 'Ghain' },
            { isolated: 'ف', initial: 'فـ', medial: 'ـفـ', final: 'ـف', name: 'Faa' },
            { isolated: 'ق', initial: 'قـ', medial: 'ـقـ', final: 'ـق', name: 'Qaf' },
            { isolated: 'ك', initial: 'كـ', medial: 'ـكـ', final: 'ـك', name: 'Kaf' },
            { isolated: 'ل', initial: 'لـ', medial: 'ـلـ', final: 'ـل', name: 'Lam' },
            { isolated: 'م', initial: 'مـ', medial: 'ـمـ', final: 'ـم', name: 'Meem' },
            { isolated: 'ن', initial: 'نـ', medial: 'ـنـ', final: 'ـن', name: 'Noon' },
            { isolated: 'ه', initial: 'هـ', medial: 'ـهـ', final: 'ـه', name: 'Haa' },
            { isolated: 'ي', initial: 'يـ', medial: 'ـيـ', final: 'ـي', name: 'Yaa' },
            // Letters that do not connect from the left side (like Alif, Dal, Ra).
            // Included with the note that their "medial" and "final" forms may be similar to the isolated or initial forms.
            { isolated: 'أ', initial: 'أ', medial: 'ـأ', final: 'ـأ', name: 'Alif' }, // Alif with hamza
            { isolated: 'د', initial: 'د', medial: 'ـد', final: 'ـد', name: 'Dal' },
            { isolated: 'ر', initial: 'ر', medial: 'ـر', final: 'ـر', name: 'Ra' },
        ];

        let currentLetterIndex = 0; // Current letter index in the array
        let score = 0; // Player's score
        let currentQuestionType = ''; // Current question type ('initial', 'medial', 'final')
        let currentCorrectAnswer = ''; // Correct answer for the current question

        // Get DOM elements
        const letterDisplay = document.getElementById('currentLetter');
        const optionsContainer = document.getElementById('optionsContainer');
        const feedbackDisplay = document.getElementById('feedback');
        const nextButton = document.getElementById('nextButton');
        const scoreDisplay = document.getElementById('score');
        const instructionText = document.getElementById('instruction');
        const progressDisplay = document.getElementById('progress'); // Get the new progress display element

        // Function to shuffle array elements randomly
        function shuffleArray(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
        }

        // Function to start a new round in the game
        function startNewRound() {
            // Clear previous feedback and styles
            feedbackDisplay.textContent = '';
            nextButton.style.display = 'none'; // Hide "Next Letter" button
            optionsContainer.innerHTML = ''; // Clear previous options
            optionsContainer.querySelectorAll('.option-button').forEach(button => {
                button.classList.remove('correct', 'incorrect'); // Remove correct/incorrect styles
                button.onclick = null; // Remove old event listeners
            });

            // Update progress display
            progressDisplay.textContent = `${currentLetterIndex + 1}/${arabicLetters.length}`;

            // Check if the game has ended
            if (currentLetterIndex >= arabicLetters.length) {
                letterDisplay.textContent = 'Game Over!'; // Changed text to English
                instructionText.textContent = `You scored ${score} points.`; // Changed text to English
                optionsContainer.innerHTML = '';
                nextButton.textContent = 'Play Again'; // Changed button text to English
                nextButton.style.display = 'block';
                nextButton.onclick = () => {
                    currentLetterIndex = 0; // Reset index and score
                    score = 0;
                    scoreDisplay.textContent = score;
                    shuffleArray(arabicLetters); // Shuffle letters again for a new game
                    startNewRound(); // Start a new round
                };
                return;
            }

            const currentLetter = arabicLetters[currentLetterIndex]; // Get the current letter
            letterDisplay.textContent = currentLetter.isolated; // Display the isolated form of the letter

            // Randomly choose question type (initial, medial, or final)
            const questionTypes = ['initial', 'medial', 'final'];
            shuffleArray(questionTypes);
            currentQuestionType = questionTypes[0]; // Choose the first question type after shuffling

            let questionPrompt = '';
            // Determine question text and correct answer based on question type
            switch (currentQuestionType) {
                case 'initial':
                    questionPrompt = `Choose the form of "${currentLetter.isolated}" at the beginning of a word:`; // Changed text to English
                    currentCorrectAnswer = currentLetter.initial;
                    break;
                case 'medial':
                    questionPrompt = `Choose the form of "${currentLetter.isolated}" in the middle of a word:`; // Changed text to English
                    currentCorrectAnswer = currentLetter.medial;
                    break;
                case 'final':
                    questionPrompt = `Choose the form of "${currentLetter.isolated}" at the end of a word:`; // Changed text to English
                    currentCorrectAnswer = currentLetter.final;
                    break;
            }
            instructionText.textContent = questionPrompt; // Display instruction text

            // Set up options, ensuring the correct answer is present
            let options = [currentLetter.initial, currentLetter.medial, currentLetter.final];

            // Filter out duplicate forms if the letter does not have distinct forms for all positions
            let uniqueOptions = Array.from(new Set(options));

            // If unique options are less than 3, fill them with random forms from other letters
            while (uniqueOptions.length < 3) {
                const randomLetter = arabicLetters[Math.floor(Math.random() * arabicLetters.length)];
                const randomFormType = questionTypes[Math.floor(Math.random() * questionTypes.length)];
                const randomForm = randomLetter[randomFormType];
                if (!uniqueOptions.includes(randomForm)) {
                    uniqueOptions.push(randomForm);
                }
            }

            // Ensure the correct answer is always among the options, then shuffle them
            if (!uniqueOptions.includes(currentCorrectAnswer)) {
                uniqueOptions.pop(); // Remove one option to make space if necessary
                uniqueOptions.push(currentCorrectAnswer);
            }
            shuffleArray(uniqueOptions);

            // Create option buttons and add them to the container
            uniqueOptions.forEach(option => {
                const button = document.createElement('button');
                button.className = 'option-button';
                button.textContent = option;
                button.onclick = () => checkAnswer(button, option); // Assign click listener
                optionsContainer.appendChild(button);
            });
        }

        // Function to check the selected answer
        function checkAnswer(selectedButton, selectedOption) {
            // Disable all buttons after selection to prevent multiple clicks
            optionsContainer.querySelectorAll('.option-button').forEach(button => {
                button.onclick = null;
            });

            if (selectedOption === currentCorrectAnswer) {
                feedbackDisplay.textContent = 'Excellent! Correct answer.'; // Changed text to English
                feedbackDisplay.style.color = '#2e7d32'; // Green color
                selectedButton.classList.add('correct'); // Add class for correct styling
                correctSynth.triggerAttackRelease('C5', '8n'); // Play positive sound
                score++; // Increase score
                scoreDisplay.textContent = score; // Update score display
            } else {
                feedbackDisplay.textContent = 'Try again. This is incorrect.'; // Changed text to English
                feedbackDisplay.style.color = '#d32f2f'; // Red color
                selectedButton.classList.add('incorrect'); // Add class for incorrect styling
                incorrectSynth.triggerAttackRelease('C4', '8n'); // Play negative sound

                // Highlight the correct answer
                optionsContainer.querySelectorAll('.option-button').forEach(button => {
                    if (button.textContent === currentCorrectAnswer) {
                        button.classList.add('correct');
                    }
                });
            }
            nextButton.style.display = 'block'; // Show "Next Letter" button
        }

        // Event listener for "Next Letter" button
        nextButton.onclick = () => {
            currentLetterIndex++; // Move to the next letter
            startNewRound(); // Start a new round
        };

        // Initial setup on page load
        window.onload = () => {
            shuffleArray(arabicLetters); // Shuffle letters for random order
            startNewRound(); // Start the first round
        };
    </script>
</body>
</html>
