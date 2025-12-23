# DevHub 개발 로드맵

> Notion을 CMS로 활용하는 개발자 지식 허브 구현 계획

## 📋 프로젝트 개요

**목표**: Notion API를 활용한 프로젝트 포트폴리오 및 학습 리소스 통합 관리 플랫폼 구축
**기술 스택**: Next.js 15, TypeScript, TailwindCSS v4, shadcn/ui, Notion API
**MVP 목표**: 프로젝트-리소스 양방향 연결 시스템을 갖춘 반응형 웹 애플리케이션

---

## 🎯 개발 단계 체크리스트

각 단계는 다음 단계의 기반이 되므로 순서대로 진행해야 합니다.

- [ ] **Phase 1**: 프로젝트 초기 설정 (골격 구축)
- [ ] **Phase 2**: 공통 모듈/컴포넌트 개발
- [ ] **Phase 3**: 핵심 기능 개발
- [ ] **Phase 4**: 추가 기능 개발
- [ ] **Phase 5**: 최적화 및 배포

---

## 📦 Phase 1: 프로젝트 초기 설정 (골격 구축)

### 🎯 목표
프로젝트의 기반 인프라를 구축하고 Notion API 연동을 완료하여 다음 단계에서 사용할 데이터 소스를 준비합니다.

### ⏱️ 예상 소요 시간
**30분 ~ 1시간**

### 📝 작업 목록

#### 1.1 Notion Integration 설정

**왜 이 작업이 먼저인가?**
모든 기능이 Notion 데이터에 의존하므로 API 연동이 최우선입니다.

- [ ] Notion 워크스페이스에서 새 Integration 생성
- [ ] API 키(secret token) 발급 및 저장
- [ ] Integration 권한 설정 (Read content, Update content, Insert content)

#### 1.2 Notion 데이터베이스 생성

**왜 이 작업이 필요한가?**
실제 데이터 구조를 먼저 정의해야 API 호출 및 타입 정의가 가능합니다.

**Projects Database 생성**:
- [ ] Name (Title)
- [ ] Description (Rich Text)
- [ ] Tech Stack (Multi-select)
- [ ] Thumbnail (Files & Media)
- [ ] GitHub URL (URL)
- [ ] Demo URL (URL)
- [ ] Status (Select: 완료/진행중/계획중)
- [ ] Completed Date (Date)
- [ ] Related Resources (Relation to Resources)
- [ ] Slug (Rich Text)

**Resources Database 생성**:
- [ ] Name (Title)
- [ ] Description (Rich Text)
- [ ] URL (URL)
- [ ] Category (Select: Frontend/Backend/Design/DevOps/General)
- [ ] Type (Select: 문서/도구/튜토리얼/라이브러리)
- [ ] Rating (Select: ⭐/⭐⭐/⭐⭐⭐)
- [ ] Tags (Multi-select)
- [ ] Used in Projects (Relation to Projects)
- [ ] Added Date (Created time)

**샘플 데이터 입력**:
- [ ] 샘플 프로젝트 3-5개 입력 (다양한 기술 스택 포함)
- [ ] 샘플 리소스 10-15개 입력 (다양한 카테고리/타입 포함)
- [ ] 양방향 Relation 설정 확인

#### 1.3 환경변수 설정

**왜 이 작업이 필요한가?**
보안을 위해 API 키와 데이터베이스 ID를 환경변수로 관리해야 합니다.

- [ ] `.env.local` 파일 생성
  ```env
  NOTION_API_KEY=secret_xxxxxxxxxxxxx
  NOTION_PROJECTS_DATABASE_ID=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
  NOTION_RESOURCES_DATABASE_ID=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
  ```

- [ ] `.env.example` 파일 업데이트 (실제 값 제외)
- [ ] `.gitignore`에 `.env.local` 포함 확인

#### 1.4 Notion SDK 설치 및 초기화

**왜 이 작업이 필요한가?**
Notion API 클라이언트를 설정해야 다음 단계에서 데이터를 가져올 수 있습니다.

- [ ] 필요한 패키지 설치
  ```bash
  npm install @notionhq/client notion-to-md
  ```

- [ ] `lib/notion.ts` 생성 - Notion 클라이언트 초기화
  ```typescript
  import { Client } from '@notionhq/client';

  export const notion = new Client({
    auth: process.env.NOTION_API_KEY,
  });

  export const DATABASES = {
    PROJECTS: process.env.NOTION_PROJECTS_DATABASE_ID!,
    RESOURCES: process.env.NOTION_RESOURCES_DATABASE_ID!,
  };
  ```

