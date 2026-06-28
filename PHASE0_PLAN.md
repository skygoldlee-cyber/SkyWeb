# Phase 0 실행계획 — TypeScript / React / Next.js 학습

> **상위 문서**: PLAN.md §10
> **기간(현실안)**: 2–3주
> **완료 기준**: 간단한 **인증 + 목록 SPA**를 외부 가이드 없이 직접 구현
> **성격**: 구현이 아니라 **역량 확보** 단계. 이후 Phase 4~7(프론트)의 선행 학습.
>
> **이번 개정 요지**: 최신 Next.js(15+)·shadcn CLI에 맞춰 에러 나는 코드를 수정하고,
> 인증을 **httpOnly 쿠키 기반**으로 재작성했으며(localStorage 제거), **Zod**(JS의 Pydantic) 섹션을 추가했다.

---

## 0. 목표 한 줄

Python 주력 개발자가 프론트엔드를 **혼자 만들고 오래 유지보수**할 수 있을 만큼의
TypeScript / React / Next.js(App Router) + 쿠키 인증 패턴을 손에 익힌다.

---

## 1. 학습 전제

### Python 개발자가 TypeScript를 배울 때의 장점
- **타입 시스템**: Python type hints + Pydantic에 익숙하므로 TS 타입 시스템은 친숙한 영역
- **함수형 패턴**: `lambda`, `map`/`filter`와 유사
- **모듈 시스템**: `import`/`export`와 유사
- **런타임 검증**: Pydantic으로 경계를 지키던 감각이 그대로 **Zod**로 옮겨간다

### Python과의 주요 차이 (사전 인지)

| 개념 | Python | TypeScript |
|---|---|---|
| 타입 | 런타임 동적 타이핑 | 컴파일 타임 정적 타이핑(빌드 후 타입은 사라짐) |
| 변수 선언 | `x = 10` | `const x = 10` (재할당 필요 시 `let x = 10`) |
| 함수 선언 | `def func() -> int:` | `function func(): number {}` |
| 화살표 함수 | `lambda a, b: a + b` | `const f = (a: number, b: number): number => a + b` |
| 클래스 | `class MyClass:` | `class MyClass {}` |
| 비동기 | `async def` / `Future` | `async function` / `Promise` |
| 리스트 | `list[int]` | `number[]` |
| 딕셔너리 | `dict[str, int]` | `Record<string, number>` |

> **주의(흔한 오타)**: 함수 반환 타입은 `()` **뒤에** 붙는다.
> `function f(): number {}` (O) / `function f(): {}` (X — "빈 객체를 반환"이라는 다른 뜻),
> `const f = (): number => {}` (O) / `const f = (): => {}` (X — 문법 에러).

---

## 2. 선행 조건

- [ ] PLAN.md 부록 A(WSL2 개발환경) 확인
- [ ] 학습 시간 확보: SkinLens 본체·SkyPredictor 유지보수와 병행 → 하루 2~3시간 기준 버퍼 포함

---

## 3. 개발 환경 설정 (Week 1 시작 전 필수)

### 3.1 Node.js 설치

Node.js는 JavaScript/TypeScript의 런타임입니다. Python처럼 설치 후 터미널에서 `node` 명령어로 실행할 수 있습니다.

```bash
# 설치 확인
node --version  # v20.x 이상 권장
npm --version   # Node.js 설치 시 자동 포함
```

**설치 방법**:
- **Windows**: https://nodejs.org/ 에서 LTS 버전 다운로드 및 설치
- **WSL2 (권장)**: PLAN.md 부록 A 참조. WSL2 내에서 `curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -` 후 `sudo apt-get install -y nodejs`

> **Python과의 비교**: Node.js ≈ Python, npm ≈ pip

### 3.2 패키지 매니저 선택

| 매니저 | 설치 명령 | 특징 | 추천 |
|---|---|---|---|
| **npm** | Node.js에 포함 | 표준, 느림 | 기본값 |
| **yarn** | `npm install -g yarn` | 빠름, lock 파일 | 선택 |
| **pnpm** | `npm install -g pnpm` | 매우 빠름, 디스크 절약 | 고급 사용자 |

이 가이드는 **npm**을 기준으로 작성했습니다. 다른 매니저를 사용하려면 명령어만 변경하면 됩니다.

```bash
# npm 업그레이드
npm install -g npm@latest
```

