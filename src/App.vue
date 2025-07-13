<template>
  <div class="app-container">
    <!-- Заголовок с общим прогрессом -->
    <header class="app-header">
      <div class="header-content">
        <h1 class="app-title">Мой идеальный график</h1>
        <div class="overall-progress">
          <div class="progress-circle" :style="{ '--progress': overallProgress }">
            <span class="progress-value">{{ overallProgress }}%</span>
          </div>
          <span class="progress-label">Недельный прогресс</span>
        </div>
      </div>
    </header>

    <!-- Основное содержимое -->
    <main class="app-main">
      <!-- Карточка текущего дня недели -->
      <div
          v-if="currentDayCard"
          class="day-card"
          :class="{ 'completed-card': progress[currentDayCard.day] === 100 }"
      >
        <div class="card-header">
          <h2 class="day-title">{{ currentDayCard.day }}</h2>
          <div class="day-progress">
            <div class="progress-container">
              <div
                  class="progress-bar"
                  :style="{ width: progress[currentDayCard.day] + '%' }"
                  :class="{
                  'low-progress': progress[currentDayCard.day] < 30,
                  'medium-progress': progress[currentDayCard.day] >= 30 && progress[currentDayCard.day] < 70,
                  'high-progress': progress[currentDayCard.day] >= 70
                }"
              ></div>
            </div>
            <span class="progress-percent">{{ progress[currentDayCard.day] }}%</span>
          </div>
        </div>

        <div class="tasks-list">
          <div
              v-for="(task, index) in parsedSchedule[currentDayCard.day]"
              :key="index"
              class="task-item"
              :class="{ 'completed': completed[currentDayCard.day][index] }"
          >
            <div class="task-checkbox" @click="toggleTask(currentDayCard.day, index)">
              <div class="checkbox-inner">
                <svg
                    v-if="completed[currentDayCard.day][index]"
                    class="check-icon"
                    viewBox="0 0 24 24"
                >
                  <path d="M5 13l4 4L19 7" />
                </svg>
              </div>
            </div>
            <span class="task-text">{{ task }}</span>
          </div>
        </div>
      </div>

      <!-- Блок с рекомендациями -->
      <div class="nutrition-card">
        <div class="nutrition-header">
          <svg class="nutrition-icon" viewBox="0 0 24 24">
            <path d="M12 2L4 12l8 10 8-10L12 2zm0 4.5l5.5 5.5H12V6.5z" />
          </svg>
          <h2 class="nutrition-title">Рекомендации по питанию</h2>
        </div>
        <ul class="nutrition-list">
          <li v-for="(item, index) in nutritionTips" :key="index" class="nutrition-item">
            <div class="nutrition-bullet"></div>
            <span>{{ item }}</span>
          </li>
        </ul>
      </div>
    </main>

    <!-- Кнопка сброса прогресса -->
    <button @click="resetProgress" class="reset-button">
      <svg class="reset-icon" viewBox="0 0 24 24">
        <path d="M17.65 6.35A8 8 0 106 7.7L4 4v8h8l-2.65-2.65a6 6 0 11-1.41 1.41z" />
      </svg>
      Сбросить прогресс
    </button>
  </div>
</template>

<script setup>
import { reactive, computed, onMounted } from 'vue';

const nutritionTips = [
  "Завтрак: 3–4 яйца, 1–2 тоста с творожным сыром, чай / кофе",
  "Обед: индейка/говядина + макароны/рис + овощи (отварные или свежие)",
  "Ужин: белковая пища (мясо/рыба/яйца) + клетчатка (овощи)",
  "Перекусы: протеиновые батончики, орехи, творог, мясные снеки",
  "Перед сном (по желанию): кефир / греческий йогурт",
  "Добавки: 1 ложка креатина с водой (утром или после тренировки)",
  "Рассмотри добавление гейнера 1 раз в день, если плохо идёт масса"
];

