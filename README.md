# React's useReducer Hook

A hook for managing more complex state. 

## Objectives

- Discuss concept of a reducer
- Implement useReducer
- Discuss useState vs. useReducer

## Reducers

In programming, a reducer is just a function that reduces a set of values to a single value.

You may have encountered this type of function in the [Stream.reduce()](https://www.baeldung.com/java-stream-reduce) operation in Java or in [Array.prototype.reduce()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce) in JavaScript.

In JavaScript, the `.reduce()` array method takes up to two arguments: 1) a reducer function that walks through the array element-by-element and stores the results of the function to an accumulator value, and 2) an initial value for the accumulator.

Take a look at the JavaScript code snippet below.

```js
const array = [1, 2, 3, 4];

const sum = array.reduce(
  (accumulator, currentValue) => accumulator + currentValue,
  0
);

console.log(sum); // => 10
```

### Knowledge Check 🤔

Based on the definition of a reducer above, why is this an example of a reducer?

## React useReducer

We've learned `useState` so far to manage state in a React application. `useReducer` is similar to `useState`, but is used for more complex state logic.

With `useState`, we don't have the built-in ability to update state in pre-defined ways. When we call the `setState` function that lets us update state, we typically only replace the existing value of state with the new one. `useReducer` helps us define specific ways in which we want to update that state, and helps us separate state management from the concerns of the rest of our component.

### useReducer Syntax

Let's break down the syntax by referencing the `useReducer` [documentation](https://beta.reactjs.org/apis/react/useReducer#usage).

Here is an excellent [mental model](https://dmitripavlutin.com/react-usereducer/#3-reducer-mental-model) of the `useReducer` hook.

## Implementing `useReducer`

Let's take this [starter code](https://codesandbox.io/s/react-to-dos-starter-for-usereducer-63pzu4?file=/src/App.js) that uses `useState` and refactor it using `useReducer`.

If you'd like to code along, feel free to fork this starter code to your own CodeSandbox account.

To make sure we're all on the same page, let's look at what the starter code is currently doing.

Here is the state of todos.

```js
const [todos, setTodos] = useState(initialTodos);
```

And the supporting functions that let us update state in two different ways: 1) by adding a todo and 2) by marking a todo in state as "done".

```js
function handleAdd(newTodo) {
  setTodos([...todos, newTodo]);
}

function handleDone(todo) {
  const todosCopy = [...todos];
  const indexToUpdate = todosCopy.findIndex((td) => td.item === todo.item);
  todosCopy[indexToUpdate].done = true;
  setTodos(todosCopy);
}
```

These functions are called in response to user interactions: `handleAdd` is called when the user submits a form to create a new todo, and `handleDone` is called when the user clicks the "Done" button to mark that todo as done.

### Our First Reducer Function

Let's refactor this a bit using a reducer function. The idea is to aggregate all the logic needed to update state into a single function.

As we saw in the React documentation, this reducer takes two arguments: the initial state value and an action to perform that will update that state. Since some action is passed that will determine how that state needs to be updated, we'll use conditional logic to determine what to do in response to different actions.

The convention for writing an action is to have both a type and a payload. While the type is the action to be performed, the payload is the value used to update state. We'll use an object to capture the value and type of action in key-value pairs.

```js
// example action object (second parameter in the reducer)
{
  type: "ADD",
  // when adding a todo, the payload is the new todo to be added
  payload: { todo: "New todo", done: false }
}

// use action.type to determine how to update state
// use action.payload to determine what value to update state with
```

```js
function todosReducer(state, action) {
  if (action.type === "ADD") {
    return [...state, action.payload];
  } else if (action.type === "MARK_DONE") {
    const todosCopy = [...state];
    const indexToUpdate = todosCopy.findIndex(
      (td) => td.item === action.payload.item
    );
    todosCopy[indexToUpdate].done = true;
    return todosCopy;
  } else {
    return state;
  }
}
```

Notice that this is a pure function -- it always returns the same result if the same arguments are passed. It doesn't depend on or alter values outside of itself.

One thing to note about the above code is that the action being passed is expected, by convention, to be uppercase. This convention is meant to highlight the action being performed and a convention that is expected when using Redux.

If the action doesn't match any condition, we default to return the unchanged state. It's very clear in the function that the action determines how state is to be updated.

### Refactor Event Handlers

Let's update our code to make use of this reducer function. Replace the `handleAdd` and `handleDone` functions with this reducer. Now we have all the logic for updating state in one place, which is pretty cool. One pure function handles all the different ways we might need to update state.

