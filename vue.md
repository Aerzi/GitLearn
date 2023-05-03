# 基础

## 创建Vue应用

每个 Vue 应用都是通过 `createApp`函数创建一个新的 **应用实例**：

```js
<!-- In JS File -->

import { createApp } from 'vue'

const app = createApp({
  /* 根组件选项 */
})
```

### 根组件

 `createApp()` 创建整个应用的根组件 

### 挂载应用

应用实例必须在调用了 `.mount()` 方法后才会渲染出来。

- 该方法接收一个“容器”参数，可以是一个实际的 DOM 元素或是一个 CSS 选择器字符串

```HTML
<!-- In HTML File -->
<div id="app"></div>
```

```js
<!-- In JS File -->
app.mount('#app')
```

## 模板语法

### 属性绑定

- `v-bind` 指令指示 Vue 将元素的 `id` attribute 与组件的 `dynamicId` 属性保持一致。

- 如果绑定的值是 `null` 或者 `undefined`，那么该 attribute 将会从渲染的元素上移除。

- 没有参数的 `v-bind` 会将一个对象的所有属性都作为 attribute 应用到目标元素上

#### 布尔型属性

布尔型 attribute 依据 true / false 值来决定 attribute 是否应该存在于该元素上

- 为真或`''`时，元素包含该属性
- 为假时，元素该属性被移除

#### 动态绑定多个值

通过不带参数的 `v-bind`命令

```js
<!-- In JS File -->
const objectOfAttrs = {
  id: 'container',
  class: 'wrapper'
}
```

```vue
<!-- In Vue File -->
<div v-bind="objectOfAttrs"></div>
```

#### 使用JS 表达式

- 在 Vue 模板内，JavaScript 表达式可以被使用在如下场景上：

  - 在文本插值中 (双大括号)

  - 在任何 Vue 指令 (以 `v-` 开头的特殊 attribute) attribute 的值中

- 每个绑定仅支持**单一表达式**，也就是一段能够被求值的 JavaScript 代码（一个简单的判断方法是是否可以合法地写在 `return` 后面）

- 可以在绑定的表达式中使用一个组件暴露的方法（函数）

#### 指令Directives![指令语法图](https://cn.vuejs.org/assets/directive.69c37117.png)

- 指令是带有 `v-` 前缀的特殊 attribute

- 指令的任务是在其表达式的值变化时响应式地更新 DOM

##### 参数 Arguments

某些指令会需要一个“参数”，在指令名后通过一个冒号隔开做标识

##### 动态参数

在指令参数上也可以使用一个 JavaScript 表达式，需要包含在一对方括号内

```vue
<!-- In Vue File -->
<a v-bind:[attributeName]="url"> ... </a>
```

存在一些限制

- 动态参数中表达式的值应当是一个字符串，或者是 `null`。特殊值 `null` 意为显式移除该绑定。其他非字符串的值会触发警告。
- 动态参数表达式因为某些字符的缘故有一些语法限制，比如空格和引号，在 HTML attribute 名称中都是不合法的。（使用计算属性替换复杂表达式）
- 免在名称中使用大写字母，因为浏览器会强制将其转换为小写

##### 修饰符 Modifiers

修饰符是以点开头的特殊后缀，表明指令需要以一些特殊的方式被绑定

## 响应式基础

`<script setup>`中的顶层的导入和变量声明可在同一组件的模板中直接使用

在 Vue 中，状态都是默认深层响应式的。这意味着即使在更改深层次的对象或数组，你的改动也能被检测到

### reactive()

创建一个响应式对象或数组，返回的是原始对象的Proxy

为保证访问代理的一致性，对同一个原始对象调用 `reactive()` 会总是返回同样的代理对象，而对一个已存在的代理对象调用 `reactive()` 会返回其本身

**局限：**

- 仅对对象类型有效（对象、数组和 `Map`、`Set` 这样的集合类型）

- 因为 Vue 的响应式系统是通过属性访问进行追踪的，因此我们必须始终保持对该响应式对象的相同引用。这意味着我们不可以随意地“替换”一个响应式对象，因为这将导致对初始引用的响应性连接丢失  ->  **当我们将响应式对象的属性赋值或解构至本地变量时，或是将该属性传入一个函数时，我们会失去响应性**

### ref()

`ref()` 将传入参数的值包装为一个带 `.value` 属性的 ref 对象.ref 的 `.value` 属性是响应式的。同时，当值为对象类型时，会用 `reactive()` 自动转换它的 `.value`。

- 一个包含对象类型值的 ref 可以响应式地替换整个对象

- ref 被传递给函数或是从一般对象上被解构时，不会丢失响应性

- 当 ref 在模板中作为**顶层属性**被访问时，它们会被自动“解包”，所以不需要使用 `.value`
- 当一个 `ref` **被嵌套在一个响应式对象中，作为属性被访问或更改时**，它会自动解包，因此会表现得和一般的属性一样
- 当 ref 作为响应式数组或像 `Map` 这种原生集合类型的元素被访问时，不会进行解包

## 计算属性

使用计算属性来描述依赖响应式状态的复杂逻辑

- 一个计算属性仅会在其响应式依赖更新时才重新计算；相比之下，方法调用**总是**会在重渲染发生时再次执行函数

- 计算属性默认是只读的，以通过同时提供 getter 和 setter 来创建可写的

```vue
<script setup>
import { ref, computed } from 'vue'

const firstName = ref('John')
const lastName = ref('Doe')

const fullName = computed({
  // getter
  get() {
    return firstName.value + ' ' + lastName.value
  },
  // setter
  set(newValue) {
    // 注意：这里使用的是解构赋值语法
    [firstName.value, lastName.value] = newValue.split(' ')
  }
})
</script>

<!-- 现在再运行 fullName.value = 'John Doe' 时，setter 会被调用而 firstName 和 lastName 会随之更新。 -->
```

## Class与Style绑定

Vue 专门为 `class` 和 `style` 的 `v-bind` 用法提供了特殊的功能增强

### 绑定HTML Class

- **绑定对象**：给 `:class` (`v-bind:class` 的缩写) 传递一个对象来动态切换 class
  - 绑定在内联样式
  - 先创建对象，再绑定
  - 绑定一个计算属性

```vue
<scrpit setup>
    const isActive = ref(true)
	const hasError = ref(false)
</scrpit>
<template>
	<div
  		class="static"
  		:class="{ active: isActive, 'text-danger': hasError }"
	></div>
</template>
<!-- 当isActive/hasError为真时，才绑定对应属性，否则渲染时就不绑定-->

<scrpit setup>
    const classObject = reactive({
  		active: true,
  		'text-danger': false
	})
</scrpit>

<template>
	<div :class="classObject"></div>
</template>

<scrpit setup>
    const isActive = ref(true)
	const error = ref(null)

	const classObject = computed(() => ({
  		active: isActive.value && !error.value,
        'text-danger': error.value && error.value.type === 'fatal'
	}))
</scrpit>

<template>
	<div :class="classObject"></div>
</template>
```