- [ ] 연결 테스트 스크립트 작성 및 실행
  ```bash
  # scripts/test-notion-connection.ts 생성 후
  npx tsx scripts/test-notion-connection.ts
  ```

### ✅ 완료 기준

- [ ] Notion Integration이 데이터베이스에 연결됨
- [ ] 샘플 데이터가 두 데이터베이스에 입력됨
- [ ] 환경변수가 올바르게 설정됨
- [ ] 테스트 스크립트로 Notion API 연결 확인
- [ ] Rate limit 에러 없이 데이터 조회 성공

### 🐛 예상 이슈 및 해결

**이슈**: "object not found" 에러
**해결**: Notion 데이터베이스 Share 메뉴에서 Integration 연결 확인

**이슈**: Rate limit 초과
**해결**: 요청 사이에 300ms 지연 추가 (3 requests/second 제한)

---

## 🔧 Phase 2: 공통 모듈/컴포넌트 개발

### 🎯 목표
개별 페이지를 구현하기 전에 공통으로 사용할 타입, API 함수, UI 컴포넌트를 먼저 만듭니다.

### ⏱️ 예상 소요 시간
**2~3시간**

### 📝 작업 목록

#### 2.1 TypeScript 타입 정의

**왜 이 작업이 먼저인가?**
타입을 먼저 정의하면 API 함수와 컴포넌트 개발 시 자동완성과 타입 체크를 활용할 수 있습니다.

- [ ] `lib/types.ts` 생성

```typescript
// Project 타입 정의
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

// Resource 타입 정의
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

// 상세 페이지용 타입
export interface ProjectDetail extends Project {
  content: string; // 마크다운 변환된 콘텐츠
  relatedResources: Resource[];
}

export interface ResourceDetail extends Resource {
  usedInProjects: Project[];
}
```

- [ ] TypeScript 컴파일 확인: `npx tsc --noEmit`

#### 2.2 Notion API 유틸리티 함수

**왜 이 작업이 필요한가?**
Notion API 응답을 우리가 정의한 타입으로 변환하는 헬퍼 함수를 만들어 코드 중복을 방지합니다.

- [ ] `lib/notion-utils.ts` 생성
  - `extractPlainText(richText)`: Notion rich text → string
  - `extractMultiSelect(property)`: Multi-select → string[]
  - `extractSelect(property)`: Select → string
  - `extractUrl(property)`: URL → string | null
  - `extractDate(property)`: Date → string | null
  - `extractRelation(property)`: Relation → string[]
  - `extractFiles(property)`: Files → string | null

- [ ] `lib/notion-to-markdown.ts` 생성
  - `notion-to-md` 라이브러리 설정
  - 커스텀 변환 로직 추가 (이미지, 코드 블록)

#### 2.3 데이터 Fetch 함수 구현

**왜 이 작업이 필요한가?**
모든 페이지에서 공통으로 사용할 데이터 조회 함수를 중앙화하여 관리합니다.

- [ ] `lib/notion-api.ts` 생성

**Projects API 함수**:
- [ ] `getAllProjects()`: 모든 프로젝트 목록
- [ ] `getProjectById(id)`: 특정 프로젝트 상세 (마크다운 포함)
- [ ] `getProjectsByTechStack(techStack)`: 기술스택 필터링
- [ ] `getRecentProjects(limit)`: 최근 프로젝트 N개

**Resources API 함수**:
- [ ] `getAllResources()`: 모든 리소스 목록
- [ ] `getResourceById(id)`: 특정 리소스 상세
- [ ] `getResourcesByCategory(category)`: 카테고리 필터링
- [ ] `getResourcesByType(type)`: 타입 필터링
- [ ] `getFeaturedResources(limit)`: 추천 리소스 (Rating 높은 순)

#### 2.4 공통 UI 컴포넌트 구현

**왜 이 작업이 필요한가?**
개별 페이지를 만들기 전에 재사용 가능한 컴포넌트를 준비하면 개발 속도가 빨라집니다.

**마크다운 렌더러**:
- [ ] `react-markdown`, `remark-gfm`, `rehype-highlight` 설치
- [ ] `components/markdown-renderer.tsx` 생성
  - Syntax highlighting 지원
  - 다크모드 스타일링
  - 보안 설정 (`allowedElements`)

