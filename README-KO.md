# Nextra 4 완전 가이드

이 문서는 이 레포(api-docs-site)의 구조를 바탕으로 Nextra 4를 이해하고 실무에서 바로 쓰기 위한 모든 핵심을 한곳에 정리했습니다. 설치/구성, 라우팅, 문서 작성(MDX), 테마 사용과 커스터마이징, 검색, 컨벤션까지 포함합니다.

## TL;DR

- Next.js 15.3.4 App Router + Nextra 4.2.17 + `nextra-theme-docs` 조합입니다.
- React 19.0.0 지원
- 실제 문서는 `content/` 폴더의 `.mdx` 파일로 관리합니다.
- `app/[[...mdxPath]]/page.jsx`가 모든 MDX 라우팅과 SEO 메타, TOC를 연결합니다.
- 전역 레이아웃은 `app/layout.tsx`에서 `nextra-theme-docs`의 `Layout`을 사용해 구성합니다.
- 검색은 Pagefind를 사용하며 `postbuild`에서 인덱스를 생성해 `public/_pagefind`에 둡니다.
- TypeScript와 JavaScript 혼용 지원 (`.tsx`와 `.jsx` 파일 모두 사용 가능)

---

## 프로젝트 구조(이 레포)

### 실제 파일 구조
```
api-docs-site/
├── app/
│   ├── [[...mdxPath]]/
│   │   └── page.jsx          # 동적 MDX 라우팅
│   └── layout.tsx            # 전역 레이아웃
├── content/
│   ├── index.mdx             # 홈페이지
│   ├── about.mdx             # About 페이지
│   └── built-in-components.mdx # 컴포넌트 문서
├── docs/
│   └── nextra4-guide.md      # 이 가이드 문서
├── public/
│   └── images/
│       └── general/
│           ├── icon.svg      # 파비콘
│           └── logo.svg      # 로고
├── .gitignore                # Git 제외 설정
├── mdx-components.js         # MDX 컴포넌트 설정
├── next-env.d.ts             # Next.js 타입 정의
├── next.config.mjs           # Next.js + Nextra 설정
├── package.json              # 패키지 정의
├── pnpm-lock.yaml           # pnpm 락 파일
├── README.md                # 프로젝트 설명
└── tsconfig.json            # TypeScript 설정
```

### 핵심 파일 역할
- `next.config.mjs`: `nextra()` 플러그인 적용, 옵션(`search`, `defaultShowCopyCode`) 설정
- `app/layout.tsx`: 전역 레이아웃, `Layout`/`Navbar`/`Footer`/`Head`/`Banner` 구성, `getPageMap()`로 사이드바 데이터 주입
- `app/[[...mdxPath]]/page.jsx`: 동적 라우트. `generateStaticParamsFor`, `importPage`를 통해 MDX 페이지 로딩 및 SEO 메타/TOC 전달
- `mdx-components.js`: 테마의 MDX 컴포넌트를 가져와 필요 시 사용자 정의 컴포넌트와 병합
- `content/`: 실제 문서(MDX) 위치. `index.mdx`는 홈(`/`)
- `public/`: 정적 자산(이미지 등)과 `public/_pagefind`(검색 인덱스 - 빌드 시 생성)
- `package.json` 스크립트: `dev`, `build`, `start`, `postbuild(pagefind 인덱싱)`
- `next-env.d.ts`: Next.js 타입 정의 파일 (자동 생성, 수정 불필요)
- `.gitignore`: `.next`, `node_modules`, `/.idea` 제외

---

## Nextra 4 핵심 개념

### 1) App Router와의 통합

- 이 레포는 Next.js App Router를 사용합니다. `app/[[...mdxPath]]/page.jsx`가 모든 MDX 경로를 흡수합니다.
- 주요 API
  - `nextra/pages`의 `generateStaticParamsFor('mdxPath')`: `content/`에 있는 모든 MDX 파일에 대응하는 정적 경로 생성
  - `nextra/pages`의 `importPage(path)`: 특정 MDX를 동적으로 로드하여 `{ default: MDXContent, toc, metadata }` 반환
  - `nextra/page-map`의 `getPageMap()`: 전체 문서 트리(사이드바/네비게이션)에 필요한 페이지 맵 생성