- **绑定数组**：给 `:class` 绑定一个数组来渲染多个 CSS class
  - 直接绑定数组
  - 在数组中使用三元表达式
  - 在数组中嵌套对象

```vue
<scrpit setup>
    const activeClass = ref('active')
	const errorClass = ref('text-danger')
</scrpit>
<template>
	<div :class="[activeClass, errorClass]"></div>
</template>

<scrpit setup>
    const isActive = ref(true)
    const activeClass = ref('active')
	const errorClass = ref('text-danger')
</scrpit>
<template>
	<div :class="[isActive ? activeClass : '', errorClass]"></div>
</template>

<scrpit setup>
    const isActive = ref(true)
	const errorClass = ref('text-danger')
</scrpit>
<template>
	<div :class="[{ active: isActive }, errorClass]"></div>
</template>
```

- **组件继承**

  - 只有一个根元素的组件，其根元素会继承组件的class

  ```vue
  <!-- 子组件模板 -->
  <p class="foo bar">Hi!</p>
  <!-- 在使用组件时 -->
  <MyComponent class="baz boo" />
  <!-- 渲染结果 -->
  <p class="foo bar baz boo">Hi</p>
  ```

  - 有多个根根元素的组件，其根元素使用`$attrs` 属性来实现指定继承

  ```vue
  <!-- MyComponent 模板使用 $attrs 时 -->
  <p :class="$attrs.class">Hi!</p>
  <span>This is a child component</span>
  <!-- 在使用组件时 -->
  <MyComponent class="baz" />
  <!-- 渲染结果 -->
  <p class="baz">Hi!</p>
  <span>This is a child component</span>
  ```

### 绑定内联样式

- **绑定对象**：`:style` 支持绑定 JavaScript 对象值，对应HTML 元素的 `style` 属性

  - 推荐使用 camelCase（小驼峰命名法）
  - 支持 kebab-cased 形式的 CSS 属性 key (对应其 CSS 中的实际名称)
  - 可以先创建对象，再绑定
  - 可以使用返回样式对象的计算属性

  ```vue
  <script setup>
      const activeColor = ref('red')
  	const fontSize = ref(30)
  </script>
  <template>
  	<div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
  	<div :style="{ 'font-size': fontSize + 'px' }"></div>
  </template>
  
  <script setup>
      const styleObject = reactive({
        	color: 'red',
        	fontSize: '13px'
     	})
  </script>
  <template>
  	<div :style="styleObject"></div>
  </template>
  ```

  

- **绑定数组**： `:style` 绑定一个包含多个样式对象的数组。这些对象会被合并后渲染到同一元素上
- **样式多值**：对一个样式属性提供多个 (不同前缀的) 值，数组仅会渲染浏览器支持的最后一个值

```vue
<template>
	<div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
</template>
<!-- 在支持不需要特别前缀的浏览器中都会渲染为 display: flex -->
```

## 条件渲染

### `v-if`

`v-if` 指令用于条件性地渲染一块内容。这块内容只会在指令的表达式返回真值时才被渲染。

### `v-else` 

使用 `v-else` 为 `v-if` 添加一个“else 区块”， `v-if` 为假时，渲染`v-else`

### `v-else-if` 

`v-else-if` 提供的是相应于 `v-if` 的“else if 区块”。它可以连续多次重复使用

```vue
<template>
	<div v-if="type === 'A'">
  		A
	</div>
	<div v-else-if="type === 'B'">
 	 	B
	</div>
	<div v-else-if="type === 'C'">
 		C
	</div>
	<div v-else>
  		Not A/B/C
	</div>
</template>
```

### `<template>` 上的条件渲染

 `<template>` 只是一个不可见的包装器元素，如果想要切换不止一个元素，可以在一个 `<template>` 元素上使用 `v-if`、`v-else`和 `v-else-if`

### `v-show`

其用法与`v-if` 基本一样。不同之处在于 `v-show` 会在 DOM 渲染中保留该元素；`v-show` 仅切换了该元素上名为 `display` 的 CSS 属性。

`v-show` 不支持在 `<template>` 元素上使用，也不能和 `v-else` 搭配使用。

### `v-if` vs. `v-show`

- `v-if` 是“真实的”按条件渲染，在切换时，条件区块内的事件监听器和子组件都会被销毁与重建；`v-if` 也是**惰性**的：如果在初次渲染时条件值为 false，则不会做任何事。条件区块只有当条件首次变为 true 时才被渲染。
- `v-show` 简单许多，元素无论初始条件如何，始终会被渲染，只有 CSS `display` 属性会被切换。
- `v-if` 有更高的**切换开销**，而 `v-show` 有更高的**初始渲染开销**。因此，如果需要频繁切换，则使用 `v-show` 较好；如果在运行时绑定条件很少改变，则 `v-if` 会更合适

## 列表渲染

### `v-for`

使用 `v-for` 指令基于一个数组来渲染一个列表。

- `v-for` 指令的值需要使用 `item in items` 形式的特殊语法，其中 `items` 是源数据的数组，而 `item` 是迭代项的别名

- `v-for` 也支持使用可选的第二个参数表示当前项的位置索引

```vue
<script setup>
    const parentMessage = ref('Parent')
	const items = ref([{ message: 'Foo' }, { message: 'Bar' }])
</script>
<template>
	<li v-for="item in items">
  		{{ item.message }}
	</li>
	<!-- 有 index 索引时 -->
	<li v-for="(item, index) in items">
  		{{ parentMessage }} - {{ index }} - {{ item.message }}
	</li>
</template>
```

- 可以在定义 `v-for` 的变量别名时使用解构，和解构函数参数类似

```vue	
<script setup>
    const parentMessage = ref('Parent')
	const items = ref([{ message: 'Foo' }, { message: 'Bar' }])
</script>
<template>
	<li v-for="{ message } in items">
  		{{ message }}
	</li>
	<!-- 有 index 索引时 -->
	<li v-for="({ message }, index) in items">
  		{{ message }} {{ index }}
	</li>
</template>
```

- 可以多层嵌套
- `v-for` 可以直接接受一个整数值。在这种用例中，会将该模板基于 `1...n` 的取值范围重复多次。

```vue
<template>
	<span v-for="n in 10">{{ n }}</span>
</template>
```

### `v-for` 与对象

可以使用 `v-for` 来遍历一个对象的所有属性。遍历的顺序会基于对该对象调用 `Object.keys()` 的返回值来决定

- 可以通过提供第二个参数表示属性名
- 可以通过提供第三个参数表示位置索引

```vue
<template>
    <li v-for="(value, key, index) in myObject">
      	{{ index }}. {{ key }}: {{ value }}
    </li>
</template>
```

### `<template>` 上的 `v-for`

可以在 `<template>` 标签上使用 `v-for` 来渲染一个包含多个元素的块

### `v-if` 和 `v-for`