**프로젝트 카드**:
- [ ] `components/project-card.tsx` 생성
  - 썸네일, 제목, 설명 (2줄 ellipsis)
  - 기술스택 배지 (최대 3개, +N 표시)
  - Status 배지, GitHub/Demo 링크
  - 호버 효과, 클릭 시 상세 페이지 이동

**리소스 카드**:
- [ ] `components/resource-card.tsx` 생성
  - 제목, 설명 (3줄 ellipsis)
  - 카테고리/타입 배지, Rating 표시
  - 태그 (최대 5개), 외부 링크 버튼

**필터 컴포넌트**:
- [ ] `components/tech-stack-filter.tsx` 생성
  - 기술스택 목록, 클릭 필터링, "전체" 옵션
- [ ] `components/category-filter.tsx` 생성
  - 카테고리 탭/버튼, 활성 상태 하이라이트

### ✅ 완료 기준

- [ ] 모든 타입이 정의되고 TypeScript 에러 없음
- [ ] Notion API 함수가 올바른 타입을 반환
- [ ] 각 유틸리티 함수가 샘플 데이터로 테스트됨
- [ ] 카드 컴포넌트가 독립적으로 렌더링 확인
- [ ] 다크모드에서 모든 컴포넌트 스타일 확인

### 🐛 예상 이슈 및 해결

**이슈**: Notion 이미지 URL 만료 (1시간)
**해결**: Next.js Image 컴포넌트 사용 또는 외부 호스팅 이동

**이슈**: Rich text 변환 오류
**해결**: notion-to-md 커스텀 변환기 설정

**이슈**: Relation 데이터 누락
**해결**: 별도 쿼리로 관련 데이터 fetch 및 populate

---

## 🎨 Phase 3: 핵심 기능 개발

### 🎯 목표
프로젝트와 리소스 페이지를 구현하여 핵심 가치를 제공합니다.

### ⏱️ 예상 소요 시간
**3~4시간**

### 📝 작업 목록

#### 3.1 프로젝트 목록 페이지

**왜 이 작업이 먼저인가?**
프로젝트 목록은 사이트의 핵심 기능이며, 상세 페이지로 진입하는 게이트웨이입니다.

- [ ] `app/projects/page.tsx` 생성
  ```typescript
  export default async function ProjectsPage() {
    const projects = await getAllProjects();
    // ...
  }
  ```

**레이아웃 구현**:
- [ ] 페이지 헤더 (제목, 설명)
- [ ] 기술스택 필터 UI
- [ ] 프로젝트 카드 그리드 (반응형: 1/2/3 columns)
- [ ] Empty state (프로젝트 없을 때)

**필터링 로직**:
- [ ] URL query parameter로 상태 관리 (`?tech=React`)
- [ ] Client Component에서 필터링 처리
- [ ] 필터 초기화 버튼

**정렬 옵션**:
- [ ] 최신순 (기본값)
- [ ] 이름순
- [ ] 기술스택별

#### 3.2 프로젝트 상세 페이지

**왜 이 작업이 필요한가?**
프로젝트의 전체 정보와 관련 리소스를 보여주는 핵심 페이지입니다.

- [ ] `app/projects/[id]/page.tsx` 생성
- [ ] `generateStaticParams` 구현 (모든 프로젝트 ID)

**레이아웃 구현**:
- [ ] 썸네일 이미지 (큰 사이즈)
- [ ] 프로젝트 제목, Status 배지
- [ ] 기술스택 배지 목록
- [ ] GitHub/Demo 링크 버튼
- [ ] 완료 날짜
- [ ] 마크다운 콘텐츠 렌더링
- [ ] 관련 리소스 섹션 (Resource 카드 그리드)
- [ ] 뒤로가기 버튼

**SEO 최적화**:
- [ ] `generateMetadata` 함수 구현
- [ ] Open Graph 이미지 설정

#### 3.3 리소스 목록 페이지

**왜 이 작업이 필요한가?**
프로젝트와 함께 사이트의 핵심 가치인 학습 리소스 라이브러리를 제공합니다.

- [ ] `app/resources/page.tsx` 생성
  ```typescript
  export default async function ResourcesPage() {
    const resources = await getAllResources();
    // ...
  }
  ```

**레이아웃 구현**:
- [ ] 페이지 헤더
- [ ] 카테고리 필터 탭 (Frontend/Backend/Design/DevOps/General)
- [ ] 타입 필터 (문서/도구/튜토리얼/라이브러리)
- [ ] 리소스 카드 그리드
- [ ] Empty state

