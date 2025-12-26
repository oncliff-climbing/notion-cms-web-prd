# DevHub 프로젝트 개발 표준

> 이 문서는 AI Agent가 DevHub 프로젝트에서 작업할 때 반드시 따라야 하는 규칙을 정의합니다.

## 프로젝트 개요

- **프로젝트명**: DevHub (개발자 지식 허브)
- **기술 스택**: Next.js 15, TypeScript, TailwindCSS v4, shadcn/ui, Notion API
- **핵심 기능**: Notion CMS 기반 프로젝트 포트폴리오 및 학습 리소스 통합 관리
- **언어**: 한국어 (코드 주석, 문서, 커밋 메시지 모두 한국어)

---

## 프로젝트 아키텍처

### 필수 디렉토리 구조

```
notion-cms-web-prd/
├── app/
│   ├── page.tsx                    # 홈페이지
│   ├── projects/
│   │   ├── page.tsx               # 프로젝트 목록
│   │   └── [id]/page.tsx          # 프로젝트 상세
│   ├── resources/
│   │   ├── page.tsx               # 리소스 목록
│   │   └── [id]/page.tsx          # 리소스 상세
│   ├── layout.tsx                  # 루트 레이아웃
│   └── globals.css                 # TailwindCSS 설정 (중요!)
├── components/
│   ├── project-card.tsx            # 프로젝트 카드
│   ├── resource-card.tsx           # 리소스 카드
│   ├── markdown-renderer.tsx       # 마크다운 렌더링
│   ├── layout/                     # 레이아웃 컴포넌트
│   └── ui/                         # shadcn/ui 컴포넌트
├── lib/
│   ├── notion.ts                   # Notion 클라이언트 설정
│   ├── notion-api.ts               # API 호출 함수
│   ├── notion-utils.ts             # Notion 응답 변환 헬퍼
│   ├── types.ts                    # TypeScript 타입 정의
│   └── utils.ts                    # cn() 유틸리티
└── docs/
    ├── PRD.md                      # 제품 요구사항 문서
    └── ROADMAP.md                  # 개발 로드맵
```

### 아키텍처 원칙

- **App Router 사용**: pages/ 디렉토리 절대 생성 금지
- **Server Component 우선**: Client Component는 인터랙션 필요 시에만 사용
- **데이터 소스**: 모든 콘텐츠는 Notion API에서 가져옴 (하드코딩 금지)

---

## Notion API 통합 규칙

### 환경변수 규칙

**필수 환경변수 (3개):**
- `NOTION_API_KEY`: Notion Integration Secret Token
- `NOTION_PROJECTS_DATABASE_ID`: Projects 데이터베이스 ID
- `NOTION_RESOURCES_DATABASE_ID`: Resources 데이터베이스 ID

**규칙:**
- `.env.local`에 실제 값 저장
- `.env.example`에 샘플 형식만 제공 (실제 값 제외)
- 환경변수 추가/변경 시 두 파일 모두 업데이트 필수

**예시:**
```env
# .env.local (실제 값)
NOTION_API_KEY=secret_abc123xyz

# .env.example (샘플)
NOTION_API_KEY=secret_xxxxxxxxxxxxx
```

### API 호출 규칙

**Rate Limit 준수:**
- Notion API는 3 requests/second 제한
- 연속 요청 시 300ms 지연 추가 필수
- 100개 이상 아이템 시 페이지네이션 처리

**파일별 역할 분리:**
- `lib/notion.ts`: Notion 클라이언트 초기화, DATABASES 상수 정의만
- `lib/notion-api.ts`: 모든 API 호출 함수 (getAllProjects, getProjectById 등)
- `lib/notion-utils.ts`: Notion 응답 → 우리 타입 변환 헬퍼 함수

**필수 API 함수 목록:**
```typescript
// Projects
getAllProjects(): Promise<Project[]>
getProjectById(id: string): Promise<ProjectDetail>
getProjectsByTechStack(techStack: string): Promise<Project[]>
getRecentProjects(limit: number): Promise<Project[]>

// Resources
getAllResources(): Promise<Resource[]>
getResourceById(id: string): Promise<ResourceDetail>
getResourcesByCategory(category: string): Promise<Resource[]>
getResourcesByType(type: string): Promise<Resource[]>
getFeaturedResources(limit: number): Promise<Resource[]>
```

### 이미지 처리 규칙

**⚠️ 중요: Notion 이미지 URL은 1시간 후 만료됨**

**해결 방법:**
1. Next.js Image 컴포넌트 사용 (우선)
2. 외부 이미지 호스팅으로 마이그레이션 (Cloudinary, S3)
3. ISR로 주기적 재생성