- 当 `v-if` 和 `v-for` 同时存在于一个元素上的时候，`v-if` 会首先被执行  ->  `v-if` 的条件将无法访问到 `v-for` 作用域内定义的变量别名

- **不推荐同时使用**，在外新包装一层 `<template>` 再在其上使用 `v-for` 可以解决这个问题

### 通过 key 管理状态

- Vue 默认按照“就地更新”的策略来更新通过 `v-for` 渲染的元素列表，确保它们在原本指定的索引位置上渲染。

- 在任何可行的时候为 `v-for` 提供一个 `key` attribute
- 使用 `<template v-for>` 时，`key` 应该被放置在这个 `<template>` 容器上
- `key` 绑定的值期望是一个基础类型的值，不要用对象作为 `v-for` 的 key

### 组件上使用 `v-for`

可以直接在组件上使用 `v-for`，和在一般的元素上使用没有区别，但是，这不会自动将任何数据传递给组件，因为组件有自己独立的作用域。为了将迭代后的数据传递到组件中，我们还需要传递 props

```vue
<template>
    <MyComponent
      v-for="(item, index) in items"
      :item="item"  // 传递props
      :index="index"  // 传递props
      :key="item.id"
    />
</template>
```

### 数组变化侦测

- **变更方法**：Vue 能够侦听响应式数组的变更方法，并在它们被调用时触发相关的更新
  - `push()`
  - `pop()`
  - `shift()`
  - `unshift()`
  - `splice()`
  - `sort()`
  - `reverse()`

- **数组替换**：遇到返回新数组的方法，需要将旧数组替换为新数组

```vue
<script setup>
    items.value = items.value.filter((item) => item.message.match(/Foo/))
</script>
```

### 展示过滤或排序后的结果

显示数组经过过滤或排序后的内容，而不实际变更或重置原始数据。在这种情况下，可以创建返回已过滤或已排序数组的计算属性

在计算属性不可行的情况下 (例如在多层嵌套的 `v-for` 循环中)，你可以使用**嵌套`v-for`**

```vue
<script setup>
    const numbers = ref([1, 2, 3, 4, 5])
	const evenNumbers = computed(() => {
  		return numbers.value.filter((n) => n % 2 === 0)
	})
    
    const sets = ref([
      	[1, 2, 3, 4, 5],
      	[6, 7, 8, 9, 10]
    ])
    function even(numbers) {
      	return numbers.filter((number) => number % 2 === 0)
    }
</script>
<template>
	<li v-for="n in evenNumbers">{{ n }}</li>

	<ul v-for="numbers in sets">
      	<li v-for="n in even(numbers)">{{ n }}</li>
    </ul>
</template>
```

在计算属性中使用 `reverse()` 和 `sort()` 的时候，这两个方法将**变更原始数组**，计算函数中不应该这么做  ->  **不定传参**

## 事件处理

使用 `v-on` 指令 (简写为 `@`) 来监听 DOM 事件，并在事件触发时执行对应的 JavaScript

### 内联事件处理器

事件被触发时执行的内联 JavaScript 语句

- 可以在内联事件处理器中调用方法

- 内联事件处理器中访问原生DOM事件

  - `$event`特殊变量
  - 箭头函数

  ```vue
  <script setup>
      function warn(message, event) {
    		// 这里可以访问原生事件
    		if (event) {
      		event.preventDefault()
    	}
    		alert(message)
  	}
  </script>
  <template>
  	<!-- 使用特殊的 $event 变量 -->
      <button @click="warn('Form cannot be submitted yet.', $event)">
        	Submit
      </button>
  
      <!-- 使用内联箭头函数 -->
      <button @click="(event) => warn('Form cannot be submitted yet.', event)">
       	Submit
      </button>
  </template>
  ```

  

### 方法事件处理器

一个指向组件上定义的方法的属性名或是路径

```vue
<script setup>
   const name = ref('Vue.js')

    function greet(event) {
      	alert(`Hello ${name.value}!`)
      	// `event` 是 DOM 原生事件
      	if (event) {
        	alert(event.target.tagName)
      	}
    }
</script>
<template>
	<button @click="greet">Greet</button>
</template>
```

- 方法事件处理器会自动接收原生 DOM 事件并触发执行，能够通过被触发事件的 `event.target.tagName` 访问到该 DOM 元素

### 事件修饰符

Vue 为 `v-on` 提供了**事件修饰符**。修饰符是用 `.` 表示的指令后缀，包含以下这些：

- `.stop`
- `.prevent`
- `.self`
- `.capture`
- `.once`
- `.passive`

`.capture`、`.once` 和 `.passive` 修饰符与原生 `addEventListener` 事件相对应

`.passive` 修饰符一般用于触摸事件的监听器，可以用来改善移动端设备的滚屏性能

```vue
<template>
	<!-- 单击事件将停止传递 -->
    <a @click.stop="doThis"></a>

    <!-- 提交事件将不再重新加载页面 -->
    <form @submit.prevent="onSubmit"></form>

    <!-- 修饰语可以使用链式书写 -->
    <a @click.stop.prevent="doThat"></a>

    <!-- 也可以只有修饰符 -->
    <form @submit.prevent></form>

    <!-- 仅当 event.target 是元素本身时才会触发事件处理器 -->
    <!-- 例如：事件处理器不来自子元素 -->
    <div @click.self="doThat">...</div>

	<!-- 添加事件监听器时，使用 `capture` 捕获模式 -->
    <!-- 例如：指向内部元素的事件，在被内部元素处理前，先被外部处理 -->
    <div @click.capture="doThis">...</div>

    <!-- 点击事件最多被触发一次 -->
    <a @click.once="doThis"></a>

    <!-- 滚动事件的默认行为 (scrolling) 将立即发生而非等待 `onScroll` 完成 -->
    <!-- 以防其中包含 `event.preventDefault()` -->
    <div @scroll.passive="onScroll">...</div>
</template>
```

### 按键修饰符

Vue 允许在 `v-on` 或 `@` 监听按键事件时添加按键修饰符

