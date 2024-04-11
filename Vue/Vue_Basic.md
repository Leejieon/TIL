# Vue 앱 인스턴스 생성 및 연결

Vue 기능을 사용할 때 가장 먼저 해야 할 일은 **HTML 코드에 Vue를 연결**하는 일

⇒ HTML 코드 중 **어떤 부분을 Vue가 관리해야 하는지** Vue가 인식하도록 설정

> Vue 앱은 하나의 HTML 요소에만 연결할 수 있다.

## 1. Vue 앱 생성하기

`Vue.createApp()` 을 통해 Vue 앱을 생성한다.

```jsx
const app = Vue.createApp();
```

## 2. HTML 연결하기

Vue 앱 생성 이후, HTML 코드 중 어떤 부분을 Vue 앱이 조정할지 설정한다.

> Vue로 HTML 요소를 제어하는 경우, 해당 요소의 **자식 요소 또한 제어할 수 있**다.

`mount()`를 호출해 해당 작업을 실행할 수 있다.

- `createApp()` 을 통해 생성한 app 객체의 메서드
- 인자로 제어하고자 하는 **CSS 선택자**가 들어간다.
- ID가 페이지의 고유한 값이므로 고유 선택자로 쓰기에 적합

```jsx
app.mount("#app");
```

# Vue 앱의 data 옵션

`createApp()` 에 **객체를 전달**해 상호 작용 기능을 사용할 수 있다.

- 객체를 통해 해당 앱의 여러 옵션을 구성할 수 있다.

## data 옵션

```jsx
const app = Vue.createApp({
  data: function () {
    return {
      key: value,
      courseGoal: "Learn Vue!",
    };
  },
});
```

- 값으로 함수를 갖는다.
- **항상 객체를 반환**한다.
- 반환된 객체에서 **Key-Value 쌍**을 설정할 수 있다.
- 위의 방식 대신 아래 방법도 사용가능하다.
  → `data() { ... }`
- data 프로퍼티에 저장된 값이 함수라는 뜻

> Data에서 반환하는 객체의 어떤 부분이든 Vue가 관리하는 HTML 코드에서 사용할 수 있다.

### {{ }} := 컨텐츠 출력 구문(Mustache)

HTML 코드에서 컨텐츠를 출력할 때, `{{ }}` 구문을 사용한다.

이 구문 사이에 반환된 **data 객체 프로퍼티를 참조**할 수 있다.

이때, 화면에 참조값이 나타나게 된다.

```html
<section id="user-goal">
  <h2>My Course Goal</h2>
  <p>{{ courseGoal }}</p>
</section>
```

⇒ 이와 같은 `{{ }}` 구문을 **Interpolation(보간법)**이라 한다.

> {{ }} 안에서 JavaScript 표현식을 실행할 수 있다.

## “v-bind”을 이용한 속성 바인딩

```jsx
const app = Vue.createApp({
  data: function () {
    return {
      vueLink: "https://vuejs.org/",
    };
  },
});
```

vueLink 프로퍼티에 입력한 링크가 HTML 코드에서 링크로 연결되도록 설정해보자.

### v-bind

> Vue에게 바인딩을 지시하는 디렉티브

- **HTML 요소의 속성**에 대한 값을 설정하도록 지시
- Vue에 어떤 속성 값을 설정해야 하는지 알려주기 위해 사용

### 사용법

`v-bind` 뒤에 콜론(:)을 입력한 뒤, 속성의 이름을 작성한다.

```jsx
<tagName v-bind:속성이름="속성값"></tagName>
```

- Vue가 HTML 요소의 속성에 대해 동적으로 값을 설정하도록 지시

```html
...
<p>Learn more <a v-bind:href="vueLink">about Vue</p>
...
```

> 정리하자면,
> HTML 태그 사이에 값을 설정 ⇒ 보간법( {{ }} )
> HTML 태그 속성에 값을 설정 ⇒ v-bind

# Vue 앱의 Methods 옵션

## 앱 객체의 methods 옵션

> 호출 및 사용자 이벤트가 발생했을 때 실행할 함수를 정의하는 용도

```jsx
const app = Vue.createApp({
	...,
	methods: {
		funcName() {
			...
		}
	}
});
```

- **JavaScript 객체**가 전달된다. → 메서드 및 함수로 만들어진 객체를 갖는다.
- 즉, 객체에서 정의하는 프로퍼티는 **_“함수”_**이다.

```jsx
const app = Vue.createApp({
	data: function() {
		return {
			courseGoal: 'Learn Vue!',
			vueLink: 'https://vuejs.org/'
		};
	},
	methods: {
		outputGoal() {
			const randomNumber = Math.random();
			...
		}
	}
})
```

## 메서드 사용하기

앱 내의 `methods` 객체에서 정의된 메서드를 HTML 코드에서 호출하기 위해 `{{ }}` 구문에 함수를 넣거나 `v-bind` 에 함수를 넣을 수 있다.

```html
<p>{{ functionName() }}</p>

<p>{{ outputGoal() }}</p>
```

## 앱 내에서 데이터 작업하기

```jsx
const app = Vue.createApp({
  data() {
    return {
      courseGoalA: "Finish the course and learn Vue!",
      courseGoalB: "Master Vue and builld amazing app!",
      vueLink: "https://vuejs.org/",
    };
  },
  methods: {
    outputGoal() {
      const randomNumber = Math.random();
      if (randomNumber < 0.5) {
        return "Learn Vue!";
      } else {
        return "Master Vue!";
      }
    },
  },
});
```

위와 같은 상황에서 `data` 에 정의된 객체를 `methods` 내에서 사용할 수 있는 방법에 대해 알아보자.

이때, `this`를 사용할 수 있다.

### this

> Vue에서 `this` 키워드는 data 내의 객체를 참조한다.

- Vue는 data 객체에서 반환하는 데이터 전체를 가져다가 데이터를 병합해 **전역 Vue 인스턴스 객체**를 만든다.
- 따라서, `this` 키워드를 사용하면 전역 Vue 인스턴스 객체에 저장된 모든 데이터에 접근할 수 있다.

```jsx
const app = Vue.createApp({
  data() {
    return {
      courseGoalA: "Finish the course and learn Vue!",
      courseGoalB: "Master Vue and builld amazing app!",
      vueLink: "https://vuejs.org/",
    };
  },
  methods: {
    outputGoal() {
      const randomNumber = Math.random();
      if (randomNumber < 0.5) {
        return this.courseGoalA;
      } else {
        return this.courseGoalB;
      }
    },
  },
});
```

# v-html 를 통한 HTML 컨텐츠 출력

위 코드에서 data 옵션 내에서 단순한 문자열이 아니라 `<h2> ... </h2>` 와 같은 HTML 코드를 출력하는 방법에 대해 알아보자.

다시말해, 출력할 데이터를 DB에서 가져왔더니 가져온 데이터가 구조화된 HTML 코드라고 가정해보자.

이 경우, 단순히 보간법을 이용하면 HTML 코드까지 그대로 문자열로 출력된다. 이때, `v-html` 디렉티브를 사용할 수 있다.

## v-html

> 태그의 속성으로 사용되며, Vue에 해당 컨텐츠는 HTML로 인식해야 한다는 것을 알리는 디렉티브

```html
<p v-html="outputGoal()"></p>
```
