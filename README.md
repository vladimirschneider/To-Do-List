# Simple To-Do list using localstorage

I think everyone is familiar with the To-Do list development.

In the post i wanna give my resolve of the task.

Tasks which I resolved in the post:

1. Storage To-Do list
2. Create, update and delete task

## Storage

To-Do list i storage in `localstorage`, so for start I create a simple interface for working with localstorage.

Lists of tasks storage as array of objects.

In the interface constructor I get data from `localstorage` and write to `this.tasks`.

```javascript
export default class LocalStorage {
  constructor() {
    // if item by key `tasks` is not defined JSON.parse return null, so we use `or empty array`
    this.tasks = JSON.parse(localStorage.getItem('tasks')) || [];
  }
}
```

Now need methods of create, update and delete tasks.

Methods very simple. For example method for create a task.

```javascript
export default class LocalStorage {
  //...

  create(data) {
    this.tasks.push(data);

    localStorage.setItem('tasks', JSON.stringify(this.tasks));
  }
}
```

In the method I update `this.tasks` and `localstorage item`.

For an update or delete a task I need find a task in the array. I create getter `get token` for generate a random token for it. Let's update the `create` method. Here I added `getIndexByToken` method for search index of task in `this.tasks` for update or delete a task.

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

So, everything that's needed for `update` method find the index of the task, check that not equal `-1`, update object by the index and rewrite `localstorage tasks`. Similarly, for `delete` method: find the index of the task, delete from `this.tasks`, rewrite `localstorage item`.

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

## Solution

Go to common solution.

So I need create class exemplar and get tasks list. Create, update and delete tasks do with this class exemplar.

```javascript
import LocalStorage from './LocalStorage.js';

const storage = new LocalStorage();

const tasks = storage.tasks;
```

In the html I created a task template with html tag `<template>`, tasks container and form for create a task.

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
    App task
  </button>
</form>
```

Find all the elements.

```javascript
//...

const container = document.querySelector('.tasks');
const template = document.querySelector('#task');

const createTaskForm = document.querySelector('.create-task');
const createTaskField = document.querySelector('.create-task__textarea');
const createTaskButton = document.querySelector('.create-task__submit');
```

So, I need iterate over tasks list from `LocalStorage`.

Create `onCreateTask` function for create task in html and elements events. I use  `onCreateTask` method for create tasks at iterate over tasks at the beginning and for create a new task in the form by submit event.

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

The method turned out to be quite simple. I search elements, set correct data and events. After it append the child in tasks container.

About `toggleTaskStatusClass` method. Tell me what do you think about it. Usually I do not do this. I did it here for example and know your opinion.

So, iterate over tasks with onCreateTask method and create form submit event.

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

That's all. We can run demo and try create tasks.

Thank you.

[Demo](https://vladimirschneider.github.io/To-Do-List/)
