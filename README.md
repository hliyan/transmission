# Transmission Archtecture (TX)

Transmission is an event driven variant of the Flux front end architecture. For a more academic treatment of the motivation and the principles behind this architecture, read the [whitepaper](./Transmission-TX-A-new-Flux-architecture.pdf). For a more practical guide, continue reading.

There is more than one way to think about *Transmission*. Here we will develop the thinking behind each layer of the architecture peacemeal.

## Contents

- [The UI layer](#the-ui-layer)
- [Event model](#event-model)
- [Engine](#the-engine)


## The UI layer

### 1. The UI is a dumb shell

UI layer is just a dumb, empty shell. It needs an "engine" to tell it what to do.

<br /><p align="center"><img src="Transmission-FE1.svg" /></p><br />

### 2. State variables must correpsond to things on screen

Every UI state variable must correspond directly to something visible on screen.

<br /><p align="center"><img src="Transmission-FE2.svg" /></p><br />

### 3. Design the UI layer API by examining the mock/wireframe

Consider this simple TODO app:

<br /><p align="center"><img src="Transmission-Mockup.svg" /></p><br />

If you carefully think about every element and how it can interact with the end user, you will end up with a UI layer API like this.

Events fired by the UI layer:

- TODO_APP_LOADED
- TODO_INPUT_BOX_EDITED
- ADD_TODO_BUTTON_CLICKED
- TODO_CHECKBOX_CLICKED
- TODO_DELETE_BUTTON_CLICKED

An "application engine" can listen to these events and use the following functions to manipulate the UI:

- updateTodoInputBox(text)
- updateTodoList(list)
- appendTodoInList(todo)
- updateTodo(todo)
- removeTodoFromList(id)

### 4. How to drive the UI

Here's a piece of sample code that illustrates how an "engine" can drive the UI by listening to user events and calling UI API functions.

```javascript
import UI from 'TodoUI';
import mock from 'mocks';

UI.addEventListener(UI.events.TODO_APP_LOADED, (e) => {
  UI.updateTodoList(mock.todoList);
});

UI.addEventListener(UI.events.TODO_CHECKBOX_CLICKED, (e) => {
  const todo = mocks.findTodo({id: e.id});
  todo.status = todo.status === true ? false : true;
  UI.appendTodoInList(todo);
});
```

### 5. Designing the internal state of the UI

Remember: every state variable in the UI's store must directly correspond to something on screen. There shouldn't be any abstract/virtual states or business state there. Such state belongs in the "engine".

Always design the state by mimicking the visual structure.

E.g.

Let's look at that TODO mockup again:

<br /><p align="center"><img src="Transmission-Mockup.svg" /></p><br />

```
app:
  title:
  todoInputBox:
    placeholderText:
    text:
  todoInputButton:
    enabled:
  todoList:
    - id: 1
      text: Buy milk
      completed: false
    - id: 2
      text: Write email
      completed: true
```

### 6. UI API manipulates UI by manipulating state

In Redux, reducers listen to actions, determine the type of action and directly perform a series of operations on the UI state. It's similar in TX, except that the UI provides a set of APIs to perform those operations, rather than allowing direct variable access to the outside.

E.g.

```javascript
function updateTodoInputBox(text) {
  state.app.todoInputBox.text = text;
}
```

### 7. UI layer summary

In summary, the UI layer of a Transmission application is a dumb, Mechanical-Turk-like shell that simply emit events in response to user actions. It expects some entity with business logic and data to listen to those events and tell it what to do using its UI manipulation API. This API does only one thing: change the state of variables within the UI layer that are wired directly to visual elements. These, taken together, constitutes the "UI layer" of the Transmission architecture.

## Event model

### 1. Events only; no promises, no callbacks

One very important aspect of Transmission is that it relies exclusively on *events* for asynchronous processing. That is, callbacks and promises are not used.

For example:

```javascript

// the component will only emit an event in response to a user action

class AddTodo extends Component {
  render() {
    return (
      <div data-todo-text={this.state.todoText} onClick={this.onClick}>
        <input value={this.state.todoText} /> <button >+</button>
      </div>);
  }
  
  onClick(e) {
    this.emitter.emit(ADD_TODO_BUTTON_CLICKED, {todoText: e.currentTarget.dataset[todo-text]});
  }
  
  constructor(props) {
    super(props);
    this.onClick = this.onClick.bind(this);
    this.emitter = new Emitter();
  }

}

// something needs to listen to those events and tell it what to do

UI.mount(document.getElementById('root'));

UI.addEventListener(ADD_TODO_BUTTON_CLICKED, (e) => {
  console.log(e.todoText);
});

```

Note that the event here is "fire-and-forget":

```javascript
  onClick(e) {
    this.emitter.emit(ADD_TODO_BUTTON_CLICKED, {todoText: e.currentTarget.dataset[todo-text]});
  }
```

The alternative would have been:

```javascript
  onClick(e) {
    const res = await Engine.addTodo({todoText: e.todoText});
    if (!res.error) {
      this.updateTodoInputBox("");
      this.appendTodoInList(res.todo);
    }
  }
```

There is no returned promise or callback. As far as the UI layer is concerned, that is the end of processing. The UI layer returns to a listening mode, waiting for either more user actions or a call to its UI API, telling it what to do. By doing this, we have achieved the following:

1. Short execution paths
2. UI layer is fully decoupled from the business/application layer -- it can effectively be developed by someone who has no knowedge of business logic.
3. Our code follows basic sequence, iteration and selection: code executes in the sequence it is written, which makes it easier to read and reason about.

### 2. Discrete events

Not virtual events like those emitted by EventEmitter. Actual discrete events in the JavaScript event queue. Achieved using either setTimeout or postMessage. Solves the problem the dispatcher tries to solve. Helps prevent re-entrant issues. Helps reduce UI stuttering.

## The Engine

The engine encapsulates business state and business logic, but has zero coupling to the UI layer. 

The idea is that the engine should be usable by *any* type of interface or application: GUI, command line, batch process, API -- anything.

Also event driven -- does not return promises or provide callbacks. Emits events instead.
