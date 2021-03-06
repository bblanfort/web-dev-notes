# More on Consuming Actions in Reducers
* We just added the `ActiveBook` reducer which produced a new piece of Application `state` whenever the **BOOK_SELECTED** `action` was triggered

## Let's work on the BookDetailView
* This will render whenever there is a **selectedBook**

### Component or a Container?
* We make Containers whenever we want to have a Component that can touch the Redux `state` directly
* We know what our **book** is and we know when it changes
* So it makes sense that this BookDetail should be a `Container`

#### App doesn't care :(
* Our `App` doesn't care about the `ActiveBook`
* As `App`s only purpose is to render the `BookList` and our soon-to-be-created `BookDetail`
* Only the `BookDetail` Component cares about what the `ActiveBook` is 
* So we will make the `BookDetail` Component a `Container`

`src/containers/BookDetail.js`

* We start off created a stateless functional component

```
import React from 'react';

const BookDetail = () => <div>Book Detail</div>;

export default BookDetail;

```

### Render BookDetail inside App
* Import and add the BookDetail instance

`src/components/App.js`

```
import React, { Component } from 'react';

import BookList from '../containers/BookList';
import BookDetail from '../containers/BookDetail';

export default class App extends Component {
  render() {
    return (
      <div>
        <BookList />
        <BookDetail />
      </div>
    );
  }
}
```

### View in browser
* Refresh and you should now see `BookDetail` text rendered to page

![current app rendered](https://i.imgur.com/2lE9TBS.png)

* We have our `BookDetail` wired up to be displayed by the `App` Component

## Hook up `BookDetail` to Redux store
* We also have to hook up the `BookDetail` to the **Redux store** so it gets told about changes to the `ActiveBook` piece of `state`

### Convert BookDetail to a Container

`BookDetail`

```
import React from 'react';
import { connect } from 'react-redux';

export const BookDetail = () => <div>Book Detail</div>;

function mapStateToProps(state) {
  return {
    book: state.activeBook,
  };
}

export default connect(mapStateToProps)(BookDetail);
```

* We'll import our `connect()` helper
* And connect our App `state` to the **props** of the `BookDetail` Container
  - **note** Remember we do that using a function called `mapStateToProps()` and then we connect `mapStateToProps()` with `BookDetail`
  - **note** Everytime we want to create a `Container` it is always the exact same number of steps

### Import our `connect()` function

`src/containers/BookDetail.js`

```js
import React, { Component } from 'react';
import { connect } from 'react-redux'; // add this line
```

### Add our helper function `mapStateToProps()`
* This has the argument of the Application `state`

`function mapStateToProps(state)`

**note** The object we return from `mapStateToProps()` will show up as `props` inside of our `BookDetail` `Container`

```js
function mapStateToProps(state) {
  return {
    book: state.activeBook
  }
}
```

**note** We are specifically referencing `activeBook` because our `ActiveBook` reducer is creating the `activeBook` piece of state

`src/reducers/index.js`

![activeBook](https://i.imgur.com/FDgnGgX.png)

### Export our Container
`src/containers/BookDetail.js`

* Change the last line from this:

`export default BookDetail;`

* To this:

`export default connect(mapStateToProps)(BookDetail);`

* We now have a connected Component

### Next up
* Conditional Rendering
