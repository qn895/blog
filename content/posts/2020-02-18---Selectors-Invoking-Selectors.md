---
title: A Case for Selectors that Invoke Selectors
date: "2019-02-18T22:40:32.169Z"
template: "post"
draft: false
slug: "/posts/selectors-invoking-selectors/"
category: "Code Snippets"
tags:
  - "React"
  - "Redux"
  - "Selectors"
description: "A surprisingly neat, yet uknown pattern for managing logic with selectors is to invoke selectors within selectors. Memoize-ception!"
---

## A Case for Selectors that Invoke Selectors


An example use case is let's say you need to switch between different data _sources_, where each source has its own shape and business logic to transform to the final set of data you need. 
Commonly, one might usually want to do this transformation/expensive calculation  (clean, reshape, process, modify, etc.) everytime an action is being dispatched, either via the api promise itself, mapDispatch, reducer, or even redux-saga. However, that means you have to watch for all the _actions_ relevant to the calculation, then react to that action.

I have found that process to be very cumbersome due to the sheer amount of dependencies that you have to manage. By abstracting away the data from the final result across many chains, it's hard to make modifications and keep the codebase clean. Instead of letting the actions do the manual work, it's much better to lift that expensive calculation into a selector and have the selector be the centralized place to do the work.  

In this case, how would you switch data sources in a selector? An interesting pattern is to have a selector that returns ... a selector! But what does that mean?  Let's look at this example below.

```typescript
export const getDataSelectorForCrossPlot = createCachedSelector(
  [getDataSource],
  (dataSource: string) => {
    switch (dataSource) {
      case "DATA_SOURCE_A":
        return getFormattedDataFromSourceA; // a selector
      case "DATA_SOURCE_B":
        return getFormattedDataFromSourceB; // another selector
      case "DATA_SOURCE_C":
        return getFormattedDataFromSourceC;
      case "DATA_SOURCE_D":
        return getFormattedDataFromSourceE;
      default:
        return getDefaultFormattedData;
    }
  }
)((state, props) => state.identifier);

export const getTransformedData = state =>
  getDataSelectorForCrossPlot(state)(state); // computes the data from the selector being returned
```

Here `getDataSelectorForCrossPlot(state)` is returning a selector. That selector then gets called again so `returnedSelector(state)` which yield the final piece of data.

So what does getFormattedDataFromSourceA do? It can do anything! Each selector can have as many or as little inputs as needed. For example, `sourceA` may only be controlled by 3 settings, while `sourceB` maybe controlled by 6 or more. This approach makes it so that you don't have to oversubscribe sourceA to changes it doesn't need. 

```typescript
export const getFormattedDataFromSourceA = createCachedSelector(
  [
    getXAxisBy,
    getYAxisBy,
    getColorBy,
    sourceA.getAsyncBlahInfo, // an asynchronous selector or data getter
    workingListSelectors.getMetadata
  ],
  (
    xAxisBy: string,
    yAxisBy: string,
    colorBy: ColorByOption,
    resp: AsyncResp,
    metadata: Metadata
  ) => {
  ...
  return results
  }
)((state, props) => state.identifier);

```
```typescript
export const getFormattedDataFromSourceB = createCachedSelector(
  [
    getXAxisBy,
    getYAxisBy,
    getColorBy,
    getSizeBy,
    getAggregateBy,
    getSmoothingFn,
    sourceB.getAsyncBlah2Data,
    workingListSelectors.getMetadata
  ],
  (
    xAxisBy: string,
    yAxisBy: string,
    colorBy: ColorByOption,
    sizeBy: SizeByOption,
    aggregateBy: AggregationOption,
    smoothingFn: SmoothingFn, 
    resp: AsyncResp,
    metadata: Metadata
  ) => {
  ...
  return results
  }
)((state, props) => state.identifier);
```

Note that this pattern works quite seamlessly with both [asynchronous](https://github.com/humflelump/async-selector) and synchronous data (and even [enhanced selectors](https://github.com/toomuchdesign/re-reselect)). It can also helps (or forces?) you to keep track better of your data dependencies. Plus, now you can easily put your APIs/business logic in all the right places. How awesome! 
