### Трекинг воронки бронирования билетов

---

## Шаг 1 — Выбор места / ценовой категории

**Страница:** https://widget.bilet.livearena.ru/show/17

### 1.1 Клик по фильтру ценовой категории

В DOM видны кнопки с классом `.button-price`. Каждая кнопка соответствует одной ценовой категории (10 000 ₽, 12 000 ₽ и т.д.).

- **Цель в Метрике:** `SEAT_CATEGORY_CLICK`
- **Куда добавить:** обработчик `@click` на компоненте кнопки ценовой категории

```javascript
// Vue-компонент кнопки ценовой категории
// Предположительно: components/TicketGroupButton.vue или аналог

function onCategoryClick(category) {
  // Ваша существующая логика фильтрации:
  filterByCategory(category);

  // Добавить вызов Метрики:
  ym(106985860, 'reachGoal', 'SEAT_CATEGORY_CLICK', {
    show_id: 17,
    category_price: category.price,   // например: 10000
    category_name: category.name,     // например: "Партер"
    category_color: category.color,   // цвет из вашей модели данных
  });
}
```

---

### 1.2 Выбор конкретного места на схеме зала (Canvas)

Схема зала рендерится через `<canvas>`.
Событие выбора места генерируется внутри Vue-компонента, управляющего Canvas.

- **Цель в Метрике:** `SEAT_SELECTED`
- **Куда добавить:** в обработчик события выбора места (click на Canvas или внутреннее событие компонента)

```javascript
// В компоненте схемы зала — там, где обрабатывается клик по месту
// Предположительно: composables/useHallMap.js или SeatMapCanvas.vue

function onSeatClick(seat) {
  // Ваша существующая логика выбора:
  selectSeat(seat);

  // Добавить вызов Метрики:
  ym(106985860, 'reachGoal', 'SEAT_SELECTED', {
    show_id: 17,
    seat_id: seat.id,
    sector: seat.sector,    // сектор/секция
    row: seat.row,          // ряд
    number: seat.number,    // место
    price: seat.price,      // стоимость
  });
}
```

---

### 1.3 Добавление в корзину

В DOM найдена кнопка с классом `.cart-action-button` и текстом «Добавить в корзину». Это главная точка перехода к бронированию.

- **Цель в Метрике:** `ADD_TO_CART`
- **Также нужно:** событие `ecommerce.add` (для отчётов по электронной торговле)

```javascript
// В компоненте корзины — CartSider.vue или аналог
// Там, где обрабатывается кнопка "Добавить в корзину"

function onAddToCart(selectedSeats) {
  // Ваша существующая логика:
  addSeatsToCart(selectedSeats);

  // reachGoal:
  ym(106985860, 'reachGoal', 'ADD_TO_CART', {
    show_id: 17,
    tickets_count: selectedSeats.length,
    total_price: selectedSeats.reduce((sum, s) => sum + s.price, 0),
  });

  // ecommerce (для отчёта "Электронная торговля"):
  window.dataLayer = window.dataLayer || [];
  window.dataLayer.push({
    ecommerce: {
      currencyCode: 'RUB',
      add: {
        products: selectedSeats.map(seat => ({
          id: String(seat.id),
          name: `Билет — ${seat.sector}, ряд ${seat.row}, место ${seat.number}`,
          price: seat.price,
          quantity: 1,
          category: seat.sector,
        }))
      }
    }
  });
}
```

---

## Шаг 2 — Бронирование (форма оплаты)

- **Цель в Метрике:** `BEGIN_CHECKOUT`

```javascript
// Вызывать при открытии страницы/шага оформления

function onCheckoutOpen(cart) {
  ym(106985860, 'reachGoal', 'BEGIN_CHECKOUT', {
    show_id: 17,
    tickets_count: cart.items.length,
    total_price: cart.totalPrice,
  });

  // ecommerce checkout step 1:
  window.dataLayer.push({
    ecommerce: {
      currencyCode: 'RUB',
      checkout: {
        actionField: { step: 1 },
        products: cart.items.map(seat => ({
          id: String(seat.id),
          name: `Билет — ${seat.sector}, ряд ${seat.row}, место ${seat.number}`,
          price: seat.price,
          quantity: 1,
        }))
      }
    }
  });
}
```

---

### 2.2 Заполнение контактных данных

- **Цель в Метрике:** `CHECKOUT_CONTACT_FILLED`

```javascript
// Вызывать после успешной валидации формы с email/телефоном
// (перед переходом к оплате)

function onContactFormFilled() {
  ym(106985860, 'reachGoal', 'CHECKOUT_CONTACT_FILLED', {
    show_id: 17,
  });
}
```

---

### 2.3 Клик «Оплатить» / инициация платежа

- **Цель в Метрике:** `PAYMENT_INITIATED`


```javascript
// Вызывать непосредственно перед открытием CloudPayments виджета

function onPaymentStart(cart) {
  ym(106985860, 'reachGoal', 'PAYMENT_INITIATED', {
    show_id: 17,
    total_price: cart.totalPrice,
    tickets_count: cart.items.length,
  });
}
```

---

## Шаг 3 — Подтверждение заказа (успешная покупка)

Это самое важное событие — конверсия. Вызывается после получения подтверждения от платёжной системы.

### 3.1 Успешная покупка

- **Цель в Метрике:** `PURCHASE`
- **Также обязательно:** событие `ecommerce.purchase`

```javascript
// Вызывать ПОСЛЕ получения успешного ответа от платёжной системы
// (в callback CloudPayments или после редиректа на страницу успеха)

function onPurchaseSuccess(order) {
  // reachGoal — обязательно:
  ym(106985860, 'reachGoal', 'PURCHASE', {
    show_id: 17,
    order_id: order.id,              // ID заказа из вашей системы
    revenue: order.totalPrice,       // итоговая сумма
    tickets_count: order.items.length,
  });

  // ecommerce purchase — для отчёта "Электронная торговля":
  window.dataLayer = window.dataLayer || [];
  window.dataLayer.push({
    ecommerce: {
      currencyCode: 'RUB',
      purchase: {
        actionField: {
          id: String(order.id),      // обязательно!
          revenue: order.totalPrice,
        },
        products: order.items.map(seat => ({
          id: String(seat.id),
          name: `Билет — ${seat.sector}, ряд ${seat.row}, место ${seat.number}`,
          price: seat.price,
          quantity: 1,
          category: seat.sector,
        }))
      }
    }
  });
}
```
