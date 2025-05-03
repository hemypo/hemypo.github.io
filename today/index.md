---
layout: single
title: "Рацион на сегодня"
permalink: /today/
---

<div id="today-content">Загрузка...</div>

<script>
(async function(){
  // 1. Вычисляем сегодняшнюю дату
  const now = new Date();
  const dow = now.getDay(); // 0=Вс … 6=Сб
  const ruDays = ['Воскресенье','Понедельник','Вторник','Среда','Четверг','Пятница','Суббота'];
  const dayName = ruDays[dow];
  const weekNum = Math.min(4, Math.ceil(now.getDate()/7)); // 1–4

  const parser = new DOMParser();
  const container = document.getElementById('today-content');
  try {
    // 2. Меню: fetch /menu/, найти <h2>Неделя X</h2> → за ним <h3>dayName</h3> → за ним <ul>
    let res = await fetch('/menu/');
    let doc = parser.parseFromString(await res.text(), 'text/html');
    let weekHeader = [...doc.querySelectorAll('h2')].find(h=>h.textContent.trim()===`Неделя ${weekNum}`);
    let mealsList;
    if (weekHeader) {
      let el = weekHeader.nextElementSibling;
      while(el){
        if (el.tagName==='H3' && el.textContent.trim()===dayName) {
          mealsList = el.nextElementSibling; break;
        }
        el = el.nextElementSibling;
      }
    }

    // 3. Тренировка: fetch /workouts/, найти <h4>Основной блок</h4> → выбрать 'День A' или 'День B'
    res = await fetch('/workouts/');
    doc = parser.parseFromString(await res.text(), 'text/html');
    let mainBlock = [...doc.querySelectorAll('h4')].find(h=>h.textContent.includes('Основной блок'));
    let type = [1,3,5].includes(dow)? 'A':'B';
    let workoutList;
    if (mainBlock) {
      let el = mainBlock.nextElementSibling;
      while(el){
        if (el.tagName==='P' && el.textContent.includes(`День ${type}`)) {
          workoutList = el.nextElementSibling; break;
        }
        el = el.nextElementSibling;
      }
    }

    // 4. Покупки: fetch /shopping/, найти <h2>Неделя X</h2> → за ним <ul>
    res = await fetch('/shopping/');
    doc = parser.parseFromString(await res.text(), 'text/html');
    weekHeader = [...doc.querySelectorAll('h2')].find(h=>h.textContent.trim()===`Неделя ${weekNum}`);
    let shopList;
    if (weekHeader) {
      shopList = weekHeader.nextElementSibling;
    }

    // 5. Рендер
    let html = `<h2>Рацион на ${dayName}, неделя ${weekNum}</h2>`;
    if (mealsList) html += mealsList.outerHTML;
    html += `<h2>Тренировка (День ${type})</h2>`;
    if (workoutList) html += workoutList.outerHTML;
    html += `<h2>Список покупок (неделя ${weekNum})</h2>`;
    if (shopList) html += shopList.outerHTML;

    container.innerHTML = html;
  } catch(e){
    console.error(e);
    container.textContent = 'Ошибка загрузки данных.';
  }
})();
</script>