可以直接使用 [`KeyboardEvent.key`](https://developer.mozilla.org/zh-CN/docs/Web/API/KeyboardEvent/key/Key_Values) 暴露的按键名称作为修饰符，但需要转为 kebab-case 形式

```vue
<template>
	<!-- 仅在 `key` 为 `Enter` 时调用 `submit` -->
	<input @keyup.enter="submit" />	

	<input @keyup.page-down="onPageDown" />
</template>
```

- **按键别名**：Vue 为一些常用的按键提供了别名
  - `.enter`
  - `.tab`
  - `.delete` (捕获“Delete”和“Backspace”两个按键)
  - `.esc`
  - `.space`
  - `.up`
  - `.down`
  - `.left`
  - `.right`

- **系统按键修饰符**：使用以下系统按键修饰符来触发鼠标或键盘事件监听器，只有当按键被按下时才会触发

  - `.ctrl`

  - `.alt`
  - `.shift`
  - ``.meta`(在 Mac 键盘上，meta 是 Command 键 (⌘)。在 Windows 键盘上，meta 键是 Windows 键 (⊞)。在 Sun 微机系统键盘上，meta 是钻石键 (◆)。在某些键盘上，特别是 MIT 和 Lisp 机器的键盘及其后代版本的键盘，如 Knight 键盘，space-cadet 键盘，meta 都被标记为“META”。在 Symbolics 键盘上，meta 也被标识为“META”或“Meta”)

```VUE
<template>
	<!-- Alt + Enter -->
    <input @keyup.alt.enter="clear" />

    <!-- Ctrl + 点击 -->
    <div @click.ctrl="doSomething">Do something</div>
</template>
```

- **`.exact` 修饰符**：允许控制触发一个事件所需的确定组合的系统按键修饰符（控制系统按键修饰符的修饰符）

```vue
<template>
	<!-- 当按下 Ctrl 时，即使同时按下 Alt 或 Shift 也会触发 -->
    <button @click.ctrl="onClick">A</button>

    <!-- 仅当按下 Ctrl 且未按任何其他键时才会触发 -->
    <button @click.ctrl.exact="onCtrlClick">A</button>

    <!-- 仅当没有按下任何系统按键时触发 -->
    <button @click.exact="onClick">A</button>
</template>
```

- **鼠标按键修饰符**：限定为特殊鼠标按键触发的事件

```vue
<template>
	<!-- 当按下鼠标左键时，才会触发 -->
    <button @click.left="doSomething">A</button>

    <!-- 当按下鼠标右键时，才会触发 -->
    <button @click.right="doSomething">A</button>

    <!-- 当按下鼠标中键时，才会触发 -->
    <button @click.middle="doSomething">A</button> 
</template>
```

## 表单输入绑定

`v-model` 指令进行表单输入绑定

### 表单绑定

#### 文本、多行文本

在 `<textarea>` 中是不支持插值表达式的。使用 `v-model` 来替代

```vue 
<script setup>
import { ref } from 'vue'

const message = ref('')
</script>

<template>
	<p>Message is: {{ message }}</p>
	<input v-model="message" placeholder="edit me" />

	<span>Multiline message is:</span>
	<p style="white-space: pre-line;">{{ message }}</p>
	<textarea v-model="message" placeholder="add multiple lines"></textarea>
</template>
```

#### 复选框

可以将多个复选框绑定到同一个数组或集合的值

```vue
<script setup>
import { ref } from 'vue'

const checkedNames = ref([])
</script>

<template>
  <div>Checked names: {{ checkedNames }}</div>

  <input type="checkbox" id="jack" value="Jack" v-model="checkedNames" />
  <label for="jack">Jack</label>
 
  <input type="checkbox" id="john" value="John" v-model="checkedNames" />
  <label for="john">John</label>
 
  <input type="checkbox" id="mike" value="Mike" v-model="checkedNames" />
  <label for="mike">Mike</label>
</template>
```

#### 单选框

```vue 
<script setup>
import { ref } from 'vue'

const picked = ref('One')
</script>

<template>
  <!-- picked的值为被选中的单选框value的值 -->
  <div>Picked: {{ picked }}</div>

	<input type="radio" id="one" value="One" v-model="picked" />
	<label for="one">One</label>

	<input type="radio" id="two" value="Two" v-model="picked" />
  <label for="two">Two</label>
</template>
```

#### 选择器

- 单选：`v-model` 表达式的初始值不匹配任何一个选择项，`<select>` 元素会渲染成一个“未选择”的状态。在 iOS 上，这将导致用户无法选择第一项，因为 iOS 在这种情况下不会触发一个 change 事件。建议提供一个空值的禁用选项

  ```vue
  <script setup>
  import { ref } from 'vue'
  
  const selected = ref('')
  </script>
  
  <template>
    <span> Selected: {{ selected }}</span>
  
    <select v-model="selected">
      
      <option>A</option>
      <option>B</option>
      <option>C</option>
    </select>
  </template>
  ```

- 多选：值绑定到一个数组

  ```vue
  <script setup>
  import { ref } from 'vue'
  
  const selected = ref([])
  </script>
  
  <template>
    <div>Selected: {{ selected }}</div>
  
    <select v-model="selected" multiple>
      <option>A</option>
      <option>B</option>
      <option>C</option>
    </select>
  </template>
  
  <style>
  select[multiple] {
    width: 300px;
  }
  </style>
  ```

选择器可以用**`v-for`动态渲染**

```vue 
<script setup>
import { ref } from 'vue'

const selected = ref('A')

const options = ref([
  { text: 'One', value: 'A' },
  { text: 'Two', value: 'B' },
  { text: 'Three', value: 'C' }
])
</script>

<template>
  <select v-model="selected">
    <option v-for="option in options" :value="option.value">
      {{ option.text }}
    </option>
  </select>

	<div>Selected: {{ selected }}</div>
</template>
```

### 值绑定

通过使用 `v-bind` 来实现将值绑定到当前组件实例上的动态数据，此外，使用 `v-bind` 还可以将选项值绑定为非字符串的数据类型

 `true-value` 和 `false-value` 是 Vue 特有的 attributes，仅支持和 `v-model` 配套使用。被绑定属性的值会在选中时被设为 `true-value `，取消选择时设为 `'false-value'`。

```vue
<script setup>
    import { ref } from 'vue'
    
    const dynamicTrueValue = ref("dynamicTrueValue")
    const dynamicFalseValue = ref("dynamicFalseValue")
</script>
<template>
	<!-- 复选按钮 -->
	<!-- 直接使用内置属性 -->
  	<input
  		type="checkbox"
  		v-model="toggle"
  		true-value="yes"
  		false-value="no" />

	<!-- 使用v-bind -->
	<input
  		type="checkbox"
  		v-model="toggle"
  		:true-value="dynamicTrueValue"
 		:false-value="dynamicFalseValue" />

	
	<!-- 单选按钮 -->
	<input type="radio" v-model="pick" :value="first" />
	<input type="radio" v-model="pick" :value="second" />


	<!-- 选择器 -->
	<select v-model="selected">
      <!-- 内联对象字面量 -->
      <option :value="{ number: 123 }">123</option>  <!-- 当选项被选中，selected 会被设为该对象字面量值 { number: 123 }。-->
    </select>
</template>
```

### 修饰符

- **`.lazy`**：在每次 `change` 事件后更新数据（默认情况`v-model` 会在每次 `input` 事件后更新数据）

- **`.number`**：将用户输入自动转换为数字
  - 如果该值无法被 `parseFloat()` 处理，那么将返回原始值
  - `number` 修饰符会在输入框有 `type="number"` 时自动启用

- **`.trim`**：默认自动去除用户输入内容中两端的空格

### 组件上的`v-model`

默认情况下，组件上的 `v-model` 使用 `modelValue` 作为 prop 和 `update:modelValue` 作为事件

## 生命周期钩子

Vue 会自动将回调函数注册到当前正被初始化的组件实例上。因此生命周期钩子应当在组件初始化时被**同步**注册。[ API查询](https://cn.vuejs.org/api/composition-api-lifecycle.html#composition-api-lifecycle-hooks)

![组件生命周期图示](https://cn.vuejs.org/assets/lifecycle.16e4c08e.png)

### onMounted()

注册一个回调函数，在组件**挂载完成后**执行

- 这个钩子通常用于执行需要访问组件所渲染的 DOM 树相关的副作用，或是在服务端渲染应用中用于确保 DOM 相关代码仅在客户端执行。

- 这个钩子在服务器端渲染期间不会被调用。

### onUpdated()

注册一个回调函数，在组件**因为响应式状态变更而更新其 DOM 树之后**调用。

- 父组件的更新钩子将在其子组件的更新钩子之后调用。
- 这个钩子会在组件的任意 DOM 更新后被调用。

### onUnmounted()

注册一个回调函数，在组件实例**被卸载之后**调用

- 可以在这个钩子中手动清理一些副作用，例如计时器、DOM 事件监听器或者与服务器的连接。

- 这个钩子在服务器端渲染期间不会被调用。

### onBeforeMount()

注册一个钩子，在组件**被挂载之前**被调用。

- 当这个钩子被调用时，组件已经完成了其响应式状态的设置，但还没有创建 DOM 节点。它即将首次执行 DOM 渲染过程

- 这个钩子在服务器端渲染期间不会被调用。

### onBeforeUpdate()

注册一个钩子，在组件**即将因为响应式状态变更而更新其 DOM 树之前**调用。

- 这个钩子可以用来在 Vue 更新 DOM 之前访问 DOM 状态。在这个钩子中更改状态也是安全的。
- 这个钩子在服务器端渲染期间不会被调用。

### onBeforeUnmount()

注册一个钩子，在组件实例**被卸载之前**调用。

- 当这个钩子被调用时，组件实例依然还保有全部的功能。

- 这个钩子在服务器端渲染期间不会被调用。

### onErrorCaptured()

注册一个钩子，在**捕获了后代组件传递的错误时**调用。

## 侦听器

使用 `watch` 函数在**每次响应式状态发生变化时**触发回调函数

- `watch` 的第一个参数可以是不同形式的“数据源”：它可以是一个 **ref (包括计算属性)**、一个**响应式对象**、一个 **getter 函数**、或**多个数据源组成的数组**
- 不能直接侦听响应式对象的属性值，需要用一个返回该属性的 getter 函数

```vue
<script setup>
    const x = ref(0)
    const y = ref(0)
    // 单个 ref
    watch(x, (newX) => {
      console.log(`x is ${newX}`)
    })
    // getter 函数
    watch(
      () => x.value + y.value,
      (sum) => {
        console.log(`sum of x + y is: ${sum}`)
      }
    )
    // 多个来源组成的数组
    watch([x, () => y.value], ([newX, newY]) => {
      console.log(`x is ${newX} and y is ${newY}`)
    })
    
    const obj = reactive({ count: 0 })
    // 提供一个 getter 函数
    watch(
      () => obj.count,
      (count) => {
        console.log(`count is: ${count}`)
      }
    )
</script>
```

### 深层侦听器

直接给 `watch()` 传入一个响应式对象，会隐式地创建一个深层侦听器——该回调函数在所有嵌套的变更时都会被触发

一个返回响应式对象的 getter 函数，只有在返回不同的对象时，才会触发回调，显式地加上 `deep` 选项，强制转成深层侦听器

```vue
<script setup>
    watch(
      () => state.someObject,
      () => {
        // 仅当 state.someObject 被替换时触发
      }
    )
    
    watch(
      () => state.someObject,
      (newValue, oldValue) => {
        // 注意：`newValue` 此处和 `oldValue` 是相等的
        // *除非* state.someObject 被整个替换了
      },
      { deep: true }
    )
</script>
```

### `watchEffect()`

`watchEffect()` 会**立即执行一遍回调函数**，如果这时函数产生了副作用，Vue 会自动追踪副作用的依赖关系，自动分析出响应源

直接写回调函数，自动追踪回调函数中涉及到的响应式变量

### 回调触发时机

默认情况下，用户创建的侦听器回调，都会在 Vue 组件更新**之前**被调用 -> 在侦听器回调中访问的 DOM 将是被 Vue 更新**之前**的状态。

- 如果想在侦听器回调中能访问被 Vue 更新**之后**的 DOM，需要指明 `flush: 'post'` 选项
- 后置刷新的 `watchEffect()` 有个更方便的别名 `watchPostEffect()`

```vue
<script set up>
    watch(source, callback, {
      flush: 'post'
    })

    watchEffect(callback, {
      flush: 'post'
    })
    watchPostEffect(() => {
      /* 在 Vue 更新后执行 */
    })
</script>
```

### 停止侦听器

用同步语句创建的侦听器，会自动绑定到宿主组件实例上，并且会在宿主组件卸载时自动停止。如果用异步回调创建一个侦听器，那么它不会绑定到当前组件上，必须手动停止。

- 要手动停止一个侦听器，调用 `watch` 或 `watchEffect` 返回的函数
- 尽可能选择同步创建。如果需要等待一些异步数据，可以使用条件式的侦听逻辑

```vue
<script setup>
    const unwatch = watchEffect(() => {})

    // ...当该侦听器不再需要时
    unwatch()
    
    
    // 需要异步请求得到的数据
    const data = ref(null)

    watchEffect(() => {
      if (data.value) {
        // 数据加载后执行某些操作...
      }
    })
</script>
```

## 模板引用

使用特殊的 `ref` attribute直接访问底层 DOM 元素

- `ref` 是一个特殊的 attribute，允许在一个特定的 DOM 元素或子组件实例被挂载后，获得对它的直接引用

### 访问模板引用

声明一个同名的 ref

```vue
<script setup>
import { ref, onMounted } from 'vue'

// 声明一个 ref 来存放该元素的引用
// 必须和模板里的 ref 同名
const input = ref(null)

onMounted(() => {
  input.value.focus()
})
</script>

<template>
  <input ref="input" />
</template>
```

- 只可以**在组件挂载后**才能访问模板引用。如果想在模板中的表达式上访问 `ref`，在初次渲染时会是 `null`

### `v-for` 中的模板引用

当在 `v-for` 中使用模板引用时，对应的 ref 中包含的值是一个数组，它将在元素被挂载后包含对应整个列表的所有元素

```vue
<script setup>
import { ref, onMounted } from 'vue'

const list = ref([1, 2, 3])

const itemRefs = ref([])

onMounted(() => console.log(itemRefs.value))  //此时itemRefs = ref([1,2,3])
</script>

<template>
  <ul>
    <li v-for="item in list" ref="itemRefs">
      {{ item }}
    </li>
  </ul>
</template>
```

### 函数模板引用

动态的 `:ref` 可以绑定为一个函数，会在每次组件更新时都被调用。该函数会收到元素引用作为其第一个参数。当绑定的元素被卸载时，函数也会**被调用一次**，此时的元素参数会是 `null`。

### 组件上的 ref

模板引用可以被用在一个子组件上。这种情况下引用中获得的值是组件实例。

## 组件基础

`.vue`文件夹默认导出暴露

- 每当使用一个组件，就创建了一个新的**实例**，因此这些组件状态不相同
- 单文件组件中，推荐为子组件使用 `PascalCase` 的标签名

### 传递 props

Props 是一种特别的 attributes，在子组件上使用`defineProps`声明注册，声明的 props 会自动暴露给模板。

- `defineProps` 是一个**仅 `<script setup>` 中可用**的编译宏命令，并不需要显式地导入
- `defineProps` 会返回一个对象，其中包含了可以传递给组件的所有 props

### 监听事件

子组件使用`defineEmits`声明需要抛出的事件

- 子组件通过调用内置的 `$emit` 方法，通过传入事件名称来抛出一个事件
- 父组件可以通过 `v-on` 或 `@` 来选择性地监听子组件上抛的事件，就像监听原生 DOM 事件那样

### 通过插槽来分配内容

和 HTML 元素一样向组件中传递内容

- 子组件使用 `<slot>` 作为一个占位符，父组件传递进来的内容就会渲染在这里

### 动态组件

有些场景会需要在两个组件间来回切换，通过 Vue 的 `<component>` 元素和特殊的 `is` attribute 实现。

- `<component>`元素要渲染的实际组件由 `is` prop 决定

- 被传给 `:is` 的值可以是以下几种：

  - 被注册的组件名

  - 导入的组件对象

- 当使用 `<component :is="...">` 来在多个组件间作切换时，被切换掉的组件会被卸载。我们可以通过 `KeepAlive>` 组件强制被切换掉的组件仍然保持“存活”的状态

### 元素位置限制

某些 HTML 元素对于放在其中的元素类型有限制，例如 `<ul>`，`<ol>`，`<table>` 和 `<select>`，相应的，某些元素仅在放置于特定元素中时才会显示，例如 `<li>`，`<tr>` 和 `<option>`，导致在使用带有此类限制元素的组件时出现问题

可以使用特殊的 `is` attribute作为一种解决方案

```vue
<template>
	<table>
      <blog-post-row></blog-post-row>  <!-- 自定义的组件 <blog-post-row> 将作为无效的内容被忽略 -->
    </table>

	<table>
      <tr is="vue:blog-post-row"></tr>
    </table>
</template>
```

# 深入组件

## 注册

一个 Vue 组件在使用前需要先被“注册”，这样 Vue 才能在渲染模板时找到其对应的实现

### 全局注册

使用 `app.component()` 方法，让组件在当前 Vue 应用中全局可用，全局注册的组件可以在此应用的任意组件的模板中使用

### 局部注册

局部注册的组件需要在使用它的父组件中显式导入，并且只能在该父组件中使用，**局部注册的组件在后代组件中并不可用**。

在使用 `<script setup>` 的单文件组件中，导入的组件可以直接在模板中使用，无需注册

### 组件命名格式

`<PascalCase />` 在模板中更明显地表明了这是一个 Vue 组件，而不是原生 HTML 元素。同时也能够将 Vue 组件和自定义元素 (web components) 区分开来

Vue 支持将模板中使用 kebab-case 的标签解析为使用 PascalCase 注册的组件。这意味着一个以 `MyComponent` 为名注册的组件，在模板中可以通过 `<MyComponent>` 或 `<my-component>` 引用。

## Props

### 声明

除了使用字符串数组来声明 prop 外，还可以使用对象的形式。对于以对象形式声明中的每个属性，key 是 prop 的名称，而值则是该 prop 预期类型的构造函数

### 传递

- 如果一个 prop 的名字很长，应使用 camelCase 形式
- 向子组件传递 props 时，通常会将其写为 kebab-case 形式

- 如果你想要将一个对象的所有属性都当作 props 传入，可以使用没有参数的 `v-bind`（即只使用 `v-bind` 而非 `:prop-name`）

### 单向数据流

所有的 props 都遵循着**单向绑定**原则，props 因父组件的更新而变化，自然地将新的状态向下流往子组件，而不会逆向传递。**不应该**在子组件中去更改一个 prop。

更改prop两种场景及解决方案

1. **prop 被用于传入初始值；而子组件想在之后将其作为一个局部数据属性**。在这种情况下，最好是新定义一个局部数据属性，从 props 上获取初始值即可

```VUE
<script setup>
    const props = defineProps(['initialCounter'])

    // 计数器只是将 props.initialCounter 作为初始值
    // 像下面这样做就使 prop 和后续更新无关了
    const counter = ref(props.initialCounter)
</script>
```

2. **需要对传入的 prop 值做进一步的转换**。在这种情况中，最好是基于该 prop 值定义一个计算属性

```vue
<script setup>
    const props = defineProps(['size'])

    // 该 prop 变更时计算属性也会自动更新
    const normalizedSize = computed(() => props.size.trim().toLowerCase())
</script>
```

当对象或数组作为 props 被传入时，虽然子组件无法更改 props 绑定，但仍然**可以**更改对象或数组内部的值

### 校验

Vue 组件可以更细致地声明对传入的 props 的校验要求。要声明对 props 的校验，可以向 `defineProps()` 宏提供一个带有 props 校验选项的对象

```vue
<script setup>
    defineProps({
      // 基础类型检查
      // （给出 `null` 和 `undefined` 值则会跳过任何类型检查）
      propA: Number,
      // 多种可能的类型
      propB: [String, Number],
      // 必传，且为 String 类型
      propC: {
        type: String,
        required: true
      },
      // Number 类型的默认值
      propD: {
        type: Number,
        default: 100
      },
      // 对象类型的默认值
      propE: {
        type: Object,
        // 对象或数组的默认值
        // 必须从一个工厂函数返回。
        // 该函数接收组件所接收到的原始 prop 作为参数。
        default(rawProps) {
          return { message: 'hello' }
        }
      },
      // 自定义类型校验函数
      propF: {
        validator(value) {
          // The value must match one of these strings
          return ['success', 'warning', 'danger'].includes(value)
        }
      },
      // 函数类型的默认值
      propG: {
        type: Function,
        // 不像对象或数组的默认，这不是一个工厂函数。这会是一个用来作为默认值的函数
        default() {
          return 'Default function'
        }
      }
    })
</script>
```

- 校验选项中的 `type` 可以是下列这些原生构造函数：
  - `String`
  - `Number`
  - `Boolean`
  - `Array`
  - `Object`
  - `Date`
  - `Function`
  - `Symbol`
- `type` 也可以是自定义的类或构造函数，Vue 将会通过 `instanceof` 来检查类型是否匹配
- 声明为 `Boolean` 类型的 props 有特别的类型转换规则

```vue
<!-- in MyComponent -->
<script setup>
    defineProps({
      disabled: Boolean
    })
</script>

<!-- in father component -->
<template>
	<!-- 等同于传入 :disabled="true" -->
    <MyComponent disabled />

    <!-- 等同于传入 :disabled="false" -->
    <MyComponent />
</template>
```

### 补充细节

- 所有 prop 默认都是可选的，除非声明了 `required: true`。
- 除 `Boolean` 外的未传递的可选 prop 将会有一个默认值 `undefined`。
- `Boolean` 类型的未传递 prop 将被转换为 `false`。这可以通过为它设置 `default` 来更改——例如：设置为 `default: undefined` 将与非布尔类型的 prop 的行为保持一致。
- 如果声明了 `default` 值，那么在 prop 的值被解析为 `undefined` 时，无论 prop 是未被传递还是显式指明的 `undefined`，都会改为 `default` 值。

## 组件事件

在模板中推荐使用 kebab-case 形式来编写监听器

### 事件参数

所有传入 `$emit()` 的额外参数都会被直接传向监听器。

在父组件中监听事件，可以先简单写一个内联的箭头函数作为监听器，此函数会接收到事件附带的参数，也可以先定义方法，然后调用

### 事件校验

要为事件添加校验，那么事件可以被赋值为一个函数，接受的参数就是抛出事件时传入 `emit` 的内容，返回一个布尔值来表明事件是否合法

```vue
<script setup>
const emit = defineEmits({
  // 没有校验
  click: null,

  // 校验 submit 事件
  submit: ({ email, password }) => {
    if (email && password) {
      return true
    } else {
      console.warn('Invalid submit event payload!')
      return false
    }
  }
})

function submitForm(email, password) {
  emit('submit', { email, password })
}
</script>
```

### 配合 `v-model` 使用

1. 直接使用，子组件内部需要做两件事：

   - 将内部原生 `input` 元素的 `value` attribute 绑定到 `modelValue` prop
   - 输入新的值时在 `input` 元素上触发 `update:modelValue` 事件

   ```vue
   <!-- CustomInput.vue -->
   <script setup>
       defineProps(['modelValue'])
       defineEmits(['update:modelValue'])
   </script>
   
   <template>
     	<input
           :value="modelValue"
           @input="$emit('update:modelValue', $event.target.value)"
       />
   </template>
   
   <!-- App.vue -->
   <template>
   	<CustomInput v-model="searchText" />
   </template>
   ```

2. 使用一个可写的，同时具有 getter 和 setter 的计算属性。`get` 方法需返回 `modelValue` prop，而 `set` 方法需触发相应的事件

   ```vue
   <!-- CustomInput.vue -->
   <script setup>
   import { computed } from 'vue'
   
   const props = defineProps(['modelValue'])
   const emit = defineEmits(['update:modelValue'])
   
   const value = computed({
     get() {
       return props.modelValue
     },
     set(value) {
       emit('update:modelValue', value)
     }
   })
   </script>
   
   <template>
     <input v-model="value" />
   </template>
   ```

- ### `v-model` 的参数

通过给 `v-model` 指定一个参数来更改props和事件

```vue
<!-- MyComponent.vue -->
<script setup>
defineProps(['title'])
defineEmits(['update:title'])
</script>

<template>
  <input
    type="text"
    :value="title"
    @input="$emit('update:title', $event.target.value)"
  />
</template>

<!-- App.vue -->
<MyComponent v-model:title="bookTitle" />
```

- 可以在一个组件上创建多个 `v-model` 双向绑定，每一个 `v-model` 都会同步不同的 prop

### 自定义修饰符 `capitalize`

对于又有参数又有修饰符的 `v-model` 绑定，生成的 prop 名将是 `arg + "Modifiers"`

```vue
<script setup>
const props = defineProps([
    'modelValue',
  	'modelModifiers'
])

const emit = defineEmits(['update:modelValue'])

function emitValue(e) {
  let value = e.target.value
  if (props.modelModifiers.capitalize) {
    value = value.charAt(0).toUpperCase() + value.slice(1)
  }
  emit('update:modelValue', value)
}
</script>

<template>
  <input type="text" :value="modelValue" @input="emitValue" />
</template>
```

## 继承

 `class`、`style` 和 `id是`透传attribute

- 当一个子组件以单个元素为根作渲染时，透传的 attribute 会自动被添加到根元素上

- **不想要**一个组件自动地继承 attribute，可以在子组件选项中设置 `inheritAttrs: false`

```vue
<script>
    // 使用普通的 <script> 来声明选项
    export default {
      inheritAttrs: false
    }
</script>

<script setup>
    // ...setup 部分逻辑
</script>
```

- 透传进来的 attribute 可以在模板的表达式中直接用 **`$attrs` 访问**到，`$attrs` 对象包含了除组件所声明的 `props` 和 `emits` 之外的所有其他 attribute，例如 `class`，`style`，`v-on` 监听器等等
  - 和 props 有所不同，透传 attributes 在 JavaScript 中**保留了它们原始的大小写**，所以像 `foo-bar` 这样的一个 attribute 需要通过 `$attrs['foo-bar']` 来访问。
  - 像 `@click` 这样的一个 **`v-on` 事件监听器将在此对象下被暴露为一个函数** `$attrs.onClick`。

- 和单根节点组件有所不同，有着多个根节点的组件**没有**自动 attribute 透传行为，需要使用 `$attrs` 被**显式绑定**

- 可以在 `<script setup>` 中使用 **`useAttrs()` API** 来访问一个组件的所有透传 attribute

## 插槽

插槽内容可以是任意合法的模板内容，不局限于文本

- 插槽内容**可以访问**到父组件的数据作用域，**无法访问**子组件的数据

- 写在 `<slot>` 标签之间来作为默认内容，父组件没有提供任何插槽内容时在子组件内渲染

### 具名插槽

`<slot>` 元素可以有一个特殊的 attribute `name`，用来给各个插槽分配唯一的 ID，以确定每一处要渲染的内容

- 要为具名插槽传入内容，我们需要使用一个含 `v-slot` 指令的 `<template>` 元素，并将目标插槽的名字传给该指令，`v-slot` 有对应的简写 `#`

```vue
<template>
	<BaseLayout>
      <template v-slot:header>
        <!-- header 插槽的内容放这里 -->
      </template>
    </BaseLayout>
</template>
```

- 当一个组件同时接收默认插槽和具名插槽时，所有位于顶级的非 `<template>` 节点都被隐式地视为默认插槽的内容
- 动态指令参数]在 `v-slot` 上依然有效