### 3.3 TypeScript 설치 (전역)

프로젝트마다 설치할 수도 있지만, 전역 설치가 편리합니다.

```bash
npm install -g typescript
tsc --version  # 설치 확인
```

> **Python과의 비교**: `npm install -g typescript` ≈ `pip install typescript`

### 3.4 VS Code 확장 설치

VS Code는 JavaScript/TypeScript 개발에 가장 적합한 IDE입니다.

**필수 확장**:
- **ESLint** (`dbaeumer.vscode-eslint`) - 코드 품질 검사
- **Prettier** (`esbenp.prettier-vscode`) - 코드 포맷팅
- **TypeScript Importer** (`pmneo.tsimporter`) - 자동 import 추가
- **Auto Rename Tag** (`formulahendry.auto-rename-tag`) - HTML 태그 자동 수정
- **Tailwind CSS IntelliSense** (`bradlc.vscode-tailwindcss`) - Tailwind 자동완성

**설치 방법**: VS Code → 확장(Ctrl+Shift+X) → 검색 → 설치

### 3.5 첫 프로젝트 생성 (실습용)

학습을 위해 간단한 프로젝트를 먼저 생성해봅니다.

#### 옵션 A: Vite (React + TypeScript) - Week 1 실습용

```bash
npm create vite@latest my-react-app -- --template react-ts
cd my-react-app
npm install
npm run dev
```

브라우저에서 `http://localhost:5173` 열면 React 앱이 실행됩니다.

#### 옵션 B: Next.js (TypeScript + Tailwind) - Week 2 이후

```bash
npx create-next-app@latest my-next-app --typescript --tailwind --app
cd my-next-app
npm run dev
```

브라우저에서 `http://localhost:3000` 열면 Next.js 앱이 실행됩니다.

> **Python과의 비교**:
> - `npm create vite@latest` ≈ `django-admin startproject` 또는 `cookiecutter`
> - `npm install` ≈ `pip install -r requirements.txt`
> - `npm run dev` ≈ `python manage.py runserver` 또는 `uvicorn main:app --reload`

### 3.6 환경 변수 설정

`.env` 파일로 환경 변수를 관리합니다.

```bash
# .env.local (프로젝트 루트)
API_URL=http://localhost:8000
NEXT_PUBLIC_API_URL=http://localhost:8000
```

> **Python과의 비교**: `.env` ≈ `python-dotenv`의 `.env` 파일

### 3.7 Windows 환경 고려사항

PLAN.md 부록 A를 참조하세요. 권장 구성:

- **WSL2 + Docker Desktop**: 리눅스 환경에서 개발 (성능, 호환성 우수)
- **VS Code Remote WSL**: WSL2에서 직접 개발
- **프로젝트 위치**: `\\wsl$\Ubuntu\home\user\projects\...` (C:\ 드라이브 X)

**순수 Windows에서 개발 시 주의사항**:
- 파일 경경: `\` 대신 `/` 사용 (Node.js 호환성)
- 라인 엔딩: CRLF (Windows 기본값)
- 포트 충돌: 3000, 5173 등 사용 중인 포트 확인

### 3.8 개발 서버 실행 및 중지

```bash
# 실행
npm run dev

# 중지: Ctrl + C
```

### 3.9 기본 명령어 정리

| 명령 | 의미 | Python 대응 |
|---|---|---|
| `npm install` | 의존성 설치 | `pip install -r requirements.txt` |
| `npm run dev` | 개발 서버 실행 | `python manage.py runserver` |
| `npm run build` | 프로덕션 빌드 | `python setup.py build` |
| `npm test` | 테스트 실행 | `pytest` |
| `tsc` | TypeScript 컴파일 | `mypy` (타입 검사) |

### 체크리스트
- [ ] Node.js v20+ 설치 확인 (`node --version`)
- [ ] npm 최신 버전 확인 (`npm --version`)
- [ ] TypeScript 전역 설치 (`tsc --version`)
- [ ] VS Code 필수 확장 설치 (ESLint, Prettier, Tailwind)
- [ ] Vite 프로젝트 생성 및 실행 (`npm create vite@latest`)
- [ ] Next.js 프로젝트 생성 및 실행 (`npx create-next-app@latest`)
- [ ] 환경 변수 파일 `.env.local` 생성
- [ ] Windows 환경 고려사항 확인 (WSL2 권장)

---

## 4. 작업 단위 (주차별)

### 4.1 Week 1 — TypeScript 기초 + React 기본

#### Day 1-2: TypeScript 기초

##### 1. 변수와 타입
```typescript
let name: string = "John";
let age: number = 30;
let isActive: boolean = true;

