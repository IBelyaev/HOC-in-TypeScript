# HOC-in-TypeScript

h1 Компоненты высшего порядка React в TypeScript.

HOC - это мощный инструмент для переиспользования кода. Однако общая проблема разработчиков использующих TS - это их типизация.


Эта статья предполагает понимание HOC(ов), ниже будут представлены примеры возрастающей сложности, на которых будет показано, как их типизировать без использования any. В этой статье компоненты высшего порядка будут условно разделены на два типа: Enhancers и Injectors.

***Enhancers: Принимает prop или функцию, но не пробрасывает ее в сам компонент.***

***Injectors: Пробрасывает в компонент как prop, так и функцию для его изменения.***

Любой HOC может относиться, как к одному из этих типов, так и к двум одновременно, что будет показано в примерах.



Мы начнем с  Enhancers, так-как они наиболее просты для добавления типов. Простейшим примером этого HOC является добавление loading prop для отображения спинера если он равен true. Вот пример без типов:

```php
const withLoading = Component =>
  class WithLoading extends React.Component {
    render() {
      const { loading, ...props } = this.props;
      return loading ? <LoadingSpinner /> : <Component {...props} />;
    }
  };
```
