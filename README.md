# todolist
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>심플 투두 리스트</title>
    <!-- Tailwind CSS 로드 -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Google Fonts (Inter) 로드 -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <!-- Phosphor Icons 로드 -->
    <script src="https://unpkg.com/@phosphor-icons/web"></script>
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #e2e8f0;
            margin: 0;
            overflow: hidden; /* 배경 애니메이션이 넘어가지 않도록 방지 */
        }
        
        /* 동적 그라데이션 배경 애니메이션 */
        .bg-shape1 {
            position: absolute;
            top: -10%; left: -10%; width: 50vw; height: 50vw;
            background: radial-gradient(circle, rgba(167,139,250,0.5) 0%, rgba(167,139,250,0) 70%);
            border-radius: 50%;
            animation: float1 15s infinite alternate ease-in-out;
            z-index: -1;
        }
        .bg-shape2 {
            position: absolute;
            bottom: -10%; right: -10%; width: 60vw; height: 60vw;
            background: radial-gradient(circle, rgba(96,165,250,0.5) 0%, rgba(96,165,250,0) 70%);
            border-radius: 50%;
            animation: float2 20s infinite alternate-reverse ease-in-out;
            z-index: -1;
        }
        .bg-shape3 {
            position: absolute;
            top: 40%; left: 30%; width: 40vw; height: 40vw;
            background: radial-gradient(circle, rgba(244,114,182,0.4) 0%, rgba(244,114,182,0) 70%);
            border-radius: 50%;
            animation: float3 18s infinite alternate ease-in-out;
            z-index: -1;
        }

        @keyframes float1 { 0% { transform: translate(0, 0) scale(1); } 100% { transform: translate(10%, 10%) scale(1.2); } }
        @keyframes float2 { 0% { transform: translate(0, 0) scale(1); } 100% { transform: translate(-10%, -10%) scale(1.1); } }
        @keyframes float3 { 0% { transform: translate(0, 0) scale(1); } 100% { transform: translate(20%, -20%) scale(1.3); } }

        /* 메인 글래스 컨테이너 */
        .glass-container {
            background: rgba(255, 255, 255, 0.7);
            backdrop-filter: blur(20px);
            -webkit-backdrop-filter: blur(20px);
            border: 1px solid rgba(255, 255, 255, 0.6);
            box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.1), 0 0 0 1px rgba(255,255,255,0.3) inset;
        }

        /* 가로 스크롤 영역 스크롤바 숨기기 (카테고리 선택 및 필터용) */
        .no-scrollbar::-webkit-scrollbar {
            display: none;
        }
        .no-scrollbar {
            -ms-overflow-style: none;  /* IE and Edge */
            scrollbar-width: none;  /* Firefox */
        }

        /* 카테고리 필터 활성화 상태 */
        .category-filter-btn.active-cat {
            background-color: #4f46e5; /* indigo-600 */
            color: white;
            border-color: transparent;
            box-shadow: 0 4px 12px rgba(79, 70, 229, 0.15);
        }

        /* 커스텀 스크롤바 */
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: rgba(156, 163, 175, 0.5); border-radius: 10px; }
        ::-webkit-scrollbar-thumb:hover { background: rgba(156, 163, 175, 0.8); }

        /* 항목 등장/퇴장 부드러운 애니메이션 */
        .todo-item {
            transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            opacity: 1;
            transform: scale(1) translateY(0);
        }
        .todo-item.fade-out {
            opacity: 0;
            transform: scale(0.9) translateX(30px);
        }
        
        /* 텍스트 취소선 애니메이션 */
        .todo-text {
            transition: color 0.3s ease;
            position: relative;
            display: inline-block;
        }
        .todo-text::after {
            content: '';
            position: absolute;
            left: 0; top: 50%; width: 0; height: 2px;
            background-color: #9ca3af;
            transition: width 0.3s cubic-bezier(0.4, 0, 0.2, 1);
        }
        .completed .todo-text::after { width: 100%; }
        
        /* 체크 아이콘 팝 애니메이션 */
        .check-icon {
            transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            transform: scale(0.5);
            opacity: 0;
        }
        .completed .check-icon {
            transform: scale(1);
            opacity: 1;
        }
        
        /* 필터 버튼 활성화 트랜지션 */
        .filter-btn {
            transition: all 0.3s ease;
        }
        .filter-btn.active {
            background: white;
            color: #4f46e5; /* indigo-600 */
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
        }
    </style>