const schedule = {
  'Понедельник / Среда / Пятница': `07:30–08:00 — Подъем, вода, зарядка
08:00–08:30 — Японский язык (базовая грамматика + лексика)
08:30–09:00 — Завтрак (белковый + углеводы)
09:00–11:30 — Программирование (пет-проект / обучение)
11:30–12:00 — Перерыв / перекус
12:00–13:00 — Работа с резюме / отклики / фриланс
13:00–13:30 — Обед (индейка, макароны, овощи)
13:30–15:00 — Практика / курс по React / Vue
15:00–16:30 — Тренировка на улице + растяжка
16:30–17:00 — Перекус + отдых
17:00–19:00 — Решение задач по NestJS / pet-проект
19:00–20:00 — Ужин
20:00–22:30 — Свободное время (друзья, скейт, музыка, ПК)
22:30–23:30 — Вечерняя рутина, чтение, отдых`,

  'Вторник / Четверг': `07:30–08:00 — Подъем, вода, легкая зарядка
08:00–08:30 — Японский язык (Anki + грамматика)
08:30–09:00 — Завтрак
09:00–11:30 — Глубокая работа (кодинг, pet-проекты)
11:30–12:00 — Отдых, прогулка
12:00–13:00 — Продолжение обучения / курс
13:00–13:30 — Обед
13:30–15:30 — Поиск работы / фриланс-платформы
15:30–17:00 — Тематическая практика (React + задачи)
17:00–19:00 — Хобби (музыка / изучение API / отдых)
19:00–20:00 — Ужин
20:00–22:30 — Игры / соц. время / лёгкий созвон
22:30–23:30 — Спокойный вечер (нет экрана за 30 мин до сна)`,

  'Суббота': `08:30–09:00 — Подъем, вода
09:00–09:30 — Японский язык
09:30–10:00 — Завтрак
10:00–12:00 — Тренировка на улице / пробежка
12:00–13:00 — Обед
13:00–15:00 — Лёгкий проект / фриланс / творчество
15:00–19:00 — Свободное окно — скейт, встреча с друзьями
19:00–20:00 — Ужин
20:00–23:00 — Игры / просмотр / музыка
23:00–23:30 — Отдых, душ, сон`,

  'Воскресенье': `~09:00 — Подъем без будильника
09:00–09:30 — Японский язык (повторение)
09:30–10:30 — Завтрак / кофе / прогулка
10:30–12:00 — Ретроспектива недели / план на следующую
12:00–19:00 — Полный отдых (друзья, игры, выход в город)
19:00–20:00 — Ужин
20:00–22:00 — Фильм / рефлексия / чтение
22:00–23:00 — Подготовка ко сну`
};

const parsedSchedule = Object.fromEntries(
    Object.entries(schedule).map(([day, text]) => [day, text.split('\n')])
);

const completed = reactive(
    Object.fromEntries(
        Object.entries(parsedSchedule).map(([day, tasks]) => [day, Array(tasks.length).fill(false)])
    )
);

const progress = reactive(
    Object.fromEntries(
        Object.entries(parsedSchedule).map(([day, tasks]) => [day, 0])
    )
);

const currentDayCard = computed(() => {
  const daysMap = {
    0: 'Воскресенье',
    1: 'Понедельник / Среда / Пятница',
    2: 'Вторник / Четверг',
    3: 'Понедельник / Среда / Пятница',
    4: 'Вторник / Четверг',
    5: 'Понедельник / Среда / Пятница',
    6: 'Суббота'
  };

  const today = new Date().getDay(); // 0 - воскресенье, 1 - понедельник и т.д.
  const dayKey = daysMap[today];

  return {
    day: dayKey,
    tasks: parsedSchedule[dayKey]
  };
});

const overallProgress = computed(() => {
  const days = Object.keys(progress);
  const total = days.reduce((sum, day) => sum + progress[day], 0);
  return Math.round(total / days.length);
});

function toggleTask(day, index) {
  completed[day][index] = !completed[day][index];
  updateProgress(day);
  saveToLocalStorage();
}

function updateProgress(day) {
  const total = completed[day].length;
  const done = completed[day].filter(Boolean).length;
  progress[day] = Math.round((done / total) * 100);
}

function saveToLocalStorage() {
  localStorage.setItem('scheduleProgress', JSON.stringify(completed));
}

function loadFromLocalStorage() {
  const saved = localStorage.getItem('scheduleProgress');
  if (saved) {
    const savedData = JSON.parse(saved);
    Object.keys(savedData).forEach(day => {
      if (completed[day]) {
        savedData[day].forEach((val, index) => {
          if (index < completed[day].length) {
            completed[day][index] = val;
          }
        });
        updateProgress(day);
      }
    });
  }
}

