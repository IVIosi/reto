## Pass Arguments to Store

You can use the `args` prop of `Provider` to pass arguments to Store.

```jsx
export function FooStore(initial = 1) {
  const [x, setX] = useState(initial)
  return {
    x,
    setX
  }
}
```

```jsx
<Provider of={FooStore} args={[5]}>
  <App/>
</Provider>
```

## Hierarchical Store Tree

Reto uses React's Context API under the hood. Thereby, you can create a hierarchical store tree.

```
- Provider of FooStore -> mark as instance A
  |- Cosumer of FooStore -> we got instance A
  |- Provider of FooStore -> mark as instance B
     |- Consumer of FooStore -> we got instance B
```

## Dependencies Between Stores

If you want to "inject" `FooStore` store into `BarStore`, you can call `useStore` in the body of `BarStore`.

```jsx
export function BarStore() {
  const fooStore = useStore(FooStore)
  const [x, setX] = useState(2)
  return {
    x,
    setX,
    fooStore,
  }
}
```

In the mean while, Reto will know `FooStore` is the dependency of BarStore. So whenever `FooStore` updates, `BarStore` will update too. 

## withProvider

Sometimes we need to use a store in the same component that provides it, for example:

```jsx
export function App(props) {
  const fooStore = useStore(FooStore) // ⚠️Can't get fooStore here
  return (
    <Provider of={FooStore}>
      <p>{fooStore.x}</p>
    </Provider>
  )
}
```

We hope to create a `Provider` of `FooStore` in `App` component, and call `useStore(FooStore)` in it at the same time. OK, we got `withProvider` for you:

```jsx
export const App = withProvider({
  of: FooStore
})((props) => {
  const fooStore = useStore(FooStore) // 🎉 Now we can get fooStore here
  return (
    <p>{fooStore.x}</p>
  )
})
```

`withProvider` is a HOC(Higher-Order Component). First, we pass a props object to it(just like what we do in jsx, but in the format of object). And then we pass our component to it.

```jsx
withProvider({
  of: FooStore,
  args: [42, 'abc'],
})(YourComponent)
```

当然，你还可以基于`withProvider`创建自己的高阶组件：

```js
const provideFooStore = withProvider({
  of: FooStore
})

export const App = provideFooStore((props) => {
  //...
})
```

## Use in Class Components

Even though Reto itself is written with hooks, it is still supported to use Reto in class components. You may wonder how to use `useStore` in class components. The answer is: No, you can't. But, there is an substitute for `useStore`, which is `Consumer` component:

```jsx
import {Consumer} from 'reto'

export class App extends Component {
  render() {
    return (
      <Consumer of={FooStore}>
        {fooStore => (
          fooStore.x
        )}
      </Consumer>
    )
  }
}
```


## How to Solve the Performance Issue

If a store is too big or it updates too frequently, there may be a performance issue.

Since `useStore` is actually `useContext` under the hood, you can solve this issue by the same way of `useContext`. Please see [this](https://github.com/facebook/react/issues/15156#issuecomment-474590693) for reference.
