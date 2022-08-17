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
      **componentDidUpdate -change the status of formIsValid**

3. 解决每次按键(keystroke)都会触发一次 useEffect 审查的问题（卸载）
   1. need to debounce the user input - 在行为外面裹一层延时 setTimeout，给个时间。
   2. 在 useEffect 里面 fn 的最后 return 一个 cleanup function，卸载定时器。卸载的前提是要给定时器起个名字，用 clearTimeout 卸载。
   3. so setFormIsValid only run once for all those keystrokes.
   4. 之前每打一个字->state 变化->触发表格合法化判断 fn。
      为了不频繁触发 fn，判断 fn 的外面裹了定时器：每打一个字->state 变化->等待 1 秒->触发判断 fn
      但如果全部都加上定时器，其实还是每打一个字都会触发 fn 只是全部延时 1 秒。因此如果在 1 秒内打了第二个字，则需要把第一个字的定时器清除，这样其实最后只执行了打最后一个字的定时器，则达到全部内容打完后统一出发判断 fn 的效果。
   5. 用在 send http request 上效果最明显。
      **componentWillUnmount - solved repeat request problem**

## useReducer() Hook - useState 高阶版，同时 manage 多组 state

1. scenario: If you update a state, which depends on another state. merging this into one state could be a good idea.
2. email 目前有两个项目需要管理：email 内容，email 是否合法。两个项目会组成一个 obj 统一由 emailState 管理
3. userReducer 包含第一个是 fn 接收 state 和 action 两个函数，第二个是 state 的初始值。
   1. fn 因为不涉及 component 内的内容页不参与 render 因此可以写在外面
   2. 因为同时管理两个 state，因此 return 一个初始 obj 包含两个项目
4. 目前 useReducer 中 emailState 就包含两个项目，各配有一个初始值。可以把之前分别使用两个 state 的地方都改为统一处理。这样就解决了 state 问题。
5. 解决 setState 问题
   1. 把 useReducer 中的 dispatchEmail 方法分别写在需要修改状态的位置，并用 type 最为暗号标注{type:暗号, val:要更新的新 state}。
   2. 回到 fn 中，action 包含所有暗号，所以增加条件: 1. 如果 action 的 type 为暗号 USER_INPUT，那么就更新 state 里面的 value 为 action.val，更新 state 里面的 isValid 为 action.val 这个内容上在判断是否.includes("@") 2. 如果 action 的 type 为暗号 INPUT_BLUR，那么更新 state 里面的 value 为目前 state 最新 value，更新 isValid 为目前 state 的 value 上直接判断是否 include @
      **use useReducer to combine entereEmail & emailIsValid state management**
      **use useReducer to combine enteredPassword & passwordIsValid state management**

## useReducer & useEffect

1. 重新打开 useEffect, 把 useState 内容改为 useReducer
2. 问题：this effect runs too often, whenever the email or the passwordState changed. 其实我们只在乎他们各自 isValid。比如输入密码，如果密码已经 valid，我再加更多的数字他还是 valid，其实就不必 run effect 了。
3. 所以我们在 useEffect 的外面把 isValid 属性分别从 emailState 和 passwordState 中 destructuring 出来并给一个单独的 alias assignment.
4. 把 dependencies 换成这两个 isValid 只监听这两个的变化即可，这样如果 isValid 为 true，后续就算你输入再多的数字，true 这个事实不变，effect 也就不会被 run。
   I’m pulling out the isValid state here, whenever just the value changes and the validity did not change, this effect will not rerun.