function resetProgress() {
  Object.keys(completed).forEach(day => {
    completed[day] = completed[day].map(() => false);
    updateProgress(day);
  });
  saveToLocalStorage();
}

onMounted(() => {
  loadFromLocalStorage();

  // Проверка PWA режима
  if (window.matchMedia('(display-mode: standalone)').matches) {
    console.log('Запущено как PWA');
    document.documentElement.classList.add('pwa-mode');
  }
});
</script>

<style>
/* Базовые стили */
:root {
  --primary-color: #4361ee;
  --secondary-color: #3a0ca3;
  --accent-color: #4cc9f0;
  --success-color: #2ec4b6;
  --warning-color: #ff9f1c;
  --danger-color: #e71d36;
  --light-color: #f8f9fa;
  --dark-color: #212529;
  --text-color: #2b2d42;
  --text-light: #8d99ae;
  --bg-gradient: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  --card-shadow: 0 10px 20px rgba(0, 0, 0, 0.1);
  --transition: all 0.3s ease;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html {
  font-size: 16px;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  color: var(--text-color);
  background: var(--bg-gradient);
  min-height: 100vh;
  line-height: 1.6;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* Контейнер приложения */
.app-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 1rem;
  padding-bottom: 4rem;
  position: relative;
}

/* Шапка приложения */
.app-header {
  background: linear-gradient(135deg, var(--primary-color), var(--secondary-color));
  border-radius: 1rem;
  padding: 1.5rem;
  margin-bottom: 2rem;
  box-shadow: var(--card-shadow);
  color: white;
  position: relative;
  overflow: hidden;
}

.app-header::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIxMDAlIiBoZWlnaHQ9IjEwMCUiPjxkZWZzPjxwYXR0ZXJuIGlkPSJwYXR0ZXJuIiB3aWR0aD0iNDAiIGhlaWdodD0iNDAiIHBhdHRlcm5Vbml0cz0idXNlclNwYWNlT25Vc2UiIHBhdHRlcm5UcmFuc2Zvcm09InJvdGF0ZSg0NSkiPjxyZWN0IHdpZHRoPSIyMCIgaGVpZ2h0PSIyMCIgZmlsbD0icmdiYSgyNTUsMjU1LDI1NSwwLjA1KSIvPjwvcGF0dGVybj48L2RlZnM+PHJlY3Qgd2lkdGg9IjEwMCUiIGhlaWdodD0iMTAwJSIgZmlsbD0idXJsKCNwYXR0ZXJuKSIvPjwvc3ZnPg==');
  opacity: 0.5;
}

.header-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  position: relative;
  z-index: 1;
}

.app-title {
  font-size: 1.5rem;
  font-weight: 700;
  margin: 0;
}