**금지:**
- ❌ Notion 이미지 URL을 로컬 변수에 저장
- ❌ 빌드 타임에만 이미지 URL 가져오기 (SSG만 사용 시)

### Relation 데이터 처리 규칙

**문제:** Notion Relation은 ID 배열만 반환

**해결:**
- 별도 쿼리로 관련 데이터 fetch (populate)
- 상세 페이지에서 전체 객체 배열로 변환

**예시:**
```typescript
// ❌ 잘못된 방법
const project = await getProjectById(id); // relatedResourceIds: string[]

// ✅ 올바른 방법
const project = await getProjectById(id);
const relatedResources = await Promise.all(
  project.relatedResourceIds.map(id => getResourceById(id))
);
```

---

## TypeScript 타입 규칙

### 필수 타입 정의 (lib/types.ts)

**기본 타입:**
```typescript
export interface Project {
  id: string;
  name: string;
  description: string;
  techStack: string[];
  thumbnail: string | null;
  githubUrl: string | null;
  demoUrl: string | null;
  status: 'completed' | 'in_progress' | 'planned';
  completedDate: string | null;
  relatedResourceIds: string[];
  slug: string | null;
}

export interface Resource {
  id: string;
  name: string;
  description: string;
  url: string;
  category: 'Frontend' | 'Backend' | 'Design' | 'DevOps' | 'General';
  type: '문서' | '도구' | '튜토리얼' | '라이브러리';
  rating: '⭐' | '⭐⭐' | '⭐⭐⭐';
  tags: string[];
  usedInProjectIds: string[];
  addedDate: string;
}
```

**상세 페이지 타입 (Relation populate):**
```typescript
export interface ProjectDetail extends Project {
  content: string; // 마크다운 변환된 콘텐츠
  relatedResources: Resource[]; // ID 배열 → 전체 객체 배열
}

export interface ResourceDetail extends Resource {
  usedInProjects: Project[]; // ID 배열 → 전체 객체 배열
}
```

### 타입 안전성 규칙

- **모든 Notion API 응답은 타입 정의 필수**
- **타입 가드 사용하여 unknown 응답 검증**
- **strict 모드 활성화 상태 유지 (tsconfig.json)**
- **any 타입 절대 사용 금지**

---

## TailwindCSS v4 규칙

### ⚠️ 중요: tailwind.config 파일 없음

**이 프로젝트는 TailwindCSS v4 CSS 기반 설정 사용**

**절대 금지:**
- ❌ `tailwind.config.js` 생성
- ❌ `tailwind.config.ts` 생성
- ❌ PostCSS 설정 변경 (`@tailwindcss/postcss` 플러그인만 사용)

### 스타일 변경 방법

**모든 스타일 설정은 `app/globals.css`에서만 수정:**

```css
@theme inline {
  /* 여기에 커스텀 색상, border-radius 등 정의 */
  --radius-sm: calc(var(--radius) - 4px);
  --color-primary: oklch(0.205 0 0);
}
```

**다크모드 설정:**
```css
@custom-variant dark (&:is(.dark *));

.dark {
  --background: oklch(0.145 0 0);
  /* 다크모드 색상 정의 */
}
```

### 색상 규칙

**OKLCH 색상 사용 필수:**
- 라이트/다크 모드 간 더 나은 지각적 균일성 제공
- 색상 대비: WCAG AA 기준 4.5:1 이상

**예시:**
```css
/* ✅ 올바른 방법 */
--primary: oklch(0.205 0 0);

/* ❌ 잘못된 방법 */
--primary: #000000;
```

### 컨테이너 스타일 규칙

**일관된 중앙 정렬:**
- 모든 페이지 컨테이너: `mx-auto max-w-screen-2xl px-4`
- 예외 없이 이 패턴 사용

---

## shadcn/ui 컴포넌트 규칙

### 컴포넌트 추가 방법

**명령:**
```bash
npx shadcn@latest add [component-name]
```

**설정 (components.json):**
- 스타일: `new-york`
- RSC: `true`
- 아이콘: `lucide-react`만 사용

### 별칭 규칙

**components.json 별칭:**
```json
{
  "components": "@/components",
  "utils": "@/lib/utils",
  "ui": "@/components/ui",
  "lib": "@/lib",
  "hooks": "@/hooks"
}
```

**규칙:**
- 컴포넌트 import 시 별칭 사용 필수
- `@/ui/button` 형태로 shadcn/ui 컴포넌트 import

---

## 페이지 구현 규칙

### App Router 필수 함수

