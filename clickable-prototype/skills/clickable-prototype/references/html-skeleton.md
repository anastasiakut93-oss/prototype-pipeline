# Этап 4. Каркас

Готовая оболочка прототипа. Бери её целиком и наполняй — она уже решает
переключение экранов, роли, состояния фокуса и уважение к системной настройке
«меньше движения».

## Устройство файла

Один `index.html`, четыре секции сверху вниз:

1. `<style>` — токены в `:root`, затем базовые стили, затем компоненты.
2. Разметка оболочки — сайдбар и шапка.
3. Экраны — по одному `<section class="screen">` на экран, активен ровно один.
4. `<script>` — сначала данные и константы, потом навигация и роли, потом
   функции отрисовки по одной на экран.

Порядок не случайный: когда файл дорастёт до двух тысяч строк, искать в нём
что-либо можно будет только по такой структуре.

## Токены

Всё оформление — через переменные. Один раз задал палитру, дальше меняешь
акцентный цвет в одном месте и весь прототип перекрашивается. Набор ниже —
нейтральный; под бренд клиента подбирай через `ui-design:color-system`.

```html
<style>
:root{
  --bg:#F5F7FA; --surface:#FFFFFF; --surface-2:#F1F5F9;
  --border:#E2E8F0; --border-strong:#CBD5E1;
  --text:#0F172A; --text-muted:#475569; --text-faint:#64748B;
  --accent:#2563EB; --accent-dark:#1D4ED8; --accent-bg:#EFF6FF;
  --warning:#F97316; --warning-bg:#FFF7ED; --warning-border:#FED7AA;
  --success:#16A34A; --success-bg:#F0FDF4; --success-border:#BBF7D0;
  --danger:#DC2626;  --danger-bg:#FEF2F2;  --danger-border:#FECACA;
  --radius-sm:8px; --radius:12px; --radius-lg:16px;
  --shadow-sm:0 1px 2px rgba(15,23,42,.06);
  --shadow:0 4px 16px rgba(15,23,42,.08);
  --dur:180ms; --sidebar-w:248px;
}
@media (prefers-reduced-motion: reduce){
  *{ animation-duration:.001ms !important; transition-duration:.001ms !important; }
}
*{box-sizing:border-box}
html,body{margin:0;padding:0}
body{
  background:var(--bg); color:var(--text);
  font-family:ui-sans-serif,-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,Arial,sans-serif;
  font-size:15px; line-height:1.5; -webkit-font-smoothing:antialiased;
}
h1,h2,h3,h4{margin:0; font-weight:600; letter-spacing:-.01em; text-wrap:balance}
p{margin:0}
button,input,select,textarea{font-family:inherit; font-size:inherit}
svg{display:block; flex-shrink:0}

/* Видимый фокус — иначе прототип нельзя пройти с клавиатуры */
button:focus-visible,a:focus-visible,input:focus-visible,
select:focus-visible,[tabindex]:focus-visible{
  outline:2px solid var(--accent); outline-offset:2px; border-radius:6px;
}

.app{display:flex; min-height:100vh}
.sidebar{
  width:var(--sidebar-w); flex-shrink:0; background:var(--surface);
  border-right:1px solid var(--border); display:flex; flex-direction:column;
  padding:20px 14px; position:sticky; top:0; height:100vh;
}
.main{flex:1; min-width:0; display:flex; flex-direction:column}
.topbar{
  height:60px; display:flex; align-items:center; gap:12px; padding:0 24px;
  background:var(--surface); border-bottom:1px solid var(--border);
  position:sticky; top:0; z-index:10;
}
.content{padding:24px; flex:1}

.nav-item{
  display:flex; align-items:center; gap:10px; padding:9px 12px;
  border-radius:var(--radius-sm); color:var(--text-muted); cursor:pointer;
  transition:background var(--dur), color var(--dur); border:0; background:none;
  width:100%; text-align:left;
}
.nav-item:hover{background:var(--surface-2); color:var(--text)}
.nav-item[aria-current="page"]{background:var(--accent-bg); color:var(--accent-dark); font-weight:500}

.screen{display:none}
.screen.active{display:block}

.card{
  background:var(--surface); border:1px solid var(--border);
  border-radius:var(--radius); box-shadow:var(--shadow-sm); padding:20px;
}
.btn{
  display:inline-flex; align-items:center; gap:8px; padding:9px 16px;
  border-radius:var(--radius-sm); border:1px solid var(--border-strong);
  background:var(--surface); color:var(--text); cursor:pointer;
  transition:background var(--dur), border-color var(--dur);
}
.btn:hover{background:var(--surface-2)}
.btn-primary{background:var(--accent); border-color:var(--accent); color:#fff}
.btn-primary:hover{background:var(--accent-dark)}
</style>
```

## Разметка оболочки