let numbers: number[] = [1, 2, 3];
let strings: Array<string> = ["a", "b"];

// 객체 (Python dict가 아니라 dataclass/TypedDict에 가까움)
interface User {
  id: string;
  email: string;
  name?: string;   // ?는 옵셔널
}

const user: User = { id: "123", email: "test@example.com" };
```

##### 2. 함수
```typescript
function add(a: number, b: number): number {
  return a + b;
}

// 화살표 함수 (Python lambda와 유사)
const multiply = (a: number, b: number): number => a * b;

// 비동기 함수
async function fetchUser(): Promise<User> {
  const res = await fetch("/api/user");
  return res.json();
}
```

##### 3. 인터페이스 vs 타입
```typescript
interface Animal { name: string; }
interface Dog extends Animal { breed: string; }   // 확장

type ID = string | number;                          // 유니온
type Coordinate = [number, number];                 // 튜플
```

##### 4. 제네릭 (Python TypeVar/Generic과 유사)
```typescript
function identity<T>(arg: T): T { return arg; }
const num = identity<number>(42);
const str = identity("hello");   // 추론되므로 <string> 생략 가능
```

##### 실습
- [ ] `.ts` 파일 생성 및 `tsc`로 컴파일
- [ ] 타입을 명시한 계산기 함수 작성
- [ ] 인터페이스 정의 + 객체 생성

---

#### Day 2.5: Zod — 런타임 검증 (Pydantic의 TS판) ⭐

TS 타입은 **컴파일 후 사라져서 런타임에는 아무것도 검증하지 않는다.** API 응답·폼 입력처럼
**외부에서 들어오는 데이터**는 `as User` 같은 단언으로 믿으면 안 된다(실제 모양을 보장 못 함).
이때 쓰는 게 **Zod** — Pydantic처럼 "스키마로 런타임 검증 + 타입까지 도출"을 한 번에 한다.

```typescript
import { z } from "zod";

// Pydantic의 BaseModel에 대응
const UserSchema = z.object({
  id: z.string(),
  email: z.string().email(),
  name: z.string().optional(),
});

type User = z.infer<typeof UserSchema>;     // 스키마에서 타입 자동 도출

// 경계에서 검증: 통과하면 타입이 확정됨 (Pydantic의 .model_validate와 동일한 감각)
const data = UserSchema.parse(await res.json());   // 안 맞으면 ZodError
```

| Pydantic | Zod |
|---|---|
| `class U(BaseModel): ...` | `const U = z.object({ ... })` |
| `U.model_validate(data)` | `U.parse(data)` |
| `Optional[str]` | `z.string().optional()` |
| `Field(gt=0)` | `z.number().positive()` |
| `EmailStr` | `z.string().email()` |

> 원칙: **경계에서 Zod로 검증 → 내부에서는 TS 타입을 신뢰.** FastAPI에서 Pydantic 모델로 요청을 받던 흐름과 동일하다.

---

#### Day 3-4: React 기본

##### 1. 컴포넌트
```tsx
function Greeting({ name }: { name: string }) {
  return <h1>Hello, {name}!</h1>;
}

const Button = ({ onClick, children }: {
  onClick: () => void;
  children: React.ReactNode;
}) => <button onClick={onClick}>{children}</button>;
```

##### 2. State (useState)
```tsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState<number>(0);
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

##### 3. Props
```tsx
interface CardProps {
  title: string;
  description: string;
  onEdit?: () => void;
}

function Card({ title, description, onEdit }: CardProps) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <p>{description}</p>
      {onEdit && <button onClick={onEdit}>Edit</button>}
    </div>
  );
}
```

##### 4. Effect (useEffect)
```tsx
import { useEffect, useState } from "react";

function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then((res) => res.json())
      .then(setUser);
    return () => { /* 클린업 (언마운트/재실행 직전) */ };
  }, [userId]);   // 의존성 배열: userId가 바뀔 때 재실행

  if (!user) return <div>Loading...</div>;
  return <div>{user.name}</div>;
}
```

> 참고: 실무에서는 이런 수동 fetch+로딩+에러 처리를 **TanStack Query**가 대체한다(Week 3에서 도입).
> 지금은 원리를 익히기 위해 손으로 한 번 해본다.