**필터링 로직**:
- [ ] URL query parameter로 상태 관리 (`?category=Frontend&type=문서`)
- [ ] 다중 필터 지원 (AND 조건)
- [ ] 필터 초기화 버튼

**정렬 옵션**:
- [ ] 최신순 (기본값)
- [ ] Rating 높은순
- [ ] 이름순

#### 3.4 리소스 상세 페이지

**왜 이 작업이 필요한가?**
리소스 정보와 양방향 연결(사용된 프로젝트)을 보여줍니다.

- [ ] `app/resources/[id]/page.tsx` 생성
- [ ] `generateStaticParams` 구현

**레이아웃 구현**:
- [ ] 리소스 제목
- [ ] 카테고리/타입/Rating 배지
- [ ] 설명 (마크다운 렌더링)
- [ ] 외부 링크 버튼 (큰 사이즈, 새 탭)
- [ ] 태그 목록
- [ ] "이 리소스를 사용한 프로젝트" 섹션
- [ ] 뒤로가기 버튼

**SEO 최적화**:
- [ ] `generateMetadata` 구현

### ✅ 완료 기준

- [ ] `/projects` 페이지에서 모든 프로젝트 표시
- [ ] 기술스택 필터 동작 확인
- [ ] `/projects/[id]` 페이지에서 상세 정보 표시
- [ ] 관련 리소스가 올바르게 연결됨
- [ ] `/resources` 페이지에서 모든 리소스 표시
- [ ] 카테고리/타입 필터 동작 확인
- [ ] `/resources/[id]` 페이지에서 상세 정보 표시
- [ ] 사용된 프로젝트가 올바르게 연결됨
- [ ] 존재하지 않는 ID 접속 시 404 페이지
- [ ] Next.js 빌드 성공: `npm run build`

### 🧪 Playwright 테스트 시나리오

```typescript
test('프로젝트 목록 및 상세 페이지', async ({ page }) => {
  // 목록 페이지
  await page.goto('/projects');
  const projectCards = page.locator('[data-testid="project-card"]');
  await expect(projectCards.first()).toBeVisible();

  // 필터링
  await page.click('text=React');
  await expect(page).toHaveURL(/tech=React/);

  // 상세 페이지
  await projectCards.first().click();
  await expect(page.locator('h1')).toBeVisible();
  await expect(page.locator('[data-testid="related-resource"]')).toBeVisible();
});

test('리소스 목록 및 상세 페이지', async ({ page }) => {
  // 목록 페이지
  await page.goto('/resources');
  const resourceCards = page.locator('[data-testid="resource-card"]');
  await expect(resourceCards.first()).toBeVisible();

  // 필터링
  await page.click('text=Frontend');
  await expect(page).toHaveURL(/category=Frontend/);

  // 상세 페이지
  await resourceCards.first().click();
  await expect(page.locator('h1')).toBeVisible();
  await expect(page.locator('[data-testid="used-in-project"]')).toBeVisible();
});
```

---

## 🏠 Phase 4: 추가 기능 개발

### 🎯 목표
홈페이지를 구현하여 사용자 경험을 향상시키고, 사이트의 첫인상을 개선합니다.

### ⏱️ 예상 소요 시간
**1~2시간**

### 📝 작업 목록

#### 4.1 Hero 섹션

**왜 이 작업이 필요한가?**
첫 화면에서 사이트의 목적과 가치를 명확히 전달합니다.

- [ ] `app/page.tsx` 업데이트
- [ ] `components/home/hero-section.tsx` 생성
  - 큰 제목: "DevHub - 개발자 지식 허브"
  - 부제목: 사이트 설명
  - CTA 버튼: "프로젝트 보기", "리소스 탐색"
  - 배경 그라디언트 또는 패턴

#### 4.2 최근 프로젝트 섹션

**왜 이 작업이 필요한가?**
최근 작업을 하이라이트하여 사용자의 관심을 유도합니다.

- [ ] `components/home/recent-projects-section.tsx` 생성
  - `getRecentProjects(4)` 호출
  - Project 카드 그리드 (최대 4개)
  - "모든 프로젝트 보기" 링크

#### 4.3 추천 리소스 섹션

**왜 이 작업이 필요한가?**
고품질 리소스를 먼저 보여줘 사용자에게 가치를 제공합니다.