### 作用域插槽

向一个插槽的出口上传递 attributes，即可在父组件访问子组件状态，也可以在 `v-slot` 中使用解构

- 默认插槽：通过子组件标签上的 **`v-slot` 指令**，直接接收到了一个插槽 props 对象

```vue
<!-- <MyComponent> 的模板 -->
<template>
	<div>
      <slot :text="greetingMessage" :count="1"></slot>
    </div>
</template>

<!-- App.vue -->
<template>
	<MyComponent v-slot="slotProps">
      {{ slotProps.text }} {{ slotProps.count }}
    </MyComponent>
</template>
```

- 具名插槽：插槽 props 可以作为 `v-slot` 指令的值被访问到：`v-slot:name="slotProps"`

```vue
<!-- <MyComponent> 的模板 -->
<template>
	<div>
      <slot name="header" message="hello"></slot>
    </div>
</template>

<!-- App.vue -->
<template>
	<MyComponent>
      <template #header="headerProps">
        {{ headerProps }}
      </template>
    </MyComponent>
</template>
```

- 如果混用了具名插槽与默认插槽，则需要为默认插槽使用显式的 `<template>` 标签

## 依赖注入

用于解决props逐级透传问题

## ![Provide/inject 模式](https://cn.vuejs.org/assets/provide-inject.3e0505e4.png)

