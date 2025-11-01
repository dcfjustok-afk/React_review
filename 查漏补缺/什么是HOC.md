HOC         ****高阶组件****
    （Higher-Order Component）是 React 中一种复用组件逻辑的模式。它是一个函数，接受一个组件作为参数，并返回一个新的组件。HOC 可以在不修改原组件代码的情况下，添加额外的功能或行为。

const EnhancedComponent = higherOrderComponent(WrappedComponent);

即：
    WrappedComponent：被包装的原始组件；
    higherOrderComponent：高阶组件函数；
    EnhancedComponent：增强后的新组件。
它本质上是一个"组件逻辑复用的模式"，不是React提供的API，而是基于函数和组件的一种高级的方法。

最经典的HOC示例是 withLogger，它用于在组件挂载时打印日志。

✅ 例 1：日志增强（打印组件挂载/卸载）
import React from "react";

function withLogger(WrappedComponent) {
  return class extends React.Component {
    componentDidMount() {
      console.log(`${WrappedComponent.name} mounted`);
    }
    componentWillUnmount() {
      console.log(`${WrappedComponent.name} will unmount`);
    }
    render() {
      return <WrappedComponent {...this.props} />;
    }
  };
}

// 使用
function Hello(props) {
  return <h1>Hello, {props.name}</h1>;
}

export default withLogger(Hello);

解释：
    withLogger 是一个高阶组件；
    它在组件挂载/卸载时输出日志；
    WrappedComponent 是被包装的 Hello；
    最后返回一个新组件，增强了功能但不改变原组件逻辑。

✅ 例 2：权限控制（Auth HOC）
const withAuth = (WrappedComponent) => {
  return (props) => {
    const isLogin = localStorage.getItem("token");
    if (!isLogin) return <div>请先登录</div>;
    return <WrappedComponent {...props} />;
  };
};


👉 这个 HOC 可以复用在所有需要登录状态的页面上。

HOC的核心思想
    输入组件->加功能->输出新组件

🔍  HOC 常见用途
场景	示例
权限控制	withAuth(WrappedComponent)
网络请求注入	withFetchData(WrappedComponent)
日志/埋点	withLogger(WrappedComponent)
样式复用	withStyle(WrappedComponent)
性能优化	withMemo(WrappedComponent)
Redux connect	connect(mapStateToProps, mapDispatchToProps)(Component)
路由注入	withRouter(Component)（v5 中）

Hooks 替代趋势
    自从 React 16.8 引入 Hooks 后，HOC 逐渐被 Hooks 替代。Hooks 提供了更简洁、更灵活的方式来复用组件逻辑，避免了 HOC 中可能出现的"嵌套地狱"问题。



面试题：
❓1. 什么是高阶组件（HOC）？

    回答：
    高阶组件（Higher-Order Component, HOC）是一个函数，它接收一个组件作为参数，并返回一个新的组件，用来复用逻辑或增强功能。

    const EnhancedComponent = higherOrderComponent(WrappedComponent);


    不修改原组件代码；

    不影响原组件使用；

    是“组件复用逻辑的一种设计模式”，而不是 React 的特性。

❓2. HOC 和 Hook 有什么区别？
    对比项	HOC	自定义 Hook
    定义方式	函数包裹组件	函数抽离逻辑
    使用范围	类组件 & 函数组件	仅函数组件
    复用逻辑方式	包裹组件并注入 props	返回状态和方法
    优点	可复用，兼容类组件	简洁，无嵌套
    缺点	容易“嵌套地狱”，props 层级复杂	仅限函数组件
    替代趋势	✅ 被 Hooks 取代中	🔥 现代主流

❓3. 为什么要用高阶组件？

    原因：
        提取公共逻辑，避免代码重复；
        可以在不改动组件源码的情况下增强功能；
        常用于权限校验、日志埋点、样式增强、性能优化等。

❓1. HOC 的实现原理是什么？

    本质是一个函数，返回一个包装了原组件的新组件。
    它利用 React 的组合特性 来实现逻辑复用。

    底层思想：

    React 允许你用 JSX 嵌套组件；
    通过函数返回新的组件，可以在中间层加入逻辑；
    用 props 将信息继续往下传。

🧭 面试总结金句

💬 HOC 是 React 中实现组件逻辑复用的一种高阶模式。
它是一个函数，接收一个组件，返回一个增强后的组件。
常用于权限、数据注入、埋点、性能优化等场景。
现代项目中更推荐用 Hook 替代，但理解 HOC 的思想非常关键。