### 2) 테마와 레이아웃

- `nextra-theme-docs`의 `Layout`, `Navbar`, `Footer` 컴포넌트를 직접 가져와 `app/layout.tsx`에서 사용합니다.
- `pageMap={await getPageMap()}`를 전달해 테마가 사이드바/목차/탐색을 렌더링할 수 있도록 합니다.
- `docsRepositoryBase`로 각 문서의 “Edit this page” 링크 기준을 설정할 수 있습니다.
- `Head` 컴포넌트로 파비콘 등 공통 `<head>` 태그를 추가합니다.

### 3) 콘텐츠 디렉터리

- 기본적으로 문서는 `content/` 폴더에 위치합니다.
- 파일 경로와 라우트 매핑
  - `content/index.mdx` → `/`
  - `content/about.mdx` → `/about`
  - `content/built-in-components.mdx` → `/built-in-components`
- 사이드바/정렬/제목 등은 `_meta.json`로 제어할 수 있습니다(아래 '사이드바와 _meta.json' 참고).
- **_meta.json이 없는 경우**: 파일명 알파벳 순서로 자동 정렬되며, 파일명이 그대로 사이드바에 표시됩니다.

### 4) MDX와 빌트인 컴포넌트

- 문서는 MDX로 작성합니다. Markdown + React 컴포넌트를 함께 사용할 수 있습니다.
- `nextra/components`에서 제공하는 빌트인 컴포넌트를 바로 import하여 사용합니다.
- 이 레포 예시: `content/index.mdx`에서 `<Cards>`, `<Steps>` 사용

빌트인 컴포넌트(공식 문서 확인)
- 출처: https://nextra.site/docs/built-ins 및 하위 페이지(아래 링크는 실제 확인 결과)

레이아웃 컴포넌트
- `Banner` — https://nextra.site/docs/built-ins/banner
- `Head` — https://nextra.site/docs/built-ins/head
- `Search` — https://nextra.site/docs/built-ins/search

콘텐츠 컴포넌트
- `Bleed` — https://nextra.site/docs/built-ins/bleed
- `Callout` — https://nextra.site/docs/built-ins/callout
- `Cards` — https://nextra.site/docs/built-ins/cards
- `FileTree` — https://nextra.site/docs/built-ins/filetree
- `Steps` — https://nextra.site/docs/built-ins/steps
- `Table` — https://nextra.site/docs/built-ins/table
- `Tabs` — https://nextra.site/docs/built-ins/tabs

기타 컴포넌트
- `MDXRemote` — https://nextra.site/docs/built-ins/mdxremote
- `Playground` — https://nextra.site/docs/built-ins/playground
- `TSDoc` — https://nextra.site/docs/built-ins/tsdoc

#### FileTree Component
A built-in component to visually represent a file tree.

Exported from `nextra/components`.

**Props**
- `...props`: `HTMLAttributes<HTMLUListElement>` - All standard HTML UL element attributes

**Usage**
Create the file tree structure by nesting `<FileTree.Folder>` and `<FileTree.File>` components within a `<FileTree>`. Name each file or folder with the `name` attribute. Use `defaultOpen` to set the folder to open on load.

```mdx
import { FileTree } from 'nextra/components'
 
<FileTree>
  <FileTree.Folder name="content" defaultOpen>
    <FileTree.File name="_meta.js" />
    <FileTree.File name="contact.md" />
    <FileTree.File name="index.mdx" />
    <FileTree.Folder name="about">
      <FileTree.File name="_meta.js" />
      <FileTree.File name="legal.md" />
      <FileTree.File name="index.mdx" />
    </FileTree.Folder>
  </FileTree.Folder>
</FileTree>
```

#### MDXRemote Component
A React component that renders compiled MDX content.

Exported from `nextra/mdx-remote`.

