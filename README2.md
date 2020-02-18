
```javascript
class ToDoView extends React.Component {
  render() {
    return (
      <TodoContainer title="Simple Todo App">
        <TodoInputForm>
          <TodoInputBox placeholder="Write todo here" />
          <TodoAddButton text="+" />
          this.getTodoList().map(todo => <TodoRow completed={todo.completed} text={todo.text} />);
        </TodoInputForm>
      </TodoContainer>
    );
  }
  
  componentDidMount() {
  }
  
  componentDidUnmount() {
  }
  
  
}
```