- [ ] `components/home/featured-resources-section.tsx` 생성
  - `getFeaturedResources(6)` 호출
  - Resource 카드 그리드 (최대 6개)
  - "모든 리소스 보기" 링크

#### 4.4 기술스택 통계 (선택사항)

**왜 이 작업이 선택사항인가?**
핵심 기능은 아니지만 사이트에 개성을 더합니다.

- [ ] `components/home/tech-stats-section.tsx` 생성
  - 모든 프로젝트에서 기술스택 추출
  - 사용 빈도 계산
  - 배지 클라우드로 표시
  - 클릭 시 해당 스택 프로젝트 목록으로 이동

### ✅ 완료 기준

- [ ] `/` 접속 시 Hero 섹션 표시
- [ ] 최근 프로젝트 4개 표시
- [ ] 추천 리소스 6개 표시
- [ ] CTA 버튼이 올바른 페이지로 이동
- [ ] 모바일/데스크톱 반응형 확인
- [ ] 다크모드 색상 확인

### 🧪 Playwright 테스트 시나리오

```typescript
test('홈페이지 섹션', async ({ page }) => {
  await page.goto('/');

  // Hero 섹션
  await expect(page.locator('h1')).toContainText('DevHub');
  await expect(page.locator('a:has-text("프로젝트 보기")')).toHaveAttribute('href', '/projects');

  // 최근 프로젝트
  const recentProjects = page.locator('[data-testid="recent-project-card"]');
  await expect(recentProjects).toHaveCount(4);

  // 추천 리소스
  const featuredResources = page.locator('[data-testid="featured-resource-card"]');
  await expect(featuredResources).toHaveCount(6);
});
```

---

## 🚀 Phase 5: 최적화 및 배포

### 🎯 목표
프로덕션 배포를 위한 스타일링, 성능 최적화, SEO 개선을 완료합니다.

### ⏱️ 예상 소요 시간
**2~3시간**

### 📝 작업 목록

#### 5.1 스타일링 통일 및 반응형 최적화

**왜 이 작업이 필요한가?**
일관된 디자인과 모든 디바이스에서의 최적 경험을 보장합니다.

**전역 스타일 통일**:
- [ ] `app/globals.css` 업데이트
  - 커스텀 색상 팔레트 확장 (OKLCH 값)
  - 타이포그래피 스케일 정의
  - 애니메이션 설정 (`tw-animate-css`)

- [ ] 공통 컴포넌트 스타일 검토
  - Button 변형 (primary, secondary, outline, ghost)
  - Badge 변형 (기술스택, 카테고리, Status)
  - Card 스타일 (hover, focus)

**반응형 레이아웃 테스트**:
- [ ] Mobile (< 640px): 1 column, 큰 터치 영역
- [ ] Tablet (640px ~ 1024px): 2 columns
- [ ] Desktop (> 1024px): 3-4 columns
- [ ] 네비게이션 반응형 (Mobile: 햄버거 메뉴)

**다크모드 색상 조정**:
- [ ] 색상 대비 검사 (WCAG AA 기준: 4.5:1)
- [ ] 텍스트, 배지, 버튼 모두 확인
- [ ] 이미지 위 오버레이 추가 (필요 시)

**애니메이션 및 인터랙션**:
- [ ] 카드 호버 효과 (scale up, shadow)
- [ ] 버튼 클릭 피드백
- [ ] 페이지 전환 애니메이션 (선택사항)

#### 5.2 빌드 최적화

**왜 이 작업이 필요한가?**
빠른 로딩 속도는 사용자 경험과 SEO에 직접적인 영향을 줍니다.

**빌드 분석**:
- [ ] Next.js 빌드 실행: `npm run build`
- [ ] 빌드 시간 및 번들 크기 확인
- [ ] `@next/bundle-analyzer` 사용 고려

**이미지 최적화**:
- [ ] Next.js Image 컴포넌트 사용 확인
- [ ] 적절한 `sizes` 속성 설정
- [ ] WebP 포맷 활용

**폰트 최적화**:
- [ ] Geist 폰트 subset 최적화
- [ ] `font-display: swap` 설정

**ISR 설정 (선택사항)**:
- [ ] 페이지에 `revalidate` 옵션 추가
  ```typescript
  export const revalidate = 3600; // 1시간마다
  ```

#### 5.3 SEO 최적화

**왜 이 작업이 필요한가?**
검색 엔진 최적화로 사이트 발견 가능성을 높입니다.