**Props**
- `components`: `MDXComponents` - An object mapping names to React components. The key used will be the name accessible to MDX.
- `scope`: `Scope` - Pass-through variables for use in the MDX content. These variables will be available in the MDX scope.
- `compiledSource`: `string` - Raw JavaScript compiled MDX source code, a result of Nextra's `compileMdx` function.

**Example**
```mdx
import { compileMdx } from 'nextra/compile'
import { MDXRemote } from 'nextra/mdx-remote'
 
<MDXRemote
  compiledSource={await compileMdx('# Hello {myVariable} <MyComponent />')}
  components={{ MyComponent: () => <div>My Component</div> }}
  scope={{ myVariable: 'World' }}
/>
```

비고
- 이 레포에서는 `Banner`, `Head`를 실제로 `nextra/components`에서 import하여 사용하고 있습니다(참고: `app/layout.tsx`).
- 각 컴포넌트의 props/슬롯/서브컴포넌트 상세는 해당 공식 페이지를 참조하십시오. 본 문서는 공식 문서에 명시된 항목만 링크로 안내합니다.

### 5) 검색(Pagefind)

- `next.config.mjs`에서 `search: true`를 켜고, `package.json`의 `postbuild`에서 Pagefind 인덱스 생성:
  - `pagefind --site .next/server/app --output-path public/_pagefind`
- 개발 서버(`next dev`)에서는 완전한 인덱스를 보장하지 않습니다. 실제 검색 동작은 `build` 후 정적으로 확인하는 것이 정확합니다.

---

## 설정 파일과 코드 흐름

### next.config.mjs

```js
import nextra from "nextra";

const withNextra = nextra({
  search: true,
  defaultShowCopyCode: true,
});

export default withNextra({
  // 필요 시 Next.js 옵션 추가
  // output: 'export'
});
```

- `search`: Nextra 검색 UI 활성화
- `defaultShowCopyCode`: 코드블록에 기본 복사 버튼 표시
- 그 외 MDX 설정(`mdxOptions`)이나 베이스 경로 등 일반 Next 옵션을 함께 지정 가능

### app/layout.tsx

핵심 포인트
- `Layout`에 `pageMap`, `navbar`, `footer`, `banner`, `docsRepositoryBase` 등을 전달
- `Head`를 사용해 공통 `<head>` 태그 주입 (children으로 추가 태그 전달)
- `Banner` 컴포넌트로 상단 알림 배너 표시 (storageKey로 닫기 상태 저장)
- Next.js `metadata` API를 함께 사용할 수 있음
- HTML 속성 필수 설정:
  - `lang="en"`: SEO를 위한 언어 설정
  - `dir="ltr"`: 텍스트 방향 설정 (필수)
  - `suppressHydrationWarning`: next-themes 패키지 호환성

### app/[[...mdxPath]]/page.jsx

핵심 포인트
- `generateStaticParams`: 모든 문서 경로를 정적으로 생성
- `generateMetadata`: 각 문서 MDX에서 읽은 `metadata`를 Next SEO 메타로 연결
- `importPage`: MDX 페이지를 불러오고 `{toc, metadata}`를 래퍼에 전달
- `wrapper`: 테마가 제공하는 페이지 래퍼(TOC/레이아웃 문맥)로 감싸 렌더링

### mdx-components.js

```js
import { useMDXComponents as getThemeComponents } from 'nextra-theme-docs'

const themeComponents = getThemeComponents()

export function useMDXComponents(components) {
  return { ...themeComponents, ...components }
}
```

- 테마 기본 MDX 컴포넌트 위에 사용자 정의 컴포넌트를 덮어씌우는 패턴입니다.
- 커스텀 컴포넌트를 MDX에서 바로 사용할 수 있게 연결 가능합니다.

---

## 문서 작성 가이드(MDX)

### 1) Frontmatter

문서 파일 상단에 YAML Frontmatter를 작성해 제목/설명/옵션을 제어합니다.

```mdx
---
title: Nextra Docs Starter
description: 이 문서는 Nextra 4 스타터의 개요를 설명합니다.
toc: true
---

# 페이지 H1 제목
본문...
```