一个父组件相对于其所有的后代组件，会作为**依赖提供者**。任何后代的组件树，无论层级有多深，都可以**注入**由父组件提供给整条链路的依赖。

### Provide (提供)

为组件后代提供数据，需要使用到 `provide()`函数

- `provide()` 函数接收两个参数。第一个参数被称为**注入名**，可以是一个字符串或是一个 `Symbol`。第二个参数是提供的值，值可以是任意类型，包括响应式的状态。

- 后代组件会用注入名来查找期望注入的值。一个组件可以多次调用 `provide()`，使用不同的注入名，注入不同的依赖值
- 可以在整个应用层面提供依赖

### Inject (注入)

注入上层组件提供的数据，需使用 `inject()`]函数

- 如果提供的值是一个 ref，注入进来的会是该 ref 对象，而**不会**自动解包为其内部的值。

- 如果在注入一个值时不要求必须有提供者，那么我们应该声明一个默认值。默认值可能需要通过调用一个函数或初始化一个类来取得，可以使用工厂函数来创建默认值

  ```vue
  <script setup>
      // 如果没有祖先组件提供 "message"
      // `value` 会是 "这是默认值"
      const value = inject('message', '这是默认值')
      const value = inject('key', () => new ExpensiveClass())
  </script>
  ```

