# Vue 컴포넌트 학습 노트

> Vue 3 · JavaScript · Composition API의 `<script setup>` 기준입니다.
> `defineModel()` 예제는 Vue 3.4 이상이 필요합니다.
> 이 파일은 학습 문서이며, 코드 블록을 지정된 파일에 옮겨 실습합니다.

## 목차

1. 컴포넌트 개념과 분리 기준
2. 실습 환경 준비
3. 첫 컴포넌트 만들기
4. 반응성과 템플릿 문법
5. Props와 Emit으로 데이터 주고받기
6. Slot으로 화면 재사용하기
7. 컴포넌트 v-model
8. 생명주기와 로직 재사용
9. 실무 활용과 자주 하는 실수
10. 연습 문제와 복습

## 1. 컴포넌트란?

컴포넌트는 화면의 일부를 독립된 단위로 묶은 것입니다. 화면 구조, 동작, 스타일을 함께 관리하고 필요한 곳에서 다시 사용합니다.

쇼핑몰을 예로 들면 다음과 같이 나눌 수 있습니다.

```text
App
├── SiteHeader
├── ProductList
│   ├── ProductCard
│   └── ProductCard
└── ShoppingCart
```

같은 ProductCard를 사용하더라도 상품 데이터는 다를 수 있습니다. 각 인스턴스 안에서 선언한 상태도 기본적으로 독립적입니다.

컴포넌트를 분리할 때는 다음을 기준으로 생각해 봅니다.

- 같은 화면과 동작이 여러 곳에서 반복되는가?
- 이름으로 역할을 설명할 수 있는가?
- 한 파일이 서로 다른 책임을 너무 많이 맡고 있는가?
- 입력 데이터와 발생시키는 이벤트를 명확하게 정의할 수 있는가?

예를 들어 주문 화면에서 상품 표시, 주소 입력, 결제 요약은 분리할 만합니다. 반대로 한 번만 쓰는 문장 하나까지 무조건 컴포넌트로 만들 필요는 없습니다.

`.vue` 파일은 SFC(Single-File Component)라고 하며 보통 세 부분으로 구성됩니다.

| 영역 | 역할 |
| --- | --- |
| `<script setup>` | 데이터와 함수, 컴포넌트 가져오기 |
| `<template>` | 화면 구조와 데이터 표시 |
| `<style scoped>` | 해당 컴포넌트에 범위를 제한한 스타일 |

`scoped`는 Shadow DOM처럼 완전한 격리가 아닙니다. 자식 컴포넌트의 루트 요소에는 부모의 scoped 스타일이 적용될 수 있습니다.