```html
<div class="app">
  <aside class="sidebar">
    <div class="brand">Название системы</div>

    <nav id="nav">
      <!-- data-roles перечисляет роли, которым виден пункт -->
      <button class="nav-item" data-target="rentals"   data-roles="point_admin line_staff"
              onclick="nav('rentals', this)">Аренда</button>
      <button class="nav-item" data-target="bikes"     data-roles="point_admin"
              onclick="nav('bikes', this)">Велосипеды</button>
      <button class="nav-item" data-target="stock"     data-roles="office_manager"
              onclick="nav('stock', this)">Остатки по сети</button>
    </nav>

    <div class="sidebar-footer" id="sidebarFooter">
      <button class="footer-account-btn" onclick="toggleAccountMenu()">
        <span id="footerAvatar">АС</span>
        <span><b id="footerName">Анастасия С.</b><br><small id="footerRole">Администратор точки</small></span>
      </button>
      <div class="account-menu" id="accountMenu"><!-- пункты ролей --></div>
    </div>
  </aside>

  <div class="main">
    <header class="topbar">
      <h1 id="topbarTitle">Аренда</h1>
      <span class="chip" id="pointChipLabel">Точка Академическая</span>
    </header>

    <main class="content">
      <section class="screen active" id="screen-rentals"></section>
      <section class="screen" id="screen-bikes"></section>
      <section class="screen" id="screen-stock"></section>
    </main>
  </div>
</div>

<div class="toast" id="toast"></div>
```

Переключатель ролей живёт внизу сайдбара, под именем пользователя — там, где в
настоящих системах меню аккаунта. Заказчик находит его сам, объяснять не нужно.

## Навигация

Экраны не перезагружают страницу — просто меняется активная секция. Заголовок в
шапке и отрисовка привязаны к тому же переходу, чтобы не рассинхронизировались.

```js
const TITLES = {
  rentals:'Аренда', bikes:'Велосипеды', stock:'Остатки по сети'
};
const RENDERERS = {
  rentals:renderRentals, bikes:renderBikes, stock:renderStock
};

function nav(target, el){
  document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
  document.getElementById('screen-' + target).classList.add('active');

  document.querySelectorAll('.nav-item').forEach(n => n.removeAttribute('aria-current'));
  if (el) el.setAttribute('aria-current','page');

  if (TITLES[target]) document.getElementById('topbarTitle').textContent = TITLES[target];
  if (RENDERERS[target]) RENDERERS[target]();

  window.scrollTo({top:0});
}
```

Отрисовка вызывается при каждом переходе, а не один раз при загрузке: данные
меняются на других экранах, и пересборка при входе избавляет от целого класса
багов с устаревшим содержимым.

## Роли

```js
const ACCOUNTS = {
  point_admin:    {initials:'АС', name:'Анастасия С.', role:'Администратор точки',
                   pointLabel:'Точка Академическая', defaultScreen:'rentals'},
  office_manager: {initials:'МК', name:'Марина К.',   role:'Менеджер по каналам',
                   pointLabel:'Все точки сети',      defaultScreen:'stock'},
};
let currentAccount = 'point_admin';

function switchAccount(key){
  currentAccount = key;
  const acc = ACCOUNTS[key];

  document.getElementById('footerAvatar').textContent = acc.initials;
  document.getElementById('footerName').textContent   = acc.name;
  document.getElementById('footerRole').textContent   = acc.role;
  document.getElementById('pointChipLabel').textContent = acc.pointLabel;

  // Пункты меню, недоступные роли, прячутся целиком
  document.querySelectorAll('.nav-item[data-roles]').forEach(el => {
    el.style.display = el.dataset.roles.split(' ').includes(key) ? 'flex' : 'none';
  });

  document.getElementById('accountMenu').classList.remove('open');
  const start = document.querySelector('.nav-item[data-target="' + acc.defaultScreen + '"]');
  nav(acc.defaultScreen, start);
}
```

## Тост

Единственный способ закрыть все «пока не сделанные» кнопки, не оставляя мёртвых
кликов. Три строки кода, а прототип перестаёт разваливаться под чужими пальцами.

```js
let toastTimer;
function toast(msg){
  const t = document.getElementById('toast');
  t.textContent = msg;
  t.classList.add('show');
  clearTimeout(toastTimer);
  toastTimer = setTimeout(() => t.classList.remove('show'), 2600);
}
```

```css
.toast{
  position:fixed; bottom:24px; left:50%; transform:translateX(-50%) translateY(12px);
  background:var(--text); color:#fff; padding:11px 18px; border-radius:var(--radius-sm);
  box-shadow:var(--shadow); opacity:0; pointer-events:none;
  transition:opacity var(--dur), transform var(--dur); z-index:100;
}
.toast.show{opacity:1; transform:translateX(-50%) translateY(0)}
```

## Запуск

```js
switchAccount('point_admin');   // расставит меню, заголовок и стартовый экран
```

## Проверка каркаса

Открой файл в браузере и убедись: переключаются экраны, меняется заголовок,
переключаются роли, меню под роль перестраивается, фокус виден при табуляции.
Покажи скриншот человеку — на пустых экранах ещё можно безболезненно
передоговориться о структуре.