</head>
<body class="min-h-screen flex items-center justify-center p-4 relative">

    <div class="bg-shape1"></div>
    <div class="bg-shape2"></div>
    <div class="bg-shape3"></div>

    <div class="glass-container rounded-[2rem] w-full max-w-md overflow-hidden flex flex-col h-[85vh] max-h-[850px] relative z-10">
        
        <!-- 프리미엄 헤더 영역 -->
        <header class="p-8 pb-5">
            <div class="flex justify-between items-end mb-2">
                <div>
                    <p class="text-indigo-500 font-semibold text-xs tracking-widest uppercase mb-1" id="date-display">로딩 중...</p>
                    <h1 class="text-4xl font-extrabold tracking-tight bg-clip-text text-transparent bg-gradient-to-r from-indigo-600 to-purple-600">My Tasks</h1>
                </div>
                <div class="flex flex-col items-end">
                    <div class="bg-indigo-100/80 text-indigo-700 px-3 py-1 rounded-full text-xs font-bold mb-1 shadow-sm">
                        <span id="active-tasks-count">0</span>개 남음
                    </div>
                    <p class="text-xs text-slate-500 font-medium">총 <span id="total-tasks-count">0</span>개</p>
                </div>
            </div>
        </header>

        <!-- 세련된 알약 형태의 필터 탭 -->
        <div class="px-8 pb-5">
            <div class="flex bg-slate-200/50 p-1.5 rounded-2xl gap-1">
                <button class="filter-btn active flex-1 py-2.5 text-sm font-semibold rounded-xl text-slate-500 hover:text-slate-800" data-filter="all">
                    전체
                </button>
                <button class="filter-btn flex-1 py-2.5 text-sm font-semibold rounded-xl text-slate-500 hover:text-slate-800" data-filter="active">
                    진행중
                </button>
                <button class="filter-btn flex-1 py-2.5 text-sm font-semibold rounded-xl text-slate-500 hover:text-slate-800" data-filter="completed">
                    완료
                </button>
            </div>
        </div>

        <!-- 세련된 가로 스크롤 카테고리 필터 -->
        <div class="px-8 pb-4">
            <div class="flex gap-1.5 overflow-x-auto no-scrollbar py-0.5" id="category-filter-container">
                <button class="category-filter-btn active-cat flex-shrink-0 px-3.5 py-1.5 text-xs font-bold rounded-full bg-white/50 text-slate-600 border border-slate-100 transition-all" data-cat-filter="all">
                    전체 카테고리
                </button>
                <button class="category-filter-btn flex-shrink-0 px-3.5 py-1.5 text-xs font-bold rounded-full bg-white/40 text-slate-500 hover:bg-white/60 border border-slate-100/50 transition-all flex items-center gap-1.5" data-cat-filter="work">
                    <span class="w-1.5 h-1.5 rounded-full bg-indigo-500"></span>업무
                </button>
                <button class="category-filter-btn flex-shrink-0 px-3.5 py-1.5 text-xs font-bold rounded-full bg-white/40 text-slate-500 hover:bg-white/60 border border-slate-100/50 transition-all flex items-center gap-1.5" data-cat-filter="personal">
                    <span class="w-1.5 h-1.5 rounded-full bg-emerald-500"></span>개인
                </button>
                <button class="category-filter-btn flex-shrink-0 px-3.5 py-1.5 text-xs font-bold rounded-full bg-white/40 text-slate-500 hover:bg-white/60 border border-slate-100/50 transition-all flex items-center gap-1.5" data-cat-filter="study">
                    <span class="w-1.5 h-1.5 rounded-full bg-purple-500"></span>공부
                </button>
                <button class="category-filter-btn flex-shrink-0 px-3.5 py-1.5 text-xs font-bold rounded-full bg-white/40 text-slate-500 hover:bg-white/60 border border-slate-100/50 transition-all flex items-center gap-1.5" data-cat-filter="shopping">
                    <span class="w-1.5 h-1.5 rounded-full bg-amber-500"></span>쇼핑
                </button>
                <button class="category-filter-btn flex-shrink-0 px-3.5 py-1.5 text-xs font-bold rounded-full bg-white/40 text-slate-500 hover:bg-white/60 border border-slate-100/50 transition-all flex items-center gap-1.5" data-cat-filter="etc">
                    <span class="w-1.5 h-1.5 rounded-full bg-slate-400"></span>기타
                </button>
            </div>
        </div>

        <!-- 새 할 일 등록 시 카테고리 선택 영역 -->
        <div class="px-8 pb-3 flex flex-col sm:flex-row gap-3 justify-between items-start sm:items-center">
            <div class="w-full sm:w-auto">
                <p class="text-[10px] font-bold text-slate-400 mb-1.5 tracking-wider uppercase">카테고리 지정</p>
                <div class="flex gap-2 overflow-x-auto no-scrollbar py-0.5" id="category-selector">
                    <button type="button" class="category-select-btn flex-shrink-0 px-3.5 py-1.5 rounded-full text-xs font-bold border transition-all flex items-center gap-1.5 bg-indigo-50 text-indigo-600 border-indigo-200 ring-2 ring-indigo-500/20" data-category="work">
                        <span class="w-1.5 h-1.5 rounded-full bg-indigo-500"></span>업무
                    </button>
                    <button type="button" class="category-select-btn flex-shrink-0 px-3.5 py-1.5 rounded-full text-xs font-bold border border-white/60 bg-white/40 text-slate-500 hover:bg-white/60 transition-all flex items-center gap-1.5" data-category="personal">
                        <span class="w-1.5 h-1.5 rounded-full bg-emerald-500"></span>개인
                    </button>
                    <button type="button" class="category-select-btn flex-shrink-0 px-3.5 py-1.5 rounded-full text-xs font-bold border border-white/60 bg-white/40 text-slate-500 hover:bg-white/60 transition-all flex items-center gap-1.5" data-category="study">
                        <span class="w-1.5 h-1.5 rounded-full bg-purple-500"></span>공부
                    </button>
                    <button type="button" class="category-select-btn flex-shrink-0 px-3.5 py-1.5 rounded-full text-xs font-bold border border-white/60 bg-white/40 text-slate-500 hover:bg-white/60 transition-all flex items-center gap-1.5" data-category="shopping">
                        <span class="w-1.5 h-1.5 rounded-full bg-amber-500"></span>쇼핑
                    </button>
                    <button type="button" class="category-select-btn flex-shrink-0 px-3.5 py-1.5 rounded-full text-xs font-bold border border-white/60 bg-white/40 text-slate-500 hover:bg-white/60 transition-all flex items-center gap-1.5" data-category="etc">
                        <span class="w-1.5 h-1.5 rounded-full bg-slate-400"></span>기타
                    </button>
                </div>
            </div>
            <div class="w-full sm:w-auto">
                <p class="text-[10px] font-bold text-slate-400 mb-1.5 tracking-wider uppercase">기한 설정 (선택)</p>
                <div class="flex items-center gap-1.5 px-3 py-1.5 rounded-xl border border-white/60 bg-white/40 text-slate-600 hover:bg-white/60 transition-all focus-within:ring-2 focus-within:ring-indigo-500/20">
                    <i class="ph ph-calendar text-indigo-500 text-sm"></i>
                    <input type="date" id="todo-date" class="bg-transparent border-none outline-none text-slate-700 text-xs font-bold cursor-pointer [color-scheme:light]">
                </div>
            </div>
        </div>

        <!-- 플로팅 느낌의 입력 폼 영역 -->
        <div class="px-8 pb-6 relative z-20">
            <form id="todo-form" class="relative flex items-center group">
                <div class="absolute left-4 text-slate-400 group-focus-within:text-indigo-500 transition-colors">
                    <i class="ph ph-plus-circle text-xl"></i>
                </div>
                <input type="text" id="todo-input" 
                    class="w-full pl-12 pr-14 py-4 rounded-2xl bg-white/60 border border-white focus:bg-white focus:border-indigo-300 focus:ring-4 focus:ring-indigo-500/10 transition-all outline-none text-slate-700 placeholder-slate-400 shadow-[0_2px_10px_rgba(0,0,0,0.02)] font-medium"
                    placeholder="새로운 할 일을 입력하세요..."
                    autocomplete="off">
                <button type="submit" 
                    class="absolute right-2 w-10 h-10 flex items-center justify-center bg-indigo-600 text-white rounded-xl hover:bg-indigo-700 hover:shadow-lg hover:shadow-indigo-500/30 hover:scale-105 active:scale-95 transition-all focus:outline-none">
                    <i class="ph ph-paper-plane-right text-lg"></i>
                </button>
            </form>
        </div>

        <!-- 리스트 영역 -->
        <div class="flex-1 overflow-y-auto px-8 pb-8 relative">
            <ul id="todo-list" class="space-y-3 pb-10">
                <!-- 할 일 항목들이 여기에 동적으로 추가됩니다 -->
            </ul>
            
            <!-- 감성적인 빈 상태 표시 -->
            <div id="empty-state" class="hidden absolute inset-0 flex-col items-center justify-center text-slate-400 pb-10">
                <div class="w-24 h-24 bg-gradient-to-br from-indigo-50 to-purple-50 rounded-full flex items-center justify-center mb-4 border border-white shadow-sm">
                    <i class="ph-fill ph-check-circle text-5xl text-indigo-300"></i>
                </div>
                <p class="text-base font-bold text-slate-600">모든 할 일을 완료했어요!</p>
                <p class="text-sm mt-1 opacity-80 text-slate-500">휴식을 취하거나 새로운 태스크를 추가해보세요.</p>
            </div>
        </div>

        <!-- 푸터 (하단 블러 효과 및 플로팅 버튼) -->
        <div class="absolute bottom-0 left-0 right-0 p-6 bg-gradient-to-t from-white/90 via-white/70 to-transparent flex justify-center pointer-events-none z-30">
            <button id="clear-completed" class="pointer-events-auto text-xs text-rose-500 hover:text-rose-600 hover:bg-rose-50 font-bold px-5 py-2.5 rounded-full shadow-sm border border-rose-100 transition-all flex items-center gap-1.5 hidden backdrop-blur-md bg-white">
                <i class="ph ph-trash-simple text-sm"></i> 완료된 항목 삭제
            </button>
        </div>
    </div>

    <script>
        // DOM 요소 선택
        const todoForm = document.getElementById('todo-form');
        const todoInput = document.getElementById('todo-input');
        const todoList = document.getElementById('todo-list');
        const emptyState = document.getElementById('empty-state');
        const totalTasksCount = document.getElementById('total-tasks-count');
        const activeTasksCount = document.getElementById('active-tasks-count');
        const dateDisplay = document.getElementById('date-display');
        const filterBtns = document.querySelectorAll('.filter-btn');
        const clearCompletedBtn = document.getElementById('clear-completed');

        // 카테고리 관련 DOM 요소 추가 선택
        const categorySelectBtns = document.querySelectorAll('.category-select-btn');
        const categoryFilterBtns = document.querySelectorAll('.category-filter-btn');
        
        // 날짜 선택 DOM 요소 추가 선택
        const todoDateInput = document.getElementById('todo-date');

        // 카테고리 설정 정보 (스타일 맵핑)
        const CATEGORIES = {
            work: { label: '업무', bg: 'bg-indigo-50/80 text-indigo-600 border-indigo-100', dot: 'bg-indigo-500' },
            personal: { label: '개인', bg: 'bg-emerald-50/80 text-emerald-600 border-emerald-100', dot: 'bg-emerald-500' },
            study: { label: '공부', bg: 'bg-purple-50/80 text-purple-600 border-purple-100', dot: 'bg-purple-500' },
            shopping: { label: '쇼핑', bg: 'bg-amber-50/80 text-amber-600 border-amber-100', dot: 'bg-amber-500' },
            etc: { label: '기타', bg: 'bg-slate-100/80 text-slate-600 border-slate-200', dot: 'bg-slate-400' }
        };

        // 상태 변수
        let todos = []; // 할 일 데이터를 저장할 배열
        let currentFilter = 'all'; // 현재 완료 여부 필터 상태 ('all', 'active', 'completed')
        let selectedCategory = 'work'; // 새 할 일을 등록할 때 사용할 선택된 카테고리
        let currentCategoryFilter = 'all'; // 목록에서 필터링할 카테고리 상태

        // 애플리케이션 초기화
        function init() {
            // 날짜 표시 설정
            updateDateDisplay();
            
            // 로컬 스토리지에서 데이터 불러오기
            loadTodos();
            
            // 이벤트 리스너 등록
            todoForm.addEventListener('submit', handleAddTodo);
            clearCompletedBtn.addEventListener('click', clearCompletedTodos);
            
            filterBtns.forEach(btn => {
                btn.addEventListener('click', () => {
                    // 활성 탭 커스텀 클래스 변경
                    filterBtns.forEach(b => b.classList.remove('active'));
                    btn.classList.add('active');
                    
                    // 필터 상태 업데이트 및 렌더링
                    currentFilter = btn.dataset.filter;
                    renderTodos();
                });
            });

            // 새 할 일 카테고리 선택 처리
            categorySelectBtns.forEach(btn => {
                btn.addEventListener('click', () => {
                    // 모든 선택 버튼 비활성화 상태로 디자인 리셋
                    categorySelectBtns.forEach(b => {
                        b.className = "category-select-btn flex-shrink-0 px-3.5 py-1.5 rounded-full text-xs font-bold border border-white/60 bg-white/40 text-slate-500 hover:bg-white/60 transition-all flex items-center gap-1.5";
                    });
                    
                    selectedCategory = btn.dataset.category;
                    
                    // 선택된 카테고리에 맞는 전용 컬러풀 테마 적용
                    const activeThemes = {
                        work: "bg-indigo-50 text-indigo-600 border-indigo-200 ring-2 ring-indigo-500/20",
                        personal: "bg-emerald-50 text-emerald-600 border-emerald-200 ring-2 ring-emerald-500/20",
                        study: "bg-purple-50 text-purple-600 border-purple-200 ring-2 ring-purple-500/20",
                        shopping: "bg-amber-50 text-amber-600 border-amber-200 ring-2 ring-amber-500/20",
                        etc: "bg-slate-100 text-slate-600 border-slate-300 ring-2 ring-slate-400/20"
                    };
                    
                    btn.className = `category-select-btn flex-shrink-0 px-3.5 py-1.5 rounded-full text-xs font-bold border transition-all flex items-center gap-1.5 ${activeThemes[selectedCategory]}`;
                });
            });

            // 카테고리 필터링 버튼 클릭 처리
            categoryFilterBtns.forEach(btn => {
                btn.addEventListener('click', () => {
                    categoryFilterBtns.forEach(b => b.classList.remove('active-cat'));
                    btn.classList.add('active-cat');
                    
                    currentCategoryFilter = btn.dataset.catFilter;
                    renderTodos();
                });
            });
            
            // 초기 렌더링
            renderTodos();
        }

        // 오늘 날짜 표시 함수
        function updateDateDisplay() {
            const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
            const today = new Date();
            dateDisplay.textContent = today.toLocaleDateString('ko-KR', options);
        }

        // 로컬 스토리지에 데이터 저장
        function saveTodos() {
            localStorage.setItem('simple-todos', JSON.stringify(todos));
            updateCounters();
        }

        // 로컬 스토리지에서 데이터 로드
        function loadTodos() {
            const savedTodos = localStorage.getItem('simple-todos');
            if (savedTodos) {
                todos = JSON.parse(savedTodos);
            }
        }

        // 카운터 및 UI 상태 업데이트
        function updateCounters() {
            const total = todos.length;
            const active = todos.filter(t => !t.completed).length;
            const completed = total - active;

            totalTasksCount.textContent = total;
            activeTasksCount.textContent = active;

            // 완료된 항목이 있으면 '완료된 항목 삭제' 버튼 표시
            if (completed > 0) {
                clearCompletedBtn.classList.remove('hidden');
            } else {
                clearCompletedBtn.classList.add('hidden');
            }
        }

        // 할 일 추가 핸들러
        function handleAddTodo(e) {
            e.preventDefault();
            const text = todoInput.value.trim();
            const dueDate = todoDateInput.value; // 선택된 날짜 (YYYY-MM-DD 형식 또는 빈 값)
            
            if (text !== '') {
                const newTodo = {
                    id: Date.now().toString(), // 고유 ID 생성
                    text: text,
                    completed: false,
                    category: selectedCategory, // 선택된 카테고리 추가 지정
                    dueDate: dueDate || null, // 기한 날짜 추가
                    createdAt: new Date().toISOString()
                };
                
                todos.unshift(newTodo); // 배열 맨 앞에 추가 (최신 항목이 위로)
                saveTodos();
                todoInput.value = ''; // 입력창 초기화
                todoDateInput.value = ''; // 날짜 선택 초기화
                renderTodos(); // 리스트 다시 그리기
            }
        }

        // D-Day 계산 배지 HTML 생성 함수
        function getDDayBadge(dueDateStr) {
            if (!dueDateStr) return '';
            
            const today = new Date();
            today.setHours(0, 0, 0, 0);
            
            const dueDate = new Date(dueDateStr);
            dueDate.setHours(0, 0, 0, 0);
            
            const diffTime = dueDate - today;
            const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));
            
            // 월/일 포맷팅 (MM/DD)
            const mmDd = dueDateStr.substring(5).replace('-', '/');
            
            let label = '';
            let bgClass = '';
            let dotClass = '';
            
            if (diffDays === 0) {
                label = `오늘까지 (${mmDd})`;
                bgClass = 'bg-rose-50/90 text-rose-600 border-rose-100';
                dotClass = 'bg-rose-500';
            } else if (diffDays === 1) {
                label = `내일까지 (${mmDd})`;
                bgClass = 'bg-amber-50/90 text-amber-600 border-amber-100';
                dotClass = 'bg-amber-500';
            } else if (diffDays < 0) {
                label = `기한 지남 (D+${Math.abs(diffDays)})`;
                bgClass = 'bg-slate-100 text-slate-400 border-slate-200';
                dotClass = 'bg-slate-400';
            } else {
                label = `D-${diffDays} (${mmDd})`;
                bgClass = 'bg-indigo-50/90 text-indigo-600 border-indigo-100';
                dotClass = 'bg-indigo-500';
            }
            
            return `
                <span class="inline-flex items-center gap-1 text-[10px] font-bold px-2 py-0.5 rounded-full border ${bgClass}">
                    <span class="w-1.5 h-1.5 rounded-full ${dotClass}"></span>
                    ${label}
                </span>
            `;
        }

        // 할 일 상태 토글 (완료/미완료)
        function toggleTodo(id) {
            todos = todos.map(todo => {
                if (todo.id === id) {
                    return { ...todo, completed: !todo.completed };
                }
                return todo;
            });
            saveTodos();
            renderTodos();
        }

        // 할 일 삭제
        function deleteTodo(id) {
            // 삭제 애니메이션 적용을 위해 DOM 요소를 찾음
            const itemElement = document.querySelector(`[data-id="${id}"]`);
            
            if (itemElement) {
                // 퇴장 애니메이션 클래스 추가
                itemElement.classList.add('fade-out');
                
                // 애니메이션이 끝난 후 실제 데이터 삭제 및 리렌더링
                setTimeout(() => {
                    todos = todos.filter(todo => todo.id !== id);
                    saveTodos();
                    renderTodos();
                }, 300); // CSS transition 시간과 일치시킴
            } else {
                // 요소를 찾지 못한 경우 (fallback)
                todos = todos.filter(todo => todo.id !== id);
                saveTodos();
                renderTodos();
            }
        }

        // 완료된 할 일 모두 삭제
        function clearCompletedTodos() {
            todos = todos.filter(todo => !todo.completed);
            saveTodos();
            renderTodos();
        }

        // 리스트 렌더링
        function renderTodos() {
            todoList.innerHTML = '';
            
            let filteredTodos = todos;
            
            // 1. 완료 상태별 필터링
            if (currentFilter === 'active') {
                filteredTodos = todos.filter(todo => !todo.completed);
            } else if (currentFilter === 'completed') {
                filteredTodos = todos.filter(todo => todo.completed);
            }

            // 2. 카테고리 종류별 필터링
            if (currentCategoryFilter !== 'all') {
                filteredTodos = filteredTodos.filter(todo => (todo.category || 'etc') === currentCategoryFilter);
            }

            // 빈 상태 처리
            if (filteredTodos.length === 0) {
                emptyState.classList.remove('hidden');
                emptyState.classList.add('flex');
            } else {
                emptyState.classList.add('hidden');
                emptyState.classList.remove('flex');
                
                filteredTodos.forEach(todo => {
                    const li = document.createElement('li');
                    // 글래스모피즘 아이템 디자인
                    li.className = `todo-item bg-white/80 backdrop-blur-sm p-4 rounded-2xl shadow-[0_2px_10px_rgba(0,0,0,0.02)] border border-white flex items-center gap-4 group hover:shadow-md transition-all ${todo.completed ? 'completed opacity-70' : ''}`;
                    li.setAttribute('data-id', todo.id);
                    
                    // 카테고리 정보 가져오기 (이전 태스크 등 비어있는 경우 'etc(기타)'로 처리)
                    const catKey = todo.category || 'etc';
                    const categoryConfig = CATEGORIES[catKey] || CATEGORIES['etc'];
                    
                    // 기한 D-Day 배지 가져오기 (완료 상태가 아닐 때만 생생하게 표시하고 완료 상태일 땐 스타일을 다르게 할 수 있음)
                    const dDayBadgeHtml = todo.completed ? '' : getDDayBadge(todo.dueDate);
                    
                    li.innerHTML = `
                        <!-- 개선된 체크박스 디자인 (바운스 애니메이션 포함) -->
                        <button onclick="toggleTodo('${todo.id}')" 
                            class="flex-shrink-0 w-7 h-7 rounded-full border-2 flex items-center justify-center transition-all focus:outline-none focus:ring-4 focus:ring-indigo-500/20
                            ${todo.completed ? 'bg-indigo-500 border-indigo-500 text-white' : 'border-slate-300 text-transparent hover:border-indigo-400 bg-white'}">
                            <i class="ph-bold ph-check text-sm check-icon"></i>
                        </button>
                        
                        <!-- 텍스트 내용 및 카테고리/기한 배지 -->
                        <div class="flex-1 min-w-0 cursor-pointer py-1" onclick="toggleTodo('${todo.id}')">
                            <div class="flex flex-col gap-1.5 items-start">
                                <div class="flex flex-wrap gap-1.5 items-center">
                                    <span class="inline-flex items-center gap-1 text-[10px] font-bold px-2 py-0.5 rounded-full border ${categoryConfig.bg}">
                                        <span class="w-1.5 h-1.5 rounded-full ${categoryConfig.dot}"></span>
                                        ${categoryConfig.label}
                                    </span>
                                    ${dDayBadgeHtml}
                                </div>
                                <span class="text-base font-medium ${todo.completed ? 'text-slate-400 line-through' : 'text-slate-700'} todo-text break-all">
                                    ${escapeHTML(todo.text)}
                                </span>
                            </div>
                        </div>
                        
                        <!-- 삭제 버튼 -->
                        <button onclick="deleteTodo('${todo.id}')" 
                            class="flex-shrink-0 w-9 h-9 rounded-xl text-slate-300 hover:text-rose-500 hover:bg-rose-50 flex items-center justify-center opacity-0 group-hover:opacity-100 transition-all focus:opacity-100 focus:outline-none bg-white border border-transparent hover:border-rose-100">
                            <i class="ph ph-trash text-lg"></i>
                        </button>
                    `;
                    todoList.appendChild(li);
                });
            }
            
            updateCounters();
        }

        // XSS 방지를 위한 HTML 이스케이프 함수
        function escapeHTML(str) {
            return str.replace(/[&<>'"]/g, 
                tag => ({
                    '&': '&amp;',
                    '<': '&lt;',
                    '>': '&gt;',
                    "'": '&#39;',
                    '"': '&quot;'
                }[tag] || tag)
            );
        }

        // 앱 시작
        init();
    </script>
</body>
</html>