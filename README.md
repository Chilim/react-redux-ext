## Redux integration using [react-redux](https://redux.js.org/basics/usage-with-react):  
1. [Provider](https://react-redux.js.org/api/provider) - `Provider  Component` makes redux container available for the whole application
```
import { Provider } from 'react-redux';
import reducers from './reducers';

const store = createStore(reducers);

render(
  <Provider store={store}>
    <TasksBox />
  </Provider>,
  document.getElementById('container'),
);
```

2. [Connect](https://react-redux.js.org/api/connect) - The `connect()` function connects a React component to a Redux store.

```
import React from 'react';
import { connect } from 'react-redux';

// This function takes necessary data from the redux store and return it to the component

const mapStateToProps = state => {
  const props = {
    users: state.users,
  }
  return props;
};

class UsersList extends React.Component {
  render() {
    const { users } = this.props;
    // render users
  }
}

export default connect(mapStateToProps)(UsersList);
```

2.1. [Connect](https://react-redux.js.org/api/connect) function provides feature to avoid of using [store.dispatch](https://redux.js.org/api/store#dispatch) method for `actions`. Instead, `actions` are passed as a second parameter of `connect()` function.

```
import React from 'react';
import { connect } from 'react-redux';
import * as actions from '../actions'; // actions are imported

const mapStateToProps = state => {
  const props = {
    users: state.users,
  }
  return props;
};

const actionCreators = {
  addUser: actions.addUser,
}; 

class UsersList extends React.Component {
  handleAddUser = (e) => {
    e.preventDefault();
    // extract addUser action and user's name from props
    const { addUser, newUserText } = this.props; 
    addTask({ text: newTaskText });
  }   

  render() {
    const { users } = this.props;
    return <div>{/* logic with this.handleAddUser */}</div>;
  }
}

export default connect(mapStateToProps, actionCreators)(UsersList);
```  

## Redux integration using [react-actions](https://redux-actions.js.org/introduction/tutorial):  

There are dozens and sometimes even hundreds actions even in small applications in JS. As the result, a lot of same code appears. Also, a mistake with an action name in a reducer might also be a headache and a system will not inform that mistake is done. By this reason, there are libraries that help to avoid this situation. Ont of the most popular is `redux-action` that incudes two aspects:  
a. Determines action
b. Determines storage  

Redux-action has three methods:
- [createAction](https://redux-actions.js.org/api/createaction)
- [handleAction](https://redux-actions.js.org/api/handleaction)
- [combineAction](https://redux-actions.js.org/api/combineactions)

`createAction` wraps an action creator so that its return value is the payload. 

```
import { createAction } from 'redux-actions';

export const addUser = createAction('USER_ADD');
export const updateUser = createAction('USER_UPDATE');

addUser('John') // { type: 'USER_ADD', payload: 'John' }
updateUser('Smith') // { type: 'USER_UPDATE', payload: 'Smith' }
```

`handleAction` wraps a reducer so that it only handles actions of a certain type.

```
import { handleActions } from 'redux-actions';
import * as actions from './actions';

const users = handleActions({
  [actions.addUser]: (state, action) => {
    const { user } = action.payload; // extract a user from payload
    return { ...state, [user.id]: user }; // 
  },
}, {});

```

`combineActions` combines a number of actions which allows to reduce multiple distinct actions with the same reducer.


```
const users = handleActions({
   [combineActions(increment, decrement)] (state, payload: { user }) => {
    return { ...state, [user.id]: user }; // 
  },
}, {});
```

## Optimizing Redux integration using [reselect](https://github.com/reduxjs/reselect): 



`Reselect` creates `selectors` that allow:
- Redux to store the minimal possible state by computing derived data
- Memoize result avoiding of recomputation of data that was not changed
- Using selectors as inputs of other selectors  

Example:

```
const mapStateToProps = (users) => {
  const props = {
    users: Object.values(users.byId),
  };
  return props;
};

```
In the given example `users` represented with commonly used structure

```
users : {
        byId : {
            "user1" : {
                username : "user1",
                name : "User 1",
            },
            "user2" : {
                username : "user2",
                name : "User 2",
            },
            "user3" : {
                username : "user3",
                name : "User 3",
            }
        },
        allIds : ["user1", "user2", "user3"]
    }
```
The function `mapStateToProps` returns an array of users where Object.values create a new object even if `users` did not change. For this purposes `selectors` are used.

```
import { createSelector } from 'reselect';

const getUsers = state => state.users;

const usersSelector = createSelector(
  getUsers,
  users => Object.values(users),
);

const mapStateToProps = (state) => {
  const props = {
    users: usersSelector(state),
  };
  return props;
};
```

## Optimizing forms in React-Redux application with [redux-forms](https://redux-form.com/8.2.2/): 

Forms in React are tough. Each new element demands synchronization with the state and adding new code within all levels of application. `Redux-form` proved to avoid this. 