**상세 페이지 (동적 라우트):**
```typescript
// app/projects/[id]/page.tsx

// 1. generateStaticParams 필수 (SSG)
export async function generateStaticParams() {
  const projects = await getAllProjects();
  return projects.map((project) => ({ id: project.id }));
}

// 2. generateMetadata 필수 (SEO)
export async function generateMetadata({ params }: { params: { id: string } }) {
  const project = await getProjectById(params.id);
  return {
    title: project.name,
    description: project.description,
  };
}
```

### 필터링 규칙

**URL Query Parameter 사용:**
- 프로젝트 기술스택: `?tech=React`
- 리소스 카테고리: `?category=Frontend`
- 리소스 타입: `?type=문서`
- 다중 필터: `?category=Frontend&type=문서`

**Client Component에서 처리:**
```typescript
'use client';
import { useSearchParams } from 'next/navigation';

export function ProjectsList({ projects }) {
  const searchParams = useSearchParams();
  const tech = searchParams.get('tech');
  // 필터링 로직
}
```

### 404 처리 규칙

**존재하지 않는 ID:**
- `notFound()` 함수 호출
- `app/not-found.tsx` 생성

---

## 성능 최적화 규칙

### 렌더링 전략 우선순위

1. **SSG (Static Site Generation)** - 기본값
2. **ISR (Incremental Static Regeneration)** - 데이터 업데이트 필요 시
3. **SSR (Server-Side Rendering)** - 실시간 데이터 필수 시

**ISR 설정:**
```typescript
export const revalidate = 3600; // 1시간마다 재생성
```

### 이미지 최적화 규칙

**Next.js Image 컴포넌트 필수:**
```typescript
import Image from 'next/image';

<Image
  src={project.thumbnail}
  alt={project.name}
  width={800}
  height={600}
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
/>
```

### Lighthouse 목표

- Performance: **90+**
- Accessibility: **90+**
- Best Practices: **90+**
- SEO: **90+**

---

## 양방향 연결 규칙

### Notion 데이터베이스 Relation

**Projects Database:**
- `Related Resources` (Relation to Resources)

**Resources Database:**
- `Used in Projects` (Relation to Projects)

### 구현 규칙

**양방향 동기화:**
- Project의 `relatedResourceIds`와 Resource의 `usedInProjectIds`는 항상 동기화
- Notion에서 자동으로 처리되므로 별도 동기화 코드 불필요

**상세 페이지 표시:**
```typescript
// 프로젝트 상세 페이지
const project = await getProjectById(id);
const relatedResources = await Promise.all(
  project.relatedResourceIds.map(id => getResourceById(id))
);

// 리소스 상세 페이지
const resource = await getResourceById(id);
const usedInProjects = await Promise.all(
  resource.usedInProjectIds.map(id => getProjectById(id))
);
```

---

## 다중 파일 동기화 규칙

### 환경변수 변경 시

**동시 업데이트 필수:**
1. `.env.local` - 실제 값 추가
2. `.env.example` - 샘플 형식만 추가

### 타입 정의 변경 시

**업데이트 순서:**
1. `lib/types.ts` - 타입 정의 수정
2. `lib/notion-api.ts` - 관련 API 함수 업데이트
3. `lib/notion-utils.ts` - 변환 헬퍼 함수 업데이트

### 새 페이지 추가 시

**동시 업데이트 필수:**
1. 페이지 파일 생성 (예: `app/about/page.tsx`)
2. `components/layout/navbar.tsx` - 네비게이션 링크 추가

---

## AI 의사결정 규칙

### 우선순위 체계

1. **Notion 데이터 우선**: 모든 콘텐츠는 Notion에서 가져옴, 하드코딩 금지
2. **타입 안전성**: TypeScript strict 모드, 타입 가드 사용
3. **성능 최적화**: SSG > ISR > SSR 순서로 선택
4. **접근성**: WCAG AA 준수, 색상 대비 4.5:1
5. **반응형**: Mobile First (1 column → 2 columns → 3-4 columns)

### 불확실한 상황 처리

| 상황 | 해결 방법 |
|------|----------|
| Notion API 응답 구조 불명확 | 타입 가드로 검증, unknown으로 받아 변환 |
| 이미지 처리 방법 불명확 | Next.js Image 컴포넌트 우선 |
| 스타일링 방법 불명확 | `app/globals.css` 확인 후 TailwindCSS 클래스 사용 |
| 컴포넌트 필요 | 기존 shadcn/ui 컴포넌트 확인 후 `npx shadcn@latest add` |
| 라우팅 방법 불명확 | App Router 사용, `app/` 디렉토리 구조 확인 |

### 구현 순서 (ROADMAP 기반)

**새 기능 구현 시 반드시 이 순서를 따름:**

