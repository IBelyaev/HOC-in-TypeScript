# HOC-in-TypeScript

Компоненты высшего порядка React в TypeScript.
-----------------------------------

HOC - это мощный инструмент для переиспользования кода. Однако общая проблема разработчиков использующих TS - это их типизация.

Эта статья предполагает понимание HOC(ов), ниже будут представлены примеры возрастающей сложности, на которых будет показано, как их типизировать без использования any. В этой статье компоненты высшего порядка будут условно разделены на два типа: Enhancers и Injectors.

***Enhancers: Принимает prop или функцию, но не пробрасывает ее в сам компонент.***

***Injectors: Пробрасывает в компонент как prop, так и функцию для его изменения.***

Любой HOC может относиться, как к одному из этих типов, так и к двум одновременно, что будет показано в примерах.



Мы начнем с  Enhancers, так-как они наиболее просты для добавления типов. Простейшим примером этого HOC является добавление loading prop для отображения спинера если он равен true. Вот пример без типов:

```js
const withLoading = Component =>
  class WithLoading extends React.Component {
    render() {
      const { loading, ...props } = this.props;
      return loading ? <LoadingSpinner /> : <Component {...props} />;
    }
  };
```

А вот тот же код, но только с добавленными типами: 


```ts
interface WithLoadingProps {
  loading: boolean;
}

const withLoading = <P extends object>(Component: React.ComponentType<P>) =>
  class WithLoading extends React.Component<P & WithLoadingProps> {
    render() {
      const { loading, ...props } = this.props as WithLoadingProps;
      return loading ? <LoadingSpinner /> : <Component {...props} />;
    }
  };

```

Давайте разберем этот пример подробнее:

```ts
interface WithLoadingProps {
  loading: boolean;
}
```

Здесь создается интерфейс с типами новых props, которые будут добавлены компоненту, после того, как он будет обернут HOC(ом).

```ts
<P extends object>(Component: React.ComponentType<P>)
```

Здесь мы используем generic; P является типами для props компонентакоторый будет обернут HOC(ом). React.ComponentType<P> это более общее название, которое включает в себя по смыслу несколько React.StatelessComponent<P> | React.ClassComponent<P>, оно говорит о том, что компонент который будет являться аргументо для HOC(а) может быть как class component, так и функциональным компонентом.
 

```ts 
class WithLoading extends React.Component<P & WithLoadingProps>
```

Здесь мы определяем HOC и указываем, что он будет ожидать как props, необходимые для оборачиваемого компонента, так и props для самого HOC(a). Объеденение типов а TS возможно при использовании оператора &.

```ts
const { loading, ...props } = this.props as WithLoadingProps;
```

Spread оператор используется для того, чтобы убрать из props те, что были необходимы HOC(у) и оставить только необходимые для оборачиваемого компонента до того, как они будут в него переданы. Здесь приходиться прибегнуть к приведению типов, это связано с проблемой  rest/spread операторов объекта в TS, которая в настоящее время решается в одном из PR в репозиторий TypeScript (уже решена).  В общем случае приведение типов это плохая практика, но в нашем случае это необходимое зло, особенно, если учесть тот факт, что остальные props, мы не используем. Если мы не будем использовать приведение типов, мы получим ошибку компилятора TS.

Наконец мы используем loading prop для отображения либо загрузочного сипнера либо компонента со всеми его props.

```ts
return loading ? <LoadingSpinner /> : <Component {...props} />;
```

Наш HOC можно переписать, чтобы он возвращал функциональный компонент: 

```ts
const withLoading = <P extends object>(
  Component: React.ComponentType<P>
): React.SFC<P & WithLoadingProps> => ({
  loading,
  ...props
}: WithLoadingProps) =>
  loading ? <LoadingSpinner /> : <Component {...props} />;
```

### Injectors


Injectors - это более распространенный тип HOC, но более сложный для типизации. Помимо добавления props в компонент, он также делает невозможным передачу их саружи в оборачиваемый компонент, пока он обернут в него, чтобы это было невозможно сделать из вне. Функция connect из пакета react-redux - это пример injectors HOC, но в этой статье будет рассмотрен более простой вариант, а именно HOC который будет передавать значения счетчика и две функции для его увеличения и уменьшения. 


```ts
import { Subtract } from 'utility-types';

export interface InjectedCounterProps {
  value: number;
  onIncrement(): void;
  onDecrement(): void;
}

interface MakeCounterState {
  value: number;
}

const makeCounter = <P extends InjectedCounterProps>(
  Component: React.ComponentType<P>
) =>
  class MakeCounter extends React.Component<
    Subtract<P, InjectedCounterProps>,
    MakeCounterState
  > {
    state: MakeCounterState = {
      value: 0,
    };

    increment = () => {
      this.setState(prevState => ({
        value: prevState.value + 1,
      }));
    };

    decrement = () => {
      this.setState(prevState => ({
        value: prevState.value - 1,
      }));
    };

    render() {
      return (
        <Component
          {...this.props}
          value={this.state.value}
          onIncrement={this.increment}
          onDecrement={this.decrement}
        />
      );
    }
  };
  
  Здесь есть несколько ключевых отличий от прошлого примера:
  
  ```ts
  export interface InjectedCounterProps {  
  value: number;  
  onIncrement(): void;  
  onDecrement(): void;
}
```

Объявляется интерфейс для props которые будут переданы в компонент с возможностью его экспортировать, чтобы  применить в компоненте.

```ts

import makeCounter, { InjectedCounterProps } from './makeCounter';

interface CounterProps extends InjectedCounterProps {
  style?: React.CSSProperties;
}

const Counter = (props: CounterProps) => (
  <div style={props.style}>
    <button onClick={props.onDecrement}> - </button>
    {props.value}
    <button onClick={props.onIncrement}> + </button>
  </div>
);

export default makeCounter(Counter);
```

```ts
<P extends InjectedCounterProps>(Component: React.ComponentType<P>)
```

Опять используется дженерик, но на этот раз он гарантирует, что компонент переданный в HOC включает в себя props передаваемые самим HOC(ом), в противном случае вы получите ошибку компиляции TS.

```ts
class MakeCounter extends React.Component<
  Subtract<P, InjectedCounterProps>,    
  MakeCounterState  
>
```

Компонент возвращаемый HOC(ом) использует Subtract от Piotrek Witek’s который можно импортировать из  utility-types, который уберет из типов передаваемых props типы для state самого HOC(а), это сделано для того, чтобы не было возможности изменить состояние компонента извне. При попытке пробросить prop value в компонент  MakeCounter получиться ошибка компиляции.

В большинстве случаев это поведение ожидается от Injector(а), но это позже будет обсуждаться в этой статье.

### Enhance + Inject
