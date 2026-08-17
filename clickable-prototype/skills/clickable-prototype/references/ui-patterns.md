# Этап 5. Паттерны экранов

Куски, из которых складывается почти любой внутренний интерфейс. Бери и наполняй
данными клиента.

Содержание:

- [Отрисовка экрана](#отрисовка-экрана)
- [Таблица с фильтрами и сортировкой](#таблица-с-фильтрами-и-сортировкой)
- [Чипы статусов](#чипы-статусов)
- [Пошаговый мастер](#пошаговый-мастер)
- [Карточка сущности](#карточка-сущности)
- [Модальное окно](#модальное-окно)
- [Пустое состояние](#пустое-состояние)
- [Подсветка проблемы](#подсветка-проблемы)

## Отрисовка экрана

Один экран — одна функция, собирающая HTML строкой и кладущая его в контейнер.
Это не самый изящный способ, но в одном файле без сборки он самый живучий:
всё, что относится к экрану, лежит в одном месте, и переписать экран целиком
дешевле, чем чинить его по кусочкам.

```js
function renderOrders(){
  const rows = ORDERS.filter(matchesFilters);

  document.getElementById('screen-orders').innerHTML = `
    <div class="toolbar">
      <input class="input" placeholder="Поиск по клиенту или номеру"
             value="${orderQuery}" oninput="setOrderQuery(this.value)">
      <button class="btn btn-primary" onclick="startOrderWizard()">Новая заявка</button>
    </div>
    ${rows.length ? renderOrderTable(rows) : emptyState('Заявок не найдено',
        'Измените условия поиска или создайте новую заявку.')}
  `;
}
```

Состояние фильтров держи в переменных модуля, а не в DOM — тогда перерисовка
ничего не теряет:

```js
let orderQuery = '';
function setOrderQuery(v){ orderQuery = v; renderOrders(); }
```

Осторожно с фокусом: перерисовка поля ввода при каждом символе сбивает каретку.
Для поиска по таблице надёжнее перерисовывать только строки:

```js
function setOrderQuery(v){
  orderQuery = v;
  document.getElementById('orderRows').innerHTML = renderOrderRows(filtered());
}
```

## Таблица с фильтрами и сортировкой

Основной экран внутренних систем. Сортировка хранится в двух переменных,
стрелка рисуется у активной колонки.

```js
let sortCol = null, sortDir = 'asc';

function setSort(col){
  if (sortCol === col) sortDir = sortDir === 'asc' ? 'desc' : 'asc';
  else { sortCol = col; sortDir = 'asc'; }
  renderStock();
}
function sortArrow(col){
  return sortCol === col ? ` <span class="sort-arrow">${sortDir === 'asc' ? '▲' : '▼'}</span>` : '';
}
function sorted(rows){
  if (!sortCol) return rows;
  const k = sortDir === 'asc' ? 1 : -1;
  return [...rows].sort((a,b) =>
    typeof a[sortCol] === 'number' ? (a[sortCol]-b[sortCol])*k
                                   : String(a[sortCol]).localeCompare(String(b[sortCol]),'ru')*k);
}
```

```css
.table{width:100%; border-collapse:collapse; font-size:14px}
.table th{
  text-align:left; font-weight:500; color:var(--text-faint); padding:10px 12px;
  border-bottom:1px solid var(--border); cursor:pointer; white-space:nowrap;
  position:sticky; top:60px; background:var(--surface);
}
.table th:hover{color:var(--text)}
.table td{padding:11px 12px; border-bottom:1px solid var(--border)}
.table tbody tr:hover{background:var(--surface-2)}
.num{text-align:right; font-variant-numeric:tabular-nums}
```

`font-variant-numeric: tabular-nums` для колонок с числами обязателен — иначе
цифры пляшут по ширине и таблица выглядит неопрятно. `position: sticky` на
заголовках спасает длинные списки.

## Чипы статусов

Один словарь подписей, один — классов. Нигде больше статусы не хардкодятся:
добавить статус тогда значит дописать две строки, а не искать по файлу.

```js
const STATUS_LABELS = {
  done:'Выполнена', active:'В работе', waiting:'Ждёт подтверждения',
  overdue:'Просрочена', draft:'Черновик'
};
const STATUS_CLASS = {
  done:'st-done', active:'st-active', waiting:'st-waiting',
  overdue:'st-overdue', draft:'st-draft'
};
const chip = s => `<span class="chip ${STATUS_CLASS[s]}">${STATUS_LABELS[s]}</span>`;
```

```css
.chip{
  display:inline-block; padding:3px 9px; border-radius:999px;
  font-size:12.5px; font-weight:500; white-space:nowrap; border:1px solid transparent;
}
.st-done{background:var(--success-bg); color:var(--success); border-color:var(--success-border)}
.st-active{background:var(--accent-bg); color:var(--accent-dark)}
.st-waiting{background:var(--warning-bg); color:var(--warning); border-color:var(--warning-border)}
.st-overdue{background:var(--danger-bg); color:var(--danger); border-color:var(--danger-border)}
.st-draft{background:var(--surface-2); color:var(--text-muted)}
```

Цвет несёт смысл, но не должен быть единственным носителем: подпись рядом есть
всегда — иначе экран нечитаем для дальтоников и на плохом проекторе.

## Пошаговый мастер

Главный сценарий почти всегда раскладывается в мастер: выбор → параметры →
подтверждение. Черновик копится в одном объекте, шаги валидируются при переходе
вперёд и не валидируются при переходе назад.

```js
let draft = {};      // копится по шагам
let step = 1;

function goToStep(n, skipValidation){
  if (!skipValidation && n > step && !validateStep(step)) return;
  step = n;
  renderStepper();
  renderStepBody();
}

function validateStep(n){
  if (n === 1 && !draft.client){  toast('Выберите клиента'); return false; }
  if (n === 2 && !draft.service){ toast('Выберите услугу'); return false; }
  return true;
}

function renderStepper(){
  const steps = ['Клиент','Услуга','Срок и стоимость','Подтверждение'];
  document.getElementById('stepper').innerHTML = steps.map((label,i) => {
    const n = i + 1;
    const state = n < step ? 'done' : n === step ? 'current' : 'todo';
    // Назад можно вернуться кликом, вперёд — только через кнопку с проверкой
    const click = n < step ? `onclick="goToStep(${n}, true)"` : '';
    return `<div class="step ${state}" ${click}><span class="step-num">${n}</span>${label}</div>`;
  }).join('');
}
```

Возврат на пройденный шаг обязателен: на демонстрации заказчик всегда просит
«а поменяйте здесь» — и мастер без возврата разваливается.

В мобильном архетипе шаги не влезают в шапку: показывай их полосой прогресса
«Шаг 2 из 4» и оставляй кнопку «назад» в шапке экрана.

Тонкости валидации и порядка полей — `interaction-design:form-design`.

## Карточка сущности

Открывается из списка, возвращается назад. Простой способ без роутинга —
отдельный экран, который наполняется перед показом.

```js
let currentOrderId = null;

function openOrder(id){
  currentOrderId = id;
  nav('order-detail');
}

function renderOrderDetail(){
  const o = ORDERS.find(x => x.id === currentOrderId);
  document.getElementById('screen-order-detail').innerHTML = `
    <button class="btn" onclick="nav('orders')">← К списку заявок</button>
    <div class="card">
      <h2>Заявка №${o.number}</h2>
      <div class="meta">${o.client} · ${o.service} · до ${fmtDate(o.due)}</div>
      ${chip(o.status)}
    </div>
  `;
}
```

Кнопка «назад» — текстом с указанием, куда именно возвращаемся. Голая стрелка
без подписи на прототипе читается плохо.

## Модальное окно

Для коротких подтверждений и форм в один-два поля. Закрытие по клику на подложку
и по Escape — без них окно ощущается сломанным.

```js
function openModal(title, body, actions){
  document.getElementById('modalRoot').innerHTML = `
    <div class="modal-overlay" onclick="if(event.target===this) closeModal()">
      <div class="modal" role="dialog" aria-modal="true" aria-label="${title}">
        <h3>${title}</h3>
        <div class="modal-body">${body}</div>
        <div class="modal-actions">${actions}</div>
      </div>
    </div>`;
  document.addEventListener('keydown', escClose);
}
function closeModal(){
  document.getElementById('modalRoot').innerHTML = '';
  document.removeEventListener('keydown', escClose);
}
function escClose(e){ if (e.key === 'Escape') closeModal(); }
```

```css
.modal-overlay{
  position:fixed; inset:0; background:rgba(15,23,42,.4);
  display:flex; align-items:center; justify-content:center; z-index:50; padding:24px;
}
.modal{
  background:var(--surface); border-radius:var(--radius-lg); padding:24px;
  width:min(520px,100%); box-shadow:var(--shadow); max-height:85vh; overflow:auto;
}
.modal-actions{display:flex; gap:8px; justify-content:flex-end; margin-top:20px}
```

## Пустое состояние

Каждый список умеет быть пустым, и на демонстрации это обязательно случится —
кто-нибудь введёт в поиск «ыыы».

```js
const emptyState = (title, hint, action = '') => `
  <div class="empty">
    <div class="empty-title">${title}</div>
    <div class="empty-hint">${hint}</div>
    ${action}
  </div>`;
```

Текст пустого состояния объясняет, что произошло и что делать дальше. «Нет
данных» — плохо; «Заявок не найдено. Измените условия поиска или создайте новую» —
хорошо. Формулировки прогоняй через `designer-toolkit:ux-writing`.

## Подсветка проблемы

То, ради чего заказчик смотрит прототип. Ноль и значения ниже порога должны
бросаться в глаза прямо в таблице.

```js
const THRESHOLD = 5;
function stockCell(n){
  if (n === 0)         return `<td class="num stock-zero">0</td>`;
  if (n < THRESHOLD)   return `<td class="num stock-low">${n}</td>`;
  return `<td class="num">${n}</td>`;
}
```

```css
.stock-zero{color:var(--danger); font-weight:600}
.stock-low{color:var(--warning); font-weight:600}
```

Порог выноси в константу и упомяни в разговоре: «подсвечиваю всё, что меньше
пяти — так правильно?» Это тот вопрос, на который у заказчика точно есть мнение,
и обсуждение сразу уходит в предметную область.
