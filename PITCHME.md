# Integrating Redux with ReactJS

---

## React Lifecycle Revisited

```JavaScript
import React from 'react';
import { View } from 'react-native';

class Component extends React.Component {

	constructor(props) {
		super(props);
		this.state = {};	// initial states
	}

	/* pure */
	render() {
		return (
			<View/>
		);
	}

	componentDidMount() {
		// side effects
	}

}
```

---

## Passing props to Child Component

Passing a state to child.

```JavaScript
/* parent component */
render() {
	return (
		<Component
			value={this.state.value}
		/>
	);
}
```

Receiving a prop.

```JavaScript
/* child component */
componentDidMount() {
	console.log(this.props.value);
}
```

---

## Invoking callbacks from Child Component

Task: create a user registration form.

Structure: (Form) -> (TextInput)

---

### Our Form Fields

```JavaScript
/* Form.jsx */
render() {
	return (
		<Field
			label="username"
			hint="pick a username"
			onChange={(text) => {
				this.setState({
					username: text
				});
			}}
		/>
	);
}
```

---

### Our TextInput within a Field

```JavaScript
/* Field.jsx */
render() {
	return (
		<React.Fragment>
			<Label>
				{this.props.label}
			</Label>
			<TextInput
				placeholder={this.props.hint}
				onChange={(text) => {
					this.props.onChange(text);	// invoke parent callback
				}}
			/>
		</React.Fragment>
	);
}
```

---

## Callback Hell

![](http://www.lukefabish.com/wp-content/uploads/2017/04/react-intro-article-component-state-render-loop.png)

---

## Motivation of Redux

- sharing of the same data across many components
- keeping states *predictable*
- keeping React Component *pure*

> pure functions: same input -> same output

---

### Key Concept: Single Source of Truth

One single *store* handling all states.

![](https://cdn.css-tricks.com/wp-content/uploads/2016/03/redux-article-3-03.svg)

---

### Key Concept: State is Read-Only

The only way to change the state is to emit an *action*, and resolved via a *reducer*.

![](https://cdn.css-tricks.com/wp-content/uploads/2016/03/redux-article-3-05.svg)

---

### Key Concept: Changes are made with Pure Functions

Reducers are just pure functions.

- take the previous state
- take an action
- return the next state.

![](http://blog.krawaller.se/img/redux-reducer.png)

---

## Reducers

A reducer is a pure function.

```JavaScript
const initialState = {
	lang: "en-HK"
};

export const LanguageReducer = (state = initialState, action) => {
	switch (action['type']) {
		case 'Language/SET': {
			return Object.assign({}, state, {  // note Object.assign
				lang: action['payload']['lang']
			});
		}
		default: {
			return state;
		}
	}
};
```

---

### Common Pitfall

Direct mutation of ```state```.

```JavaScript
state['lang'] = action['payload']['lang'];
return state;
```

Not realizing ```Object.assign``` mutates the first argument.

```JavaScript
return Object.assign(state, {
	lang: action['payload']['lang']
});
```

> Key Concept: State is Read-Only

---