자주 쓰는 필드(테마/버전에 따라 일부 차이 가능)
- `title`: 페이지 제목(사이드바/헤더에 사용)
- `description`: SEO 및 미리보기 설명
- `toc`: 페이지 내 목차 표시 여부(true/false)

### 2) 링크와 이미지

- 내부 링크: `[문자열](/about)` 처럼 라우트 경로로 연결
- 이미지: `public/` 아래 정적 파일을 `/images/...`처럼 절대 경로로 참조

```mdx
![로고](/images/general/logo.svg)
```

### 3) 코드블록

- 언어 지정으로 하이라이팅: ```ts, ```bash 등
- `defaultShowCopyCode: true` 옵션으로 복사 버튼 표시

```ts
export function add(a: number, b: number) {
  return a + b
}
```

### 4) 빌트인 컴포넌트 예시

컴포넌트는 MDX 상단에서 import 후 사용합니다.

```mdx
import { Callout, Steps, Cards, Tabs, FileTree } from 'nextra/components'

# 빌트인 예시

<Callout type="info" emoji="ℹ️">
  이 박스는 정보성 콜아웃입니다.
  </Callout>

<Steps>
  ### 첫 번째 단계
  내용을 적습니다.

  ### 두 번째 단계
  또 다른 내용을 적습니다.
</Steps>

<Cards>
  <Cards.Card title="Nextra 4" href="https://nextra.site" />
  <Cards.Card title="Pagefind" href="https://pagefind.app" />
</Cards>

<Tabs items={["npm", "pnpm", "yarn"]}>
  <Tabs.Tab>
    ```bash
    npm i nextra nextra-theme-docs
    ```
  </Tabs.Tab>
  <Tabs.Tab>
    ```bash
    pnpm add nextra nextra-theme-docs
    ```
  </Tabs.Tab>
  <Tabs.Tab>
    ```bash
    yarn add nextra nextra-theme-docs
    ```
  </Tabs.Tab>
</Tabs>

<FileTree>
  <FileTree.Folder name="content">
    <FileTree.File name="index.mdx" />
    <FileTree.File name="about.mdx" />
  </FileTree.Folder>
</FileTree>
```

더 많은 컴포넌트: https://nextra.site/docs/built-ins

### 5) 사이드바와 `_meta.json`

- `content/` 하위 각 폴더에 `_meta.json`을 두면 사이드바 순서/제목/숨김 등을 제어할 수 있습니다.

`content/_meta.json` 예시
```json
{
  "index": {
    "title": "소개"
  },
  "about": {
    "title": "About"
  },
  "built-in-components": {
    "title": "빌트인"
  }
}
```

- 키는 파일명(확장자 제외). `title`로 표시명을 바꾸고, 필요 시 `display: false`로 숨길 수 있습니다.

### 6) 문서 구조 컨벤션(추천)

- 파일명은 소문자-케밥케이스(`my-feature.mdx`). URL과 사이드바 키가 일관됩니다.
- 각 섹션은 의미 있는 H2/H3로 나눠 자동 TOC 품질을 높입니다.
- 긴 페이지는 분할하고 상위 폴더에 `_meta.json`으로 순서를 명시합니다.
- 이미지/다이어그램은 `public/images/<도메인>/<파일>`에 저장합니다.
- 코드블록은 언어를 반드시 명시합니다.
- 내부 링크는 최대한 절대경로(`/경로`)를 사용해 이동의 안정성을 높입니다.

---

## 검색(Pagefind) 설정과 동작

- `package.json`:

```json
{
  "scripts": {
    "build": "next build",
    "postbuild": "pagefind --site .next/server/app --output-path public/_pagefind"
  }
}
```

- 빌드 후 `.next/server/app`을 소스로 인덱스를 만들어 `public/_pagefind`에 출력합니다.
- 로컬 개발 중에는 인덱스가 최신이 아닐 수 있으니 검색 검증은 `build` 결과를 기준으로 합니다.

---

## 배포/Export 팁

