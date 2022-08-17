## useEffect() Hook

1. 解决用户登录后刷新需要重新登陆的问题(挂载)

   1. App.js 中使用 localStorage.setItem("isLoggedIn",1) 生成键值对储存用户状态
   2. useEffect 中，dependencies 为[]，相当于 componentDidMount(组件挂载后，发送网络请求时放于此勾子函数中，也可以进行异步任务)。run this effect function once when the app starts up.
      1. const 拿到 isLoggedIn 对应的键值
      2. 如果为 1,则把 isLoggedIn 状态直接改为 true。
      3. 页面每次 render 之后会先触发这个 hook，就可以先确认用户登录状态，如果浏览器记录为 1，那么就直接显示已登陆后的状态。
   3. 在 logoutHandler 中把 stored 的 Item remove 掉，这样就清空本地缓存，重新登陆。
      **stored login status**

2. 解决非法输入重复判断问题：根据 email 或 psw 输入内容是否合法，自动设置 formIsValid 的状态（更新）
   1. want to reevaluate and rerun the form validation state setting function for every keystroke in email and password change handler.
   2. 设置 uesEffect，2 个 dependencies，一旦这些 dependencies 有改变，则触发前面的 function（判断 email 和 psw 是否同时符合规范）
      after every login component function execution, it will rerun this useEffect function but only if either enteredEmail or enteredPassword changed in the last component rerender cycle.
   3. useEffect 中，dependencies 有值则相当于 componentDidUpdate（组件更新后）：super important hook that helps you deal with code that should be executed in response to something.
   4. So long story short: You must add all "things" you use in your effect function if those "things" could change because your component (or some parent component) re-rendered. That's why variables or state defined in component functions, props or functions defined in component functions have to be added as dependencies!
