# Vue의 기본 문법과 사용법 정리

> Vue 3 `<script setup>` 기준\
> 기존 Java/JSP/jQuery 개발자가 Vue 소스를 읽을 때 이해하기 쉽도록 정리

------------------------------------------------------------------------

## 1. Vue란?

Vue는 화면(UI)을 **데이터와 연결해서 관리하는 프론트엔드
프레임워크**이다.

jQuery에서는 HTML 요소를 직접 찾아서 변경하는 방식이 많다.

``` javascript
// jQuery 방식
$('.display').hide();
$('.display').show();
```

Vue에서는 DOM을 직접 조작하기보다 **상태값을 변경하고 Vue가 화면을
변경하도록 한다.**

``` javascript
// Vue 방식
visible.value = false; // 숨김
visible.value = true;  // 보임
```

``` vue
<p v-show="visible">
  보이지롱
</p>
```

핵심 흐름:

``` text
사용자 이벤트
    ↓
상태값 변경
    ↓
Vue가 변경 감지
    ↓
화면 자동 변경
```

------------------------------------------------------------------------

## 2. Vue 파일의 기본 구조

하나의 `.vue` 파일은 보통 다음 세 영역으로 구성된다.

``` vue
<script setup>
// JavaScript 영역
// 변수, 상태값, 함수, API 호출 등을 작성
</script>

<template>
  <!-- 실제 화면 HTML 작성 -->
</template>

<style scoped>
/* 현재 컴포넌트의 CSS */
</style>
```

예제:

``` vue
<script setup>
import { ref } from 'vue'

// 반응형 변수
const name = ref('홍길동')

// 함수 정의
const changeName = () => {
  name.value = '김철수'
}
</script>

<template>
  <!-- 변수 출력 -->
  <p>{{ name }}</p>

  <!-- 클릭 이벤트 -->
  <button @click="changeName">
    이름 변경
  </button>
</template>
```

------------------------------------------------------------------------

## 3. `ref()` - 반응형 변수

`ref()`는 값이 변경되었을 때 Vue가 그 변경을 감지할 수 있도록 만든다.

문자열, 숫자, Boolean, 객체, 배열 등을 모두 담을 수 있다.

``` javascript
import { ref } from 'vue'

// 문자열
const name = ref('홍길동')

// 숫자
const count = ref(0)

// Boolean
const visible = ref(true)

// 객체
const form = ref({
  title: '',
  content: ''
})

// 배열
const users = ref([])
```

### script에서 접근

`ref`의 실제 값에 접근하려면 `.value`를 사용한다.

``` javascript
name.value = '김철수'
count.value = 10

form.value.title = '공지사항'
```

### template에서 접근

`template`에서는 `.value`를 사용하지 않는다.

``` vue
<template>
  <p>{{ name }}</p>
  <p>{{ form.title }}</p>
</template>
```

정리:

``` text
<script>
form.value.title

<template>
form.title
```

------------------------------------------------------------------------

## 4. `reactive()` - 객체를 반응형으로 관리

`reactive()`는 객체 자체를 반응형으로 만든다.

``` javascript
import { reactive } from 'vue'

// 여러 데이터를 하나의 객체로 관리
const form = reactive({
  title: '',
  content: '',
  writer: ''
})
```

`reactive`는 `.value`가 필요 없다.

``` javascript
form.title = '공지사항'
form.content = '공지 내용'
form.writer = '홍길동'
```

### ref 객체와 reactive 객체의 차이

``` javascript
// ref
const form1 = ref({
  title: ''
})

form1.value.title = '공지사항'


// reactive
const form2 = reactive({
  title: ''
})

form2.title = '공지사항'
```

핵심:

``` text
ref({...})
→ form.value.title

reactive({...})
→ form.title
```

`reactive()` 자체가 서버로 데이터를 전달하는 기능은 아니다.\
여러 값을 객체로 묶어서 **반응형으로 관리**하는 기능이다.