- 정적 Export가 필요하면 `next.config.mjs`에 `output: 'export'`를 고려하세요(프로젝트 요구사항에 맞게).
- 정적 Export 시 일부 동적 기능이 제한될 수 있습니다.

---

## 자주 겪는 이슈 체크리스트

- 사이드바 순서가 이상함: 해당 폴더의 `_meta.json` 구성 누락 여부 확인 (없으면 알파벳 순 정렬)
- 검색이 비어 있음: `next build` 후 `postbuild`가 정상 실행되어 `public/_pagefind`가 생성되었는지 확인
- 이미지가 안 보임: 경로가 `/images/...`(public 기준)인지 확인
- MDX 컴포넌트 충돌: `mdx-components.js`에서 겹치는 키가 없는지 확인하고, 필요한 경우 고유 이름으로 등록
- TypeScript 에러: `next-env.d.ts`가 존재하는지 확인 (자동 생성 파일)
- 빌드 실패: Node.js 18+ 및 pnpm 설치 확인

---

## 빠른 작성 예시

`content/getting-started.mdx`
```mdx
---
title: 시작하기
description: 프로젝트 구조와 빌드/검색 흐름을 소개합니다.
---

import { Callout } from 'nextra/components'

# 시작하기

이 문서는 레포의 구조와 Nextra 4의 핵심 개념을 간단히 안내합니다.

<Callout type="info">문서는 모두 content/ 아래에 위치합니다.</Callout>
```

`content/_meta.json`
```json
{
  "index": { "title": "홈" },
  "getting-started": { "title": "시작하기" },
  "about": { "title": "About" }
}
```

---

## 버전 정보 및 호환성

### 현재 사용 버전 (2024년 기준)
```json
{
  "dependencies": {
    "next": "^15.3.4",
    "nextra": "^4.2.17",
    "nextra-theme-docs": "^4.2.17",
    "react": "^19.0.0",
    "react-dom": "^19.0.0"
  },
  "devDependencies": {
    "@types/node": "22.13.10",
    "@types/react": "19.0.10",
    "pagefind": "^1.3.0",
    "typescript": "^5.7.3"
  }
}
```

### 요구사항
- Node.js 18.0.0 이상
- pnpm 권장 (npm, yarn도 사용 가능)
- TypeScript 5.0+ (선택사항)

## 배포 가이드

### 정적 Export

`next.config.mjs`에 추가:
```js
export default withNextra({
  output: 'export'  // 정적 HTML 출력
})
```

주의: 정적 export 시 일부 동적 기능 제한

## TypeScript/JavaScript 혼용

이 프로젝트는 TypeScript와 JavaScript를 함께 사용합니다:

- **TypeScript 파일**: `app/layout.tsx`, `tsconfig.json` 설정
- **JavaScript 파일**: `app/[[...mdxPath]]/page.jsx`, `mdx-components.js`
- **자동 타입 체크**: `strict: false`로 점진적 마이그레이션 지원
- **Path 별칭**: `@/*`로 프로젝트 루트 참조 가능

## 최소 구조 Nextra 4 프로젝트

가장 간단한 Nextra 4 프로젝트 구조:
```
my-docs/
├── app/
│   ├── [[...mdxPath]]/
│   │   └── page.jsx
│   └── layout.tsx
├── content/
│   └── index.mdx
├── mdx-components.js
├── next.config.mjs
└── package.json
```

이 5개 파일만으로도 작동하는 문서 사이트 구축 가능

## 참고 링크

- Nextra 공식 문서: https://nextra.site
- 테마(Docs) 옵션: https://nextra.site/docs/docs-theme/configuration
- 빌트인 컴포넌트: https://nextra.site/docs/built-ins
- Pagefind: https://pagefind.app
- 이 스타터 템플릿: https://github.com/phucbm/nextra-docs-starter
- 라이브 데모: https://nextra-docs-starter.vercel.app

이 문서만으로 레포 복제 → 문서 작성 → 빌드/검색까지 한 번에 진행할 수 있도록 구성했습니다. 추가로 팀 규칙이나 린팅/프리티어 규칙이 있다면 본 컨벤션 섹션에 합쳐 운영하세요.
