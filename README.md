# Simple To-Do list using localstorage 📝👨🏼‍💻

I think everyone has probably faced the To-Do list development. In this post I'd like to share how to build a simple one using localstorage. 

Our application will allow:

1. To store To-Do list
2. To create, update and delete tasks

## Storage

I am going to keep our list in`localstorage`, so I will start with a simple interface for working with tasks and `localstorage`. I am going to store our tasks in an array of objects. 

I will get our data from `localstorage` and write to`this.tasks`. in the interface constructor:

```javascript
export default class LocalStorage {
  constructor() {
    // if item by key `tasks` is not defined JSON.parse return null, so I use `or empty array`
    this.tasks = JSON.parse(localStorage.getItem('tasks')) || [];
  }
}
```

Now I need methods to create, update and delete tasks.

Methods are going to be very simple. For example, here is the method for creating a task:

```javascript
export default class LocalStorage {
  //...

  create(data) {
    this.tasks.push(data);

    localStorage.setItem('tasks', JSON.stringify(this.tasks));
  }
}
```

In this method I update `this.tasks` and `localstorage item`.

To update or delete a task I need to find it in the array first. I will create a getter `get token` to generate a random token for it and will update our `create` method. Here I are also adding a`getIndexByToken` method to search for a task index in `this.tasks` in order to update or delete it.

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

Now the only thing I need to update a task is to find its index, check if it's not  `-1`, update the object by an index found and rewrite `localstorage tasks`. Similarly, for `delete` method I need to find an index, delete a respective object from `this.tasks`, and rewrite `localstorage item`.

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

## Application

Now lets build our application.

First, I need to create an instance of our class and get a list of task. I will create, update and delete the tasks using this instance.

```javascript
import LocalStorage from './LocalStorage.js';

const storage = new LocalStorage();

const tasks = storage.tasks;
```

Then I will create a html task template with html tag `<template>`, tasks container, and a task creation form.

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

Here I will get all these elements:

```javascript
//...

const container = document.querySelector('.tasks');
const template = document.querySelector('#task');

const createTaskForm = document.querySelector('.create-task');
const createTaskField = document.querySelector('.create-task__textarea');
const createTaskButton = document.querySelector('.create-task__submit');
```

Now I just need to iterate over task list from `LocalStorage`.

I will create `onCreateTask` function that will create tasks and add main events there:

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

The method turned out to be quite simple. I search for objects, set correct data and events, and append the child to container.

Let's look at  `toggleTaskStatusClass` method - usually this is not how I code things like that, and I shared it as example only. What's your take on it? Please don't hesitate to comment. 

Now I will iterate over tasks with onCreateTask method and create a form submit event.

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

That's all! Time for a demo. 

[Demo](https://vladimirschneider.github.io/To-Do-List/)

Thank you.
