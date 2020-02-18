## Writing a React app with vanilla JS, view models and event emitters (no Redux)

```javascript
const Todos = require("todo-service");

class ToDoView extends React.Component { 
  constructor() {
    super();  // container component has only state, no props
    bindFunctionsStartingWith("on", this); // a util function
    this.setState({ // state = WYSIWYG (only visual state, not business state)
      title: "Simple Todo App",
      todoInputBox: {                          
        placeholderText: "Write todo here",
        text: ""
      }
      todoInputButton: {
        enabled: false
      }
      todoList: [
        {
          id: 1,
          text: "Email Bob",
          completed: false 
        }
        {
          id: 2,
          text: "Buy paper",
          completed: true
        }
       ]
    });
  }
  
  render() {
    return (
      <TodoContainer title={this.state.title}> // sub components have only props, no state
        <TodoInputForm>
          <TodoInputBox placeholder={this.state.todoInputBox.placeholderText} onChange={this.onChangeInputBox} />
          <TodoAddButton text="+" onClick={this.onClickAddButton} />
          this.getTodoList().map(todo => <TodoRow completed={todo.completed} text={todo.text} />);
        </TodoInputForm>
      </TodoContainer>
    );
  }
  
  componentDidMount() { // instead of connecting to an immutable store, listen to business events
    Todos.addEventListener(this.onTodoEvent);
  }
  
  onTodoEvent(e) { // business events
    switch(e.type) {
      case TODO_CREATED:
        this.appendTodoRow({
          text: e.data.text,
          completed: e.data.completed
        })
      break;
    }
  }
  
  componentDidUnmount() { // don't receive events when unmounted
    Todos.removeEventListener(this.onTodoEvent);
  }
  
  onChangeInputBox(e) {
    this.updateTodoInputBox(e.target.text); // overwrites what's already typed, with the same value, so that state is kept in sync
  }
  
  onClickAddButton(e) {
    Todos.createTodo({
      text: this.getTodoInputBoxText()
    });
  }
  
  
  
}
```