API 호출 시 이 객체를 전달하면 여러 데이터를 한 번에 서버로 보낼 수
있다.

``` javascript
await axios.post('/api/board/save', form)
```

------------------------------------------------------------------------

## 5. `{{ }}` - 화면에 값 출력

JavaScript의 값을 template에 출력한다.

``` vue
<script setup>
import { ref } from 'vue'

const name = ref('홍길동')

const user = ref({
  name: '김철수',
  age: 30
})
</script>

<template>
  <!-- 단일 값 출력 -->
  <p>{{ name }}</p>

  <!-- 객체의 속성 출력 -->
  <p>{{ user.name }}</p>

  <!-- 간단한 표현식도 가능 -->
  <p>{{ user.age + 1 }}</p>
</template>
```

------------------------------------------------------------------------

## 6. `v-model` - 입력값과 데이터 연결

`v-model`은 input 등의 입력값과 Vue 데이터를 양방향으로 연결한다.

``` vue
<script setup>
import { reactive } from 'vue'

const form = reactive({
  title: '',
  content: ''
})
</script>

<template>
  <!--
    사용자가 입력
        ↓
    form.title 자동 변경
  -->
  <input v-model="form.title">

  <textarea v-model="form.content"></textarea>

  <!-- 입력한 값 확인 -->
  <p>{{ form.title }}</p>
  <p>{{ form.content }}</p>
</template>
```

흐름:

``` text
사용자가 input에 입력
        ↓
      v-model
        ↓
form.title 변경
        ↓
화면 자동 반영
```

------------------------------------------------------------------------

## 7. `v-if / v-else` - 조건에 따른 화면 생성/제거

조건이 `true`일 때만 DOM을 생성한다.

``` vue
<script setup>
import { ref } from 'vue'

const loginYn = ref(true)
</script>

<template>
  <!-- true이면 생성 -->
  <div v-if="loginYn">
    로그인 상태
  </div>

  <!-- false이면 생성 -->
  <div v-else>
    로그인이 필요합니다.
  </div>
</template>
```

여러 조건:

``` vue
<div v-if="score >= 90">
  A
</div>

<div v-else-if="score >= 80">
  B
</div>

<div v-else>
  C
</div>
```

------------------------------------------------------------------------

## 8. `v-show` - show / hide

`v-show`는 조건값에 따라 요소를 보여주거나 숨긴다.

``` vue
<script setup>
import { ref } from 'vue'

// 처음에는 보임
const visible = ref(true)

const hide = () => {
  // false가 되면 v-show가 display:none 처리
  visible.value = false
}

const show = () => {
  // true가 되면 다시 표시
  visible.value = true
}
</script>

<template>
  <!--
    visible = true  → 보임
    visible = false → display: none
  -->
  <p v-show="visible">
    보이지롱
  </p>

  <button @click="show">
    보이기
  </button>

  <button @click="hide">
    숨기기
  </button>
</template>
```

### jQuery와 비교

``` javascript
// jQuery
$('.display').hide();
```

Vue:

``` javascript
visible.value = false
```

``` vue
<p v-show="visible">
  내용
</p>
```

차이:

``` text
jQuery
이벤트 → DOM 직접 조작 → hide()

Vue
이벤트 → 상태값 변경 → Vue가 DOM 처리
```

### v-if와 v-show 차이

``` text
v-if
→ 조건이 false이면 DOM 자체를 제거

v-show
→ DOM은 그대로 존재
→ display: none으로 숨김
```

------------------------------------------------------------------------

## 9. `v-for` - 반복문

배열의 데이터를 반복해서 화면에 출력한다.

``` vue
<script setup>
import { ref } from 'vue'

const users = ref([
  { id: 1, name: '홍길동' },
  { id: 2, name: '김철수' },
  { id: 3, name: '이영희' }
])
</script>

<template>
  <div
    v-for="user in users"
    :key="user.id"
  >
    {{ user.name }}
  </div>
</template>
```