1. **타입 정의** (`lib/types.ts`)
2. **Notion 유틸리티** (`lib/notion-utils.ts`)
3. **API 함수** (`lib/notion-api.ts`)
4. **UI 컴포넌트** (`components/`)
5. **페이지** (`app/`)

**예시: 프로젝트 목록 기능 구현**
1. Project 타입 정의
2. Notion 응답 → Project 변환 함수
3. getAllProjects() API 함수
4. ProjectCard 컴포넌트
5. /projects 페이지

---

## 금지 사항

### ❌ 절대 금지

1. **tailwind.config.js/ts 생성**
   - TailwindCSS v4는 CSS 기반 설정 사용
   - 모든 스타일은 `app/globals.css`에서만

2. **PostCSS 설정 변경**
   - `@tailwindcss/postcss` 플러그인만 사용
   - 다른 플러그인 추가 금지

3. **한국어 이외 언어 사용**
   - 코드 주석: 한국어
   - 커밋 메시지: 한국어
   - 문서: 한국어
   - 변수/함수명은 영어 (코드 표준)

4. **Turbopack 설정 제거**
   - `package.json`의 `--turbopack` 플래그 유지 필수
   - `next.config.ts`에서 Turbopack 설정 제거 금지

5. **Notion 이미지 URL 로컬 저장**
   - 1시간 후 만료되므로 저장 무의미
   - Next.js Image 컴포넌트로 처리

6. **pages/ 디렉토리 생성**
   - App Router 사용 (app/ 디렉토리만)

7. **일반 CSS import**
   - TailwindCSS만 사용
   - `import './styles.css'` 금지

8. **any 타입 사용**
   - TypeScript strict 모드 준수
   - 타입 가드로 unknown 검증

### ⚠️ 주의 사항

1. **Notion Rate Limit**
   - 3 requests/second 초과 금지
   - 연속 요청 시 300ms 지연 추가

2. **Relation 데이터 처리**
   - ID 배열만 반환됨
   - 별도 쿼리로 populate 필수

3. **환경변수 노출**
   - `.env.local`은 절대 커밋 금지
   - `.env.example`만 커밋

4. **빌드 타임 vs 런타임**
   - 이미지 URL은 런타임에도 접근 가능해야 함
   - ISR 또는 SSR 고려

---

## 예시 코드

### ✅ 올바른 예시

**Notion API 호출:**
```typescript
// lib/notion-api.ts
import { notion, DATABASES } from './notion';
import { extractPlainText, extractMultiSelect } from './notion-utils';

export async function getAllProjects(): Promise<Project[]> {
  const response = await notion.databases.query({
    database_id: DATABASES.PROJECTS,
  });

  return response.results.map((page: any) => ({
    id: page.id,
    name: extractPlainText(page.properties.Name.title),
    techStack: extractMultiSelect(page.properties['Tech Stack']),
    // ...
  }));
}
```

**페이지 구현:**
```typescript
// app/projects/page.tsx
import { getAllProjects } from '@/lib/notion-api';
import { ProjectCard } from '@/components/project-card';

export default async function ProjectsPage() {
  const projects = await getAllProjects();

  return (
    <div className="mx-auto max-w-screen-2xl px-4 py-8">
      <h1>프로젝트</h1>
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        {projects.map((project) => (
          <ProjectCard key={project.id} project={project} />
        ))}
      </div>
    </div>
  );
}
```

### ❌ 잘못된 예시

**tailwind.config.js 생성 (금지):**
```javascript
// ❌ 절대 금지
module.exports = {
  theme: {
    extend: {
      colors: {
        primary: '#000000',
      },
    },
  },
};
```

**하드코딩된 데이터 (금지):**
```typescript
// ❌ 잘못된 방법
const projects = [
  { id: '1', name: 'Project 1' },
  { id: '2', name: 'Project 2' },
];

// ✅ 올바른 방법
const projects = await getAllProjects();
```

**영어 주석 (금지):**
```typescript
// ❌ Wrong: Fetch all projects
const projects = await getAllProjects();

// ✅ 올바른 방법: 모든 프로젝트 가져오기
const projects = await getAllProjects();
```

---

## 참고 문서

- **PRD**: `docs/PRD.md` - 제품 요구사항 및 Notion 데이터베이스 구조
- **ROADMAP**: `docs/ROADMAP.md` - 단계별 개발 계획 및 구현 순서
- **CLAUDE.md**: 프로젝트 컨텍스트 및 기본 아키텍처

**새 기능 구현 전 반드시 확인:**
1. `docs/PRD.md` - 요구사항 확인
2. `docs/ROADMAP.md` - 구현 순서 확인
3. 본 문서 - 규칙 준수

---

**이 문서는 AI Agent 전용입니다. 일반 개발 가이드는 README.md를 참조하세요.**
