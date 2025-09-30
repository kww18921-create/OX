# OX
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>생태천 OX 스피드 퀴즈 (실시간 공유)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Jua&family=Noto+Sans+KR:wght@400;700&display=swap" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <style>
        body {
            font-family: 'Noto Sans KR', sans-serif;
            overflow: hidden; 
        }
        .font-jua {
            font-family: 'Jua', sans-serif;
        }
        /* 기본 카드 스타일 (글래스모피즘) */
        .card {
            background-color: rgba(255, 255, 255, 0.7);
            backdrop-filter: blur(10px);
            padding: 2rem;
            border-radius: 1.5rem; 
            box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
            border: 1px solid rgba(255, 255, 255, 0.2);
            transition: all 0.5s cubic-bezier(0.25, 0.8, 0.25, 1); 
            width: 100%;
            max-width: 42rem;
            position: absolute; 
            opacity: 0;
            transform: translateY(20px);
            pointer-events: none; 
        }
        .card.active {
            opacity: 1;
            transform: translateY(0);
            pointer-events: auto;
        }
        .btn-choice {
            transition: all 0.2s ease-in-out;
        }
        .btn-choice:hover {
            transform: scale(1.05) translateY(-5px);
            box-shadow: 0 10px 20px rgba(0,0,0,0.1);
        }
        .btn-choice:active {
            transform: scale(0.98);
        }

        /* 랭킹 테이블 스타일 */
        .rank-table th, .rank-table td {
            padding: 0.75rem 0.75rem;
            text-align: center;
        }
        .rank-table tbody tr {
            transition: background-color 0.2s;
        }
        .rank-table tbody tr:hover {
            background-color: rgba(0,0,0,0.05);
        }
        
        /* 모달 스타일 */
        .modal {
            position: fixed; top: 0; left: 0; right: 0; bottom: 0;
            background-color: rgba(0, 0, 0, 0.6);
            display: flex; align-items: center; justify-content: center;
            z-index: 1000;
            opacity: 0;
            transition: opacity 0.3s ease;
            pointer-events: none;
        }
        .modal.active {
            opacity: 1;
            pointer-events: auto;
        }
        .modal-content {
            transform: scale(0.95);
            transition: transform 0.3s ease;
        }
        .modal.active .modal-content {
             transform: scale(1);
        }

        /* 애니메이션 효과 */
        @keyframes shake {
            0%, 100% { transform: translateX(0); }
            10%, 30%, 50%, 70%, 90% { transform: translateX(-5px); }
            20%, 40%, 60%, 80% { transform: translateX(5px); }
        }
        .animate-shake {
            animation: shake 0.5s ease-in-out;
        }

        /* 정답/오답 피드백 오버레이 */
        .feedback-overlay {
            position: absolute;
            top: 0; left: 0; right: 0; bottom: 0;
            border-radius: 1.5rem;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 10rem;
            color: white;
            opacity: 0;
            transition: opacity 0.3s;
            pointer-events: none;
        }
        .feedback-overlay.correct { background-color: rgba(16, 185, 129, 0.7); }
        .feedback-overlay.incorrect { background-color: rgba(239, 68, 68, 0.7); }
        .feedback-overlay.show { opacity: 1; }

        /* 생물 애니메이션 - 최소화 (Firestore 코드 공간 확보를 위해) */
        .creature-container {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%; overflow: hidden; z-index: 0; pointer-events: none;
        }
        .duck { width: 100px; bottom: 25%; position: absolute; left: -150px; animation: float 25s linear infinite; animation-delay: 2s; }
        @keyframes float {
            0% { transform: translateX(0) translateY(0) rotate(-2deg); }
            50% { transform: translateX(calc(100vw + 100px)) translateY(-10px) rotate(2deg); }
            100% { transform: translateX(0) translateY(0) rotate(-2deg); }
        }
    </style>