Let's refactor the `handleSubmit` a bit:

```js
function handleSubmit(event) {
  event.preventDefault();
  const temp = { item: newTodo, done: false };
  // update this line
  setTodos(todosReducer(todos, { payload: temp, type: "ADD" }));
  setNewTodo("");
}
```

And the other place we're updating our todos state is the mark done button:

```js
onClick={() =>
  setTodos(
    todosReducer(todos, { payload: todo, type: "MARK_DONE" })
    )
  }
```

Test out your app and make sure your todo functionality is still intact.

### Switch Statements

Switch statements are typically used to write conditional logic for reducers, as they're easier to read than extended if/else statements.

Let's refactor our reducer a bit:

```js
function todosReducer(state, action) {
  switch (action.type) {
    case "ADD":
      return [...state, action.payload];
    case "MARK_DONE":
      const todosCopy = [...state];
      const indexToUpdate = todosCopy.findIndex(
        (td) => td.item === action.payload.item
      );
      todosCopy[indexToUpdate].done = true;
      return todosCopy;
    default:
      return state;
  }
}
```

This makes it a bit easier to think and reason about our logic.

### Knowledge Check 🤔

What benefits do we get by using a `reducer` function?

### useReducer

Let's replace `useState` with the `useReducer` hook. We still need `useState` to manage our form state, so add the following import:

```js
import { useState, useReducer } from "react";
```

`useReducer` works very similar to `useState` but with some differences. `useReducer` takes in a reducer function as the first argument and the initial state value as the second.

Like `useState`, it returns an array with two items:

1. The current value of this state variable
2. A dispatch function that lets you change it in response to user interaction

```js
// const [todos, setTodos] = useState(initialTodos);
const [todos, dispatch] = useReducer(todosReducer, initialTodos);
```

`Dispatch` is the conventional name of the function returned from the `useReducer` call, just like `setState` is conventional.

Now, all we have left to do is update the reducer calls to use dispatch instead. Dispatch already knows about the state it's associated with, so all we have to pass it is the action object with the type and payload key/value pairs.

For adding a todo:

```js
function handleSubmit(event) {
  event.preventDefault();
  const temp = { item: newTodo, done: false };
  // update this line:...isn't it beautiful???
  dispatch({ payload: temp, type: "ADD" });
  setNewTodo("");
}
```

For marking a todo done:

```js
onClick={() => dispatch({ payload: todo, type: "MARK_DONE" })}
```

Now all the logic for managing the todos state in this component is isolated to one pure, testable reducer function that returns a single updated state value for different actions. When we want to update this state, we simply call the dispatch function and tell it what type of action is being performed and what the payload is.

Completed code [here](https://codesandbox.io/s/react-to-dos-starter-for-usereducer-esins-working-copy-kvr89f?file=/src/App.js).

## useReducer vs useState: Which should I use?

Let's take a look at the React [documentation](https://beta.reactjs.org/learn/extracting-state-logic-into-a-reducer#comparing-usestate-and-usereducer) on this topic.

Kent C. Dodds, a terrific React blogger, advises that when your state is an independent element (like our newTodo form state), to use the `useState` hook. When elements of state are interdependent, `useReducer` is probably a better solution.

Our job is programmers is to assess the tools at our disposal for solving different types of problems, and make as informed a choice as possible as to which one is best suited for the taskk.

### Historical Note

`useReducer` was introduced in [React Version 16.8 in 2019](https://reactjs.org/blog/2019/02/06/react-v16.8.0.html). Before the introduction of this hook, more complex state management was typically handled with the help of third-party libraries like [Redux](https://react-redux.js.org/). New hooks like `useReducer` and `useContext` enable us to manage state in more sophisticated and powerful ways, somewhat lessening the need for third-party libraries. But you're still likely to see something like Redux or [Recoil](https://recoiljs.org/) (a newer, more lightweight state management solution) in real-world React code. Redux is highly worth learning if you want to pursue React professionally, and many of the concepts of Redux have been translated into `useReducer`, like dispatch functions and reducers.

### Knowledge Check 🤔

What are some of the ways that `useReducer` is different from `useState`?

## Resources

- React (beta) [docs](https://beta.reactjs.org/apis/react/useReducer) on useReducer
- Try out some `useReducer` [challenges](https://beta.reactjs.org/learn/extracting-state-logic-into-a-reducer#challenges)
- [Kent C. Dodds on useReducer](https://kentcdodds.com/blog/should-i-usestate-or-usereducer)
- [Dmitri Pavlutin on useReducer](https://dmitripavlutin.com/react-usereducer/)