여기서:

``` vue
v-for="user in users"
```

의 의미:

``` text
users 배열에서
데이터를 하나씩 꺼내서
현재 데이터를 user라고 부르겠다.
```

`user`는 미리 선언할 필요가 없다.\
`v-for` 안에서 만들어지는 임시 변수이다.

Java의 향상된 for문과 비슷하다.

``` java
for (User user : users) {
    System.out.println(user.getName());
}
```

Vue:

``` vue
<div v-for="user in users">
  {{ user.name }}
</div>
```

`user`라는 이름은 직접 정할 수 있다.

``` vue
<div v-for="abc in users">
  {{ abc.name }}
</div>
```

------------------------------------------------------------------------

## 10. `:key` - 반복 데이터의 고유 식별값

`v-for`와 함께 자주 사용한다.

``` vue
<div
  v-for="abc in users"
  :key="abc.id"
>
  {{ abc.name }}
</div>
```

`key`는 Vue가 반복된 각각의 요소를 구분하기 위한 고유 식별값이다.

데이터:

``` javascript
const users = ref([
  { id: 1, name: '홍길동' },
  { id: 2, name: '김철수' },
  { id: 3, name: '이영희' }
])
```

Vue 입장에서는:

``` text
key = 1 → 홍길동
key = 2 → 김철수
key = 3 → 이영희
```

김철수가 삭제되면:

``` text
기존               변경 후

key=1 홍길동   →    key=1 홍길동
key=2 김철수   →    삭제
key=3 이영희   →    key=3 이영희
```

즉 `:key="abc.id"`는:

``` text
"Vue야, 이 데이터는 abc.id를 고유번호로 사용해서 구분해."
```

라는 의미로 이해하면 된다.

------------------------------------------------------------------------

## 11. `:` - `v-bind` 속성 바인딩

`:`는 `v-bind:`의 축약형이다.

``` vue
<script setup>
import { ref } from 'vue'

const disabled = ref(true)
</script>

<template>
  <!-- disabled 변수의 값을 HTML 속성에 연결 -->
  <button :disabled="disabled">
    저장
  </button>
</template>
```

원래 문법:

``` vue
<button v-bind:disabled="disabled">
```

축약:

``` vue
<button :disabled="disabled">
```

컴포넌트에서도 많이 사용한다.

``` vue
<GridWrapper
  :column-defs="columnDefs"
  :pagination="false"
/>
```

의미:

``` text
:column-defs="columnDefs"
→ columnDefs 데이터를 컴포넌트에 전달

:pagination="false"
→ pagination 속성에 false 전달
```

------------------------------------------------------------------------

## 12. `@` - 이벤트

`@`는 `v-on:`의 축약형이다.

``` vue
<script setup>
const save = () => {
  console.log('저장')
}
</script>

<template>
  <!-- 버튼 클릭 시 save 함수 실행 -->
  <button @click="save">
    저장
  </button>
</template>
```

원래 문법:

``` vue
<button v-on:click="save">
```

축약:

``` vue
<button @click="save">
```

자주 보는 이벤트:

``` vue
<!-- 클릭 -->
<button @click="save">

<!-- 값 변경 -->
<select @change="changeValue">

<!-- 입력 -->
<input @input="inputValue">

<!-- submit -->
<form @submit="save">
```

중요:

``` javascript
const save = () => {
  console.log('저장')
}
```

이 코드만 있으면 **함수를 정의한 것뿐이며 실행된 것이 아니다.**

``` vue
<button @click="save">
```

처럼 이벤트와 연결하거나 다른 함수에서 `save()`를 호출해야 실제로
실행된다.

------------------------------------------------------------------------

## 13. 이벤트 → 상태 변경 → 화면 변경

Vue에서 매우 중요한 흐름이다.