참고: [컴포넌트 기초](https://vuejs.org/guide/essentials/component-basics.html)

## 2. 실습 환경 준비

HTML, CSS, JavaScript의 변수·함수·배열·객체·import를 알고 있으면 따라가기 쉽습니다.

설치 없이 해보려면 [Vue Playground](https://play.vuejs.org/)에서 파일을 추가해 실습하세요. Playground에서는 컴포넌트 파일을 같은 위치에 만들고 import 경로를 `./CounterButton.vue`처럼 맞춥니다.

로컬에서는 [공식 빠른 시작](https://vuejs.org/guide/quick-start.html)의 Node.js 요구 버전을 확인하고 아래 명령을 실행합니다.

```sh
npm create vue@latest
```

프로젝트 이름을 `vue-component-study`로 지정하고, 처음에는 TypeScript·Router·Pinia 등 추가 기능을 선택하지 않아도 됩니다.

```sh
cd vue-component-study
npm install
npm run dev
```

터미널에 표시되는 주소를 브라우저에서 엽니다. 다음 예제는 프로젝트의 `src/App.vue`를 교체하며 순서대로 실습합니다. 각 절의 App.vue 예제는 별도로 실행합니다.

## 3. 첫 컴포넌트 만들기

`src/components/CounterButton.vue`:

```vue
<script setup>
import { ref } from 'vue'

const count = ref(0)

function increase() {
  count.value += 1
}
</script>

<template>
  <button type="button" @click="increase">
    클릭 횟수: {{ count }}
  </button>
</template>

<style scoped>
button {
  padding: 8px 16px;
  cursor: pointer;
}
</style>
```

`src/App.vue`:

```vue
<script setup>
import CounterButton from './components/CounterButton.vue'
</script>

<template>
  <h1>컴포넌트 실습</h1>
  <CounterButton />
  <CounterButton />
</template>
```

`<script setup>`에서 import한 컴포넌트는 template에서 바로 사용할 수 있습니다.

**확인:** 첫 번째 버튼만 세 번 클릭해 보세요. 첫 번째는 3, 두 번째는 0입니다. 같은 컴포넌트 정의로 만들어도 인스턴스마다 count를 따로 가집니다.

## 4. 반응성과 템플릿 문법

반응성은 데이터 변경이 화면 갱신으로 이어지는 성질입니다.

```js
import { ref, computed } from 'vue'

const quantity = ref(1)
const unitPrice = ref(10000)
const totalPrice = computed(() => quantity.value * unitPrice.value)
```

JavaScript에서 ref의 값은 `.value`로 접근합니다. template에서는 최상위 ref가 자동으로 풀리므로 `{{ quantity }}`처럼 씁니다. computed는 기존 데이터로부터 계산되는 값을 표현할 때 사용합니다.

| 문법 | 예시 | 의미 |
| --- | --- | --- |
| 보간 | `{{ title }}` | 값을 텍스트로 표시 |
| `v-bind` / `:` | `:disabled="loading"` | 속성을 표현식에 연결 |
| `v-on` / `@` | `@click="save"` | 이벤트 처리 |
| `v-if` | `v-if="visible"` | 조건에 따라 생성·제거 |
| `v-show` | `v-show="visible"` | 요소를 유지하고 CSS로 표시 전환 |
| `v-for` | `v-for="item in items"` | 목록 반복 |
| `v-model` | `v-model="keyword"` | 입력값과 상태 연결 |

리스트의 `key`에는 항목의 안정적인 고유 ID를 사용합니다. 항목을 삭제하거나 순서를 바꾸는 목록에서는 배열 인덱스를 key로 쓰지 않는 편이 좋습니다.

참고: [반응성 기초](https://vuejs.org/guide/essentials/reactivity-fundamentals.html)

## 5. Props와 Emit: 상품 선택 실습

**Props는 부모가 자식에게 전달하는 입력값이고, Emit은 자식이 부모에게 보내는 이벤트입니다.**

```text
App.vue ── product (props) ──▶ ProductCard.vue
App.vue ◀── add (event) ───── ProductCard.vue
```

다음 두 파일을 만들면 상품 카드를 클릭해 선택 수량과 합계를 확인할 수 있습니다.

### 자식: src/components/ProductCard.vue

```vue
<script setup>
defineProps({
  product: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['add'])
</script>

<template>
  <article class="product-card">
    <h2>{{ product.name }}</h2>
    <p>{{ product.price.toLocaleString() }}원</p>
    <button type="button" @click="emit('add', product.id)">
      담기
    </button>
  </article>
</template>

<style scoped>
.product-card {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 16px;
  margin-bottom: 12px;
}
</style>
```

`defineProps`와 `defineEmits`는 `<script setup>`의 컴파일러 매크로이므로 import하지 않습니다. 여기서 Object 타입 선언은 product 내부의 name·price 필드까지 검증하지는 않습니다.

### 부모: src/App.vue

```vue
<script setup>
import { ref, computed } from 'vue'
import ProductCard from './components/ProductCard.vue'

const products = [
  { id: 1, name: 'Vue 학습 노트', price: 8000 },
  { id: 2, name: '개발자 머그컵', price: 12000 }
]

const cart = ref([])

function addToCart(productId) {
  const product = products.find(item => item.id === productId)
  if (product) {
    cart.value.push(product)
  }
}

const totalPrice = computed(() =>
  cart.value.reduce((sum, item) => sum + item.price, 0)
)
</script>

<template>
  <main>
    <h1>상품 목록</h1>
    <ProductCard
      v-for="product in products"
      :key="product.id"
      :product="product"
      @add="addToCart"
    />
    <p>선택 수량: {{ cart.length }}개</p>
    <p>합계: {{ totalPrice.toLocaleString() }}원</p>
    <p v-if="cart.length === 0">상품을 담아 보세요.</p>
  </main>
</template>
```

### 실행 흐름

1. 부모가 product를 자식에게 전달합니다.
2. 자식이 상품 정보를 표시합니다.
3. 담기 버튼을 누르면 자식이 add 이벤트와 상품 ID를 전달합니다.
4. 부모의 addToCart가 실행되어 cart를 변경합니다.
5. 계산된 합계와 화면이 갱신됩니다.

**확인:** 노트 한 번, 머그컵 두 번을 담으면 수량은 3개, 합계는 32,000원입니다. 이 예제는 같은 상품도 클릭할 때마다 배열에 추가합니다.

### 꼭 기억할 규칙

- `title="문자열"`은 고정 문자열, `:title="변수"`는 표현식의 결과를 전달합니다.
- `count="3"`은 문자열이고 `:count="3"`은 숫자입니다.
- 자식에서 props 자체를 다시 할당하지 않습니다.
- 객체·배열 props의 내부는 기술적으로 변경할 수 있어도 부모 상태를 몰래 바꾸지 말고 이벤트로 변경을 요청합니다.
- 컴포넌트 이벤트는 DOM 이벤트처럼 조상에게 자동으로 버블링하지 않습니다. 필요한 중간 부모가 다시 전달해야 합니다.

참고: [Props](https://vuejs.org/guide/components/props.html), [컴포넌트 이벤트](https://vuejs.org/guide/components/events.html)

## 6. Slot: 같은 틀에 다른 내용 넣기

Props는 값 전달에, Slot은 부모가 작성한 화면 조각을 삽입하는 데 사용합니다. 예를 들어 카드의 테두리는 같지만 제목·본문·버튼 배치를 바꾸고 싶을 때 유용합니다.

`src/components/BaseCard.vue`:

```vue
<template>
  <section class="card">
    <header><slot name="header">기본 제목</slot></header>
    <div><slot /></div>
    <footer><slot name="footer" /></footer>
  </section>
</template>

<style scoped>
.card {
  border: 1px solid #ccc;
  padding: 16px;
  border-radius: 8px;
}
</style>
```

`src/App.vue`:

```vue
<script setup>
import BaseCard from './components/BaseCard.vue'
</script>

<template>
  <BaseCard>
    <template #header><h1>오늘의 학습</h1></template>
    <p>Props와 Emit 예제를 직접 작성하기</p>
    <template #footer><small>예상 소요: 20분</small></template>
  </BaseCard>
</template>
```

`#header`는 `v-slot:header`의 축약입니다. 이름 없는 내용은 기본 slot으로 들어갑니다. 부모가 넣은 slot 내용은 부모의 데이터 범위를 사용합니다. 자식의 데이터를 slot으로 노출해야 한다면 scoped slot을 추가로 학습하세요.

참고: [Slots](https://vuejs.org/guide/components/slots.html)

## 7. 컴포넌트 v-model: 공통 입력창

검색창이나 입력 필드를 컴포넌트로 감쌀 때 사용합니다.

`src/components/SearchInput.vue` — Vue 3.4 이상:

```vue
<script setup>
const model = defineModel({ type: String, required: true })
</script>

<template>
  <label>
    검색어
    <input v-model="model" placeholder="상품명을 입력하세요" />
  </label>
</template>
```

`src/App.vue`:

```vue
<script setup>
import { ref } from 'vue'
import SearchInput from './components/SearchInput.vue'

const keyword = ref('')
</script>

<template>
  <SearchInput v-model="keyword" />
  <p>입력한 검색어: {{ keyword }}</p>
</template>
```

부모의 keyword와 입력값이 연결됩니다. 내부적으로는 `modelValue` prop과 `update:modelValue` 이벤트를 이용하는 규약입니다.

Vue 3.4 미만에서는 SearchInput.vue를 다음처럼 작성할 수 있습니다.

```vue
<script setup>
defineProps({ modelValue: { type: String, required: true } })
const emit = defineEmits(['update:modelValue'])
</script>

<template>
  <label>
    검색어
    <input
      :value="modelValue"
      @input="emit('update:modelValue', $event.target.value)"
    />
  </label>
</template>
```

참고: [컴포넌트 v-model](https://vuejs.org/guide/components/v-model.html)

## 8. 생명주기와 로직 재사용

### 생명주기

컴포넌트는 생성되고, 화면에 연결되고, 갱신되며, 제거됩니다.

- `onMounted`: DOM에 연결된 뒤 실행합니다. DOM 관련 초기화 등에 사용합니다.
- `onUnmounted`: 제거될 때 실행합니다. 타이머·이벤트 리스너 같은 자원을 정리합니다.

`src/components/StudyTimer.vue`:

```vue
<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const seconds = ref(0)
let timerId

onMounted(() => {
  timerId = setInterval(() => {
    seconds.value += 1
  }, 1000)
})

onUnmounted(() => {
  clearInterval(timerId)
})
</script>

<template>
  <p>학습 시간: {{ seconds }}초</p>
</template>
```

부모에서 import한 뒤 `v-if`로 표시를 켜고 끄면, 제거될 때 타이머가 정리되고 새로 표시할 때 0부터 시작합니다. `v-show`는 숨기기만 하므로 타이머가 계속 동작합니다.

### Composable

화면을 재사용하면 컴포넌트, 반응형 상태와 동작을 재사용하면 composable로 분리할 수 있습니다.

`src/composables/useCounter.js`:

```js
import { ref } from 'vue'

export function useCounter(initialValue = 0) {
  const count = ref(initialValue)
  const increase = () => { count.value += 1 }
  const reset = () => { count.value = initialValue }

  return { count, increase, reset }
}
```

컴포넌트의 `<script setup>`에서 다음처럼 사용합니다.

```js
import { useCounter } from './composables/useCounter'

const { count, increase, reset } = useCounter()
```

위 경로는 src/App.vue 기준입니다. src/components 안에서는 `../composables/useCounter`로 바꿉니다. 이 함수는 호출할 때마다 ref를 새로 만들기 때문에 각각 독립된 상태를 반환합니다.

참고: [생명주기](https://vuejs.org/guide/essentials/lifecycle.html), [Composables](https://vuejs.org/guide/reusability/composables.html)

## 9. 활용 방법과 자주 하는 실수

### 상황별 설계 선택

| 상황 | 접근 방법 | 예시 |
| --- | --- | --- |
| 같은 UI를 반복 표시 | 컴포넌트 + props | 상품 카드, 사용자 프로필 |
| 자식의 동작을 부모에 알림 | emit | 삭제 요청, 선택 완료 |
| 같은 레이아웃에 다른 내용 | slot | 카드, 대화상자, 페이지 틀 |
| 공통 입력 필드 | 컴포넌트 v-model | 검색창, 텍스트 입력 |
| 형제끼리 데이터 공유 | 공통 부모가 상태를 소유 | 목록과 선택 결과 |
| 여러 화면에 걸친 공유 상태 | 상태 관리 도구 검토 | 로그인 정보, 장바구니 |
| 반복되는 반응형 로직 | composable | 카운터, 데이터 조회 상태 |

상품 카드가 직접 전체 장바구니를 관리하도록 만들기보다, 상품 표시와 선택 알림을 맡기면 검색 결과·추천 상품 영역에서도 재사용하기 쉽습니다.

### 흔한 오류

| 증상 | 점검할 내용 |
| --- | --- |
| 컴포넌트가 표시되지 않음 | import 경로·파일 이름의 대소문자·태그 이름 |
| JavaScript에서 상태 변경이 안 됨 | ref에 `.value`를 사용했는지 |
| 숫자 prop에 타입 경고 | `count="3"` 대신 `:count="3"`인지 |
| 부모 데이터가 예상치 않게 변경됨 | 자식이 객체 props 내부를 변경하는지 |
| 이벤트가 처리되지 않음 | emit 이름과 부모의 @이벤트 이름이 일치하는지 |
| 목록 삭제 후 입력값이 엉뚱한 행에 남음 | key가 고유하고 안정적인 ID인지 |
| 숨겼는데 타이머가 계속 실행됨 | v-show인지, onUnmounted에서 정리하는지 |

## 10. 연습 문제

### 실습 A — 카운터 확장

CounterButton에 감소·초기화 버튼을 추가하세요.

완료 기준: 증가·감소가 모두 동작하고, 초기화하면 0이며, 두 인스턴스가 서로 영향을 주지 않습니다.

힌트: 별도 함수에서 `count.value -= 1`, `count.value = 0`을 실행합니다.

### 실습 B — 상품 품절 처리

상품에 stock을 추가하고 0이면 담기 버튼을 비활성화하세요.

완료 기준: 품절 안내가 보이고 버튼 클릭으로 장바구니가 증가하지 않습니다.

힌트: `:disabled="product.stock === 0"`을 사용합니다. 추가 과제로 부모가 재고를 줄이고 0 이하로 내려가지 않게 해보세요.

### 실습 C — 검색 기능 결합

7절의 SearchInput을 5절 상품 목록과 결합하세요.

완료 기준: 검색어에 해당하는 상품만 보이고, 검색어를 지우면 전체 상품이 보입니다.

힌트: keyword와 computed를 사용해 products.filter 결과를 만든 뒤 그 결과를 v-for로 표시합니다.

### 실습 D — 삭제 이벤트

장바구니 항목을 별도 컴포넌트로 만들고 삭제 이벤트를 연결하세요.

완료 기준: 선택한 한 항목만 삭제되고 합계도 갱신됩니다.

힌트: 같은 상품이 여러 번 들어갈 수 있으므로, 상품 ID와 별개인 장바구니 항목 ID를 부여하세요.

### 스스로 답해 보기

1. 같은 컴포넌트를 두 번 사용하면 왜 상태가 독립적일까요?
2. props를 직접 바꾸는 대신 emit을 사용하는 이유는 무엇일까요?
3. props와 slot의 역할은 어떻게 다를까요?
4. computed와 일반 ref 중 합계에는 무엇이 적합할까요?
5. v-if와 v-show는 타이머가 있는 컴포넌트에 어떤 차이를 만들까요?

<details>
<summary>답 확인</summary>

1. 각 컴포넌트 인스턴스가 자신의 setup을 실행하며 상태를 생성하기 때문입니다.
2. 데이터 소유자가 변경을 처리하도록 하여 변경 경로를 추적하기 쉽기 때문입니다.
3. props는 값, slot은 부모가 작성한 화면 내용을 전달합니다.
4. 합계는 원본 데이터에서 계산되므로 computed가 적합합니다.
5. v-if가 false가 되면 제거되어 정리 훅이 실행됩니다. v-show는 숨기기만 합니다.

</details>

## 다음 학습과 공식 문서

기본 예제를 설명할 수 있게 되면 아래 순서로 확장해 보세요.

1. [Provide / Inject](https://vuejs.org/guide/components/provide-inject.html): 깊은 컴포넌트 트리에서 데이터 전달
2. [동적 컴포넌트](https://vuejs.org/guide/essentials/component-basics.html#dynamic-components): 탭 등에 따라 표시할 컴포넌트 교체
3. [Vue Router](https://router.vuejs.org/): URL에 따라 화면 전환
4. [Pinia](https://pinia.vuejs.org/): 여러 컴포넌트·화면의 상태 공유

학습 순서: **직접 작성 → 실행 결과 확인 → 값을 바꿔 실험 → 코드 없이 데이터 흐름 설명**.
