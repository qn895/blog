---
title: Common React/Redux Performance Pitfalls & How To Fix Them
date: "2019-08-14T22:40:32.169Z"
template: "post"
draft: false
slug: "/posts/react-redux-performance/"
category: "Code Snippets"
tags:
  - "React"
  - "Redux"
  - "Performance"
description: "Troubleshooting React/Redux performance problems can be dauting. Here are some common pitfalls and how to fix them."
---

In my experience troubleshooting React/Redux performance issues for several teams, I have found several comments pitfalls and mistakes that essentially boil down to:

1. Your selectors are not memoized correctly and are therefore over-computing.
2. Your React component have props that changed in shallow preference even though the content should be the same.
3. You're not taking advantage of memoizing tools like React.Memo, virtualization, or hash maps for look-up operations.

## Selectors Problems

**Root problem**: Selectors are over-computing or running even though the inputs _should_ be the same.

Usually you use selectors for when operations to calculate derived data get expensive, therefore, you should minimize how many times the inside functions run as much as possible.

**A simple quick check** to see if your selectors are over-computing is to add [`.recomputations`](https://github.com/reduxjs/reselect#q-how-do-i-test-a-selector) in `mapStateToProps` or in your tests to check how many times did your selectors actually ran.

```javascript
const mapStateToProps = state => {
  console.log(
    "Selector recomputed #",
    selectors.getDerivedChildren.recomputations()
  );
  return {
    object: declaredObject,
    test: selectors.getDerivedChildren(state)
  };
};
```

If your number of recomputations is higher than you expected, then you know one of the inputs in your selector _is_ indeed changing, even though in console.log the value looks the same.

If you get to this point, this means at least one of your earlier selectors is messed up to begin with. This could be because:

### 1. You're using Immutable and is using `.toJS()` too liberally.

**DO NOT call `.toJS` in mapStateToProps!** `toJS` creates a new object entirely, so if you put it in mapStateToProps, it's going to be called every time anything is done in a Redux app.

You should _only_ call `toJS`in selectors any no where else.

Also, `toJS` is overall an expensive operation to avoid. If you find yourself having to use `toJS` everywhere, consider whether or not you really need the library.

### 2. You're not memoizing selectors across multiple instances of the same components correctly.

Let's say you multiple `VisibleTodoList` instances that you want to display, each list has its own `listId`, and you're trying to show the lists together.

**A common pitfall is to have only _one_ selector for all your rows.** Everytime an instance of the `VisibleTodoList` is rendered, it's calling the `getVisibleTodoList` selector with a different `props.listId`, thus invalidating the cache and causing the selector to recompute.

```javascript
// WRONG
import { createSelector } from "reselect";

const getVisibilityFilter = (state, props) =>
  state.todoLists[props.listId].visibilityFilter;

const getTodos = (state, props) => state.todoLists[props.listId].todos;

const getVisibleTodoList = createSelector(
  [getVisibilityFilter, getTodos],
  (visibilityFilter, todos) => {
    switch (visibilityFilter) {
      case "SHOW_COMPLETED":
        return todos.filter(todo => todo.completed);
      case "SHOW_ACTIVE":
        return todos.filter(todo => !todo.completed);
      default:
        return todos;
    }
  }
);

export default makeGetVisibleTodos;
```

**Fix:** Make a [selector and makeStateToProps factory](https://github.com/reduxjs/reselect#sharing-selectors-with-props-across-multiple-component-instances) or [re-reselect](https://github.com/toomuchdesign/re-reselect). I'm a big fan of the re-select library and so that's what I personally use.

```javascript
// CORRECT with selector/mapState factory
// A bit cumbersome

// selectors.js
import { createSelector } from "reselect";

const getVisibilityFilter = (state, props) =>
  state.todoLists[props.listId].visibilityFilter;

const getTodos = (state, props) => state.todoLists[props.listId].todos;

const makeGetVisibleTodos = () => {
  return createSelector(
    [getVisibilityFilter, getTodos],
    (visibilityFilter, todos) => {
      switch (visibilityFilter) {
        case "SHOW_COMPLETED":
          return todos.filter(todo => todo.completed);
        case "SHOW_ACTIVE":
          return todos.filter(todo => !todo.completed);
        default:
          return todos;
      }
    }
  );
};

// Component.js
const makeMapStateToProps = () => {
  const getVisibleTodos = makeGetVisibleTodos();
  const mapStateToProps = (state, props) => {
    return {
      todos: getVisibleTodos(state, props)
    };
  };
  return mapStateToProps;
};
```

```javascript
// CORRECT using re-reselect

// selectors.js
const getVisibilityFilter = (state, props) =>
  state.todoLists[props.listId].visibilityFilter;

const getTodos = (state, props) => state.todoLists[props.listId].todos;

export const getVisibleTodos = createCatchedSelector([getVisibilityFilter, getTodos],
(visibilityFilter, todos) =>  {
      switch (visibilityFilter) {
        case "SHOW_COMPLETED":
          return todos.filter(todo => todo.completed);
        case "SHOW_ACTIVE":
          return todos.filter(todo => !todo.completed);
        default:
          return todos;
      }
    }
)(state, props) => `${props.listId}`



// Component.js
const mapStateToProps = (state, props) => {
  console.log(
    "getVisibleTodos recomputed #",
    selectors.getVisibleTodos.recomputations()
  );
  return {
    object: declaredObject,
    test: selectors.getVisibleTodos(state, props)
  };
};
```

## Rendering Problems

Believe it or not, there's not a better tool to troubleshoot this more quick and dirtier than your good ole `console.log()`.

```javascript
const getDisplayName = (WrappedComponent)  => {
  return WrappedComponent.displayName ||
         WrappedComponent.name ||
         ‘Component’
}

const log = (title) => (WrappedComponent) => {
   return class extends React.Component {
      render() {
        console.log(
          `%c${getDisplayName(WrappedComponent)} is being re-rendered`,
          "background: #0000ff; color: #bada55"
        );
        return <WrappedComponent {...this.props} />
      }
   }
}

@log()
class extends React.Component {
  render() {
      return <div />
  }
}

```

### Pitfall: Creating new objects or arrays in mapStateToProps or selectors

Constructing a new object `{}` or array `[]` in the mapStateToProps will also lead to re-rendering because a new reference is being recreated everytime mapStateToProps is run anywhere in the application.

```javascript
// WRONG
const mapStateToProps = state => ({
  /* Passing a new object here will cause the props.object to be new everytime */
  object: {
    value: "Component With Object In mapStateToProps (Wrong)"
  }
});
```

```javascript
// ALSO WRONG
const mapStateToProps = state => {
  const declaredObject = {
    value: "Component With Object In mapStateToProps (Wrong)"
  };
  return {
    /* Passing a new object here will cause the props.object to be new everytime */
    object: declaredObject
  };
};
```

**Fix**: The correct way is to declare it OUTSIDE of mapState or in a selector.

```javascript
// CORRECT
const declaredObject = {
  value: "Component With Object In mapStateToProps (Correct)"
};
const mapStateToProps = state => ({
  object: declaredObject
});
```

```javascript
// CORRECT
const mapStateToProps = state => ({
  object: selectors.getClaredObject(state)
});
```

Similarly, if you're passing in an object or array in the props:

```javascript
// WRONG
// This will re-render Component
    render(){
        return <Component extra={[]} />
    }

```

## Some Extra Toolings To Help with Performance

If you're rendering a large list of complicated items, there are several things you can do to help speed up your app.

### Use virtualization to render large lists

I recommend checking out [react-window](https://github.com/bvaughn/react-window) (which is the latest library from the same author as react-virtualized)

### Use reselect or memoizeOne _inside_ your React component (not just Redux)

If you have pure React components that need to do some extra heavy lifting, don't be afraid to memoize them!

The example shown below is utilizing `buildLayout`, `buildMainData`, and `buildHistogram` data based on props being received. These are very expensive functions, and as such it's completely advisable to memoize them. `memoizeOne` will cache the most recent one result, so if none of the props changed, the expensive functions won't be called.

```javascript
import React from "react";
import Plot from "./Plot";
import PropTypes from "prop-types";
import memoizeOne from "memoize-one";

const fakeData = Array.from(Array(600).keys()).map(d => ({
  title: `Item ${d}`,
  value: `item-${d}`,
  key: `key-${d}`
}));

export class CrossPlotWrapper extends React.Component {
  constructor(props) {
    super(props);

    this.buildMainData = this.buildMainData.bind(this);
    this.buildHistogramData = this.buildHistogramData.bind(this);
  }

  /* I'm using memoizeOne here but you can also use reselect as well */
  buildLayout = memoizeOne(
    (
      xAxisTitle,
      yAxisTitle,
      xHistogramVisible,
      yHistogramVisible,
      textColor,
      backgroundColor,
      gridColor,
      color
    ) => {
      // ...
      return layout;
    }
  );

  buildMainData = memoizeOne((chartData, markerColor) => {
    // ...
    return data;
  });

  buildHistogramData = memoizeOne(
    (chartData, markerColor, type, numberOfBins, showHistogram) => {
      // ...
      return histogramData;
    }
  );

  calculateHistogram(cData, min, max, type, numberOfBins) {
    //...
    return bins;
  }

  render() {
    const {
      xAxisTitle,
      yAxisTitle,
      numberOfXBins,
      numberOfYBins,
      chartData,
      theme,
      xHistogramVisible,
      yHistogramVisible,
      scrollZoom
    } = this.props;

    const palette = !!theme ? theme.palette : undefined;
    const themeText = !!palette ? palette.type : "light";
    const grey = !!palette ? palette.grey : undefined;

    const backgroundColor = "transparent";
    const textColor = themeText === "dark" ? grey[50] : grey[900];
    const color = themeText === "dark" ? grey[200] : grey[700];
    const gridColor = themeText === "dark" ? grey[700] : grey[300];

    const layout = this.buildLayout(
      xAxisTitle,
      yAxisTitle,
      xHistogramVisible,
      yHistogramVisible,
      textColor,
      backgroundColor,
      gridColor,
      color
    );

    const mainData = this.buildMainData(chartData, color);
    const xHistogramData = this.buildHistogramData(
      chartData,
      color,
      this.HORIZONTAL_TYPE,
      numberOfXBins,
      xHistogramVisible
    );
    const yHistogramData = this.buildHistogramData(
      chartData,
      color,
      undefined,
      numberOfYBins,
      yHistogramVisible
    );
    const data = [...mainData, ...xHistogramData, ...yHistogramData]; // would be even better if this was a memoized function

    return (
      <Plot
        config={{
          scrollZoom: scrollZoom
        }}
        data={data}
        layout={layout}
        useResizeHandler
        style={{ width: "100%", height: "100%" }}
      />
    );
  }
}
```

### Use React.Memo

I'm not going to delve on this too much because there's a lot of resources such as:

- [React's official documentation](https://reactjs.org/docs/react-api.html#reactmemo)
- [Use React.memo() wisely
  ](https://dmitripavlutin.com/use-react-memo-wisely/)

### Use Indexed Objects Instead of array.find() if you need it often

Often time when you need to look up information from an array and display it. If you find yourself keep having to use `yourArray.find()`, it might be best to convert that array to an object where you can easily access with an `id`.

```javascript
import _ from "lodash";

interface Datum {
  userId: string;
  //...
}

interface IndexedData {
  [key: string]: Datum;
}
export const getIndexedData = createSelector(
  [getData],
  (data: Datum[]) => {
    return _.keyBy(data, "userId");
  }
);
export const getUserData = createSelector(
  [getUserId, getData],
  (userId: UserId, data: IndexedData) => {
    return indexedData[userId];
  }
);

export const getFriendsData = createSelector(
  [getFriendIds, getIndexedData],
  (friendIds: UserId[], indexedData: IndexedData) => {
    return friendIds.map(id => indexedData[id]);
  }
);
```

### Use Redux-Visualize to check for performance and re-rendering of child components

[Redux-Visualize](<(https://github.com/humflelump/redux-visualize)>) is an awesome library/toolkit a co-worker of mine worked on to easily visualize the dependency graph of a Javascript application. It will help you detect if one of your selectors is taking long, or show you the current value catched inside your selector.

![](https://github.com/humflelump/redux-visualize/raw/master/gif.gif)

I hope this post gives you some quick & dirty ways to trouble shoot problems, especially if you are new to the world of React/Redux.