``` vue
<script setup>
import { ref } from 'vue'

const visible = ref(true)

const hide = () => {
  // 이벤트가 발생했을 때 상태값 변경
  visible.value = false
}
</script>

<template>
  <!-- 상태값에 따라 화면 표시 여부 결정 -->
  <p v-show="visible">
    내용
  </p>

  <!-- 클릭 이벤트 -->
  <button @click="hide">
    숨기기
  </button>
</template>
```

흐름:

``` text
버튼 클릭
   ↓
@click 이벤트
   ↓
hide() 실행
   ↓
visible.value = false
   ↓
v-show가 변경 감지
   ↓
display: none
```

------------------------------------------------------------------------

## 14. Props - 부모 → 자식 데이터 전달

부모 컴포넌트에서 자식 컴포넌트로 데이터를 전달할 때 사용한다.

부모:

``` vue
<UserInfo
  :name="userName"
  :age="30"
/>
```

자식 `UserInfo.vue`:

``` vue
<script setup>

// 부모에게 받을 데이터 정의
const props = defineProps({
  name: String,
  age: Number
})
</script>

<template>
  <p>{{ name }}</p>
  <p>{{ age }}</p>
</template>
```

흐름:

``` text
부모 컴포넌트
     ↓
    Props
     ↓
자식 컴포넌트
```

------------------------------------------------------------------------

## 15. `emit` - 자식 → 부모 이벤트 전달

자식 컴포넌트에서 부모에게 이벤트를 전달할 때 사용한다.

자식:

``` vue
<script setup>

// 발생시킬 이벤트 정의
const emit = defineEmits(['save'])

const saveData = () => {

  // 부모에게 save 이벤트 발생
  emit('save')
}
</script>

<template>
  <button @click="saveData">
    저장
  </button>
</template>
```

부모:

``` vue
<MyComponent
  @save="handleSave"
/>
```

흐름:

``` text
자식 버튼 클릭
     ↓
saveData()
     ↓
emit('save')
     ↓
부모 @save
     ↓
handleSave()
```

------------------------------------------------------------------------

## 16. `computed()` - 계산된 값

기존 상태값을 이용해 계산된 값을 만들 때 사용한다.

``` vue
<script setup>
import { ref, computed } from 'vue'

const price = ref(10000)
const count = ref(3)

// price 또는 count가 변경되면 자동 재계산
const totalPrice = computed(() => {
  return price.value * count.value
})
</script>

<template>
  <p>가격 : {{ price }}</p>
  <p>수량 : {{ count }}</p>
  <p>총 가격 : {{ totalPrice }}</p>
</template>
```

------------------------------------------------------------------------

## 17. `watch()` - 값 변경 감시

특정 값이 변경됐을 때 로직을 실행한다.

``` javascript
import { ref, watch } from 'vue'

const selectedCity = ref('서울')

// selectedCity 값이 변경되는지 감시
watch(
  selectedCity,

  (newValue, oldValue) => {

    // 변경 후 값
    console.log('새로운 값:', newValue)

    // 변경 전 값
    console.log('기존 값:', oldValue)
  }
)
```

실무 예:

``` text
시/도 변경
   ↓
watch 감지
   ↓
구/군 목록 다시 조회
```

------------------------------------------------------------------------

## 18. `onMounted()` - 화면 최초 실행

컴포넌트가 화면에 처음 표시된 후 실행된다.

``` vue
<script setup>
import { onMounted } from 'vue'

const search = () => {
  console.log('데이터 조회')
}

// 화면 최초 진입 시 실행
onMounted(() => {
  search()
})
</script>
```

흐름:

``` text
화면 접속
   ↓
컴포넌트 생성
   ↓
onMounted()
   ↓
search()
   ↓
초기 데이터 조회
```

화면을 열자마자 자동 조회되는 코드라면 `onMounted()`를 확인해본다.

------------------------------------------------------------------------

## 19. Axios - 서버 호출

Axios는 Vue 자체 문법이 아니라 HTTP 통신을 위한 라이브러리이다.

``` javascript
const response = await axios.get('/api/board/1')
```

주석으로 풀면:

