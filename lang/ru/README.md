# Простой список дел с использованием localstorage

Каждый наверняка сталкивался с задачей создания списков дел, как в рамках обучения, так и в рамках рабочей задачи. В этом посте я хочу рассказать о том, как я решил эту задачу с использованием localstorage.

Итак, задачи, которые я решил:

1. Хранение списка задач в браузере
2. Создание, редактирование и удаление задач

## Хранение

Список задач я буду хранить в `localstorage`, поэтому для начала я создам простой интерфейс для работы с задачами и `localstorage`.

Хранить задачи мы будем как массив объектов.

В конструкторе интерфейса для работы с `localstorage` я буду получать данные из `localstorage` и записывать в `this.tasks`.

```javascript
export default class LocalStorage {
  constructor() {
    // Если элемента по ключу tasks нет, то JSON.parse вернёт null, оэтому мы используем конструкцию `|| []`
    this.tasks = JSON.parse(localStorage.getItem('tasks')) || [];
  }
}
```

Теперь нам нужны методы для добавления, обновления и удаления задач.

Методы очень простые. Например, вот метод для добавления задачи:

```javascript
export default class LocalStorage {
  //...

  create(data) {
    this.tasks.push(data);

    localStorage.setItem('tasks', JSON.stringify(this.tasks));
  }
}
```

Здесь мы актуализируем наш список задач в `this.tasks` и перезаписываем его в `localstorage`.

Чтобы обновить задачу или удалить ее, нам нужен некий идентификатор для поиска задачи в `this.tasks` и обновления списка в `localstorage`. Создадим геттер `get token` для получения сгенерированного токена, обновим текущий метод `add`, а также метод для поиска индекса задачи по токену в `this.task`, нужный нам для обновления и удаления задач.

```javascript
export default class LocalStorage {
  //...

  create(data) {
    data.token = this.token;

    //...
  }

  getIndexByToken(token) {
    for (let i = 0; i < this.tasks.length; i++) {
      if (this.tasks[i].token === token) {
        return i;
      }
    }
      
    return -1;
  }

  get token() {
    return Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15);
  }
}
```

Теперь всё, что нам нужно для обновления данных задачи, это найти ее индекс в `this.tasks`, проверить, что он не равен `-1`, обновить объект по индексу и перезаписать данные в `localstorage`. Аналогично и с удалением: ищем, удаляем, обновляем. Опишем недостающие методы.

```javascript
export default class LocalStorage {
  //...

  update(data) {
    let index = this.getIndexByToken(data.token);

    if (index !== -1) {
      this.tasks[index] = data;

      localStorage.setItem('tasks', JSON.stringify(this.tasks));
    }
  }

  delete(data) {
    let index = this.getIndexByToken(data.token);

    if (index !== -1) {
      this.tasks.splice(index, 1);

      localStorage.setItem('tasks', JSON.stringify(this.tasks));
    }
  }

  //...
}
```

## Реализация

Перейдём к основной реализации.

Теперь нужно создать экземпляр класса, и получить список задач. С помощью созданного экземпляра класса будут производиться все основные манипуляции (добавление, обновление, удаление).

```javascript
import LocalStorage from './LocalStorage.js';

const storage = new LocalStorage();

const tasks = storage.tasks;
```

В html я создал шаблон для задачи с помощью тега `<template>`, разметил место для списка и форму для создания задачи.

```html
<template id="task">
  <li class="task tasks__task">
    <label class="task__label">
      <input class="task__checkbox" type="checkbox" />

      <span class="task__checkbox-custom"></span>
    </label>

    <p class="task__text" contenteditable></p>

    <button class="btn btn--red task__delete" type="button">
      Delete
    </button>
  </li>
</template>

<ul class="tasks"></ul>

<form class="create-task">
  <textarea class="create-task__textarea" placeholder="Task title"></textarea>

  <button class="btn btn--bg-blue create-task__submit" type="submit" disabled>
    Create
  </button>
</form>
```

Находим все эти элементы.

```javascript
//...

const container = document.querySelector('.tasks');
const template = document.querySelector('#task');

const createTaskForm = document.querySelector('.create-task');
const createTaskField = document.querySelector('.create-task__textarea');
const createTaskButton = document.querySelector('.create-task__submit');
```

Теперь нужно всего лишь перебрать список задач, полученный из `LocalStorage`.

Создаем функцию для создания задачи. В ней же объявляем события для обновления и удаления, чтобы использовать её в переборе задач из `LocalStorage` и в создании новой задачи из формы.

```javascript
function onCreateTask({data}) {
  const clone = template.content.cloneNode(true);

  const task = clone.querySelector('.task');
  const checkbox = clone.querySelector('.task__checkbox');
  const title = clone.querySelector('.task__text');
  const del = clone.querySelector('.task__delete');

  title.innerHTML = data.value;
  checkbox.checked = data.checked;

  toggleTaskStatusClass({checked: data.checked, task});

  checkbox.addEventListener('input', () => {
    data.checked = checkbox.checked;

    toggleTaskStatusClass({checked: data.checked, task});

    storage.update(data);
  });

  title.addEventListener('input', () => {
    data.value = title.innerHTML;

    storage.update(data);
  });

  del.addEventListener('click', (e) => {
    storage.delete(data);

    task.remove();
  });

  container.appendChild(clone);
}

function toggleTaskStatusClass({checked, task}) {
  task.classList[checked ? 'add' : 'remove']('task--done');
}
```

Метод получился довольно простым. Мы ищем элементы, записываем в них данные, навешиваем события на чекбокс, тайтл и кнопку "удалить", после чего вставлем элементы в наш контейнер.

По поводу метода `toggleTaskStatusClass` – полагаю, вы практикуете его не через точку, а через тернарный оператор. Это менее очевидно, но короче. Обычно я не делаю так, но здесь хотел показать такую возможность и узнать ваше мнение.

Теперь нам осталось перебрать `tasks`, используя этот метод, и навесить событие на форму.

```javascript
tasks.forEach((data) => {
  onCreateTask({data});
});

createTaskForm.addEventListener('submit', (e) => {
  e.preventDefault();

  const value = createTaskField.value;

  if (value) {
    const data = {
      value,
      checked: false
    };

    storage.create(data);

    onCreateTask({data});

    createTaskForm.reset();
  }
});
```

На этом всё. Можно запускать и создавать задачи.

Спасибо вам.

[Demo](https://vladimirschneider.github.io/To-Do-List/)
