---
layout: single
title: "Рацион на сегодня"
permalink: /today/
---

<div id="today-content">Загрузка...</div>

<script>
(async function(){
  const now = new Date();
  const dow = now.getDay(); // 0=Вс … 6=Сб
  const ruDays = ['Воскресенье','Понедельник','Вторник','Среда','Четверг','Пятница','Суббота'];
  const dayKey = ruDays[dow];
  const weekNum = Math.min(4, Math.ceil(now.getDate()/7));

  // Словарь для ссылок
  const recipeMap = {
    'овсянка с яблоком и орехами': '/recipes/ovsyanka-s-yablokom-i-orehami/',
    'гречка с творогом и ягодами': '/recipes/grechka-s-tvorogom-i-yagodami/',
    'овсяноблин с ягодами': '/recipes/ovsyanoblin-s-yagodami/',
    'овсяноблин со шпинатом': '/recipes/ovsyanoblin-so-shpinatom/',
    'перловка с творогом и ягодами': '/recipes/perlovka-s-tvorogom-i-yagodami/',
    'пшеничная каша с чиа и ягодами': '/recipes/pshenichnaya-kasha-s-chia-i-yagodami/',
    'цельнозерновой тост с авокадо и яйцом': '/recipes/tost-s-avokado-i-yaitsom/',
    'тост с творогом и зеленью': '/recipes/tost-s-tvorogom-i-zelenyu/',
    'омлет из белков со шпинатом': '/recipes/omlet-iz-belykov-so-shpinatom/',
    'омлет с помидорами и зеленью': '/recipes/omlet-s-pomidorami-i-zelenyu/',
    'омлет с овощами': '/recipes/omlet-s-ovoshchami/',
    'смузи «банан–йогурт–шпинат»': '/recipes/smuzi-banan-jogurt-shpinat/',
    'смузи «груша–йогурт–шпинат»': '/recipes/smuzi-grusha-jogurt-shpinat/',
    'творожная запеканка с ягодами': '/recipes/zapekanka-tvorozhnaya-s-yagodami/',
    'греческий йогурт с ягодами': '/recipes/grecheskij-jogurt-s-yagodami/',
    'чиа-пудинг на йогурте': '/recipes/chia-puding-na-yogurte/',
    'творог с мёдом и ягодами': '/recipes/tvorog-s-mjodom-i-yagodami/',
    'фрукты + орехи': '/recipes/frukty-i-orehi/',
    'киноа с яйцом-пашот и овощами': '/recipes/kinoa-s-yaitsom-pashot-i-ovoshchami/',
    'гриль-куриная грудка с киноа и салатом': '/recipes/gril-kuriceskaya-grudka-s-kinoa-i-salatom/',
    'куриные фрикадельки с киноа и фасолью': '/recipes/kurinye-frikadelki-s-kinoa-i-fasolyu/',
    'фрикадельки из индейки': '/recipes/frikadelki-iz-indeyki/',
    'стир-фрай из курицы с овощами': '/recipes/stir-fry-iz-kuritsy-s-ovoshchami/',
    'стир-фрай из индейки с овощами': '/recipes/stir-fry-iz-indeyki-s-ovoshchami/',
    'говяжий гуляш с рисом и брокколи': '/recipes/govyazhj-gulyash-s-risom-i-brokkoli/',
    'говяжьи тефтели': '/recipes/govyazhji-tefteli/',
    'стейк говядины': '/recipes/stejk-govyadiny/',
    'филе рыбы на пару': '/recipes/file-ryby-na-paru/',
    'рыба в фольге': '/recipes/ryba-v-folge/',
    'запечённые овощи с нутом': '/recipes/zapechennye-ovoshchi-s-nutom/',
    'суп из цветной капусты': '/recipes/sup-iz-tsvetnoj-kapusty/',
    'суп-пюре кабачковый': '/recipes/sup-pjure-kabachkovyj/',
    'суп-пюре шпинатный': '/recipes/sup-pjure-shpinatnyj/',
    'суп с нутом': '/recipes/sup-s-nutom/',
    'суп с горошком': '/recipes/sup-s-goroshkom/',
    'овощной суп-пюре': '/recipes/ovoshhnoj-sup-pjure/',
    'салат с фасолью': '/recipes/salat-s-fasolyu/',
    'салат с лососем': '/recipes/salat-s-lososem/',
    'салат «кобб» без авокадо': '/recipes/salat-kobb-bez-avokado/',
    'салат «нисуаз» без яиц': '/recipes/salat-nisaz-bez-yaits/',
    'салат с тунцом и перцем': '/recipes/salat-s-tuntsom-i-pertsem/'
  };

  const parser = new DOMParser();
  const container = document.getElementById('today-content');

  try {
    // меню
    let r = await fetch('/menu/');
    let d = parser.parseFromString(await r.text(),'text/html');
    let h2 = [...d.querySelectorAll('h2')].find(x=>x.textContent.trim()==='Неделя '+weekNum);
    let mealsEl, el=h2.nextElementSibling;
    while(el){ if(el.tagName==='H3' && el.textContent.trim()===dayKey){ mealsEl=el.nextElementSibling; break;} el=el.nextElementSibling; }
    const meals = mealsEl?Array.from(mealsEl.querySelectorAll('li')).map(li=>li.textContent.split(':')[1].trim()):[];

    // тренировки
    r = await fetch('/workouts/');
    d = parser.parseFromString(await r.text(),'text/html');
    const mb = [...d.querySelectorAll('h4')].find(x=>x.textContent.includes('Основной блок'));
    const type = [1,3,5].includes(dow)?'A':'B';
    let woEl, e=mb.nextElementSibling;
    while(e){ if(e.tagName==='P'&&e.textContent.includes('День '+type)){ woEl=e.nextElementSibling; break;} e=e.nextElementSibling;}
    
    // покупки
    r = await fetch('/shopping/');
    d = parser.parseFromString(await r.text(),'text/html');
    h2=[...d.querySelectorAll('h2')].find(x=>x.textContent.trim()==='Неделя '+weekNum);
    const shopEl = h2.nextElementSibling;
    const shopping = shopEl?Array.from(shopEl.querySelectorAll('li')).map(li=>li.textContent.trim()):[];

    // рендер
    let html = `<h2>Рацион на ${dayKey}, неделя ${weekNum}</h2>`;
    ['Завтрак','Перекус','Обед','Перекус','Ужин'].forEach((lbl,i)=>{
      let dish=meals[i]||'', key=dish.toLowerCase(), href=recipeMap[key]||'';
      html+=href?`<p><strong>${lbl}:</strong> <a href="${href}">${dish}</a></p>`:
               `<p><strong>${lbl}:</strong> ${dish}</p>`;
    });
    html+=`<h2>Утренняя тренировка (День ${type})</h2>`+(woEl?woEl.outerHTML:'');
    html+=`<h2>Список покупок (неделя ${weekNum})</h2><ul>`+shopping.map(i=>`<li>${i}</li>`).join('')+'</ul>';

    container.innerHTML=html;
  }
  catch(e){
    console.error(e);
    container.textContent='Ошибка загрузки данных.';
  }
})();
</script>
