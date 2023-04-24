---
layout  : wiki
title   : Vue CLI 프로젝트 생성 옵션 알아보기 
summary : 
date    : 2023-04-24 14:29:22 +0900
updated : 2023-04-24 15:45:19 +0900
tag     : Vue_js
resource: C9/03A904-A11B-474A-BD09-4CD0C32F4ED3
toc     : true
public  : true
parent  : [[/Vue_js]]
latex   : false
---
* TOC
{:toc}

## 개요

> Vue CLI는 Vue.js 프로젝트를 시작하기 위한 공식 커맨드라인 인터페이스(Command-Line Interface) 도구 
 
- Vue.js 프로젝트의 기본 구조를 설정하고, Webpack을 비롯한 빌드 도구를 설정하는 것을 돕는다.

## 프로젝트 생성 및 세부 옵션

> vue create '프로젝트명'

- vue create 명령어를 실행한다.

>  Please pick a preset: \
> ❯ Default ([Vue 3] babel, eslint) \
> Default ([Vue 2] babel, eslint) \
> Manually select features 

- Manually select features 선택 

> ◉ Babel \
> ◯ TypeScript \
> ◯ Progressive Web App (PWA) Support \
> ◯ Router \
> ◯ Vuex \
> ◯ CSS Pre-processors \
> ◉ Linter / Formatter \
> ◯ Unit Testing \
> ◯ E2E Testing

- Defult로 Babel과 eslint가 선택되어있다.

> Choose a version of Vue.js that you want to start the project with (Use arrow keys) \
> ❯ 3.x \
> 2.x

- 선택을 마치면, Vue의 버전을 선택한다.
- 수동 선택 시, 어떤 옵션을 선택할 수 있는지 확인해보자.

> Use class-style component syntax?

- 클래스 스타일 컴포넌트 구문을 사용할 것인지 여부를 묻는 옵션이다.
- VUe.js 2.x 버전에서 도입된 옵션으로, 선택 시 ES6의 클래스 문법을 사용하여 Vue 컴포넌트를 작성하는 방식이다.
- 비 선택 시, 일반적인 객체 리터럴 기반의 컴포넌트 구문을 사용하게 된다.

### Babel

- Babel은 ES6+ 코드를 ES5 코드로 변환하는 JavaScript 컴파일러이다.

> Use Babel alongside TypeScript (required for modern mode, auto-detected polyfills, transpiling JSX)?

- 옵션 선택 시, Typescript와 함께 Babel을 사용하도록 구성한다.
- 이를 통해, 모던 모드, 자동 감지 폴리필(polyfill), JSX 변환 등 기능을 사용할 수 있다.

### TypeScript

> add TypeScript 

- Vue 프로젝트에 TypeScript를 추가한다.

### Progressive Web App (PWA) Support

> PWA features via @vue/cli-plugin-pwa

- PWA(Progressive Web App)은 웹과 네이티브 앱의 기능 모두의 이점을 갖도록 수 많은 특정 기술과 표준 패턴을 사용해 개발된 웹 앱이다.
- 옵션 선택 시, 프로젝트에 PWA를 지원하도록 의존성을 추가한다.

### Router

- Router는 Routing 기능을 제공한다. 
- Routing 기능을 통해 SPA(Single Page Application)을 구축할 수 있다.
- 이를 통해 어떤 페이지에서 다른 페이지로 이동할 때, 페이지 전체의 정보를 읽을 필요가 없어진다.

> Use history mode for router? (Requires proper server setup for index fallback in production)

- Router를 History 모드로 사용한다면, URL에 해시(#)가 포함되지 않으며 일반적인 URL 경로로 Vue js 애플리케이션을 탐색할 수 있다.

### Vuex

- Vue js 애플리케이션에서 상태 관리를 위한 공식 라이브러리이다.
- 애플리케이션의 모든 컴포넌트에 대한 중앙 집중식 저장소 역할을 하며, 상태를 변경할 수 있다.
- 옵션 선택 시, 라이브러리에 Vuex를 추가한다.

### CSS Pre-processors

> Pick a CSS pre-processor (PostCSS, Autoprefixer and CSS Modules are supported by default): \
> ❯ Sass/SCSS (with dart-sass) \
> Less \
> Stylus


- CSS 전처리기는 CSS 작성을 보다 효율적으로 할 수 있도록 도와주는 도구이다.
- 옵션 선택 시 Sass/SCSS, Less, Stylus 등 프로젝트에 CSS 전처리기를 추가한다.


### Linter / Formatter

> Pick a linter / formatter config: \
> ❯ ESLint with error prevention only \
> ESLint + Airbnb config \
> ESLint + Standard config \
> ESLint + Prettier \

- Linter는 코드 스타일과 잠재적 오류를 검사하는 도구이다. 
- formatter는 코드를 일관된 스타일로 자동으로 정렬하여 가독성을 향상시키는 도구이다. 

> Pick additional lint features: \
> ❯◉ Lint on save \
> ◯ Lint and fix on commit

- Linter에 추가할 기능 여부를 묻는 옵션이다.
- Lint on save의 경우 코드를 저장할 때마다, Linter를 실행하여 파일과 품질을 검사한다.
- Lint and fix on commit은 코드를 commit할 때마다 Linter를 실행한다.


### Unit Testing

> Pick a unit testing solution: \
> ❯ Jest \
> Mocha + Chai

- 단위 테스트 작성을 위한 Mocha + Chai, Jest와 같은 테스트 프레임워크를 추가한다.

### E2E Testing

>  Pick an E2E testing solution: \
> ❯ Cypress (Test in Chrome, Firefox, MS Edge, and Electron) \
> Nightwatch (WebDriver-based) \
> WebdriverIO (WebDriver/DevTools based) 

- 사용자가 애플리케이션을 사용하는 것과 유사한 시나리오에서 전체 애플리케이션을 테스트하는 사용된다.
- 옵션 선택 시, 해당 테스트 프레임워크를 추가한다.

### 나머지

> Where do you prefer placing config for Babel, ESLint, etc.? \
> In dedicated config files \
> In package.json

- In dedicated config files는 Babel, ESLint 등의 구성 파일을 별도의 파일에 저장한다. 
  - 선택 시, Vue CLI가 자동으로 해당 파일을 생성한다.
- In package.json: Babel, ESLint 등의 구성 파일을 package.json 파일 내부에 저장한다.
  - 선택 시, package.json 파일 내부의 "babel", "eslint" 등의 속성에 구성 정보가 저장된다.

> Use config files \
> Use separate config files

- 옵션 선택 시, Vue CLI가 프로젝트의 구성 파일을 별도의 파일로 분리한다.

> Save this as a preset for future projects? \
> Yes
> No

- 프로젝트의 설정을 Vue CLI의 프리셋으로 저장할지 여부를 묻는다.
- 프리셋으로 저장하면 나중에 비슷한 설정을 사용하는 새 프로젝트를 쉽게 생성할 수 있다.
- 