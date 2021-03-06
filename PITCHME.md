# Integrating Redux with ReactJS

---

## React Lifecycle Revisited

```JavaScript
class Component extends React.Component {
	constructor(props) {
		super(props);
		// initial states
		this.state = {};
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
					// invoke parent callback
					this.props.onChange(text);
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

> pure functions:

> same input -> same output

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

## Actions

Actions are payloads of information that send data from your application to your store.

- a plain JavaScript object
- the only source of information for the store
- must have a type property
- optional payload property

```json
{
  "type": "Language/SET",
  "payload": {
		"lang": "en-HK"
	}
}
```

---

### Dispatcher

A helper function for easy action dispatching.

```JavaScript
export class LanguageAction {
	/**
	 * Sets the current language
	 * @param lang language
	 */
	static setLocale = (lang) => {
		store.dispatch({
			type: 'Language/SET',
			payload: {
				lang: lang
			}
		});
	}
}
```

---

## Reducers

A reducer is a pure function.

```JavaScript
const init = {
	lang: "en-HK"
};
export const LanguageReducer = (state = init, action) => {
	switch (action['type']) {
		// capture ACTION
		case 'Language/SET': {
			return Object.assign({}, state, {
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

Not aware that ```Object.assign``` mutates the first argument.

```JavaScript
return Object.assign(state, {
	lang: action['payload']['lang']
});
```

> Key Concept:

> State is Read-Only

---

## Store

The *store* is the object that brings reducers together.

- only ONE single store in an app
- holds the application state
- allows access to state via ```getState()```
- allows state to be updated via ```dispatch(action)```

```JavaScript
export const store = createStore(combineReducers({
	GithubReducer: GithubReducer,
	LanguageReducer: LanguageReducer
	// ...
}), applyMiddleware(saga));
```

---

## Redux Flow

![](https://image.slidesharecdn.com/reactreduxintroduction-151124165017-lva1-app6891/95/react-redux-introduction-33-638.jpg?cb=1448383914)

---

## A Glance of Redux

- Action -> pure JSON
- Reducer -> pure function
- Store -> plain state repository

> How about side effects?

---

## Side-Effect Model

- listen for actions
- perform side effects
- optionally dispatch another action

---

## Introduction to redux-saga

A saga is a generator function.

```JavaScript
yield takeLatest('Github/RELEASE-FETCH', function* (action) {
	try {
		let res = yield call((payload) => {
			return new Promise((resolve, reject) => {
				// do whatever side-effect
			});
		}, action['payload']);
		yield put({
			type: 'Github/RELEASE-FETCH-COMPLETE',
			payload: res
		});
	} catch (err) {
		console.log(err);
	}
});
```

---

### redux-saga API

Saga Helpers

```JavaScript
import { takeEvery, takeLatest } from 'redux-saga/effects';

takeEvery(ACTION, GENERATOR, PAYLOAD);
takeEvery(ACTION, GENERATOR, PAYLOAD);
```

---

### redux-saga API

Effects Creator

```JavaScript
import { call, put } from 'redux-saga/effects';

let result = yield call((payload) => {
	// return a Promise
});
yield put(ACTION);
```

---

## Combining Reducers

```JavaScript
import { combineReducers } from 'redux';

combineReducers({
	// ... reducers
})
```

---

## Combining Sagas

```JavaScript
import createSagaMiddleware from 'redux-saga';
import { all, fork } from 'redux-saga/effects';

const saga = createSagaMiddleware();

saga.run(function* () {
	yield all([
		// ... sagas
	].map(fork));
});
```

---

## Register Helpers

```JavaScript
/* store.js */

const saga = createSagaMiddleware();

export const store = createStore(combineReducers({
	GithubReducer: GithubReducer,
	LanguageReducer: LanguageReducer
}), applyMiddleware(saga));

saga.run(function* () {
	yield all([
		GithubSaga
	].map(fork));
});
```