##### 실습
- [ ] Vite로 프로젝트 생성: `npm create vite@latest my-app -- --template react-ts`
- [ ] 카운터, Todo 리스트(추가/삭제)
- [ ] useEffect로 API 데이터 가져오기 + Zod로 응답 검증

---

#### Day 5-7: React 심화

> **변경**: 기존 가이드의 React Router 실습은 제거했다. 목표가 Next.js라면 라우팅은
> Next의 App Router(파일 기반, Week 2)가 대체하므로 React Router 학습은 대부분 버려진다.

##### 1. Context API (전역 상태)
```tsx
import { createContext, useContext, useState } from "react";

interface AuthContextType {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => Promise<void>;
}

const AuthContext = createContext<AuthContextType | null>(null);

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  // 토큰을 JS에서 직접 다루지 않는다 (쿠키 기반 — Day 15-18 참고)
  const login = async (email: string, password: string) => {
    const res = await fetch("/api/auth/login", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email, password }),
    });
    const { user } = await res.json();
    setUser(user);
  };

  const logout = async () => {
    await fetch("/api/auth/logout", { method: "POST" });
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error("useAuth must be used within AuthProvider");
  return ctx;
}
```

##### 2. 조건부 렌더링 / 리스트 렌더링
```tsx
function UserStatus({ user }: { user: User | null }) {
  if (!user) return <div>Please login</div>;
  return <p>Welcome, {user.name}</p>;
}

function UserList({ users }: { users: User[] }) {
  return <ul>{users.map((u) => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

##### 실습
- [ ] AuthContext 구현
- [ ] 사용자 목록 페이지
- [ ] (라우팅은 Next에서 — 여기선 단일 화면으로 충분)

---

### 4.2 Week 2 — Next.js 기초

#### Day 8-9: 프로젝트 구조

##### 1. 생성 (TypeScript + Tailwind + App Router)
```bash
npx create-next-app@latest my-app --typescript --tailwind --app
```

##### 2. App Router 구조
```
app/
├── layout.tsx          # 루트 레이아웃
├── page.tsx            # 홈 (/)
├── auth/login/page.tsx        # /auth/login
├── auth/register/page.tsx     # /auth/register
├── analyses/[id]/page.tsx     # /analyses/:id
├── api/                # Route Handlers — BFF로 활용 (아래 설명)
└── globals.css
```

> **`api/`에 대한 정정**: 별도 FastAPI 백엔드가 있어도 Next의 Route Handler는 유용하다.
> **BFF(Backend-for-Frontend)** 역할로, 여기서 httpOnly 쿠키를 굽고, 백엔드 URL을 숨기고,
> CORS를 피하고, 토큰을 서버 쪽에서 다룬다. (Day 15-18에서 실제로 사용)

##### 3. 레이아웃과 페이지
```tsx
// app/layout.tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ko">
      <body>
        <nav>Navigation</nav>
        {children}
      </body>
    </html>
  );
}
```

##### 4. 동적 라우팅 — `params`는 이제 **Promise** ⭐ 수정
```tsx
// app/analyses/[id]/page.tsx
// Next.js 15+에서 params는 Promise이므로 await로 풀어야 한다 (14 방식은 타입 에러)
export default async function AnalysisDetail({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  return <div>Analysis ID: {id}</div>;
}

// 클라이언트 컴포넌트라면:
//   'use client';
//   import { use } from "react";
//   const { id } = use(params);
```

##### 실습
- [ ] Next 프로젝트 생성, 레이아웃 구조 파악
- [ ] 동적 라우팅 페이지(`await params`) 작성

---

#### Day 10-11: 데이터 가져오기 + 환경변수

##### 0. 환경변수 ⭐ 신규
URL을 하드코딩하지 않는다.
```bash
# .env.local
API_URL=http://localhost:8000              # 서버 전용 (브라우저 노출 X)
NEXT_PUBLIC_API_URL=http://localhost:8000  # 클라이언트에서도 필요할 때만
```
- 서버 컴포넌트/Route Handler: `process.env.API_URL`
- 클라이언트 컴포넌트: `process.env.NEXT_PUBLIC_API_URL` (반드시 `NEXT_PUBLIC_` 접두사)

##### 1. Server Component (기본) — 쿠키를 읽어 인증 요청
```tsx
// app/analyses/page.tsx  (기본이 Server Component)
import { cookies } from "next/headers";

async function getAnalyses() {
  const token = (await cookies()).get("access_token")?.value;  // Next 15: cookies()도 async
  const res = await fetch(`${process.env.API_URL}/api/analyses`, {
    headers: { Authorization: `Bearer ${token}` },
    cache: "no-store",
  });
  return res.json();
}

export default async function AnalysesPage() {
  const analyses = await getAnalyses();
  return <ul>{analyses.map((a: any) => <li key={a.id}>{a.id}</li>)}</ul>;
}
```

> 이게 **쿠키 기반 인증을 택한 이유**다. 토큰이 httpOnly 쿠키에 있으면 서버 컴포넌트가
> 그것을 읽어 백엔드에 인증 요청을 보낼 수 있다. (localStorage 토큰은 서버에서 못 읽어 모델이 깨진다.)

##### 2. Client Component (인터랙티브)
```tsx
"use client";
import { useState } from "react";
export default function InteractiveButton() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

##### 3. 로딩/에러 (`loading.tsx`, `error.tsx`)
```tsx
// app/analyses/loading.tsx
export default function Loading() { return <div>Loading...</div>; }

// app/analyses/error.tsx
"use client";
export default function Error({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div>
      <h2>Error: {error.message}</h2>
      <button onClick={reset}>Retry</button>
    </div>
  );
}
```

##### 실습
- [ ] Server Component로 데이터 가져오기(+쿠키)
- [ ] Client Component 인터랙션
- [ ] 로딩/에러 상태

---

#### Day 12-14: TailwindCSS + shadcn/ui

##### 1. Tailwind 기본
```tsx
<div className="flex items-center justify-between p-4 bg-white rounded-lg shadow">
  <h1 className="text-xl font-bold">Title</h1>
  <button className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600">Button</button>
</div>
```

##### 2. shadcn/ui 설치 — 패키지명은 **`shadcn`** ⭐ 수정
```bash
# 'shadcn-ui'는 폐기됨. 'shadcn'으로 변경됨.
npx shadcn@latest init
npx shadcn@latest add button card input form
```

##### 3. 폼: shadcn `form` = react-hook-form + Zod ⭐ 일관성 수정
shadcn의 `form`은 **react-hook-form + Zod** 기반이다. 앞서 배운 Zod가 여기서 그대로 쓰인다.
(raw `useState` 폼은 학습용으로만; 실제 폼은 아래 패턴 권장)

```tsx
"use client";
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

const LoginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});
type LoginInput = z.infer<typeof LoginSchema>;

export function LoginForm() {
  const form = useForm<LoginInput>({ resolver: zodResolver(LoginSchema) });

  const onSubmit = async (values: LoginInput) => {
    await fetch("/api/auth/login", {   // BFF 라우트 (쿠키 설정은 서버에서)
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(values),
    });
  };

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      <input {...form.register("email")} type="email" placeholder="Email" />
      <input {...form.register("password")} type="password" placeholder="Password" />
      <button type="submit">Login</button>
    </form>
  );
}
```

##### 실습
- [ ] Tailwind 클래스 익히기
- [ ] `npx shadcn@latest add ...`로 컴포넌트 설치
- [ ] react-hook-form + Zod로 로그인 폼

---

### 4.3 Week 3 — 실전: 인증 + 목록 SPA

#### Day 15-18: 쿠키 기반 인증 (BFF) ⭐ 재작성

> **변경 핵심**: JWT를 **localStorage에 저장하지 않는다.** localStorage는 XSS에 토큰이 노출되기 쉽고,
> 이 서비스는 얼굴 이미지(개인정보보호법상 민감정보 가능성)를 다루므로 더 보수적으로 간다.
> 대신 Next Route Handler에서 받은 토큰을 **httpOnly 쿠키**로 굽는다(=JS에서 못 읽음 → XSS에 강함).

##### 흐름
```
브라우저 ──(email/pw)──▶ Next Route Handler(/api/auth/login)
                              │  FastAPI에 로그인 요청
                              ▼
                         FastAPI ──(JWT)──▶ Route Handler
                              │  httpOnly 쿠키로 Set-Cookie
                              ▼
                         브라우저(쿠키 보관, JS 접근 불가)
서버 컴포넌트/Route Handler가 이후 요청 때 쿠키를 읽어 FastAPI에 Authorization 전달
```

##### 로그인 라우트 (쿠키 굽기)
```ts
// app/api/auth/login/route.ts
import { NextResponse } from "next/server";

export async function POST(request: Request) {
  const body = await request.json();

  const res = await fetch(`${process.env.API_URL}/api/auth/login`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(body),
  });

  if (!res.ok) {
    return NextResponse.json({ error: "로그인 실패" }, { status: 401 });
  }

  const data = await res.json();          // { access_token, user }
  const response = NextResponse.json({ user: data.user });
  response.cookies.set("access_token", data.access_token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === "production",
    sameSite: "lax",
    path: "/",
    maxAge: 60 * 30,                       // 30분
  });
  return response;
}
```

##### 로그아웃 라우트 (쿠키 삭제)
```ts
// app/api/auth/logout/route.ts
import { NextResponse } from "next/server";

export async function POST() {
  const response = NextResponse.json({ ok: true });
  response.cookies.delete("access_token");
  return response;
}
```

##### 보호 라우트 가드 (middleware)
```ts
// middleware.ts
import { NextResponse, type NextRequest } from "next/server";

export function middleware(req: NextRequest) {
  const token = req.cookies.get("access_token")?.value;
  if (!token) return NextResponse.redirect(new URL("/auth/login", req.url));
  return NextResponse.next();
}

export const config = { matcher: ["/analyses/:path*", "/profile"] };
```

##### 체크리스트
- [ ] 회원가입 페이지(`/auth/register`)
- [ ] 로그인 페이지(`/auth/login`) + react-hook-form + Zod
- [ ] `/api/auth/login`·`/logout` Route Handler (httpOnly 쿠키)
- [ ] AuthContext(현재 사용자 상태만 보관, 토큰은 JS가 안 만짐)
- [ ] middleware로 보호 라우트 가드

---

#### Day 19-21: API 연동 + 히스토리

> **쿠키 만료/갱신**: 현재는 30분 만료로 설정. v2에서 refreshToken 패턴 도입 예정.
> (access_token 만료 시 백그라운드 갱신 → 사용자 무중단 경험)

##### 1. TanStack Query 기본 — 클라이언트 측 캐싱/리패칭

서버 컴포넌트로 초기 데이터를 가져오면 좋지만, 클라이언트에서 **캐싱·리패칭·무효화**가 필요할 때
TanStack Query가 표준 솔루션이다. (Python의 requests.Session + 캐싱과 유사한 역할)

```bash
npm install @tanstack/react-query
```

```tsx
// app/providers.tsx (루트 레이아웃에서 감싸기)
"use client";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { useState } from "react";

export function Providers({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(() => new QueryClient());
  return <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>;
}
```

```tsx
// app/layout.tsx
import { Providers } from "./providers";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ko">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

##### 2. useQuery — 데이터 가져오기 (GET)

```tsx
"use client";
import { useQuery } from "@tanstack/react-query";

const AnalysisSchema = z.object({
  id: z.string(),
  status: z.enum(["queued", "processing", "done", "failed"]),
  overall_score: z.number().optional(),
  // ... 기타 필드
});
type Analysis = z.infer<typeof AnalysisSchema>;

async function fetchAnalyses(): Promise<Analysis[]> {
  const res = await fetch("/api/analyses");  // BFF 라우트 (쿠키를 백엔드에 전달)
  if (!res.ok) throw new Error("Failed to fetch");
  const data = await res.json();
  return z.array(AnalysisSchema).parse(data);  // Zod 검증
}

export function AnalysesList() {
  const {
    data: analyses,
    isLoading,
    error,
    refetch,
  } = useQuery({
    queryKey: ["analyses"],           // 캐시 키 (Python의 cache_key와 유사)
    queryFn: fetchAnalyses,           // 데이터 가져오기 함수
    staleTime: 5 * 60 * 1000,        // 5분 동안 "신선"으로 간주 (재요청 안 함)
    gcTime: 10 * 60 * 1000,           // 10분 후 캐시 삭제 (이전 cacheTime)
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <ul>
      {analyses?.map((a) => (
        <li key={a.id}>{a.status} — {a.overall_score}</li>
      ))}
    </ul>
  );
}
```

##### 3. useMutation — 데이터 변경 (POST/PUT/DELETE)

```tsx
"use client";
import { useMutation, useQueryClient } from "@tanstack/react-query";

const UploadSchema = z.object({
  image: z.instanceof(File),
});
type UploadInput = z.infer<typeof UploadSchema>;

async function uploadImage(input: UploadInput): Promise<Analysis> {
  const formData = new FormData();
  formData.append("image", input.image);

  const res = await fetch("/api/analyses", { method: "POST", body: formData });
  if (!res.ok) throw new Error("Upload failed");
  const data = await res.json();
  return AnalysisSchema.parse(data);
}

export function ImageUploader() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: uploadImage,
    onSuccess: (data) => {
      // 성공 시 관련 캐시 무효화 → 자동 재요청
      queryClient.invalidateQueries({ queryKey: ["analyses"] });
      console.log("Uploaded:", data.id);
    },
    onError: (error) => {
      console.error("Upload failed:", error);
    },
  });

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const form = e.currentTarget;
    const fileInput = form.querySelector<HTMLInputElement>('input[type="file"]');
    if (fileInput?.files?.[0]) {
      mutation.mutate({ image: fileInput.files[0] });
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="file" accept="image/*" />
      <button type="submit" disabled={mutation.isPending}>
        {mutation.isPending ? "Uploading..." : "Upload"}
      </button>
      {mutation.error && <div>Error: {mutation.error.message}</div>}
    </form>
  );
}
```

##### 4. 에러 처리 + 401 리다이렉트

```tsx
"use client";
import { useQuery } from "@tanstack/react-query";
import { useRouter } from "next/navigation";

