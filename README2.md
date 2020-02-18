## An example todo app

```javascript
const Todos = require("todo-service");

class ToDoView extends React.Component { 
  constructor() {
    super();                                   // container component has only state, no props
    this.setState({ 
      title: "Simple Todo App",
      todoInputBox: {                          // state = WYSIWYG (only visual state, not business state)
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
          <TodoInputBox placeholder={this.state.todoInputBox.placeholderText} />
          <TodoAddButton text="+" />
          this.getTodoList().map(todo => <TodoRow completed={todo.completed} text={todo.text} />);
        </TodoInputForm>
      </TodoContainer>
    );
  }
  
  componentDidMount() {
    Todos.addEventListener((e) => {
      switch(e.type) {
        case TODO_CREATED:
        break;
      }
    });
  }
  
  componentDidUnmount() {
  }
  
  
}
```