``` javascript
// axios
// → HTTP 통신 라이브러리

// get()
// → GET 방식으로 서버 요청

// '/api/board/1'
// → 호출할 서버 API 주소

// await
// → 서버 응답을 기다림

// response
// → 서버가 보내준 응답을 저장
const response = await axios.get('/api/board/1')
```

서버에서 다음 데이터를 반환했다고 가정:

``` json
{
  "title": "공지사항",
  "content": "공지 내용입니다.",
  "writer": "홍길동"
}
```

Vue에서는:

``` javascript
const response = await axios.get('/api/board/1')

// 서버가 보내준 실제 데이터
console.log(response.data)

// 각각 접근
console.log(response.data.title)
console.log(response.data.content)
```

------------------------------------------------------------------------

## 20. GET / POST 예제

### GET - 데이터 조회

``` javascript
const search = async () => {

  // 서버에 GET 요청
  const response = await axios.get('/api/board')

  // 서버 응답 확인
  console.log(response.data)
}
```

### POST - 데이터 전달

``` javascript
const form = reactive({
  title: '',
  content: '',
  writer: ''
})

const save = async () => {

  // form 객체 전체를 서버에 전달
  await axios.post(
    '/api/board/save',
    form
  )
}
```

전달되는 데이터의 형태:

``` json
{
  "title": "공지사항",
  "content": "공지 내용",
  "writer": "홍길동"
}
```

------------------------------------------------------------------------

## 21. Vue → Spring 백엔드 흐름

Vue 소스를 분석할 때 중요한 흐름이다.

Vue:

``` javascript
const response = await axios.get('/api/board/1')
```

Spring:

``` java
@RestController
@RequestMapping("/api/board")
public class BoardController {

    // GET /api/board/1 요청을 받음
    @GetMapping("/{id}")
    public BoardDto getBoard(@PathVariable Long id) {

        return boardService.getBoard(id);
    }
}
```

전체 흐름:

``` text
Vue 화면
   ↓
@click / @change 등 이벤트
   ↓
JavaScript 함수
   ↓
axios.get() / axios.post()
   ↓
Spring Controller
   ↓
Service
   ↓
Mapper / Repository
   ↓
DB
   ↓
서버 응답
   ↓
response.data
   ↓
Vue 상태값 변경
   ↓
화면 자동 갱신
```

실무에서 백엔드까지 추적하려면 Axios의 URL을 잡고 Java 프로젝트에서
검색하면 된다.

``` text
axios.get('/api/board')
           ↓
"/api/board" 검색
           ↓
@GetMapping
@PostMapping
@RequestMapping
           ↓
Controller 찾기
```

------------------------------------------------------------------------

## 22. Vue + JavaScript의 타입

Vue 자체가 타입을 없애는 것은 아니다.

JavaScript 기반 Vue에서는 변수 선언 시 타입을 명시하지 않는 경우가 많다.

``` javascript
const name = ref('홍길동') // String
const age = ref(30)       // Number
const useYn = ref(true)   // Boolean
const users = ref([])     // Array
const form = ref({})      // Object
```

Java와 비교:

``` java
String name = "홍길동";
int age = 30;
boolean useYn = true;
```

Vue 프로젝트가 TypeScript를 사용한다면 다음과 같이 타입을 명시할 수도
있다.

``` vue
<script setup lang="ts">
```

``` typescript
const name = ref<string>('홍길동')
const age = ref<number>(30)
```

즉:

``` text
Vue
→ 프론트엔드 프레임워크

JavaScript
→ 동적 타입 언어

TypeScript
→ JavaScript에 타입 기능 추가
```

------------------------------------------------------------------------

## 23. 실무에서 자주 보는 종합 예제