async function fetchProtectedData() {
  const res = await fetch("/api/protected");
  if (res.status === 401) {
    throw new Error("UNAUTHORIZED");
  }
  if (!res.ok) throw new Error("Failed");
  return res.json();
}

export function ProtectedComponent() {
  const router = useRouter();

  const { data, error } = useQuery({
    queryKey: ["protected"],
    queryFn: fetchProtectedData,
    retry: false,  // 401 같은 인증 에러는 재시도 안 함
  });

  if (error?.message === "UNAUTHORIZED") {
    router.push("/auth/login");  // 로그인 페이지로 리다이렉트
    return null;
  }

  if (error) return <div>Error: {error.message}</div>;
  return <div>{JSON.stringify(data)}</div>;
}
```

##### 5. 페이지네이션

```tsx
"use client";
import { useQuery } from "@tanstack/react-query";

async function fetchAnalysesPage(page: number): Promise<{ data: Analysis[]; total: number }> {
  const res = await fetch(`/api/analyses?page=${page}`);
  return res.json();
}

export function AnalysesWithPagination() {
  const [page, setPage] = useState(1);

  const { data, isLoading } = useQuery({
    queryKey: ["analyses", page],
    queryFn: () => fetchAnalysesPage(page),
  });

  return (
    <div>
      {isLoading && <div>Loading...</div>}
      <ul>
        {data?.data.map((a) => <li key={a.id}>{a.id}</li>)}
      </ul>
      <button onClick={() => setPage((p) => Math.max(1, p - 1))} disabled={page === 1}>
        Previous
      </button>
      <span>Page {page}</span>
      <button onClick={() => setPage((p) => p + 1)} disabled={!data || data.data.length === 0}>
        Next
      </button>
    </div>
  );
}
```

##### 체크리스트
- [ ] TanStack Query 설치 + QueryClientProvider 설정
- [ ] useQuery로 히스토리 목록 가져오기 (+ Zod 검증)
- [ ] useMutation으로 이미지 업로드 (+ 캐시 무효화)
- [ ] 401 에러 시 로그인 리다이렉트
- [ ] 페이지네이션 구현
- [ ] 서버 컴포넌트에서 쿠키 읽어 FastAPI 호출 (Day 10-11 패턴)
- [ ] 히스토리 목록(`/analyses`) + 상세(`/analyses/[id]`, `await params`)

---

## 5. 학습 자료

1. **TypeScript Handbook**: https://www.typescriptlang.org/docs/
2. **React 공식 문서(Learn React)**: https://react.dev/
3. **Next.js 문서**: https://nextjs.org/docs
4. **Zod**: https://zod.dev/
5. **TanStack Query**: https://tanstack.com/query/latest
6. **TailwindCSS**: https://tailwindcss.com/docs
7. **shadcn/ui**: https://ui.shadcn.com/docs

---

## 6. Python 개발자를 위한 팁

### 타입 매핑
- `Optional[str]` → `string | undefined`(또는 `| null`)
- `list[int]` → `number[]`
- `dict[str, int]` → `Record<string, number>`
- `Union[A, B]` → `A | B`

### 비동기
- `async/await`는 거의 동일, `Future` → `Promise`

### 클래스 vs 인터페이스 vs Zod
- Python은 클래스/Pydantic으로 구조 정의
- TS `interface`는 **컴파일 후 사라지는** 타입 선언(런타임 검증 X)
- 런타임 검증이 필요하면 **Zod**(=Pydantic 역할)

### 디버깅
- VS Code 디버거, `console.table`/`console.group`, React DevTools

---

## 7. 자주 하는 실수

### 1. `useEffect` 의존성 누락
```tsx
useEffect(() => { fetchData(userId); }, []);        // ❌ userId 변경 시 재실행 안 됨
useEffect(() => { fetchData(userId); }, [userId]);  // ✅
```

### 2. 상태 직접 수정
```tsx
user.name = "New";                    // ❌ 리렌더 안 됨
setUser({ ...user, name: "New" });    // ✅ 새 객체
```

### 3. 이벤트 핸들러에 async 함수 즉시 실행
```tsx
<button onClick={fetchData()}>X</button>        // ❌ 렌더 시 즉시 호출됨
<button onClick={() => fetchData()}>X</button>  // ✅
```

### 4. 타입 단언 남용 → **Zod로 검증**
```tsx
const user = data as User;            // ❌ 런타임 보장 없음
const user = UserSchema.parse(data);  // ✅ 검증 + 타입 확정
```

### 5. `params`를 동기 객체로 취급 (Next 15+)
```tsx
function Page({ params }: { params: { id: string } }) { params.id }   // ❌ 타입 에러
async function Page({ params }: { params: Promise<{ id: string }> }) { // ✅
  const { id } = await params;
}
```

---

## 8. 학습 체크리스트

**TypeScript**: 기본 타입 · 인터페이스/타입 · 함수 타입 · 제네릭 · 유니온 · **Zod 검증**
**React**: 컴포넌트 · useState · useEffect · Props · Context · 폼(rhf+zod)
**Next.js**: App Router · Server/Client 구분 · 데이터 가져오기 · 동적 라우팅(`await params`) · Route Handler(BFF) · 쿠키 인증 · loading/error
**스타일링**: Tailwind · shadcn/ui(`shadcn@latest`) · 반응형

---

## 9. 산출물

1. **인증 + 목록 SPA** (Next.js App Router)
   - 회원가입/로그인/로그아웃 (httpOnly 쿠키)
   - 보호 라우트 가드(middleware)
   - 목록 조회(useQuery + Zod) + 페이지네이션
   - *백엔드는 mock 또는 임시 FastAPI 스텁으로 대체 가능*
2. 학습 메모(선택): TS↔Python 타입 매핑, 자주 한 실수 기록

---

## 10. 완료 기준 (Definition of Done)

- [ ] 공식 문서 외 가이드 없이 위 SPA의 **로그인→목록→로그아웃** 플로우를 직접 구현
- [ ] httpOnly 쿠키 인증 흐름(BFF)을 말로 설명 가능
- [ ] Server Component / Client Component 구분 기준을 설명 가능
- [ ] Zod 검증을 "경계에서" 적용하는 이유를 설명 가능

---

## 11. 리스크 / 주의

- **함정 1**: localStorage에 토큰 저장 → XSS 취약. 처음부터 쿠키로 간다.
- **함정 2**: `params`/`cookies()`를 동기로 취급 → Next 15+ 타입 에러. `await` 필수.
- **함정 3**: 타입 단언(`as User`) 남용 → 런타임 보장 없음. `Schema.parse()`로.
- **함정 4**: `useEffect` 의존성 누락 → 갱신 누락. 의존성 배열 점검.
- **시간 리스크**: Week 3가 가장 무겁다. 막히면 공식문서→SO→AI도구→GitHub Issues 순.

---

## 12. 다음 단계 연결

- Phase 0의 SPA는 **버리는 코드가 아니다.** Phase 4(Next.js 인증/레이아웃)에서
  쿠키 인증·middleware·TanStack Query 패턴을 그대로 재사용한다.
- 백엔드 mock으로 연습한 부분은 Phase 1 FastAPI가 완성되면 실제 엔드포인트로 교체.
