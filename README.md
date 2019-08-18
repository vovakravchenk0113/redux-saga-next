# next-redux-saga

[![npm version](https://img.shields.io/npm/v/next-redux-saga.svg)](https://npmjs.org/package/next-redux-saga)
[![npm downloads](https://img.shields.io/npm/dm/next-redux-saga.svg)](https://npmjs.org/package/next-redux-saga)
[![XO code style](https://img.shields.io/badge/code_style-XO-5ed9c7.svg)](https://github.com/sindresorhus/xo)
[![styled with prettier](https://img.shields.io/badge/styled_with-prettier-ff69b4.svg)](https://github.com/prettier/prettier)
[![Build Status](https://travis-ci.com/bmealhouse/next-redux-saga.svg?branch=master)](https://travis-ci.com/bmealhouse/next-redux-saga)
[![All Contributors](https://img.shields.io/badge/all_contributors-4-orange.svg)](#contributors)

> redux-saga HOC for [Next.js](https://github.com/zeit/next.js/)

> **Attention:** Synchronous HOC is no longer supported since version 4.0.0!

## Installation

```sh
yarn add next-redux-saga
```

## Getting Started

Check out the official [Next.js example](https://github.com/zeit/next.js/tree/canary/examples/with-redux-saga) or clone this repository and run the local example.

### Try the local example

1. Clone this repository
1. Install dependencies: `yarn`
1. Start the project: `yarn start`
1. Open [http://localhost:3000](http://localhost:3000)

## Usage

`next-redux-saga` uses the redux store created by [next-redux-wrapper](https://github.com/kirill-konshin/next-redux-wrapper). Please refer to their documentation for more information.

### Configure Store

```js
import {createStore, applyMiddleware} from 'redux'
import createSagaMiddleware from 'redux-saga'
import rootReducer from './root-reducer'
import rootSaga from './root-saga'

function configureStore(preloadedState) {

  /**
   * Recreate the stdChannel (saga middleware) with every context.
   */

  const sagaMiddleware = createSagaMiddleware()

  /**
   * Since Next.js does server-side rendering, you are REQUIRED to pass
   * `preloadedState` when creating the store.
   */

  const store = createStore(
    rootReducer,
    preloadedState,
    applyMiddleware(sagaMiddleware)
  )

  /**
   * next-redux-saga depends on `sagaTask` being attached to the store.
   * It is used to await the rootSaga task before sending results to the client.
   */

  store.sagaTask = sagaMiddleware.run(rootSaga)

  return store
}

export default configureStore
```

### Configure Custom `_app.js` Component

```js
import React from 'react'
import {Provider} from 'react-redux'
import App, {Container} from 'next/app'
import withRedux from 'next-redux-wrapper'
import withReduxSaga from 'next-redux-saga'
import configureStore from './configure-store'

class ExampleApp extends App {
  static async getInitialProps({Component, ctx}) {
    let pageProps = {}

    if (Component.getInitialProps) {
      pageProps = await Component.getInitialProps(ctx)
    }

    return {pageProps}
  }

  render() {
    const {Component, pageProps, store} = this.props
    return (
      <Container>
        <Provider store={store}>
          <Component {...pageProps} />
        </Provider>
      </Container>
    )
  }
}

export default withRedux(configureStore)(withReduxSaga(ExampleApp))
```

### Connect Page Components

```js
import React, {Component} from 'react'
import {connect} from 'react-redux'

class ExamplePage extends Component {
  static async getInitialProps({store}) {
    store.dispatch({type: 'SOME_ASYNC_ACTION_REQUEST'})
    return {staticData: 'Hello world!'}
  }

  render() {
    return <div>{this.props.staticData}</div>
  }
}

export default connect(state => state)(ExamplePage)
```