``` vue
<script setup>

import { ref, reactive, onMounted } from 'vue'
import axios from 'axios'


// ========================================
// 검색 조건
// ========================================

const form = reactive({
  title: '',
  writer: ''
})


// ========================================
// 조회 결과
// ========================================

const rows = ref([])


// ========================================
// 화면 표시 상태
// ========================================

const visible = ref(true)


// ========================================
// 조회
// ========================================

const search = async () => {

  // 서버 호출
  const response = await axios.get(
    '/api/board',
    {
      params: {
        title: form.title,
        writer: form.writer
      }
    }
  )

  // 서버 결과를 rows에 저장
  rows.value = response.data
}


// ========================================
// 상세 영역 숨기기
// ========================================

const hideDetail = () => {

  // false → v-show 영역 display:none
  visible.value = false
}


// ========================================
// 화면 최초 실행
// ========================================

onMounted(() => {

  // 화면이 열리면 자동 조회
  search()
})

</script>


<template>

  <!-- 검색조건 -->
  <input
    v-model="form.title"
    placeholder="제목"
  >

  <input
    v-model="form.writer"
    placeholder="작성자"
  >


  <!-- 이벤트 → 함수 호출 -->
  <button @click="search">
    조회
  </button>


  <!-- 반복 출력 -->
  <div
    v-for="row in rows"
    :key="row.id"
  >
    {{ row.title }}
  </div>


  <!-- 상태값에 따라 show/hide -->
  <div v-show="visible">
    상세 내용
  </div>


  <!-- 이벤트 → 상태값 변경 -->
  <button @click="hideDetail">
    상세 숨기기
  </button>

</template>
```

------------------------------------------------------------------------

## 24. 자주 쓰이는 문법 빠른 정리

``` text
ref()
→ 반응형 변수
→ script에서는 .value 사용

reactive()
→ 반응형 객체
→ .value 사용하지 않음

{{ 변수 }}
→ 화면에 값 출력

v-model
→ 입력값과 데이터 연결

v-if
→ 조건에 따라 DOM 생성 / 제거

v-show
→ 조건에 따라 show / hide
→ false이면 display:none

v-for
→ 배열 반복

:key
→ 반복 데이터의 고유 식별값

:
→ v-bind 축약
→ 속성에 JavaScript 값 연결

@
→ v-on 축약
→ 이벤트 연결

computed()
→ 기존 데이터를 이용한 계산값

watch()
→ 특정 값의 변경 감시

onMounted()
→ 화면 최초 진입 시 실행

defineProps()
→ 부모 → 자식 데이터 전달

defineEmits()
→ 자식 → 부모 이벤트 전달

axios.get()
→ 서버 GET 요청

axios.post()
→ 서버 POST 요청
```

------------------------------------------------------------------------

## 25. Vue 소스 읽는 순서

기존 프로젝트를 분석할 때는 다음 순서로 추적하면 편하다.

``` text
① template에서 버튼 / 컴포넌트 확인

        ↓

② @click, @change 등의 이벤트 확인

        ↓

③ 연결된 함수 찾기

@click="search"
        ↓
const search = ...

        ↓

④ 함수 안에서 API 호출 확인

axios.get(...)
axios.post(...)

        ↓

⑤ API URL 확인

'/api/board'

        ↓

⑥ Spring 프로젝트에서 URL 검색

@GetMapping
@PostMapping
@RequestMapping

        ↓

⑦ Controller → Service

        ↓

⑧ Mapper / Repository

        ↓

⑨ SQL / DB 확인
```

### 가장 먼저 익혀둘 핵심

``` text
v-model  = 입력값 연결

:        = 속성 / 데이터 연결

@        = 이벤트

ref      = 반응형 변수

.value   = ref의 실제 값 접근

reactive = 반응형 객체

v-show   = true/false로 show/hide

v-for    = 반복

:key     = 반복 데이터 고유 식별값

onMounted = 화면 최초 실행

axios    = 서버와 HTTP 통신
```

이 문법들을 먼저 익히면 기존 Vue 프로젝트의 화면 → 이벤트 → 함수 → API →
Java 백엔드 흐름을 추적하기 훨씬 쉬워진다.