</head>
<body class="bg-gradient-to-br from-emerald-100 via-cyan-100 to-sky-200 flex items-center justify-center min-h-screen p-4">

    <div id="app-container" class="relative container mx-auto flex justify-center items-center h-full" style="height: 600px;">
        
        <div id="start-screen" class="card active text-center">
            
            <div class="creature-container">
                <!-- 단순화된 오리 SVG -->
                <svg viewBox="0 0 128 128" class="duck">
                    <path fill="#FFD95A" d="M107.2,62.7c0.2,1.3-0.9,4.4-1.1,4.6c-1,1.1-6,2.2-6.2,2.2c-0.2,0-3.2-1.2-5.1-2.4c-4-2.5-5.6-5.8-5.6-5.8s0.3,4.6-2.9,9.2c-2.3,3.3-6.1,6-11,7.2c-8.5,2.1-16.1-0.2-22.3-5.2c-5.7-4.6-9.5-11.2-10.4-18.4c-0.9-7.2,1-15.1,5.3-20.9c4.3-5.8,10.7-9,17.7-8.5c7,0.5,13.4,4.2,17.7,10c0.9,1.2,1.6,2.4,2.3,3.7c0,0-2.3-1.4-3.7-1.2c-1.4,0.2-2.1,1.1-2.1,1.1s1.8-0.9,2.9,0c1.1,0.9,0.7,2.1,0.7,2.1l4.2,3.3C104.9,59.2,107,61.4,107.2,62.7z"/>
                    <circle fill="#2F333A" cx="99.5" cy="59.9" r="2.5"/>
                </svg>
            </div>

            <h1 class="text-5xl font-jua text-emerald-700 mb-4 flex items-center justify-center gap-3">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-10 w-10" viewBox="0 0 20 20" fill="currentColor"><path fill-rule="evenodd" d="M17.707 9.293a1 1 0 010 1.414l-7 7a1 1 0 01-1.414 0l-7-7A.997.997 0 012 10V5a1 1 0 011-1h14a1 1 0 011 1v4.293zM5 6a1 1 0 100 2 1 1 0 000-2zm4 0a1 1 0 100 2 1 1 0 000-2zm4 0a1 1 0 100 2 1 1 0 000-2z" clip-rule="evenodd" /></svg>
                생태천 OX 스피드 퀴즈
            </h1>
            <p class="text-gray-600 mb-8 text-lg">실시간 랭킹으로 모두와 경쟁하세요!</p>
            <div class="space-y-4">
                <input type="text" id="userId" placeholder="아이디 (예: 홍길동)" class="w-full p-3 border-2 border-gray-200 rounded-lg focus:outline-none focus:ring-4 focus:ring-emerald-300 transition-all duration-300">
                <input type="tel" id="userPhone" placeholder="전화번호 (예: 01012345678)" class="w-full p-3 border-2 border-gray-200 rounded-lg focus:outline-none focus:ring-4 focus:ring-emerald-300 transition-all duration-300">
                <button id="start-btn" class="w-full bg-emerald-600 text-white font-bold py-4 px-6 rounded-lg hover:bg-emerald-700 transition-all duration-300 text-xl shadow-lg hover:shadow-xl transform hover:-translate-y-1">퀴즈 시작!</button>
            </div>
             <p id="start-error" class="text-red-500 mt-4 text-sm h-5"></p>
             <div class="mt-6 flex justify-between items-center text-sm">
                <p id="current-user-id" class="text-xs text-gray-500">User ID: N/A (연결 중...)</p>
                <button id="admin-mode-btn" class="text-gray-500 hover:text-gray-700 underline">관리자 모드</button>
             </div>
        </div>

        <div id="quiz-screen" class="card">
            <div class="feedback-overlay" id="feedback-overlay"></div>
            <div class="flex justify-between items-center mb-6">
                <div id="question-counter" class="text-xl font-bold text-gray-700 bg-white/50 px-4 py-2 rounded-full"></div>
                <div id="timer" class="text-3xl font-jua text-emerald-600">0.00초</div>
            </div>
            <div class="bg-white/50 p-6 rounded-lg min-h-[200px] flex items-center justify-center mb-6 shadow-inner">
                <p id="question-text" class="text-3xl font-bold text-center text-gray-800 leading-normal"></p>
            </div>
            <div class="grid grid-cols-2 gap-6">
                <button data-answer="true" class="btn-choice text-8xl font-jua bg-blue-500 text-white rounded-2xl py-12 hover:bg-blue-600 shadow-lg">O</button>
                <button data-answer="false" class="btn-choice text-8xl font-jua bg-red-500 text-white rounded-2xl py-12 hover:bg-red-600 shadow-lg">X</button>
            </div>
        </div>

        <div id="result-screen" class="card text-center">
            <h2 class="text-4xl font-jua text-emerald-700 mb-6">퀴즈 완료! 수고하셨습니다!</h2>
            <div class="flex justify-center gap-6 mb-8">
                <div class="bg-white/50 p-6 rounded-lg flex-1">
                    <p class="text-lg text-gray-600 mb-2">정답 개수</p> 
                    <span id="score" class="font-bold text-5xl text-blue-600 font-jua"></span>
                </div>
                <div class="bg-white/50 p-6 rounded-lg flex-1">
                    <p class="text-lg text-gray-600 mb-2">소요 시간</p> 
                    <span id="time-taken" class="font-bold text-5xl text-red-600 font-jua"></span>
                </div>
            </div>

            <h3 class="text-2xl font-jua text-gray-800 mb-4 flex items-center justify-center gap-2">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-amber-500" viewBox="0 0 20 20" fill="currentColor"><path d="M9.049 2.927c.3-.921 1.603-.921 1.902 0l1.07 3.292a1 1 0 00.95.69h3.462c.969 0 1.371 1.24.588 1.81l-2.8 2.034a1 1 0 00-.364 1.118l1.07 3.292c.3.921-.755 1.688-1.54 1.118l-2.8-2.034a1 1 0 00-1.175 0l-2.8 2.034c-.784.57-1.838-.197-1.539-1.118l1.07-3.292a1 1 0 00-.364-1.118L2.98 8.72c-.783-.57-.38-1.81.588-1.81h3.461a1 1 0 00.951-.69l1.07-3.292z" /></svg>
                명예의 전당 (TOP 10)
            </h3>
            <div class="overflow-x-auto max-h-60">
                <table id="rank-table" class="rank-table w-full text-base text-left text-gray-700">
                    <thead class="text-sm text-gray-800 uppercase bg-gray-200/70 sticky top-0">
                        <tr><th>순위</th><th>아이디</th><th>정답</th><th>기록(초)</th></tr>
                    </thead>
                    <tbody id="rank-body">
                        <tr><td colspan="4">랭킹을 불러오는 중...</td></tr>
                    </tbody>
                </table>
            </div>
            <button id="restart-btn" class="mt-8 w-full bg-emerald-600 text-white font-bold py-4 px-6 rounded-lg hover:bg-emerald-700 transition-all duration-300 text-xl shadow-lg hover:shadow-xl transform hover:-translate-y-1">처음으로 돌아가기</button>
        </div>

        <div id="admin-screen" class="card">
            <h2 class="text-3xl font-jua text-blue-700 mb-6">관리자 페이지 - 전체 기록</h2>
            <div class="overflow-y-auto max-h-[450px]">
                <table id="admin-rank-table" class="rank-table w-full text-sm text-left text-gray-600">
                    <thead class="text-xs text-gray-700 uppercase bg-blue-100/70 sticky top-0">
                        <tr><th>순위</th><th>아이디</th><th>전화번호</th><th>정답</th><th>기록(초)</th></tr>
                    </thead>
                    <tbody id="admin-rank-body">
                        <tr><td colspan="5">기록을 불러오는 중...</td></tr>
                    </tbody>
                </table>
            </div>
            <button id="admin-back-btn" class="mt-8 w-full bg-gray-600 text-white font-bold py-3 px-6 rounded-lg hover:bg-gray-700 text-lg">처음으로</button>
        </div>
    </div>

    <div id="admin-login-modal" class="modal">
        <div class="modal-content bg-white p-8 rounded-lg shadow-xl w-full max-w-sm text-center">
            <h3 class="text-2xl font-jua mb-4">관리자 로그인</h3>
            <input type="password" id="admin-password" placeholder="비밀번호" class="w-full p-3 border-2 border-gray-300 rounded-lg mb-2 focus:outline-none focus:ring-4 focus:ring-blue-300">
            <p id="admin-error" class="text-red-500 text-sm h-5 mb-4"></p>
            <div class="flex gap-4">
                <button id="admin-login-btn" class="w-full bg-blue-600 text-white font-bold py-2 rounded-lg hover:bg-blue-700">로그인</button>
                <button id="admin-close-btn" class="w-full bg-gray-300 text-gray-800 font-bold py-2 rounded-lg hover:bg-gray-400">닫기</button>
            </div>
        </div>
    </div>


    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, query, onSnapshot, setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // --- Firebase 초기 설정 (필수 전역 변수 사용) ---
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : '';
        
        // Firebase 초기화
        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        const auth = getAuth(app);
        
        // Firestore 컬렉션 참조 (모두가 공유하는 공개 데이터)
        const RANKINGS_COLLECTION = `artifacts/${appId}/public/data/quiz_rankings`;
        const rankingsCollectionRef = collection(db, RANKINGS_COLLECTION);
        
        setLogLevel('error'); // 배포 시에는 'silent'로 변경 가능
        
        // --- 퀴즈 데이터 ---
        const quizData = [
            { question: "생태하천은 오염된 물을 스스로 정화하는 능력이 있다.", answer: true },
            { question: "모든 하천의 물고기는 1급수에서만 살 수 있다.", answer: false },
            { question: "수질 정화를 위해 하천 바닥을 모두 콘크리트로 덮는 것이 좋다.", answer: false },
            { question: "버드나무와 갈대는 수질 정화에 도움을 주는 대표적인 수생식물이다.", answer: true },
            { question: "생태하천 복원 사업은 동식물의 서식지를 파괴하는 주된 원인이다.", answer: false },
            { question: "빗물이 하천으로 바로 유입되면 수질 오염의 원인이 될 수 있다.", answer: true },
            { question: "하천에 사는 잠자리의 애벌레는 물 밖 풀숲에서 생활한다.", answer: false },
            { question: "생활하수를 정화 없이 하천으로 흘려보내는 것은 생태계를 풍요롭게 한다.", answer: false },
            { question: "생태하천의 돌과 자갈은 미생물이 살 공간을 제공하여 물을 깨끗하게 한다.", answer: true },
            { question: "생태하천에서는 어떠한 경우에도 낚시나 물놀이가 금지되어 있다.", answer: false }
        ];

        // --- DOM 요소 ---
        const screens = {
            start: document.getElementById('start-screen'),
            quiz: document.getElementById('quiz-screen'),
            result: document.getElementById('result-screen'),
            admin: document.getElementById('admin-screen'),
        };
        const adminLoginModal = document.getElementById('admin-login-modal');
        
        const startBtn = document.getElementById('start-btn');
        const restartBtn = document.getElementById('restart-btn');
        const choiceBtns = document.querySelectorAll('.btn-choice');
        const adminModeBtn = document.getElementById('admin-mode-btn');
        const adminLoginBtn = document.getElementById('admin-login-btn');
        const adminCloseBtn = document.getElementById('admin-close-btn');
        const adminBackBtn = document.getElementById('admin-back-btn');
        
        const userIdInput = document.getElementById('userId');
        const userPhoneInput = document.getElementById('userPhone');
        const startError = document.getElementById('start-error');
        const questionCounter = document.getElementById('question-counter');
        const timerDisplay = document.getElementById('timer');
        const questionText = document.getElementById('question-text');
        const scoreDisplay = document.getElementById('score');
        const timeTakenDisplay = document.getElementById('time-taken');
        const rankTableBody = document.getElementById('rank-body');
        const adminPasswordInput = document.getElementById('admin-password');
        const adminError = document.getElementById('admin-error');
        const adminRankBody = document.getElementById('admin-rank-body');
        const feedbackOverlay = document.getElementById('feedback-overlay');
        const currentUserIdDisplay = document.getElementById('current-user-id');
        
        let currentQuestionIndex, score, timerInterval, startTime;
        let isAnswering = false; 
        let currentUserId = null; // 인증된 User ID

        /**
         * rawPhone: 하이픈이 있거나 없는 전화번호 입력 문자열
         * 반환: 하이픈이 포함된 정규화된 전화번호 문자열 (저장용)
         */
        function getFormattedPhone(rawPhone) {
            const cleaned = rawPhone.replace(/[^0-9]/g, '');
            if (cleaned.length === 11 && cleaned.startsWith('01')) { 
                return cleaned.replace(/(\d{3})(\d{4})(\d{4})/, '$1-$2-$3');
            } else if (cleaned.length === 10 && cleaned.startsWith('01')) { 
                return cleaned.replace(/(\d{3})(\d{3})(\d{4})/, '$1-$2-$3');
            }
            return cleaned; 
        }

        // --- 화면 전환 함수 ---
        function showScreen(screenName) {
            Object.values(screens).forEach(screen => screen.classList.remove('active'));
            screens[screenName].classList.add('active');
        }
        
        // --- 게임 로직 ---
        function startGame() {
            if (!currentUserId) {
                startError.textContent = '아직 사용자 연결이 완료되지 않았습니다. 잠시 후 다시 시도해주세요.';
                return;
            }

            const id = userIdInput.value.trim();
            const phone = userPhoneInput.value.trim();
            
            // 1. 입력 유효성 검사
            if (!id || !phone) {
                startError.textContent = '아이디와 전화번호를 모두 입력해주세요.'; return;
            }

            // 2. 전화번호 정리 및 길이 검증
            const cleanedPhone = phone.replace(/[^0-9]/g, '');
            if (cleanedPhone.length < 10 || cleanedPhone.length > 11) {
                startError.textContent = '전화번호는 10~11자리 숫자여야 합니다.';
                userIdInput.parentElement.classList.add('animate-shake'); 
                setTimeout(() => userIdInput.parentElement.classList.remove('animate-shake'), 500);
                return;
            }
            
            startError.textContent = '';
            currentQuestionIndex = 0;
            score = 0;
            isAnswering = false;

            showScreen('quiz');
            displayQuestion();
            startTimer();
        }

        function displayQuestion() {
            if (currentQuestionIndex < quizData.length) {
                const currentQuestion = quizData[currentQuestionIndex];
                questionText.textContent = currentQuestion.question;
                questionCounter.textContent = `${currentQuestionIndex + 1} / ${quizData.length}`;
                questionText.parentElement.classList.remove('animate-shake'); 
            } else {
                endGame();
            }
        }

        function selectAnswer(e) {
            if (isAnswering) return; 
            isAnswering = true;

            const selectedAnswer = e.target.dataset.answer === 'true';
            const correctAnswer = quizData[currentQuestionIndex].answer;

            if (selectedAnswer === correctAnswer) {
                score++;
                showFeedback(true);
            } else {
                showFeedback(false);
            }
            
            // 0.5초 후 다음 문제로
            setTimeout(() => {
                currentQuestionIndex++;
                displayQuestion();
                isAnswering = false; 
            }, 500);
        }

        function showFeedback(isCorrect) {
            feedbackOverlay.innerHTML = isCorrect ? 'O' : 'X';
            feedbackOverlay.className = 'feedback-overlay show';
            feedbackOverlay.classList.add(isCorrect ? 'correct' : 'incorrect');

            if (!isCorrect) {
                questionText.parentElement.classList.add('animate-shake');
            }

            setTimeout(() => {
                feedbackOverlay.classList.remove('show');
            }, 500);
        }

        function startTimer() {
            startTime = Date.now();
            timerDisplay.textContent = '0.00초';
            timerInterval = setInterval(() => {
                const elapsedTime = (Date.now() - startTime) / 1000;
                timerDisplay.textContent = `${elapsedTime.toFixed(2)}초`;
            }, 10);
        }

        async function endGame() {
            clearInterval(timerInterval);
            const totalTime = ((Date.now() - startTime) / 1000);
            
            showScreen('result');

            // 점수와 시간 애니메이션
            animateValue(scoreDisplay, 0, score, 500, ` / ${quizData.length}`);
            animateValue(timeTakenDisplay, 0, totalTime, 500, '초', true);

            await saveRecord(totalTime);
            
            // 축하 효과
            confetti({ particleCount: 150, spread: 90, origin: { y: 0.6 } });
        }

        function animateValue(obj, start, end, duration, suffix = '', isFloat = false) {
            let startTimestamp = null;
            const step = (timestamp) => {
                if (!startTimestamp) startTimestamp = timestamp;
                const progress = Math.min((timestamp - startTimestamp) / duration, 1);
                let currentValue = progress * (end - start) + start;
                obj.innerHTML = (isFloat ? currentValue.toFixed(2) : Math.floor(currentValue)) + suffix;
                if (progress < 1) {
                    window.requestAnimationFrame(step);
                }
            };
            window.requestAnimationFrame(step);
        }
        
        /** Firestore에 랭킹 데이터를 저장 */
        async function saveRecord(time) {
            const rawPhone = userPhoneInput.value.trim();
            const formattedPhone = getFormattedPhone(rawPhone);

            const record = {
                id: userIdInput.value.trim(),
                phone: formattedPhone, // 정규화된 전화번호 사용
                score: score,
                time: parseFloat(time.toFixed(2)),
                createdAt: new Date().toISOString(),
                userId: currentUserId // 현재 사용자 ID 저장
            };

            try {
                // Firestore에 새 문서 추가
                await addDoc(rankingsCollectionRef, record);
                console.log("Record successfully written to Firestore!");
            } catch (e) {
                console.error("Error adding document: ", e);
                // 사용자에게 오류 메시지 표시
                startError.textContent = '⚠️ 기록 저장 중 오류가 발생했습니다. (콘솔 확인)';
            }
        }

        /** 랭킹 Snapshot을 처리하고 UI를 업데이트하는 함수 */
        function handleRankingsSnapshot(snapshot) {
            const rawRankings = [];
            snapshot.forEach(doc => {
                rawRankings.push({ docId: doc.id, ...doc.data() });
            });
            
            // 1. 데이터 정렬 (점수 내림차순, 시간 오름차순)
            const sortedRankings = rawRankings.sort((a, b) => {
                if (b.score !== a.score) {
                    return b.score - a.score;
                }
                return a.time - b.time;
            });

            // 2. 일반 사용자 랭킹 업데이트 (TOP 10)
            displayRankings(sortedRankings);
            
            // 3. 관리자 랭킹 업데이트 (전체)
            displayAdminRankings(sortedRankings);
        }

        /** 실시간 랭킹 리스너 설정 */
        function setupRankingsListener() {
            // 쿼리 (정렬을 하지 않고 모든 데이터를 가져온 후 JS에서 정렬)
            const q = query(rankingsCollectionRef);
            
            // onSnapshot 리스너를 설정하여 실시간 업데이트
            onSnapshot(q, handleRankingsSnapshot, (error) => {
                console.error("Error listening to rankings:", error);
                rankTableBody.innerHTML = '<tr><td colspan="4" class="text-red-500">데이터 로드 중 오류 발생</td></tr>';
            });
        }


        /** 일반 사용자용 랭킹 (TOP 10) 표시 */
        function displayRankings(rankings) {
            rankTableBody.innerHTML = '';
            if (rankings.length === 0) {
                 rankTableBody.innerHTML = '<tr><td colspan="4">아직 등록된 기록이 없습니다.</td></tr>'; return;
            }
            // 상위 10개만 표시
            rankings.slice(0, 10).forEach((rank, i) => {
                const tr = document.createElement('tr');
                // 현재 사용자 기록은 강조
                const isCurrentUser = rank.userId === currentUserId;
                tr.className = isCurrentUser ? 'bg-yellow-100 font-bold' : '';

                tr.innerHTML = `
                    <td class="font-bold text-lg">${i + 1}</td>
                    <td>${rank.id}</td>
                    <td class="font-bold text-blue-600">${rank.score}</td>
                    <td class="text-red-600">${rank.time.toFixed(2)}</td>
                `;
                rankTableBody.appendChild(tr);
            });
        }

        function restartGame() {
            showScreen('start');
            userIdInput.value = '';
            userPhoneInput.value = '';
        }

        // --- 관리자 기능 ---
        function showAdminLogin() {
            adminPasswordInput.value = '';
            adminError.textContent = '';
            adminLoginModal.classList.add('active');
        }

        function handleAdminLogin() {
            // 관리자 비밀번호: 1212
            if (adminPasswordInput.value === '1212') { 
                adminLoginModal.classList.remove('active');
                showScreen('admin');
                // displayAdminRankings는 onSnapshot에서 이미 처리됨
            } else {
                adminPasswordInput.parentElement.classList.add('animate-shake');
                adminError.textContent = '비밀번호가 올바르지 않습니다.';
                setTimeout(() => adminPasswordInput.parentElement.classList.remove('animate-shake'), 500);
            }
        }

        /** 관리자용 전체 랭킹 표시 */
        function displayAdminRankings(rankings) {
            adminRankBody.innerHTML = '';
            if (rankings.length === 0) {
                 adminRankBody.innerHTML = '<tr><td colspan="5">등록된 기록이 없습니다.</td></tr>'; return;
            }
            rankings.forEach((rank, index) => {
                const tr = document.createElement('tr');
                 const isCurrentUser = rank.userId === currentUserId;
                tr.className = isCurrentUser ? 'bg-yellow-100 font-bold' : '';
                tr.innerHTML = `
                    <td class="font-bold">${index + 1}</td>
                    <td>${rank.id}</td>
                    <td>${rank.phone}</td>
                    <td class="font-bold text-blue-600">${rank.score}</td>
                    <td class="text-red-600">${rank.time.toFixed(2)}</td>
                `;
                adminRankBody.appendChild(tr);
            });
        }


        // --- 초기화 및 이벤트 리스너 설정 ---

        // Firebase 인증 및 리스너 설정
        async function initFirebase() {
            try {
                // 1. 초기 인증 시도 (커스텀 토큰 사용 또는 익명 로그인)
                if (initialAuthToken.length > 0) {
                    await signInWithCustomToken(auth, initialAuthToken).catch(e => {
                        console.warn("Custom token sign in failed, falling back to anonymous:", e);
                        return signInAnonymously(auth);
                    });
                } else {
                    await signInAnonymously(auth);
                }

                // 2. 인증 상태 변경 리스너 설정
                onAuthStateChanged(auth, (user) => {
                    if (user) {
                        currentUserId = user.uid;
                        currentUserIdDisplay.textContent = `User ID: ${currentUserId}`;
                        setupRankingsListener(); // 인증 완료 후 실시간 리스너 시작
                    } else {
                        currentUserId = null;
                        currentUserIdDisplay.textContent = 'User ID: N/A (로그인 실패)';
                    }
                });
            } catch (error) {
                console.error("Firebase Initialization or Sign-in Error:", error);
                currentUserIdDisplay.textContent = 'User ID: N/A (DB 연결 오류)';
                startError.textContent = '데이터베이스 연결에 문제가 발생했습니다.';
            }
        }


        // 이벤트 리스너
        startBtn.addEventListener('click', startGame);
        restartBtn.addEventListener('click', restartGame);
        choiceBtns.forEach(button => button.addEventListener('click', selectAnswer));
        adminModeBtn.addEventListener('click', showAdminLogin);
        adminCloseBtn.addEventListener('click', () => adminLoginModal.classList.remove('active'));
        adminLoginBtn.addEventListener('click', handleAdminLogin);
        adminBackBtn.addEventListener('click', restartGame);
        adminPasswordInput.addEventListener('keyup', (e) => e.key === 'Enter' && handleAdminLogin());
        
        // 앱 시작
        initFirebase();
    </script>
</body>
</html>