.overall-progress {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.progress-circle {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background: conic-gradient(var(--accent-color) calc(var(--progress) * 3.6deg), #ffffff33 calc(var(--progress) * 3.6deg));
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 0 0 3px rgba(255, 255, 255, 0.2);
  transition: var(--transition);
}

.progress-value {
  font-weight: 700;
  font-size: 0.9rem;
  color: white;
}

.progress-label {
  font-size: 0.7rem;
  margin-top: 0.3rem;
  opacity: 0.8;
}

/* Основное содержимое */
.app-main {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

/* Карточка дня */
.day-card {
  background: white;
  border-radius: 1rem;
  overflow: hidden;
  box-shadow: var(--card-shadow);
  transition: var(--transition);
}

.day-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 15px 30px rgba(0, 0, 0, 0.15);
}

.day-card.completed-card {
  border-left: 5px solid var(--success-color);
}

.card-header {
  padding: 1.2rem;
  background: linear-gradient(to right, #f8f9fa, #e9ecef);
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.day-title {
  font-size: 1.1rem;
  font-weight: 600;
  margin: 0;
  color: var(--text-color);
}

.day-progress {
  display: flex;
  align-items: center;
  gap: 0.8rem;
}

.progress-container {
  width: 100px;
  height: 8px;
  background: #e9ecef;
  border-radius: 4px;
  overflow: hidden;
}

.progress-bar {
  height: 100%;
  border-radius: 4px;
  transition: width 0.6s ease;
}

.low-progress {
  background: linear-gradient(to right, #ff758c, #ff7eb3);
}

.medium-progress {
  background: linear-gradient(to right, #ffb347, #ffcc33);
}

.high-progress {
  background: linear-gradient(to right, #56ab2f, #a8e063);
}

.progress-percent {
  font-size: 0.9rem;
  font-weight: 600;
  min-width: 35px;
  text-align: right;
}

/* Список задач */
.tasks-list {
  padding: 0.5rem 1.2rem 1.2rem;
}

.task-item {
  display: flex;
  align-items: flex-start;
  gap: 0.8rem;
  padding: 0.8rem 0;
  border-bottom: 1px solid #f1f3f5;
  transition: var(--transition);
}

.task-item:last-child {
  border-bottom: none;
}

.task-item.completed {
  opacity: 0.7;
}

.task-checkbox {
  flex-shrink: 0;
  width: 22px;
  height: 22px;
  border-radius: 6px;
  border: 2px solid #adb5bd;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: var(--transition);
  position: relative;
  margin-top: 2px;
}

.task-item.completed .task-checkbox {
  background: var(--success-color);
  border-color: var(--success-color);
}

.checkbox-inner {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.check-icon {
  width: 14px;
  height: 14px;
  stroke: white;
  stroke-width: 3;
  opacity: 0;
  transform: scale(0.8);
  transition: var(--transition);
}

.task-item.completed .check-icon {
  opacity: 1;
  transform: scale(1);
}

.task-text {
  font-size: 0.9rem;
  transition: var(--transition);
}

.task-item.completed .task-text {
  text-decoration: line-through;
  color: var(--text-light);
}

/* Карточка с рекомендациями */
.nutrition-card {
  background: white;
  border-radius: 1rem;
  padding: 1.5rem;
  box-shadow: var(--card-shadow);
  margin-top: 1rem;
}

.nutrition-header {
  display: flex;
  align-items: center;
  gap: 0.8rem;
  margin-bottom: 1.2rem;
}

.nutrition-icon {
  width: 24px;
  height: 24px;
  fill: var(--primary-color);
}

.nutrition-title {
  font-size: 1.2rem;
  font-weight: 600;
  margin: 0;
  color: var(--text-color);
}

.nutrition-list {
  list-style: none;
}

.nutrition-item {
  display: flex;
  align-items: flex-start;
  gap: 0.8rem;
  padding: 0.5rem 0;
  font-size: 0.9rem;
}

.nutrition-bullet {
  flex-shrink: 0;
  width: 8px;
  height: 8px;
  background: var(--primary-color);
  border-radius: 50%;
  margin-top: 6px;
}

/* Кнопка сброса */
.reset-button {
  position: fixed;
  bottom: 1.5rem;
  left: 50%;
  transform: translateX(-50%);
  background: white;
  color: var(--danger-color);
  border: none;
  border-radius: 50px;
  padding: 0.8rem 1.5rem;
  font-size: 0.9rem;
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
  cursor: pointer;
  transition: var(--transition);
  z-index: 10;
}

.reset-button:hover {
  background: #f8f9fa;
  transform: translateX(-50%) scale(1.05);
}

.reset-icon {
  width: 16px;
  height: 16px;
  fill: var(--danger-color);
}

/* Адаптация для iPhone */
@supports (-webkit-touch-callout: none) {
  body {
    padding-bottom: env(safe-area-inset-bottom);
  }

  .app-container {
    padding-bottom: calc(4rem + env(safe-area-inset-bottom));
  }

  .reset-button {
    bottom: calc(1.5rem + env(safe-area-inset-bottom));
  }
}

/* Темная тема */
@media (prefers-color-scheme: dark) {
  :root {
    --text-color: #f8f9fa;
    --text-light: #adb5bd;
    --light-color: #212529;
    --dark-color: #f8f9fa;
    --bg-gradient: linear-gradient(135deg, #212529 0%, #343a40 100%);
    --card-shadow: 0 10px 20px rgba(0, 0, 0, 0.3);
  }

  .day-card,
  .nutrition-card {
    background: #343a40;
  }

  .card-header {
    background: linear-gradient(to right, #495057, #343a40);
  }

  .day-title {
    color: white;
  }

  .progress-container {
    background: #495057;
  }

  .task-item {
    border-bottom-color: #495057;
  }

  .reset-button {
    background: #495057;
    color: white;
  }

  .reset-icon {
    fill: white;
  }
}
</style>