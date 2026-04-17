# Яндекс.Метрика — JS-события для рекламы и трекинга

Документация: https://yandex.ru/support/metrica/

---

## Инициализация счётчика

Перед вызовом любых событий счётчик должен быть инициализирован на странице.

```html
<!-- Стандартный тег счётчика (вставляется в <head>) -->
<script>
(function(m,e,t,r,i,k,a){m[i]=m[i]||function(){(m[i].a=m[i].a||[]).push(arguments)};
m[i].l=1*new Date();
for (var j = 0; j < document.scripts.length; j++) {if (document.scripts[j].src === r) { return; }}
k=e.createElement(t),a=e.getElementsByTagName(t)[0],k.async=1,k.src=r,a.parentNode.insertBefore(k,a)})
(window, document, "script", "https://mc.yandex.ru/metrika/tag.js", "ym");

ym(XXXXXXXX, "init", {
  clickmap: true,
  trackLinks: true,
  accurateTrackBounce: true,
  webvisor: true,       // Вебвизор
  ecommerce: "dataLayer" // для e-commerce
});
</script>
```

> Замените `XXXXXXXX` на номер вашего счётчика.

Документация по инициализации: https://yandex.ru/support/metrica/code/counter-initialize.html

---

## 1. Просмотр страницы (hit)

Используется для SPA (React, Vue, Next.js) — когда URL меняется без перезагрузки страницы.

```javascript
ym(XXXXXXXX, 'hit', window.location.href, {
  title: document.title,
  referer: document.referrer,
  params: {
    page_type: 'product' // произвольные параметры
  }
});
```

Документация: https://yandex.ru/support/metrica/objects/hit.html

---

## 2. Достижение цели (reachGoal)

Основной метод для трекинга любых пользовательских действий.  
Цель должна быть предварительно создана в интерфейсе Метрики: **Настройки → Цели → Добавить цель → JavaScript-событие**.

```javascript
// Базовый вызов
ym(XXXXXXXX, 'reachGoal', 'GOAL_NAME');

// С параметрами визита
ym(XXXXXXXX, 'reachGoal', 'GOAL_NAME', {
  order_id: '12345',
  value: 2990
});

// С callback после отправки
ym(XXXXXXXX, 'reachGoal', 'GOAL_NAME', {}, function() {
  console.log('Цель отправлена');
});
```

Документация: https://yandex.ru/support/metrica/objects/reachgoal.html

---

## 3. Типовые события — примеры вызовов

### Клик по кнопке / CTA

```javascript
document.querySelector('#cta-button').addEventListener('click', () => {
  ym(XXXXXXXX, 'reachGoal', 'CLICK_CTA', {
    button_label: 'Оставить заявку',
    page: window.location.pathname
  });
});
```

---

### Отправка формы (лид)

```javascript
document.querySelector('#lead-form').addEventListener('submit', (e) => {
  ym(XXXXXXXX, 'reachGoal', 'FORM_SUBMIT', {
    form_id: 'lead_form_homepage'
  });
});
```

---

### Глубина скролла

```javascript
const tracked = {};

window.addEventListener('scroll', () => {
  const pct = Math.round(
    (window.scrollY / (document.body.scrollHeight - window.innerHeight)) * 100
  );
  [25, 50, 75, 90].forEach(depth => {
    if (pct >= depth && !tracked[depth]) {
      tracked[depth] = true;
      ym(XXXXXXXX, 'reachGoal', `SCROLL_${depth}`, {
        depth_percent: depth,
        page: window.location.pathname
      });
    }
  });
}, { passive: true });
```

---

### Поиск по сайту

```javascript
document.querySelector('#search-form').addEventListener('submit', () => {
  const query = document.querySelector('#search-input').value;
  ym(XXXXXXXX, 'reachGoal', 'SITE_SEARCH', {
    search_query: query
  });
});
```

---

### Начало оформления заказа

```javascript
ym(XXXXXXXX, 'reachGoal', 'BEGIN_CHECKOUT', {
  cart_value: 5980,
  items_count: 2
});
```

---

### Успешная покупка

```javascript
ym(XXXXXXXX, 'reachGoal', 'PURCHASE', {
  order_id: 'ORD-7891',
  revenue: 5980,
  currency: 'RUB'
});
```

---

### Открытие чата / звонок

