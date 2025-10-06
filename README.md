<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Лабиринт Алфавита</title>
    <!-- Подключение Tailwind CSS для адаптивности и базового стиля -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@600;800&display=swap');
        
        body {
            font-family: 'Inter', sans-serif;
            background-color: #264653; /* Глубокий сине-зеленый фон */
            overflow-x: hidden;
        }
        
        /* Стилизация кнопки ответа */
        .choice-button {
            transition: all 0.15s ease-in-out;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -2px rgba(0, 0, 0, 0.1);
        }
        .choice-button:hover:not(:disabled) {
            transform: translateY(-2px);
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -4px rgba(0, 0, 0, 0.1);
        }

        /* Анимация движения вперед (для перехода между уровнями) */
        @keyframes success-move {
            0% { transform: translateX(0); }
            50% { transform: translateX(100vw); opacity: 0; }
            51% { transform: translateX(-100vw); opacity: 0; }
            100% { transform: translateX(0); opacity: 1; }
        }

        .moving {
            animation: success-move 0.8s ease-out;
        }

        /* Анимация при ошибке (тряска) */
        @keyframes error-shake {
            0%, 100% { transform: translateX(0); }
            10%, 30%, 50%, 70%, 90% { transform: translateX(-5px); }
            20%, 40%, 60%, 80% { transform: translateX(5px); }
        }

        .shaking {
            animation: error-shake 0.5s;
        }

        /* Стили бокового меню настроек */
        #settings-panel {
            position: fixed;
            top: 0;
            right: 0;
            width: 80%; /* На мобильных */
            max-width: 350px;
            height: 100%;
            background-color: #f7f7f7;
            transform: translateX(100%);
            transition: transform 0.3s ease-in-out;
            z-index: 50;
            box-shadow: -4px 0 15px rgba(0, 0, 0, 0.3);
            overflow-y: auto;
        }

        #settings-panel.open {
            transform: translateX(0);
        }

    </style>
