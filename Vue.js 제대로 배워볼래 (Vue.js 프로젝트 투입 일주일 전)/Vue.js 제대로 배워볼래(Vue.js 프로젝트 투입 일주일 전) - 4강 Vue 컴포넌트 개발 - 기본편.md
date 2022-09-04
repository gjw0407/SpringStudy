# Vue.js 제대로 배워볼래?(Vue.js 프로젝트 투입 일주일 전) - 4강 Vue 컴포넌트 개발 - 기본편

# Vue 컴포넌트 기본구조 이해하기

- <template></template>
    - html 영역
- <script></script>
    - javascript 영역
- <style></style>
    - css 영역

## Script

- `components:`
- `data()`
    - template 영역에서 {{ }} 형태로 쓰임
- `setup()`
- `created()`
    - 컴포넌트가 실행이 되는 순간에 자동으로 created안에 만든 요소들이 생성
- `mounted()`
    - html DOM이 화면에 로딩 되는 순가(mount) mounted()안에 있는 js 코드 실행
    - `this`으로 `data()` 접근 가능
- `unmounted()`
    - 다른 컴포넌트로 이동할 때 unmounted()안에 있는 코드 실행
- `methods: {}`
    - 함수들 정의

### 양방향 데이터 바인딩 가능

```jsx
<h1> {{title}} </h1>
<input type="text" v-model="title"/>
```
data(){
	return {
		title: 'hi'
	};
}
```

## v-html

```jsx
<div v-html="htmlString"> </div>
```
data(){
	return {
		htmlString: '<p style="color:red;"> Red String </p>'
	};
}
```

# 입력폼 데이터 바인딩

## checkbox

```jsx
<label><input type="checkbox" value="서울" v-model="checked"> 서울 </label>
<label><input type="checkbox" value="부산" v-model="checked"> 부산 </label>
<label><input type="checkbox" value="제주" v-model="checked"> 제주 </label>
<span> 체크한 지역 : {{checked}} </span>
```
data(){
	return {
		checked: []
	};
}
```

- 체크 박스를 체크할 때마다 `checked` 배열에 값이 입력이 됨

## radio

```jsx
<label><input type="radio" v-bind:value="radio1" v-model="picked"> 서울 </label>
<label><input type="radio" v-bind:value="radio2" v-model="picked"> 부산 </label>
<label><input type="radio" v-bind:value="radio3" v-model="picked"> 제주 </label>
<span> 체크한 지역 : {{picked}} </span>
```
data(){
	return {
		picked: "",
		radio1: "서울2",
		radio2: "부산2",
		radio3: "제주2",		
	};
}
```
체크한 지역 : 서울2
```

- `v-bind` 되어 있기에 picked에는 data 값이 들어감