**메타태그 설정**:
- [ ] 모든 페이지에 `generateMetadata` 구현
- [ ] title, description, keywords
- [ ] Open Graph 이미지

**Sitemap 및 Robots**:
- [ ] `app/sitemap.ts` 생성
  - 모든 프로젝트/리소스 페이지 포함
- [ ] `robots.txt` 생성
  ```txt
  User-agent: *
  Allow: /
  Sitemap: https://your-domain.com/sitemap.xml
  ```

**구조화된 데이터**:
- [ ] Project → SoftwareSourceCode schema
- [ ] Resource → Article schema

#### 5.4 배포 및 검증

**왜 이 작업이 마지막인가?**
모든 기능과 최적화가 완료된 후 프로덕션 환경에 배포합니다.

**환경변수 설정**:
- [ ] Vercel 프로젝트 생성
- [ ] 환경변수 등록
  - `NOTION_API_KEY`
  - `NOTION_PROJECTS_DATABASE_ID`
  - `NOTION_RESOURCES_DATABASE_ID`

**배포**:
- [ ] Vercel에 배포: `vercel --prod`
- [ ] 프로덕션 URL 확인

**프로덕션 테스트**:
- [ ] 모든 페이지 접근 가능 확인
- [ ] Notion 데이터 올바르게 표시
- [ ] 링크 동작 확인
- [ ] 이미지 로딩 확인
- [ ] 다양한 디바이스/브라우저 테스트

**성능 검증**:
- [ ] Lighthouse 실행
  - Performance: 90+
  - Accessibility: 90+
  - Best Practices: 90+
  - SEO: 90+
- [ ] Core Web Vitals 확인
  - LCP < 2.5s
  - FID < 100ms
  - CLS < 0.1

### ✅ 완료 기준

- [ ] 모든 페이지가 일관된 스타일
- [ ] 반응형 레이아웃이 모든 디바이스에서 동작
- [ ] 다크모드 색상 대비 WCAG AA 준수
- [ ] Lighthouse 성능 점수 90+
- [ ] Next.js 빌드 성공
- [ ] Vercel 배포 완료
- [ ] 프로덕션 환경에서 모든 기능 동작
- [ ] SEO 메타태그 모든 페이지 적용

---

## 📊 테스트 전략 요약

### 단위 테스트
- `lib/notion-utils.ts` 함수 테스트
- `lib/notion-api.ts` 함수 테스트 (모킹 활용)

### 통합 테스트
- Notion API 실제 호출 테스트 (테스트 데이터베이스)
- 페이지 렌더링 테스트

### E2E 테스트 (Playwright)
- 주요 사용자 플로우
  - 홈 → 프로젝트 목록 → 프로젝트 상세
  - 홈 → 리소스 목록 → 리소스 상세
  - 필터링 및 정렬 기능
- 반응형 레이아웃 테스트
- 다크모드 전환 테스트

### 성능 테스트
- Lighthouse CI 설정
- Core Web Vitals 모니터링

---

## 🔄 MVP 이후 확장 기능

### Phase 6: 검색 및 고급 필터링 (1~2시간)
- [ ] 프로젝트/리소스 통합 검색
- [ ] 자동완성 (Fuse.js)
- [ ] 검색 결과 하이라이팅
- [ ] 태그 기반 필터링
- [ ] 태그 클라우드 UI

### Phase 7: 통계 및 분석 (1시간)
- [ ] 프로젝트 수, 리소스 수 대시보드
- [ ] 기술스택 분포 차트
- [ ] 최근 활동 타임라인
- [ ] Vercel Analytics 연동

### Phase 8: 소셜 기능 (2~3시간)
- [ ] RSS 피드 (`/feed/projects.xml`, `/feed/resources.xml`)
- [ ] Giscus 댓글 시스템 연동
- [ ] 공유 버튼 (Twitter, LinkedIn)
- [ ] 좋아요 카운트 (로컬 스토리지)

---

## 🐛 예상 이슈 및 해결 가이드

### Notion API 관련

**이슈**: Rate limit 초과 (429 Too Many Requests)
- **원인**: 3 requests/second 제한
- **해결**: 요청 사이 지연, 배치 처리, 캐싱

**이슈**: 이미지 URL 만료
- **원인**: Notion 이미지 URL은 1시간 후 만료
- **해결**:
  1. Next.js Image `unoptimized` 사용 (임시)
  2. Cloudinary/S3로 이미지 마이그레이션 (권장)
  3. ISR로 주기적 재생성