```javascript
// Клик по номеру телефона
document.querySelectorAll('a[href^="tel:"]').forEach(el => {
  el.addEventListener('click', () => {
    ym(XXXXXXXX, 'reachGoal', 'PHONE_CLICK', {
      phone: el.getAttribute('href')
    });
  });
});

// Открытие онлайн-чата
document.querySelector('#open-chat').addEventListener('click', () => {
  ym(XXXXXXXX, 'reachGoal', 'CHAT_OPEN');
});
```

---

## 4. Параметры визита (params)

Позволяют передавать произвольные данные о визите — сегментируются в отчётах.

```javascript
// Передача параметров визита (без привязки к цели)
ym(XXXXXXXX, 'params', {
  user_type: 'registered',
  plan: 'pro',
  city: 'Moscow'
});

// Параметры пользователя (UserID — для cross-device трекинга)
ym(XXXXXXXX, 'setUserID', 'user_12345');
```

Документация: https://yandex.ru/support/metrica/objects/params-method.html  
UserID: https://yandex.ru/support/metrica/general/user-id.html

---

## 5. E-commerce трекинг

Требует `ecommerce: "dataLayer"` в инициализации счётчика.

### Просмотр товара

```javascript
window.dataLayer = window.dataLayer || [];
window.dataLayer.push({
  ecommerce: {
    currencyCode: 'RUB',
    detail: {
      products: [{
        id: 'SKU_123',
        name: 'Футболка белая',
        price: 2990,
        brand: 'BrandName',
        category: 'Одежда',
        variant: 'XL'
      }]
    }
  }
});
```

### Добавление в корзину

```javascript
window.dataLayer.push({
  ecommerce: {
    currencyCode: 'RUB',
    add: {
      products: [{
        id: 'SKU_123',
        name: 'Футболка белая',
        price: 2990,
        quantity: 1
      }]
    }
  }
});
```

### Покупка

```javascript
window.dataLayer.push({
  ecommerce: {
    currencyCode: 'RUB',
    purchase: {
      actionField: {
        id: 'ORD-7891',     // ID заказа
        revenue: 5980,      // сумма с доставкой
        shipping: 300,
        coupon: 'SALE10'
      },
      products: [{
        id: 'SKU_123',
        name: 'Футболка белая',
        price: 2990,
        quantity: 2,
        category: 'Одежда'
      }]
    }
  }
});
```

Документация по E-commerce: https://yandex.ru/support/metrica/data/e-commerce.html

---

## 6. Отслеживание внешних ссылок и файлов

Яндекс.Метрика автоматически отслеживает внешние ссылки при `trackLinks: true`.  
Для файлов (PDF, ZIP и др.) нужна ручная разметка:

```javascript
document.querySelectorAll('a[href$=".pdf"], a[href$=".zip"]').forEach(el => {
  el.addEventListener('click', () => {
    ym(XXXXXXXX, 'reachGoal', 'FILE_DOWNLOAD', {
      file: el.getAttribute('href')
    });
  });
});
```

Документация: https://yandex.ru/support/metrica/general/download-links.html

---

## 7. Вебвизор — дополнительные настройки

```javascript
ym(XXXXXXXX, 'init', {
  webvisor: true,
  // Не записывать поля с паролями и картами:
  webvisor_settings: {
    inputExclusions: ['input[type=password]', '#card-number']
  }
});
```

Документация: https://yandex.ru/support/metrica/general/webvisor.html

---

## 8. Отладка

```javascript
// Просмотр всех отправленных событий в консоли браузера
// Добавьте в URL параметр: ?_ym_debug=1
// Или вызовите в консоли:
window._ym_debug = 1;
```

Также можно использовать браузерное расширение **Яндекс.Метрика Debugger**:  
https://chromewebstore.google.com/detail/яндексметрика-debugger/haflbbkfmocnfklkfohgekhgkhikhkli

---

## Ссылки на документацию

| Раздел | Ссылка |
|---|---|
| Инициализация счётчика | https://yandex.ru/support/metrica/code/counter-initialize.html |
| reachGoal | https://yandex.ru/support/metrica/objects/reachgoal.html |
| hit (SPA) | https://yandex.ru/support/metrica/objects/hit.html |
| params | https://yandex.ru/support/metrica/objects/params-method.html |
| setUserID | https://yandex.ru/support/metrica/general/user-id.html |
| E-commerce | https://yandex.ru/support/metrica/data/e-commerce.html |
| Цели | https://yandex.ru/support/metrica/general/goals.html |
| Вебвизор | https://yandex.ru/support/metrica/general/webvisor.html |
| Отслеживание файлов | https://yandex.ru/support/metrica/general/download-links.html |
| Общая документация | https://yandex.ru/support/metrica/ |
