# Этап 4. Каркас

Готовые оболочки прототипа. Общая часть — токены, фокус, тост — одна для всех.
Дальше выбирается архетип под тип продукта.

Содержание:

- [Выбор архетипа](#выбор-архетипа)
- [Устройство файла](#устройство-файла)
- [Общая часть: токены и база](#общая-часть-токены-и-база)
- [Архетип A: рабочая система с боковым меню](#архетип-a-рабочая-система-с-боковым-меню)
- [Архетип B: мобильное приложение](#архетип-b-мобильное-приложение)
- [Архетип C: публичная страница](#архетип-c-публичная-страница)
- [Навигация](#навигация)
- [Роли](#роли)
- [Тост](#тост)

## Выбор архетипа

| Архетип | Когда | Оболочка |
|---|---|---|
| **A. Рабочая система** | CRM, админка, панель оператора, ЛК сотрудника, дашборд, кабинет клиента с разделами | Боковое меню, шапка, широкое рабочее поле |
| **B. Мобильное приложение** | Приложение для клиента или курьера, всё, что показывают «как на телефоне» | Рамка телефона, нижний таб-бар, экран поверх экрана |
| **C. Публичная страница** | Лендинг, витрина, страница услуги, промо новой фичи | Обычный поток блоков, шапка со скроллом, без меню разделов |

Архетип определяется из карты экранов и почти никогда не обсуждается с человеком
отдельно: если в карте роли и разделы — это A, если «экран заказа» и «экран
профиля» на телефоне — B, если «первый экран, преимущества, цены» — C.

Смешанные случаи бывают: у публичного сайта есть личный кабинет. Тогда собирай
две оболочки в одном файле и переключай их так же, как экраны.

## Устройство файла

Один `index.html`, четыре секции сверху вниз:

1. `<style>` — токены в `:root`, базовые стили, компоненты.
2. Разметка оболочки.
3. Экраны — по одному `<section class="screen">`, активен ровно один.
4. `<script>` — данные и константы, затем навигация и роли, затем функции
   отрисовки по одной на экран.

Порядок не случайный: когда файл дорастёт до двух тысяч строк, ориентироваться
в нём можно будет только по такой структуре.

## Общая часть: токены и база

Всё оформление — через переменные. Значения подставляются из подбора
`ui-ux-pro-max` (см. `design-system.md`); ниже — нейтральный набор по умолчанию.

```html
<style>
:root{
  --bg:#F5F7FA; --surface:#FFFFFF; --surface-2:#F1F5F9;
  --border:#E2E8F0; --border-strong:#CBD5E1;
  --text:#0F172A; --text-muted:#475569; --text-faint:#64748B;
  --accent:#2563EB; --accent-dark:#1D4ED8; --accent-bg:#EFF6FF;
  --on-accent:#FFFFFF; --cta:var(--accent); --ring:var(--accent);
  --warning:#F97316; --warning-bg:#FFF7ED; --warning-border:#FED7AA;
  --success:#16A34A; --success-bg:#F0FDF4; --success-border:#BBF7D0;
  --danger:#DC2626;  --danger-bg:#FEF2F2;  --danger-border:#FECACA;
  --radius-sm:8px; --radius:12px; --radius-lg:16px;
  --shadow-sm:0 1px 2px rgba(15,23,42,.06);
  --shadow:0 4px 16px rgba(15,23,42,.08);
  --dur:180ms;
  --font-body: ui-sans-serif,-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,Arial,sans-serif;
  --font-head: var(--font-body);
}
@media (prefers-reduced-motion: reduce){
  *{ animation-duration:.001ms !important; transition-duration:.001ms !important; }
}
*{box-sizing:border-box}
html,body{margin:0;padding:0}
body{
  background:var(--bg); color:var(--text); font-family:var(--font-body);
  font-size:15px; line-height:1.5; -webkit-font-smoothing:antialiased;
}
h1,h2,h3,h4{margin:0; font-family:var(--font-head); font-weight:600;
            letter-spacing:-.01em; text-wrap:balance}
p{margin:0}
button,input,select,textarea{font-family:inherit; font-size:inherit}
svg{display:block; flex-shrink:0}

/* Видимый фокус — иначе прототип нельзя пройти с клавиатуры */
button:focus-visible,a:focus-visible,input:focus-visible,
select:focus-visible,[tabindex]:focus-visible{
  outline:2px solid var(--ring); outline-offset:2px; border-radius:6px;
}

.screen{display:none}
.screen.active{display:block}

.card{
  background:var(--surface); border:1px solid var(--border);
  border-radius:var(--radius); box-shadow:var(--shadow-sm); padding:20px;
}
.btn{
  display:inline-flex; align-items:center; justify-content:center; gap:8px;
  padding:9px 16px; min-height:44px; border-radius:var(--radius-sm);
  border:1px solid var(--border-strong); background:var(--surface); color:var(--text);
  cursor:pointer; transition:background var(--dur), border-color var(--dur);
}
.btn:hover{background:var(--surface-2)}
.btn-primary{background:var(--accent); border-color:var(--accent); color:var(--on-accent)}
.btn-primary:hover{background:var(--accent-dark)}
.btn-cta{background:var(--cta); border-color:var(--cta); color:var(--on-accent)}
</style>
```

`min-height: 44px` на кнопках — не эстетика, а размер, в который попадает палец.
На мобильном архетипе это критично, на десктопном не мешает.

## Архетип A: рабочая система с боковым меню

```html
<div class="app">
  <aside class="sidebar">
    <div class="brand">Название системы</div>

    <nav id="nav">
      <!-- data-roles перечисляет роли, которым виден пункт -->
      <button class="nav-item" data-target="orders"  data-roles="operator manager"
              onclick="nav('orders', this)">Заявки</button>
      <button class="nav-item" data-target="clients" data-roles="operator"
              onclick="nav('clients', this)">Клиенты</button>
      <button class="nav-item" data-target="reports" data-roles="manager"
              onclick="nav('reports', this)">Отчёты</button>
    </nav>

    <div class="sidebar-footer" id="sidebarFooter">
      <button class="footer-account-btn" onclick="toggleAccountMenu()">
        <span id="footerAvatar">ОП</span>
        <span><b id="footerName">Ольга П.</b><br><small id="footerRole">Оператор</small></span>
      </button>
      <div class="account-menu" id="accountMenu"><!-- пункты ролей --></div>
    </div>
  </aside>

  <div class="main">
    <header class="topbar">
      <h1 id="topbarTitle">Заявки</h1>
      <span class="chip" id="scopeChipLabel">Москва</span>
    </header>
    <main class="content">
      <section class="screen active" id="screen-orders"></section>
      <section class="screen" id="screen-clients"></section>
      <section class="screen" id="screen-reports"></section>
    </main>
  </div>
</div>
<div class="toast" id="toast"></div>
```

```css
.app{display:flex; min-height:100vh}
.sidebar{
  width:248px; flex-shrink:0; background:var(--surface);
  border-right:1px solid var(--border); display:flex; flex-direction:column;
  padding:20px 14px; position:sticky; top:0; height:100vh;
}
.sidebar-footer{margin-top:auto; position:relative}
.main{flex:1; min-width:0; display:flex; flex-direction:column}
.topbar{
  height:60px; display:flex; align-items:center; gap:12px; padding:0 24px;
  background:var(--surface); border-bottom:1px solid var(--border);
  position:sticky; top:0; z-index:10;
}
.content{padding:24px; flex:1}
.nav-item{
  display:flex; align-items:center; gap:10px; padding:9px 12px; width:100%;
  border:0; background:none; text-align:left; border-radius:var(--radius-sm);
  color:var(--text-muted); cursor:pointer;
  transition:background var(--dur), color var(--dur);
}
.nav-item:hover{background:var(--surface-2); color:var(--text)}
.nav-item[aria-current="page"]{background:var(--accent-bg); color:var(--accent-dark); font-weight:500}
```

Переключатель ролей — внизу бокового меню, под именем пользователя, там же, где
меню аккаунта в настоящих системах. Заказчик находит его сам.

## Архетип B: мобильное приложение

Прототип показывают на ноутбуке, поэтому телефон рисуется рамкой по центру. Так
сразу понятно, что это мобильный интерфейс, и никто не спрашивает, почему всё
такое узкое.

```html
<div class="phone">
  <div class="phone-screen">
    <header class="app-bar">
      <button class="back-btn" id="backBtn" onclick="goBack()" hidden>←</button>
      <h1 id="appBarTitle">Заказы</h1>
    </header>

    <main class="app-body">
      <section class="screen active" id="screen-orders"></section>
      <section class="screen" id="screen-order"></section>
      <section class="screen" id="screen-profile"></section>
    </main>

    <nav class="tabbar">
      <button class="tab" data-target="orders"  onclick="nav('orders', this)">Заказы</button>
      <button class="tab" data-target="catalog" onclick="nav('catalog', this)">Каталог</button>
      <button class="tab" data-target="profile" onclick="nav('profile', this)">Профиль</button>
    </nav>
  </div>
</div>
<div class="toast" id="toast"></div>
```

```css
body{background:var(--surface-2); display:flex; justify-content:center; padding:32px 16px}
.phone{
  width:390px; height:844px; background:var(--bg); border-radius:44px;
  border:10px solid #1E293B; overflow:hidden; box-shadow:var(--shadow);
  display:flex; flex-direction:column;
}
.phone-screen{display:flex; flex-direction:column; height:100%}
.app-bar{
  height:52px; flex-shrink:0; display:flex; align-items:center; gap:8px;
  padding:0 16px; background:var(--surface); border-bottom:1px solid var(--border);
}
.app-body{flex:1; overflow-y:auto; padding:16px; -webkit-overflow-scrolling:touch}
.tabbar{
  flex-shrink:0; display:flex; background:var(--surface);
  border-top:1px solid var(--border); padding-bottom:8px;
}
.tab{
  flex:1; min-height:52px; border:0; background:none; cursor:pointer;
  color:var(--text-faint); font-size:12px;
}
.tab[aria-current="page"]{color:var(--accent); font-weight:600}
```

Экраны второго уровня — карточка заказа, оформление — открываются поверх и
возвращаются кнопкой «назад» в шапке. Стек хранится массивом:

```js
let stack = [];
function openScreen(target){ stack.push(currentScreen); nav(target); updateBack(); }
function goBack(){ nav(stack.pop() || 'orders'); updateBack(); }
function updateBack(){ document.getElementById('backBtn').hidden = stack.length === 0; }
```

Аппаратной кнопки «назад» в браузере нет, поэтому своя обязательна на каждом
экране второго уровня — иначе прототип превращается в ловушку.

## Архетип C: публичная страница

Ни бокового меню, ни экранов: обычный поток блоков и якорная навигация. Структуру
блоков — что за чем идёт и где призыв к действию — бери из подбора
`ui-ux-pro-max --domain landing`, а не из головы.

```html
<header class="site-head">
  <div class="site-head-inner">
    <span class="brand">Название</span>
    <nav>
      <a href="#features">Возможности</a>
      <a href="#pricing">Тарифы</a>
      <a class="btn btn-primary" href="#cta">Оставить заявку</a>
    </nav>
  </div>
</header>

<main>
  <section class="hero">…</section>
  <section id="features" class="section">…</section>
  <section id="pricing" class="section">…</section>
  <section id="cta" class="section">…</section>
</main>
<div class="toast" id="toast"></div>
```

```css
.site-head{
  position:sticky; top:0; z-index:20; background:color-mix(in srgb, var(--surface) 88%, transparent);
  backdrop-filter:blur(8px); border-bottom:1px solid var(--border);
}
.site-head-inner{
  max-width:1140px; margin:0 auto; padding:14px 24px;
  display:flex; align-items:center; justify-content:space-between; gap:24px;
}
.section{max-width:1140px; margin:0 auto; padding:80px 24px}
.hero{max-width:1140px; margin:0 auto; padding:96px 24px 72px}
html{scroll-behavior:smooth}
@media (prefers-reduced-motion: reduce){ html{scroll-behavior:auto} }
```

Формы на публичной странице никуда не отправляются — по кнопке показывается
экран благодарности или тост. Это надо проговорить человеку заранее, иначе на
встрече возникнет неловкая пауза.

Здесь адаптив обязателен: публичную страницу обязательно откроют с телефона прямо
на встрече. Одной точки перелома на 768px обычно достаточно.

## Навигация

Общая для A и B. Экраны не перезагружают страницу — меняется активная секция.
Заголовок и отрисовка привязаны к тому же переходу, чтобы не рассинхронизировались.

```js
const TITLES    = { orders:'Заявки', clients:'Клиенты', reports:'Отчёты' };
const RENDERERS = { orders:renderOrders, clients:renderClients, reports:renderReports };
let currentScreen = 'orders';

function nav(target, el){
  document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
  document.getElementById('screen-' + target).classList.add('active');

  document.querySelectorAll('[data-target]').forEach(n => n.removeAttribute('aria-current'));
  const tab = el || document.querySelector('[data-target="' + target + '"]');
  if (tab) tab.setAttribute('aria-current','page');

  const title = document.getElementById('topbarTitle') || document.getElementById('appBarTitle');
  if (title && TITLES[target]) title.textContent = TITLES[target];
  if (RENDERERS[target]) RENDERERS[target]();

  currentScreen = target;
  window.scrollTo({top:0});
}
```

Отрисовка вызывается при каждом переходе, а не один раз при загрузке: данные
меняются на других экранах, и пересборка при входе избавляет от целого класса
багов с устаревшим содержимым.

## Роли

Нужны, когда в карте экранов больше одной роли — обычно в архетипе A, иногда в B
(клиент и курьер в одном приложении).

```js
const ACCOUNTS = {
  operator: {initials:'ОП', name:'Ольга П.',  role:'Оператор',
             scopeLabel:'Москва',    defaultScreen:'orders'},
  manager:  {initials:'ДК', name:'Дмитрий К.', role:'Руководитель',
             scopeLabel:'Все города', defaultScreen:'reports'},
};
let currentAccount = 'operator';

function switchAccount(key){
  currentAccount = key;
  const acc = ACCOUNTS[key];

  document.getElementById('footerAvatar').textContent = acc.initials;
  document.getElementById('footerName').textContent   = acc.name;
  document.getElementById('footerRole').textContent   = acc.role;
  document.getElementById('scopeChipLabel').textContent = acc.scopeLabel;

  // Пункты меню, недоступные роли, прячутся целиком
  document.querySelectorAll('[data-roles]').forEach(el => {
    el.style.display = el.dataset.roles.split(' ').includes(key) ? 'flex' : 'none';
  });

  document.getElementById('accountMenu').classList.remove('open');
  nav(acc.defaultScreen);
}
```

Права доступа в прототипе не моделируются. Достаточно, чтобы при смене роли
менялись набор пунктов меню и стартовый экран — этого хватает, чтобы заказчик
увидел разницу.

## Тост

Единственный способ закрыть все «пока не сделанные» кнопки, не оставляя мёртвых
кликов. Нужен во всех трёх архетипах.

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

## Запуск и проверка

```js
switchAccount('operator');   // архетипы A и B с ролями
nav('orders');               // если роль одна
```

Открой файл в браузере и убедись: переключаются экраны, меняется заголовок,
работают роли и меню под роль, виден фокус при табуляции, в мобильном архетипе
возвращает кнопка «назад». Покажи скриншот — на пустых экранах ещё можно
безболезненно передоговориться о структуре.