**이슈**: Relation 데이터 누락
- **원인**: Relation은 ID만 반환
- **해결**: 별도 쿼리로 populate

### Next.js 빌드 관련

**이슈**: 빌드 시 Notion API 실패
- **원인**: 환경변수 미설정
- **해결**: `.env.local` 확인, 빌드 로그 분석

**이슈**: Static Generation 시간 초과
- **원인**: 많은 페이지, 느린 API
- **해결**: 병렬 처리, 페이지 수 제한, 일부 SSR 전환

### TypeScript 관련

**이슈**: Notion API 타입 불일치
- **원인**: 복잡한 API 응답 구조
- **해결**: 타입 가드 작성, `unknown`으로 받아 검증

---

## ✅ 최종 체크리스트

### 기능 완성도
- [ ] Notion API 연동 및 데이터 fetch
- [ ] 프로젝트 목록/상세 페이지
- [ ] 리소스 목록/상세 페이지
- [ ] 프로젝트-리소스 양방향 연결
- [ ] 홈페이지 모든 섹션
- [ ] 필터링 기능
- [ ] 반응형 레이아웃
- [ ] 다크모드 지원

### 품질 보증
- [ ] TypeScript 타입 에러 0개
- [ ] Next.js 빌드 성공
- [ ] Lighthouse 성능 점수 90+
- [ ] Playwright 테스트 통과
- [ ] 다양한 디바이스 수동 테스트
- [ ] WCAG AA 색상 대비 준수

### 배포 준비
- [ ] 프로덕션 환경변수 설정
- [ ] SEO 메타태그 모든 페이지
- [ ] Sitemap 생성
- [ ] robots.txt 설정
- [ ] 에러 페이지 (404, 500)
- [ ] 프로덕션 배포 완료
- [ ] 도메인 연결 (선택사항)

### 문서화
- [ ] README.md 업데이트
- [ ] 환경변수 설명
- [ ] 아키텍처 다이어그램 (선택사항)
- [ ] 기여 가이드 (선택사항)

---

## 🎉 마일스톤

1. **Phase 1 완료**: Notion 연동 완료 ✅ (예상: 1시간)
2. **Phase 2 완료**: 공통 모듈 구현 ✅ (예상: 2-3시간)
3. **Phase 3 완료**: 핵심 페이지 구현 ✅ (예상: 3-4시간)
4. **Phase 4 완료**: 홈페이지 구현 ✅ (예상: 1-2시간)
5. **Phase 5 완료**: 배포 및 최적화 ✅ (예상: 2-3시간)
6. **MVP 출시**: 사용자에게 공개 🚀 (총 예상: 9-13시간)

---

## 📚 참고 자료

- [Notion API 공식 문서](https://developers.notion.com/)
- [Next.js 15 문서](https://nextjs.org/docs)
- [TailwindCSS v4 문서](https://tailwindcss.com/docs)
- [shadcn/ui 컴포넌트](https://ui.shadcn.com/)
- [Playwright 문서](https://playwright.dev/)
- [Vercel 배포 가이드](https://vercel.com/docs)

---

## 🔍 개발 순서 체크리스트

- [x] **Phase 1: 프로젝트 골격이 가장 먼저인가?**
  ✅ Notion 연동과 환경 설정이 최우선입니다. 모든 기능이 Notion 데이터에 의존하기 때문입니다.

- [x] **Phase 2: 공통 모듈/컴포넌트가 개별 기능보다 먼저인가?**
  ✅ 타입 정의, API 함수, UI 컴포넌트를 먼저 만들어야 개별 페이지 개발 시 재사용할 수 있습니다.

- [x] **Phase 3~4: 개별 기능이 공통 모듈 이후에 있는가?**
  ✅ 공통 모듈을 활용하여 프로젝트/리소스 페이지와 홈페이지를 구현합니다.

- [x] **각 단계마다 왜 이 순서인지 설명이 있는가?**
  ✅ 각 Phase와 작업마다 "왜 이 작업이 먼저/필요한가?" 설명이 포함되어 있습니다.

---

**이 로드맵을 따라 단계별로 구현하면 완전히 작동하는 DevHub MVP를 완성할 수 있습니다.**

각 Phase를 순서대로 완료하고, 체크리스트를 활용하여 진행 상황을 추적하세요. 🚀