</head>
<body class="flex items-center justify-center min-h-screen p-4">

    <!-- Кнопка открытия меню -->
    <button id="open-settings-btn" class="fixed top-4 right-4 z-40 p-3 bg-yellow-500 hover:bg-yellow-600 text-white rounded-full shadow-lg transition choice-button">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.942 3.333.993 2.404 2.573a1.724 1.724 0 00-1.066 2.573c.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.942-3.333-.993-2.404-2.573a1.724 1.724 0 001.066-2.573z" />
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
        </svg>
    </button>
    
    <!-- Основной контейнер игры -->
    <div id="game-container" class="w-full max-w-lg bg-white rounded-xl shadow-2xl p-6 md:p-10 border-b-8 border-yellow-500">
        
        <!-- Заголовок и индикатор уровня -->
        <h1 class="text-2xl md:text-3xl font-extrabold text-center text-gray-800 mb-6">
            Лабиринт Алфавита
        </h1>

        <!-- Индикатор прогресса -->
        <div id="progress-bar" class="w-full h-2 bg-gray-200 rounded-full mb-8">
            <div id="progress-fill" class="h-2 bg-yellow-500 rounded-full transition-all duration-300" style="width: 0%;"></div>
        </div>

        <!-- Сценарий/Кот -->
        <div id="cat-scene" class="text-center mb-8 relative">
            <div class="text-6xl mb-4" id="cat-emoji">🐈</div>
            <p id="cat-message" class="text-lg text-gray-700 font-semibold h-6">
                Какой путь правильный?
            </p>
        </div>

        <!-- Центральный вопрос (Буква) -->
        <div id="letter-display" class="text-center my-8 p-6 bg-yellow-500 rounded-xl transform rotate-1 shadow-lg">
            <span id="current-letter" class="text-9xl md:text-[120px] font-black text-white leading-none">A</span>
        </div>

        <!-- Кнопки с вариантами ответов -->
        <div id="options-container" class="grid grid-cols-1 gap-4 mt-8">
            <!-- Кнопки будут добавлены сюда с помощью JavaScript -->
        </div>

        <!-- Модальное окно завершения игры -->
        <div id="completion-modal" class="hidden absolute inset-0 bg-white bg-opacity-95 rounded-xl flex flex-col items-center justify-center p-8 text-center">
            <div class="text-7xl mb-4">🏆</div>
            <h2 class="text-3xl font-bold text-gray-800 mb-3">Поздравляем!</h2>
            <p class="text-xl text-gray-600 mb-6">Вы успешно прошли алфавитный лабиринт!</p>
            <button onclick="startGame()" class="bg-green-600 hover:bg-green-700 text-white font-bold py-3 px-6 rounded-lg choice-button transition duration-200">
                Начать заново
            </button>
        </div>

    </div>

    <!-- ---------------------------------------------------- -->
    <!-- БОКОВОЕ МЕНЮ НАСТРОЕК -->
    <!-- ---------------------------------------------------- -->
    <div id="settings-panel">
        <div class="p-6">
            <h2 class="text-2xl font-bold text-gray-800 mb-4 flex justify-between items-center">
                Настройки теста
                <button id="close-settings-btn" class="text-gray-500 hover:text-gray-800">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                    </svg>
                </button>
            </h2>

            <p class="text-sm text-gray-600 mb-4">Выберите буквы, которые будут включены в лабиринт.</p>

            <div id="letter-checkboxes" class="grid grid-cols-4 gap-2 border p-3 rounded-lg bg-white shadow-inner">
                <!-- Чекбоксы будут здесь -->
            </div>

            <div class="mt-6 flex justify-between space-x-2">
                <button onclick="selectAllLetters(true)" class="w-1/2 bg-gray-200 hover:bg-gray-300 text-gray-800 font-semibold py-2 rounded-lg transition choice-button text-sm">
                    Выбрать все
                </button>
                <button onclick="selectAllLetters(false)" class="w-1/2 bg-gray-200 hover:bg-gray-300 text-gray-800 font-semibold py-2 rounded-lg transition choice-button text-sm">
                    Сбросить все
                </button>
            </div>

            <button onclick="saveSettingsAndRestart()" class="w-full mt-6 bg-green-600 hover:bg-green-700 text-white font-bold py-3 rounded-lg choice-button transition duration-200">
                Сохранить и начать тест
            </button>
        </div>
    </div>
    <!-- ---------------------------------------------------- -->

    <script>
        // ----------------------------------------------------
        // 1. ДАННЫЕ ИГРЫ (Буквы и их произношения)
        // ----------------------------------------------------

        const alphabetData = [
            { letter: 'A', correct: 'эй', wrong1: 'ай', wrong2: 'йа' },
            { letter: 'B', correct: 'би', wrong1: 'ба', wrong2: 'вэ' },
            { letter: 'C', correct: 'си', wrong1: 'ца', wrong2: 'ка' },
            { letter: 'D', correct: 'ди', wrong1: 'да', wrong2: 'дэ' },
            { letter: 'E', correct: 'и', wrong1: 'е', wrong2: 'э' },
            { letter: 'F', correct: 'эф', wrong1: 'фа', wrong2: 'фе' },
            { letter: 'G', correct: 'джи', wrong1: 'га', wrong2: 'ге' },
            { letter: 'H', correct: 'эйч', wrong1: 'ха', wrong2: 'аш' },
            { letter: 'I', correct: 'ай', wrong1: 'и', wrong2: 'ей' },
            { letter: 'J', correct: 'джей', wrong1: 'жа', wrong2: 'йот' },
            { letter: 'K', correct: 'кей', wrong1: 'ка', wrong2: 'ки' },
            { letter: 'L', correct: 'эл', wrong1: 'ла', wrong2: 'ли' },
            { letter: 'M', correct: 'эм', wrong1: 'ма', wrong2: 'ми' },
            { letter: 'N', correct: 'эн', wrong1: 'на', wrong2: 'ни' },
            { letter: 'O', correct: 'оу', wrong1: 'о', wrong2: 'у' },
            { letter: 'P', correct: 'пи', wrong1: 'па', wrong2: 'пе' },
            { letter: 'Q', correct: 'кью', wrong1: 'ку', wrong2: 'ква' },
            { letter: 'R', correct: 'ар', wrong1: 'ра', wrong2: 'эр' },
            { letter: 'S', correct: 'эс', wrong1: 'са', wrong2: 'си' },
            { letter: 'T', correct: 'ти', wrong1: 'та', wrong2: 'тэ' },
            { letter: 'U', correct: 'ю', wrong1: 'у', wrong2: 'йу' },
            { letter: 'V', correct: 'ви', wrong1: 'ва', wrong2: 'вэ' },
            { letter: 'W', correct: 'дабл-ю', wrong1: 'да-да-бу', wrong2: 'вэ-вэ' },
            { letter: 'X', correct: 'экс', wrong1: 'кса', wrong2: 'икс' },
            { letter: 'Y', correct: 'уай', wrong1: 'и', wrong2: 'яй' },
            { letter: 'Z', correct: 'зи' , wrong1: 'за', wrong2: 'зе' }
        ];

        // ----------------------------------------------------
        // 2. СОСТОЯНИЕ ИГРЫ И НАСТРОЙКИ
        // ----------------------------------------------------
        let currentLevel = 0;
        let isProcessing = false;
        let activeLetters = []; // Буквы, выбранные в настройках
        let shuffledGameData = []; // Отфильтрованный и перемешанный набор букв для текущей игры

        // ----------------------------------------------------
        // 3. ССЫЛКИ НА ЭЛЕМЕНТЫ DOM
        // ----------------------------------------------------
        const catScene = document.getElementById('cat-scene');
        const catEmoji = document.getElementById('cat-emoji');
        const catMessage = document.getElementById('cat-message');
        const currentLetter = document.getElementById('current-letter');
        const optionsContainer = document.getElementById('options-container');
        const progressBar = document.getElementById('progress-fill');
        const completionModal = document.getElementById('completion-modal');
        const settingsPanel = document.getElementById('settings-panel');
        const openSettingsBtn = document.getElementById('open-settings-btn');
        const closeSettingsBtn = document.getElementById('close-settings-btn');
        const letterCheckboxes = document.getElementById('letter-checkboxes');

        // ----------------------------------------------------
        // 4. ФУНКЦИИ УТИЛИТ
        // ----------------------------------------------------

        /**
         * Перемешивает массив.
         * @param {Array} array - массив для перемешивания.
         * @returns {Array} Перемешанный массив.
         */
        function shuffle(array) {
            // Создаем копию, чтобы не менять исходный массив
            const arr = [...array]; 
            let currentIndex = arr.length, randomIndex;
            while (currentIndex !== 0) {
                randomIndex = Math.floor(Math.random() * currentIndex);
                currentIndex--;
                [arr[currentIndex], arr[randomIndex]] = [
                    arr[randomIndex], arr[currentIndex]];
            }
            return arr;
        }

        // ----------------------------------------------------
        // 5. ЛОГИКА НАСТРОЕК
        // ----------------------------------------------------

        /**
         * Инициализирует UI настроек на основе alphabetData.
         */
        function initializeSettingsUI() {
            // Попытка загрузить сохраненные настройки или использовать все по умолчанию
            const savedSettings = JSON.parse(localStorage.getItem('alphabetSettings'));
            
            letterCheckboxes.innerHTML = '';
            alphabetData.forEach(item => {
                const letter = item.letter;
                const isChecked = savedSettings ? savedSettings.includes(letter) : true;
                
                const label = document.createElement('label');
                label.className = 'flex items-center space-x-1 cursor-pointer bg-blue-100 hover:bg-blue-200 p-2 rounded-md transition text-sm font-semibold';
                label.innerHTML = `
                    <input type="checkbox" data-letter="${letter}" ${isChecked ? 'checked' : ''} class="h-4 w-4 text-blue-600 rounded">
                    <span>${letter}</span>
                `;
                letterCheckboxes.appendChild(label);
            });

            // Обновляем activeLetters
            updateActiveLettersFromUI();
        }

        /**
         * Обновляет массив activeLetters на основе состояния чекбоксов.
         */
        function updateActiveLettersFromUI() {
            activeLetters = Array.from(letterCheckboxes.querySelectorAll('input[type="checkbox"]:checked'))
                .map(input => input.dataset.letter);
        }

        /**
         * Выбирает или сбрасывает все чекбоксы.
         * @param {boolean} select - true для выбора всех, false для сброса.
         */
        function selectAllLetters(select) {
            letterCheckboxes.querySelectorAll('input[type="checkbox"]').forEach(input => {
                input.checked = select;
            });
            updateActiveLettersFromUI();
        }

        /**
         * Сохраняет настройки в localStorage и перезапускает игру.
         */
        function saveSettingsAndRestart() {
            updateActiveLettersFromUI();

            if (activeLetters.length === 0) {
                catMessage.textContent = 'Выберите хотя бы одну букву!';
                return;
            }

            localStorage.setItem('alphabetSettings', JSON.stringify(activeLetters));
            settingsPanel.classList.remove('open');
            startGame();
        }

        // ----------------------------------------------------
        // 6. ОСНОВНАЯ ИГРОВАЯ ЛОГИКА
        // ----------------------------------------------------

        /**
         * Начинает или перезапускает игру.
         */
        function startGame() {
            // 1. Фильтруем и перемешиваем данные на основе настроек
            const filteredLetters = alphabetData.filter(item => activeLetters.includes(item.letter));
            shuffledGameData = shuffle(filteredLetters);
            
            if (shuffledGameData.length === 0) {
                catMessage.textContent = 'Пожалуйста, выберите буквы в настройках. 😥';
                currentLetter.textContent = '?';
                completionModal.classList.add('hidden');
                return;
            }

            currentLevel = 0;
            completionModal.classList.add('hidden');
            catMessage.textContent = `Лабиринт начат (${shuffledGameData.length} букв)!`;
            catEmoji.classList.remove('shaking', 'moving');
            renderLevel();
        }

        /**
         * Обновляет полосу прогресса.
         */
        function updateProgress() {
            if (shuffledGameData.length === 0) return;
            const percentage = (currentLevel / shuffledGameData.length) * 100;
            progressBar.style.width = `${percentage}%`;
        }

        /**
         * Рендерит текущий уровень (букву и варианты ответов).
         */
        function renderLevel() {
            if (currentLevel >= shuffledGameData.length) {
                endGame();
                return;
            }

            isProcessing = false;
            updateProgress();
            optionsContainer.innerHTML = '';
            
            const levelData = shuffledGameData[currentLevel];
            
            // Случайный регистр (заглавная или прописная)
            const displayLetter = Math.random() < 0.5 ? levelData.letter.toLowerCase() : levelData.letter.toUpperCase();
            currentLetter.textContent = displayLetter;
            catMessage.textContent = 'Какой путь правильный?';

            // Формируем список вариантов
            const options = [
                { text: levelData.correct, isCorrect: true },
                { text: levelData.wrong1, isCorrect: false },
                { text: levelData.wrong2, isCorrect: false }
            ];

            // Перемешиваем варианты (требование 1 и 4)
            shuffle(options).forEach(option => {
                const button = document.createElement('button');
                button.textContent = option.text.toUpperCase();
                button.className = 'choice-button w-full py-4 px-4 text-xl font-bold rounded-lg text-white ' + 
                                   'bg-blue-500 hover:bg-blue-600 active:bg-blue-700 disabled:opacity-75';
                
                button.addEventListener('click', () => handleChoice(button, option.isCorrect));
                optionsContainer.appendChild(button);
            });
        }

        /**
         * Обрабатывает выбор пользователя.
         */
        function handleChoice(button, isCorrect) {
            if (isProcessing) return;
            isProcessing = true;
            
            // Отключаем все кнопки, чтобы предотвратить повторные клики
            Array.from(optionsContainer.children).forEach(btn => btn.disabled = true);
            
            if (isCorrect) {
                handleSuccess(button);
            } else {
                handleFailure(button);
            }
        }

        /**
         * Обрабатывает правильный ответ.
         */
        function handleSuccess(button) {
            button.className = button.className.replace('bg-blue-500', 'bg-green-500');
            button.className = button.className.replace('hover:bg-blue-600', 'hover:bg-green-500');
            catMessage.textContent = 'Правильно! Идем дальше! 🎉';
            
            // Анимация движения кота
            catScene.classList.add('moving');

            setTimeout(() => {
                catScene.classList.remove('moving');
                currentLevel++;
                renderLevel(); // Переходим к следующему уровню
            }, 800);
        }

        /**
         * Обрабатывает неправильный ответ.
         */
        function handleFailure(button) {
            button.className = button.className.replace('bg-blue-500', 'bg-red-500');
            button.className = button.className.replace('hover:bg-blue-600', 'hover:bg-red-500');
            
            // Находим и подсвечиваем правильный ответ
            const correctText = shuffledGameData[currentLevel].correct.toUpperCase();
            Array.from(optionsContainer.children).forEach(btn => {
                if (correctText === btn.textContent) {
                    btn.className = btn.className.replace('bg-blue-500', 'bg-green-500');
                    btn.className = btn.className.replace('hover:bg-blue-600', 'hover:bg-green-500');
                }
            });

            catMessage.textContent = 'Ой! Неверно. Начнем сначала. 😟';
            
            // Анимация тряски кота
            catEmoji.classList.add('shaking');

            setTimeout(() => {
                catEmoji.classList.remove('shaking');
                startGame(); // Начинаем игру заново (с новым порядком букв и вариантов)
            }, 1500);
        }

        /**
         * Завершает игру.
         */
        function endGame() {
            updateProgress(); // Заполняем полосу до 100%
            completionModal.classList.remove('hidden');
            catMessage.textContent = 'Финиш!';
        }

        // ----------------------------------------------------
        // 7. ОБРАБОТЧИКИ СОБЫТИЙ МЕНЮ
        // ----------------------------------------------------
        openSettingsBtn.addEventListener('click', () => {
            initializeSettingsUI(); // Обновляем чекбоксы перед открытием
            settingsPanel.classList.add('open');
        });

        closeSettingsBtn.addEventListener('click', () => {
            settingsPanel.classList.remove('open');
            // При закрытии без сохранения, сбрасываем состояние к последнему сохраненному
            initializeSettingsUI();
        });


        // ----------------------------------------------------
        // 8. ЗАПУСК ПРИЛОЖЕНИЯ
        // ----------------------------------------------------
        window.onload = () => {
            // Инициализируем настройки в первый раз и запускаем игру
            initializeSettingsUI();
            if (activeLetters.length === 0) {
                 // Если нет сохраненных настроек, начинаем со всеми буквами
                selectAllLetters(true); 
            }
            startGame();
        };

    </script>

</body>
</html>