## 和响应式数据配合使用

当提供 / 注入响应式的数据时，**建议尽可能将任何对响应式状态的变更都保持在供给方组件中**。

- 在注入方组件中更改数据时，应在供给方组件内声明并提供一个更改数据的方法函数

```vue
<!-- 在供给方组件内 -->
<script setup>
    import { provide, ref } from 'vue'

    const location = ref('North Pole')

    function updateLocation() {
      location.value = 'South Pole'
    }

    provide('location', {
      location,
      updateLocation
    })
</script>

<!-- 在注入方组件 -->
<script setup>
    import { inject } from 'vue'

    const { location, updateLocation } = inject('location')
</script>

<template>
  	<button @click="updateLocation">{{ location }}</button>
</template>
```

- 想确保提供的数据不能被注入方的组件更改，可以使用 `readonly()`]来包装提供的值

```vue
<script setup>
    import { ref, provide, readonly } from 'vue'

    const count = ref(0)
    provide('read-only-count', readonly(count))
</script>
```

# 逻辑复用

## 组合式函数

“组合式函数”(Composables) 是一个利用 Vue 的组合式 API 来封装和复用**有状态逻辑**的函数

```js
// mouse.js
import { ref, onMounted, onUnmounted } from 'vue'

// 按照惯例，组合式函数名以“use”开头
export function useMouse() {
  // 被组合式函数封装和管理的状态
  const x = ref(0)
  const y = ref(0)

  // 组合式函数可以随时更改其状态。
  function update(event) {
    x.value = event.pageX
    y.value = event.pageY
  }

  // 一个组合式函数也可以挂靠在所属组件的生命周期上
  // 来启动和卸载副作用
  onMounted(() => window.addEventListener('mousemove', update))
  onUnmounted(() => window.removeEventListener('mousemove', update))

  // 通过返回值暴露所管理的状态
  return { x, y }
}
```

