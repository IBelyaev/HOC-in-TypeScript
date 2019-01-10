# HOC-in-TypeScript

h1 Компоненты высшего порядка React в TypeScript.

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

