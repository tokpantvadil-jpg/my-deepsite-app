# my-deepsite-app
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>World Medicine Kazakhstan - Трекер посещений</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        .hour-slot {
            min-height: 60px;
            border-bottom: 1px solid #e5e7eb;
            position: relative;
        }
        .hour-slot:hover {
            background-color: #f3f4f6;
        }
        .event-dot {
            width: 10px;
            height: 10px;
            border-radius: 50%;
            position: absolute;
            top: 5px;
            right: 5px;
        }
        .calendar-day {
            transition: all 0.2s ease;
        }
        .calendar-day:hover {
            transform: scale(1.05);
        }
        .calendar-day.active {
            background-color: #3b82f6;
            color: white;
        }
        .note-card {
            transition: all 0.2s ease;
        }
        .note-card:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
        }
        @media (max-width: 768px) {
            .calendar-header {
                font-size: 0.8rem;
            }
            .hour-label {
                font-size: 0.7rem;
            }
        }
    </style>
</head>
<body class="bg-gray-50 font-sans">
    <div class="container mx-auto px-4 py-6 max-w-7xl">
        <!-- Шапка приложения -->
        <header class="bg-blue-600 text-white rounded-lg shadow-md p-6 mb-6">
            <div class="flex flex-col md:flex-row justify-between items-start md:items-center">
                <div class="mb-4 md:mb-0">
                    <h1 class="text-2xl md:text-3xl font-bold">World Medicine Kazakhstan</h1>
                    <p class="text-blue-100">Трекер посещений медицинских учреждений</p>
                    <p class="text-blue-200 text-sm mt-1"><i class="fas fa-map-marker-alt mr-1"></i> Город Костанай</p>
                </div>
                <div class="bg-white text-blue-800 rounded-lg p-3 shadow-inner">
                    <div class="flex items-center">
                        <i class="fas fa-calendar-day text-xl mr-2"></i>
                        <div>
                            <p class="font-semibold" id="current-date">Загрузка...</p>
                            <p class="text-sm" id="current-time">00:00:00</p>
                        </div>
                    </div>
                </div>
            </div>
        </header>

        <!-- Основное содержимое -->
        <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
            <!-- Левая колонка - Календарь и статистика -->
            <div class="lg:col-span-2 space-y-6">
                <!-- Навигация по периодам -->
                <div class="bg-white rounded-lg shadow-md p-4">
                    <div class="flex justify-between items-center mb-4">
                        <h2 class="text-xl font-semibold text-gray-800">Планирование посещений</h2>
                        <div class="flex space-x-2">
                            <button id="prev-month" class="p-2 rounded-full hover:bg-gray-100">
                                <i class="fas fa-chevron-left"></i>
                            </button>
                            <button id="next-month" class="p-2 rounded-full hover:bg-gray-100">
                                <i class="fas fa-chevron-right"></i>
                            </button>
                        </div>
                    </div>
                    
                    <div class="flex justify-center space-x-4 mb-4">
                        <button data-period="day" class="period-btn active px-4 py-2 rounded-lg bg-blue-100 text-blue-700 font-medium">День</button>
                        <button data-period="week" class="period-btn px-4 py-2 rounded-lg hover:bg-gray-100 font-medium">Неделя</button>
                        <button data-period="month" class="period-btn px-4 py-2 rounded-lg hover:bg-gray-100 font-medium">Месяц</button>
                    </div>
                    
                    <!-- Месячный календарь -->
                    <div id="month-view" class="hidden">
                        <div class="grid grid-cols-7 gap-1 mb-2">
                            <div class="text-center font-medium text-gray-500 text-sm py-2">Пн</div>
                            <div class="text-center font-medium text-gray-500 text-sm py-2">Вт</div>
                            <div class="text-center font-medium text-gray-500 text-sm py-2">Ср</div>
                            <div class="text-center font-medium text-gray-500 text-sm py-2">Чт</div>
                            <div class="text-center font-medium text-gray-500 text-sm py-2">Пт</div>
                            <div class="text-center font-medium text-gray-500 text-sm py-2">Сб</div>
                            <div class="text-center font-medium text-gray-500 text-sm py-2">Вс</div>
                        </div>
                        <div id="month-days" class="grid grid-cols-7 gap-1"></div>
                    </div>
                    
                    <!-- Недельный календарь -->
                    <div id="week-view" class="hidden">
                        <div class="grid grid-cols-8 gap-1">
                            <div class="col-span-1"></div>
                            <div class="text-center font-medium text-gray-800 py-2 border-b">Понедельник</div>
                            <div class="text-center font-medium text-gray-800 py-2 border-b">Вторник</div>
                            <div class="text-center font-medium text-gray-800 py-2 border-b">Среда</div>
                            <div class="text-center font-medium text-gray-800 py-2 border-b">Четверг</div>
                            <div class="text-center font-medium text-gray-800 py-2 border-b">Пятница</div>
                            <div class="text-center font-medium text-gray-800 py-2 border-b">Суббота</div>
                            <div class="text-center font-medium text-gray-800 py-2 border-b">Воскресенье</div>
                        </div>
                        <div id="week-hours" class="grid grid-cols-8 gap-1"></div>
                    </div>
                    
                    <!-- Дневной календарь -->
                    <div id="day-view">
                        <div class="flex justify-between items-center mb-4">
                            <h3 id="selected-date" class="text-lg font-semibold text-gray-700">Сегодня, 1 июня</h3>
                            <div class="flex space-x-2">
                                <button id="prev-day" class="p-2 rounded-full hover:bg-gray-100">
                                    <i class="fas fa-chevron-left"></i>
                                </button>
                                <button id="today-btn" class="px-3 py-1 rounded-lg bg-gray-100 hover:bg-gray-200 text-sm">
                                    Сегодня
                                </button>
                                <button id="next-day" class="p-2 rounded-full hover:bg-gray-100">
                                    <i class="fas fa-chevron-right"></i>
                                </button>
                            </div>
                        </div>
                        
                        <div class="border rounded-lg overflow-hidden">
                            <div id="day-hours" class="divide-y divide-gray-200"></div>
                        </div>
                    </div>
                </div>
                
                <!-- Форма добавления посещения -->
                <div class="bg-white rounded-lg shadow-md p-4">
                    <h2 class="text-xl font-semibold text-gray-800 mb-4">Добавить посещение</h2>
                    <form id="visit-form" class="space-y-4">
                        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                            <div>
                                <label for="visit-type" class="block text-sm font-medium text-gray-700 mb-1">Тип посещения</label>
                                <select id="visit-type" class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500">
                                    <option value="doctor">Врач</option>
                                    <option value="pharmacy">Аптека</option>
                                    <option value="hospital">Больница</option>
                                    <option value="clinic">Клиника</option>
                                </select>
                            </div>
                            <div>
                                <label for="institution" class="block text-sm font-medium text-gray-700 mb-1">Учреждение/Врач</label>
                                <input type="text" id="institution" class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500" placeholder="Название или ФИО">
                            </div>
                        </div>
                        
                        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                            <div>
                                <label for="visit-date" class="block text-sm font-medium text-gray-700 mb-1">Дата</label>
                                <input type="date" id="visit-date" class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500">
                            </div>
                            <div>
                                <label for="visit-time" class="block text-sm font-medium text-gray-700 mb-1">Время</label>
                                <input type="time" id="visit-time" class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500">
                            </div>
                        </div>
                        
                        <div>
                            <label for="visit-notes" class="block text-sm font-medium text-gray-700 mb-1">Заметки</label>
                            <textarea id="visit-notes" rows="2" class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500" placeholder="Детали посещения..."></textarea>
                        </div>
                        
                        <div class="flex justify-end">
                            <button type="submit" class="px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2">
                                <i class="fas fa-plus mr-2"></i> Добавить посещение
                            </button>
                        </div>
                    </form>
                </div>
            </div>
            
            <!-- Правая колонка - Заметки и статистика -->
            <div class="space-y-6">
                <!-- Статистика -->
                <div class="bg-white rounded-lg shadow-md p-4">
                    <h2 class="text-xl font-semibold text-gray-800 mb-4">Статистика посещений</h2>
                    <div class="space-y-3">
                        <div class="flex justify-between items-center p-3 bg-blue-50 rounded-lg">
                            <div>
                                <p class="text-sm text-gray-600">Сегодня</p>
                                <p class="font-bold text-blue-800" id="today-count">0</p>
                            </div>
                            <div class="p-2 bg-blue-100 rounded-full">
                                <i class="fas fa-calendar-day text-blue-600"></i>
                            </div>
                        </div>
                        <div class="flex justify-between items-center p-3 bg-green-50 rounded-lg">
                            <div>
                                <p class="text-sm text-gray-600">На этой неделе</p>
                                <p class="font-bold text-green-800" id="week-count">0</p>
                            </div>
                            <div class="p-2 bg-green-100 rounded-full">
                                <i class="fas fa-calendar-week text-green-600"></i>
                            </div>
                        </div>
                        <div class="flex justify-between items-center p-3 bg-purple-50 rounded-lg">
                            <div>
                                <p class="text-sm text-gray-600">В этом месяце</p>
                                <p class="font-bold text-purple-800" id="month-count">0</p>
                            </div>
                            <div class="p-2 bg-purple-100 rounded-full">
                                <i class="fas fa-calendar-alt text-purple-600"></i>
                            </div>
                        </div>
                    </div>
                    
                    <div class="mt-4 pt-4 border-t">
                        <h3 class="font-medium text-gray-700 mb-2">Топ посещаемых мест</h3>
                        <div id="top-institutions" class="space-y-2">
                            <!-- Динамически заполняется -->
                        </div>
                    </div>
                </div>
                
                <!-- Заметки -->
                <div class="bg-white rounded-lg shadow-md p-4">
                    <div class="flex justify-between items-center mb-4">
                        <h2 class="text-xl font-semibold text-gray-800">Мои заметки</h2>
                        <button id="add-note-btn" class="p-2 rounded-full bg-blue-100 text-blue-600 hover:bg-blue-200">
                            <i class="fas fa-plus"></i>
                        </button>
                    </div>
                    
                    <div id="notes-container" class="space-y-3">
                        <!-- Пример заметки -->
                        <div class="note-card bg-yellow-50 border-l-4 border-yellow-400 rounded-lg p-3 shadow-sm">
                            <div class="flex justify-between items-start">
                                <div>
                                    <h3 class="font-medium text-gray-800">Важная встреча с главврачом</h3>
                                    <p class="text-sm text-gray-600">12 июня, 10:00</p>
                                </div>
                                <button class="text-gray-400 hover:text-gray-600">
                                    <i class="fas fa-times"></i>
                                </button>
                            </div>
                            <p class="mt-2 text-gray-700 text-sm">Обсудить поставки новых препаратов и условия сотрудничества.</p>
                        </div>
                    </div>
                    
                    <!-- Форма добавления заметки (скрыта по умолчанию) -->
                    <div id="note-form-container" class="hidden mt-4">
                        <form id="note-form" class="space-y-3">
                            <div>
                                <input type="text" id="note-title" class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500" placeholder="Заголовок заметки" required>
                            </div>
                            <div>
                                <textarea id="note-content" rows="3" class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500" placeholder="Содержание заметки..." required></textarea>
                            </div>
                            <div class="flex justify-end space-x-2">
                                <button type="button" id="cancel-note-btn" class="px-3 py-1 bg-gray-200 text-gray-700 rounded-md hover:bg-gray-300">
                                    Отмена
                                </button>
                                <button type="submit" class="px-3 py-1 bg-blue-600 text-white rounded-md hover:bg-blue-700">
                                    Сохранить
                                </button>
                            </div>
                        </form>
                    </div>
                </div>
                
                <!-- Комментарии -->
                <div class="bg-white rounded-lg shadow-md p-4">
                    <h2 class="text-xl font-semibold text-gray-800 mb-4">Комментарии</h2>
                    <div id="comments-container" class="space-y-3 mb-4 max-h-60 overflow-y-auto">
                        <!-- Пример комментария -->
                        <div class="bg-gray-50 rounded-lg p-3">
                            <div class="flex justify-between items-start">
                                <div>
                                    <p class="font-medium text-gray-800">Айгуль Сапарбаева</p>
                                    <p class="text-xs text-gray-500">15 мая, 14:30</p>
                                </div>
                                <button class="text-gray-400 hover:text-gray-600 text-sm">
                                    <i class="fas fa-reply mr-1"></i> Ответить
                                </button>
                            </div>
                            <p class="mt-2 text-gray-700 text-sm">В аптеке "Авиценна" заинтересованы в увеличении ассортимента наших препаратов. Нужно подготовить коммерческое предложение.</p>
                        </div>
                    </div>
                    
                    <form id="comment-form" class="mt-4">
                        <div class="flex space-x-2">
                            <input type="text" id="comment-input" class="flex-1 px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500" placeholder="Написать комментарий..." required>
                            <button type="submit" class="px-3 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700">
                                <i class="fas fa-paper-plane"></i>
                            </button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>

    <!-- Модальное окно для просмотра посещения -->
    <div id="visit-modal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 hidden">
        <div class="bg-white rounded-lg shadow-xl w-full max-w-md mx-4">
            <div class="flex justify-between items-center border-b px-6 py-4">
                <h3 class="text-lg font-semibold text-gray-800" id="modal-title">Детали посещения</h3>
                <button id="close-modal" class="text-gray-400 hover:text-gray-600">
                    <i class="fas fa-times"></i>
                </button>
            </div>
            <div class="p-6 space-y-4">
                <div>
                    <p class="text-sm text-gray-600">Тип</p>
                    <p class="font-medium" id="modal-type">Врач</p>
                </div>
                <div>
                    <p class="text-sm text-gray-600">Учреждение/Врач</p>
                    <p class="font-medium" id="modal-institution">Доктор Ахметов</p>
                </div>
                <div>
                    <p class="text-sm text-gray-600">Дата и время</p>
                    <p class="font-medium" id="modal-datetime">15 июня, 10:00</p>
                </div>
                <div>
                    <p class="text-sm text-gray-600">Заметки</p>
                    <p class="font-medium" id="modal-notes">Обсуждение эффективности препарата X</p>
                </div>
                <div>
                    <p class="text-sm text-gray-600">Статус</p>
                    <p class="font-medium" id="modal-status"><span class="px-2 py-1 bg-green-100 text-green-800 rounded-full text-xs">Завершено</span></p>
                </div>
            </div>
            <div class="border-t px-6 py-4 flex justify-end space-x-3">
                <button id="edit-visit" class="px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700">
                    <i class="fas fa-edit mr-2"></i> Редактировать
                </button>
                <button id="delete-visit" class="px-4 py-2 bg-red-100 text-red-700 rounded-md hover:bg-red-200">
                    <i class="fas fa-trash mr-2"></i> Удалить
                </button>
            </div>
        </div>
    </div>

    <script>
        // Текущая дата и время
        function updateDateTime() {
            const now = new Date();
            const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
            document.getElementById('current-date').textContent = now.toLocaleDateString('ru-RU', options);
            
            function updateTime() {
                const now = new Date();
                const timeStr = now.toLocaleTimeString('ru-RU');
                document.getElementById('current-time').textContent = timeStr;
            }
            updateTime();
            setInterval(updateTime, 1000);
        }
        
        // Инициализация календаря
        document.addEventListener('DOMContentLoaded', function() {
            updateDateTime();
            
            // Текущая дата
            let currentDate = new Date();
            let selectedDate = new Date();
            
            // Данные приложения
            let visits = JSON.parse(localStorage.getItem('wm-visits')) || [];
            let notes = JSON.parse(localStorage.getItem('wm-notes')) || [
                {
                    id: 1,
                    title: "Важная встреча с главврачом",
                    date: "2023-06-12",
                    time: "10:00",
                    content: "Обсудить поставки новых препаратов и условия сотрудничества."
                }
            ];
            let comments = JSON.parse(localStorage.getItem('wm-comments')) || [
                {
                    id: 1,
                    author: "Айгуль Сапарбаева",
                    date: "2023-05-15T14:30:00",
                    content: "В аптеке 'Авиценна' заинтересованы в увеличении ассортимента наших препаратов. Нужно подготовить коммерческое предложение."
                }
            ];
            
            // Сохранить данные в localStorage
            function saveData() {
                localStorage.setItem('wm-visits', JSON.stringify(visits));
                localStorage.setItem('wm-notes', JSON.stringify(notes));
                localStorage.setItem('wm-comments', JSON.stringify(comments));
                updateStatistics();
            }
            
            // Обновить статистику
            function updateStatistics() {
                const today = new Date().toISOString().split('T')[0];
                const todayVisits = visits.filter(v => v.date === today).length;
                document.getElementById('today-count').textContent = todayVisits;
                
                // Недельные посещения
                const weekStart = getWeekStartDate(new Date());
                const weekEnd = new Date(weekStart);
                weekEnd.setDate(weekStart.getDate() + 6);
                
                const weekVisits = visits.filter(v => {
                    const visitDate = new Date(v.date);
                    return visitDate >= weekStart && visitDate <= weekEnd;
                }).length;
                document.getElementById('week-count').textContent = weekVisits;
                
                // Месячные посещения
                const monthStart = new Date(currentDate.getFullYear(), currentDate.getMonth(), 1);
                const monthEnd = new Date(currentDate.getFullYear(), currentDate.getMonth() + 1, 0);
                
                const monthVisits = visits.filter(v => {
                    const visitDate = new Date(v.date);
                    return visitDate >= monthStart && visitDate <= monthEnd;
                }).length;
                document.getElementById('month-count').textContent = monthVisits;
                
                // Топ учреждений
                const institutionCounts = {};
                visits.forEach(v => {
                    institutionCounts[v.institution] = (institutionCounts[v.institution] || 0) + 1;
                });
                
                const topInstitutions = Object.entries(institutionCounts)
                    .sort((a, b) => b[1] - a[1])
                    .slice(0, 3);
                
                const topInstitutionsContainer = document.getElementById('top-institutions');
                topInstitutionsContainer.innerHTML = '';
                
                if (topInstitutions.length === 0) {
                    topInstitutionsContainer.innerHTML = '<p class="text-gray-500 text-sm">Нет данных о посещениях</p>';
                } else {
                    topInstitutions.forEach(([institution, count]) => {
                        const div = document.createElement('div');
                        div.className = 'flex justify-between items-center p-2 bg-gray-50 rounded';
                        div.innerHTML = `
                            <p class="text-sm font-medium">${institution}</p>
                            <span class="px-2 py-1 bg-blue-100 text-blue-800 rounded-full text-xs">${count}</span>
                        `;
                        topInstitutionsContainer.appendChild(div);
                    });
                }
            }
            
            // Получить начало недели для даты
            function getWeekStartDate(date) {
                const day = date.getDay();
                const diff = date.getDate() - day + (day === 0 ? -6 : 1); // Понедельник как начало недели
                return new Date(date.setDate(diff));
            }
            
            // Форматирование даты
            function formatDate(date) {
                const options = { weekday: 'long', day: 'numeric', month: 'long' };
                return date.toLocaleDateString('ru-RU', options);
            }
            
            // Форматирование времени
            function formatTime(timeStr) {
                const [hours, minutes] = timeStr.split(':');
                return `${hours}:${minutes}`;
            }
            
            // Отобразить день
            function renderDay(date) {
                const dayContainer = document.getElementById('day-hours');
                dayContainer.innerHTML = '';
                
                const dateStr = date.toISOString().split('T')[0];
                const dayVisits = visits.filter(v => v.date === dateStr);
                
                // Заголовок с датой
                document.getElementById('selected-date').textContent = formatDate(date);
                
                // Часы с 8:00 до 20:00
                for (let hour = 8; hour <= 20; hour++) {
                    const hourDiv = document.createElement('div');
                    hourDiv.className = 'hour-slot';
                    
                    const hourLabel = document.createElement('div');
                    hourLabel.className = 'hour-label flex items-center px-3 py-1 text-sm text-gray-500';
                    hourLabel.innerHTML = `<span class="w-10">${hour}:00</span>`;
                    
                    const hourContent = document.createElement('div');
                    hourContent.className = 'px-3 pb-2';
                    
                    // Найти посещения для этого часа
                    const hourVisits = dayVisits.filter(v => {
                        const visitHour = parseInt(v.time.split(':')[0]);
                        return visitHour === hour;
                    });
                    
                    if (hourVisits.length > 0) {
                        hourVisits.forEach(visit => {
                            const visitDiv = document.createElement('div');
                            visitDiv.className = 'mb-2 last:mb-0';
                            
                            const dotColor = visit.type === 'doctor' ? 'bg-blue-500' : 
                                            visit.type === 'pharmacy' ? 'bg-green-500' :
                                            visit.type === 'hospital' ? 'bg-purple-500' : 'bg-yellow-500';
                            
                            visitDiv.innerHTML = `
                                <div class="flex items-start">
                                    <div class="w-2 h-2 mt-1.5 rounded-full ${dotColor} mr-2"></div>
                                    <div class="flex-1">
                                        <p class="text-sm font-medium">${visit.institution}</p>
                                        <p class="text-xs text-gray-500">${formatTime(visit.time)}</p>
                                    </div>
                                </div>
                            `;
                            
                            visitDiv.addEventListener('click', () => openVisitModal(visit));
                            hourContent.appendChild(visitDiv);
                        });
                    } else {
                        hourContent.innerHTML = '<p class="text-xs text-gray-400">Нет запланированных посещений</p>';
                    }
                    
                    hourDiv.appendChild(hourLabel);
                    hourDiv.appendChild(hourContent);
                    dayContainer.appendChild(hourDiv);
                }
            }
            
            // Отобразить неделю
            function renderWeek(date) {
                const weekContainer = document.getElementById('week-hours');
                weekContainer.innerHTML = '';
                
                const weekStart = getWeekStartDate(new Date(date));
                const weekEnd = new Date(weekStart);
                weekEnd.setDate(weekStart.getDate() + 6);
                
                // Часы с 8:00 до 20:00
                for (let hour = 8; hour <= 20; hour++) {
                    const hourRow = document.createElement('div');
                    hourRow.className = 'grid grid-cols-8 gap-1';
                    
                    // Метка часа
                    const hourLabel = document.createElement('div');
                    hourLabel.className = 'flex items-center justify-center text-sm text-gray-500 border-r';
                    hourLabel.textContent = `${hour}:00`;
                    hourRow.appendChild(hourLabel);
                    
                    // Дни недели
                    for (let i = 0; i < 7; i++) {
                        const day = new Date(weekStart);
                        day.setDate(weekStart.getDate() + i);
                        const dayStr = day.toISOString().split('T')[0];
                        
                        const dayCell = document.createElement('div');
                        dayCell.className = 'p-1 border min-h-12';
                        
                        // Найти посещения для этого часа и дня
                        const hourVisits = visits.filter(v => {
                            const visitDate = v.date;
                            const visitHour = parseInt(v.time.split(':')[0]);
                            return visitDate === dayStr && visitHour === hour;
                        });
                        
                        if (hourVisits.length > 0) {
                            hourVisits.forEach(visit => {
                                const dotColor = visit.type === 'doctor' ? 'bg-blue-500' : 
                                                visit.type === 'pharmacy' ? 'bg-green-500' :
                                                visit.type === 'hospital' ? 'bg-purple-500' : 'bg-yellow-500';
                                
                                const visitDot = document.createElement('div');
                                visitDot.className = `w-2 h-2 rounded-full ${dotColor} mx-auto`;
                                visitDot.title = `${visit.institution} в ${formatTime(visit.time)}`;
                                visitDot.addEventListener('click', () => openVisitModal(visit));
                                dayCell.appendChild(visitDot);
                            });
                        }
                        
                        hourRow.appendChild(dayCell);
                    }
                    
                    weekContainer.appendChild(hourRow);
                }
            }
            
            // Отобразить месяц
            function renderMonth(date) {
                const monthContainer = document.getElementById('month-days');
                monthContainer.innerHTML = '';
                
                const year = date.getFullYear();
                const month = date.getMonth();
                
                // Первый день месяца
                const firstDay = new Date(year, month, 1);
                // Последний день месяца
                const lastDay = new Date(year, month + 1, 0);
                
                // День недели первого дня месяца (0 - воскресенье, 1 - понедельник и т.д.)
                const firstDayOfWeek = firstDay.getDay();
                // Корректировка для отображения понедельника первым
                const startOffset = firstDayOfWeek === 0 ? 6 : firstDayOfWeek - 1;
                
                // Дата в заголовке
                const monthName = date.toLocaleDateString('ru-RU', { month: 'long', year: 'numeric' });
                
                // Заполнить предыдущие дни (если месяц начинается не с понедельника)
                for (let i = 0; i < startOffset; i++) {
                    const prevDay = new Date(firstDay);
                    prevDay.setDate(firstDay.getDate() - (startOffset - i));
                    
                    const dayElement = document.createElement('div');
                    dayElement.className = 'calendar-day text-center py-2 text-gray-400';
                    dayElement.textContent = prevDay.getDate();
                    monthContainer.appendChild(dayElement);
                }
                
                // Заполнить дни месяца
                for (let i = 1; i <= lastDay.getDate(); i++) {
                    const currentDay = new Date(year, month, i);
                    const dayStr = currentDay.toISOString().split('T')[0];
                    
                    const dayVisits = visits.filter(v => v.date === dayStr);
                    
                    const dayElement = document.createElement('div');
                    dayElement.className = 'calendar-day text-center py-2 cursor-pointer';
                    
                    // Выделить сегодняшний день
                    const today = new Date();
                    if (currentDay.toDateString() === today.toDateString()) {
                        dayElement.classList.add('bg-blue-100', 'font-medium');
                    }
                    
                    // Выделить выбранный день
                    if (currentDay.toDateString() === selectedDate.toDateString()) {
                        dayElement.classList.add('active');
                    }
                    
                    dayElement.textContent = i;
                    
                    // Добавить точки для дней с посещениями
                    if (dayVisits.length > 0) {
                        const dotsContainer = document.createElement('div');
                        dotsContainer.className = 'flex justify-center space-x-1 mt-1';
                        
                        dayVisits.slice(0, 3).forEach(visit => {
                            const dotColor = visit.type === 'doctor' ? 'bg-blue-500' : 
                                            visit.type === 'pharmacy' ? 'bg-green-500' :
                                            visit.type === 'hospital' ? 'bg-purple-500' : 'bg-yellow-500';
                            
                            const dot = document.createElement('div');
                            dot.className = `w-2 h-2 rounded-full ${dotColor}`;
                            dot.title = `${visit.institution} в ${formatTime(visit.time)}`;
                            dotsContainer.appendChild(dot);
                        });
                        
                        if (dayVisits.length > 3) {
                            const moreDot = document.createElement('div');
                            moreDot.className = 'w-2 h-2 rounded-full bg-gray-300';
                            moreDot.title = `Ещё ${dayVisits.length - 3} посещений`;
                            dotsContainer.appendChild(moreDot);
                        }
                        
                        dayElement.appendChild(dotsContainer);
                    }
                    
                    dayElement.addEventListener('click', () => {
                        selectedDate = currentDay;
                        renderMonth(date);
                        setView('day');
                        renderDay(selectedDate);
                    });
                    
                    monthContainer.appendChild(dayElement);
                }
                
                // Заполнить оставшиеся дни (чтобы сетка была ровной)
                const totalCells = startOffset + lastDay.getDate();
                const remainingCells = 7 - (totalCells % 7);
                
                if (remainingCells < 7) {
                    for (let i = 1; i <= remainingCells; i++) {
                        const nextDay = new Date(lastDay);
                        nextDay.setDate(lastDay.getDate() + i);
                        
                        const dayElement = document.createElement('div');
                        dayElement.className = 'calendar-day text-center py-2 text-gray-400';
                        dayElement.textContent = nextDay.getDate();
                        monthContainer.appendChild(dayElement);
                    }
                }
            }
            
            // Установить вид (день, неделя, месяц)
            function setView(view) {
                document.querySelectorAll('.period-btn').forEach(btn => {
                    btn.classList.remove('active', 'bg-blue-100', 'text-blue-700');
                    btn.classList.add('hover:bg-gray-100');
                });
                
                document.getElementById(`${view}-view`).classList.remove('hidden');
                
                if (view === 'day') {
                    document.querySelector('.period-btn[data-period="day"]').classList.add('active', 'bg-blue-100', 'text-blue-700');
                    document.getElementById('week-view').classList.add('hidden');
                    document.getElementById('month-view').classList.add('hidden');
                } else if (view === 'week') {
                    document.querySelector('.period-btn[data-period="week"]').classList.add('active', 'bg-blue-100', 'text-blue-700');
                    document.getElementById('day-view').classList.add('hidden');
                    document.getElementById('month-view').classList.add('hidden');
                } else if (view === 'month') {
                    document.querySelector('.period-btn[data-period="month"]').classList.add('active', 'bg-blue-100', 'text-blue-700');
                    document.getElementById('day-view').classList.add('hidden');
                    document.getElementById('week-view').classList.add('hidden');
                }
            }
            
            // Открыть модальное окно посещения
            function openVisitModal(visit) {
                document.getElementById('modal-title').textContent = `Посещение ${visit.institution}`;
                document.getElementById('modal-type').textContent = 
                    visit.type === 'doctor' ? 'Врач' : 
                    visit.type === 'pharmacy' ? 'Аптека' : 
                    visit.type === 'hospital' ? 'Больница' : 'Клиника';
                
                document.getElementById('modal-institution').textContent = visit.institution;
                
                const visitDate = new Date(visit.date);
                const options = { day: 'numeric', month: 'long', hour: '2-digit', minute: '2-digit' };
                document.getElementById('modal-datetime').textContent = 
                    `${visitDate.toLocaleDateString('ru-RU', { day: 'numeric', month: 'long' })}, ${visit.time}`;
                
                document.getElementById('modal-notes').textContent = visit.notes || 'Нет заметок';
                
                // Проверить статус посещения (прошло или нет)
                const now = new Date();
                const visitDateTime = new Date(`${visit.date}T${visit.time}`);
                const status = now > visitDateTime ? 'Завершено' : 'Запланировано';
                const statusClass = now > visitDateTime ? 'bg-green-100 text-green-800' : 'bg-blue-100 text-blue-800';
                
                document.getElementById('modal-status').innerHTML = 
                    `<span class="px-2 py-1 ${statusClass} rounded-full text-xs">${status}</span>`;
                
                // Установить ID посещения для редактирования/удаления
                document.getElementById('visit-modal').setAttribute('data-visit-id', visit.id);
                
                // Показать модальное окно
                document.getElementById('visit-modal').classList.remove('hidden');
            }
            
            // Закрыть модальное окно
            document.getElementById('close-modal').addEventListener('click', () => {
                document.getElementById('visit-modal').classList.add('hidden');
            });
            
            // Удалить посещение
            document.getElementById('delete-visit').addEventListener('click', () => {
                const visitId = parseInt(document.getElementById('visit-modal').getAttribute('data-visit-id'));
                visits = visits.filter(v => v.id !== visitId);
                saveData();
                renderDay(selectedDate);
                document.getElementById('visit-modal').classList.add('hidden');
            });
            
            // Редактировать посещение (заполнить форму)
            document.getElementById('edit-visit').addEventListener('click', () => {
                const visitId = parseInt(document.getElementById('visit-modal').getAttribute('data-visit-id'));
                const visit = visits.find(v => v.id === visitId);
                
                if (visit) {
                    document.getElementById('visit-type').value = visit.type;
                    document.getElementById('institution').value = visit.institution;
                    document.getElementById('visit-date').value = visit.date;
                    document.getElementById('visit-time').value = visit.time;
                    document.getElementById('visit-notes').value = visit.notes || '';
                    
                    // Изменить текст кнопки на "Обновить"
                    const submitBtn = document.querySelector('#visit-form button[type="submit"]');
                    submitBtn.innerHTML = '<i class="fas fa-save mr-2"></i> Обновить';
                    submitBtn.setAttribute('data-edit-id', visit.id);
                    
                    document.getElementById('visit-modal').classList.add('hidden');
                    
                    // Прокрутить к форме
                    document.getElementById('visit-form').scrollIntoView({ behavior: 'smooth' });
                }
            });
            
            // Обработчик формы посещения
            document.getElementById('visit-form').addEventListener('submit', function(e) {
                e.preventDefault();
                
                const type = document.getElementById('visit-type').value;
                const institution = document.getElementById('institution').value.trim();
                const date = document.getElementById('visit-date').value;
                const time = document.getElementById('visit-time').value;
                const notes = document.getElementById('visit-notes').value.trim();
                
                if (!institution) {
                    alert('Пожалуйста, укажите учреждение или врача');
                    return;
                }
                
                const submitBtn = this.querySelector('button[type="submit"]');
                const isEdit = submitBtn.hasAttribute('data-edit-id');
                
                if (isEdit) {
                    // Редактирование существующего посещения
                    const editId = parseInt(submitBtn.getAttribute('data-edit-id'));
                    const visitIndex = visits.findIndex(v => v.id === editId);
                    
                    if (visitIndex !== -1) {
                        visits[visitIndex] = {
                            id: editId,
                            type,
                            institution,
                            date,
                            time,
                            notes
                        };
                    }
                    
                    // Вернуть кнопку в исходное состояние
                    submitBtn.innerHTML = '<i class="fas fa-plus mr-2"></i> Добавить посещение';
                    submitBtn.removeAttribute('data-edit-id');
                } else {
                    // Добавление нового посещения
                    const newId = visits.length > 0 ? Math.max(...visits.map(v => v.id)) + 1 : 1;
                    
                    visits.push({
                        id: newId,
                        type,
                        institution,
                        date,
                        time,
                        notes
                    });
                }
                
                saveData();
                renderDay(selectedDate);
                this.reset();
                
                // Установить текущую дату по умолчанию
                document.getElementById('visit-date').value = selectedDate.toISOString().split('T')[0];
            });
            
            // Обработчик кнопок периода
            document.querySelectorAll('.period-btn').forEach(btn => {
                btn.addEventListener('click', function() {
                    const period = this.getAttribute('data-period');
                    setView(period);
                    
                    if (period === 'day') {
                        renderDay(selectedDate);
                    } else if (period === 'week') {
                        renderWeek(selectedDate);
                    } else if (period === 'month') {
                        renderMonth(currentDate);
                    }
                });
            });
            
            // Кнопки навигации по дням
            document.getElementById('prev-day').addEventListener('click', () => {
                selectedDate.setDate(selectedDate.getDate() - 1);
                renderDay(selectedDate);
            });
            
            document.getElementById('next-day').addEventListener('click', () => {
                selectedDate.setDate(selectedDate.getDate() + 1);
                renderDay(selectedDate);
            });
            
            document.getElementById('today-btn').addEventListener('click', () => {
                selectedDate = new Date();
                renderDay(selectedDate);
            });
            
            // Кнопки навигации по месяцам
            document.getElementById('prev-month').addEventListener('click', () => {
                currentDate.setMonth(currentDate.getMonth() - 1);
                renderMonth(currentDate);
            });
            
            document.getElementById('next-month').addEventListener('click', () => {
                currentDate.setMonth(currentDate.getMonth() + 1);
                renderMonth(currentDate);
            });
            
            // Заметки
            function renderNotes() {
                const notesContainer = document.getElementById('notes-container');
                notesContainer.innerHTML = '';
                
                if (notes.length === 0) {
                    notesContainer.innerHTML = '<p class="text-gray-500 text-sm py-4 text-center">Нет заметок</p>';
                    return;
                }
                
                // Сортировка по дате (новые сначала)
                const sortedNotes = [...notes].sort((a, b) => {
                    const dateA = new Date(`${a.date}T${a.time || '00:00'}`);
                    const dateB = new Date(`${b.date}T${b.time || '00:00'}`);
                    return dateB - dateA;
                });
                
                sortedNotes.forEach(note => {
                    const noteElement = document.createElement('div');
                    noteElement.className = 'note-card bg-yellow-50 border-l-4 border-yellow-400 rounded-lg p-3 shadow-sm';
                    noteElement.innerHTML = `
                        <div class="flex justify-between items-start">
                            <div>
                                <h3 class="font-medium text-gray-800">${note.title}</h3>
                                <p class="text-sm text-gray-600">${note.date}${note.time ? ', ' + note.time : ''}</p>
                            </div>
                            <button class="text-gray-400 hover:text-gray-600 delete-note" data-note-id="${note.id}">
                                <i class="fas fa-times"></i>
                            </button>
                        </div>
                        <p class="mt-2 text-gray-700 text-sm">${note.content}</p>
                    `;
                    
                    notesContainer.appendChild(noteElement);
                });
                
                // Обработчики удаления заметок
                document.querySelectorAll('.delete-note').forEach(btn => {
                    btn.addEventListener('click', function() {
                        const noteId = parseInt(this.getAttribute('data-note-id'));
                        notes = notes.filter(n => n.id !== noteId);
                        saveData();
                        renderNotes();
                    });
                });
            }
            
            // Показать/скрыть форму добавления заметки
            document.getElementById('add-note-btn').addEventListener('click', function() {
                document.getElementById('note-form-container').classList.remove('hidden');
                document.getElementById('note-title').focus();
            });
            
            document.getElementById('cancel-note-btn').addEventListener('click', function() {
                document.getElementById('note-form-container').classList.add('hidden');
                document.getElementById('note-form').reset();
            });
            
            // Обработчик формы заметки
            document.getElementById('note-form').addEventListener('submit', function(e) {
                e.preventDefault();
                
                const title = document.getElementById('note-title').value.trim();
                const content = document.getElementById('note-content').value.trim();
                
                if (!title || !content) {
                    alert('Пожалуйста, заполните все поля');
                    return;
                }
                
                const newId = notes.length > 0 ? Math.max(...notes.map(n => n.id)) + 1 : 1;
                const today = new Date().toISOString().split('T')[0];
                
                notes.push({
                    id: newId,
                    title,
                    content,
                    date: today,
                    time: new Date().toLocaleTimeString('ru-RU', { hour: '2-digit', minute: '2-digit' })
                });
                
                saveData();
                renderNotes();
                this.reset();
                document.getElementById('note-form-container').classList.add('hidden');
            });
            
            // Комментарии
            function renderComments() {
                const commentsContainer = document.getElementById('comments-container');
                commentsContainer.innerHTML = '';
                
                if (comments.length === 0) {
                    commentsContainer.innerHTML = '<p class="text-gray-500 text-sm py-4 text-center">Нет комментариев</p>';
                    return;
                }
                
                // Сортировка по дате (новые сначала)
                const sortedComments = [...comments].sort((a, b) => new Date(b.date) - new Date(a.date));
                
                sortedComments.forEach(comment => {
                    const commentDate = new Date(comment.date);
                    const formattedDate = commentDate.toLocaleDateString('ru-RU', { day: 'numeric', month: 'long' }) + ', ' + 
                                          commentDate.toLocaleTimeString('ru-RU', { hour: '2-digit', minute: '2-digit' });
                    
                    const commentElement = document.createElement('div');
                    commentElement.className = 'bg-gray-50 rounded-lg p-3';
                    commentElement.innerHTML = `
                        <div class="flex justify-between items-start">
                            <div>
                                <p class="font-medium text-gray-800">${comment.author}</p>
                                <p class="text-xs text-gray-500">${formattedDate}</p>
                            </div>
                            <button class="text-gray-400 hover:text-gray-600 text-sm delete-comment" data-comment-id="${comment.id}">
                                <i class="fas fa-trash mr-1"></i>
                            </button>
                        </div>
                        <p class="mt-2 text-gray-700 text-sm">${comment.content}</p>
                    `;
                    
                    commentsContainer.appendChild(commentElement);
                });
                
                // Обработчики удаления комментариев
                document.querySelectorAll('.delete-comment').forEach(btn => {
                    btn.addEventListener('click', function() {
                        const commentId = this.getAttribute('data-comment-id');
                        comments = comments.filter(c => c.id !== commentId);
                        saveData();
                        renderComments();
                    });
                });
            }
            
            // Обработчик формы комментария
            document.getElementById('comment-form').addEventListener('submit', function(e) {
                e.preventDefault();
                
                const content = document.getElementById('comment-input').value.trim();
                
                if (!content) {
                    alert('Пожалуйста, введите комментарий');
                    return;
                }
                
                const newId = comments.length > 0 ? Math.max(...comments.map(c => c.id)) + 1 : 1;
                const now = new Date();
                
                comments.push({
                    id: newId,
                    author: 'Вы',
                    date: now.toISOString(),
                    content
                });
                
                saveData();
                renderComments();
                this.reset();
            });
            
            // Инициализация
            function init() {
                // Установить текущую дату в форме
                const todayStr = new Date().toISOString().split('T')[0];
                document.getElementById('visit-date').value = todayStr;
                
                // Установить текущее время + 1 час
                const now = new Date();
                now.setHours(now.getHours() + 1);
                const hours = now.getHours().toString().padStart(2, '0');
                const minutes = now.getMinutes().toString().padStart(2, '0');
                document.getElementById('visit-time').value = `${hours}:${minutes}`;
                
                // Начальный вид - день
                setView('day');
                renderDay(selectedDate);
                renderMonth(currentDate);
                renderWeek(selectedDate);
                renderNotes();
                renderComments();
                updateStatistics();
            }
            
            init();
        });
    </script>
</body>
</html>