- 组合式函数约定用驼峰命名法命名，并以“use”作为开头

- 组合式函数可接收 ref 参数

- 组合式函数始终返回一个包含多个 ref 的普通的非响应式对象，这样该对象在组件中被解构为 ref 之后仍可以保持响应性
- 组合式函数在 `<script setup>` 或 `setup()` 钩子中，应始终被**同步地**调用

## 自定义指令

自定义指令主要是为了重用涉及普通元素的底层 DOM 访问的逻辑

- 一个自定义指令由一个包含**类似组件生命周期钩子的对象**来定义。钩子函数会接收到指令所**绑定元素**作为其参数。

- 在 `<script setup>` 中，任何以 `v` 开头的**驼峰式命名**的变量都可以被用作一个自定义指令。

### 指令钩子

一个指令的定义对象可以提供几种钩子函数 (都是可选的)：

```vue
<script setup>
    const myDirective = {
      // 在绑定元素的 attribute 前
      // 或事件监听器应用前调用
      created(el, binding, vnode, prevVnode) {
      },
      // 在元素被插入到 DOM 前调用
      beforeMount(el, binding, vnode, prevVnode) {},
      // 在绑定元素的父组件
      // 及他自己的所有子节点都挂载完成后调用
      mounted(el, binding, vnode, prevVnode) {},
      // 绑定元素的父组件更新前调用
      beforeUpdate(el, binding, vnode, prevVnode) {},
      // 在绑定元素的父组件
      // 及他自己的所有子节点都更新后调用
      updated(el, binding, vnode, prevVnode) {},
      // 绑定元素的父组件卸载前调用
      beforeUnmount(el, binding, vnode, prevVnode) {},
      // 绑定元素的父组件卸载后调用
      unmounted(el, binding, vnode, prevVnode) {}
    }
</script>
```

- **钩子参数**：指令的钩子会传递以下几种参数，除了 `el` 外，其他参数都是**只读**的，不要更改它们。
  - `el`：指令绑定到的元素。这可以用于直接操作 DOM。
  - `binding`：一个对象，包含以下属性。
    - `value`：传递给指令的值。例如在 `v-my-directive="1 + 1"` 中，值是 `2`。
    - `oldValue`：之前的值，仅在 `beforeUpdate` 和 `updated` 中可用。无论值是否更改，它都可用。
    - `arg`：传递给指令的参数 (如果有的话)。例如在 `v-my-directive:foo` 中，参数是 `"foo"`。
    - `modifiers`：一个包含修饰符的对象 (如果有的话)。例如在 `v-my-directive.foo.bar` 中，修饰符对象是 `{ foo: true, bar: true }`。
    - `instance`：使用该指令的组件实例。
    - `dir`：指令的定义对象。
  - `vnode`：代表绑定元素的底层 VNode。
  - `prevNode`：之前的渲染中代表指令所绑定元素的 VNode。仅在 `beforeUpdate` 和 `updated` 钩子中可用。

- **不推荐**在组件上使用自定义指令









