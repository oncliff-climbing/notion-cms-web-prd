# DevHub - 개발자 지식 허브 PRD

## 프로젝트 개요

- **프로젝트명**: DevHub (개발자 지식 허브)
- **목적**: Notion을 CMS로 활용하여 개발 프로젝트와 학습 리소스를 통합 관리하고 공유하는 개인 지식 플랫폼 구축
- **CMS 선택 이유**:
  - Notion API를 활용하여 비개발자도 콘텐츠 관리 가능
  - 데이터베이스 관계 설정으로 프로젝트-리소스 연결 자동화
  - 실시간 콘텐츠 업데이트 가능
  - 마크다운 기반 리치 텍스트 지원

## 주요 기능

1. **프로젝트 포트폴리오 쇼케이스**
   - 완성한 프로젝트를 카드 그리드로 표시
   - 기술스택별 필터링 및 검색
   - 프로젝트 상세 페이지에서 사용한 리소스 연결 표시

2. **학습 리소스 라이브러리**
   - 북마크한 도구, 문서, 튜토리얼을 카테고리별로 정리
   - 타입별(문서/도구/튜토리얼/라이브러리) 필터링
   - 각 리소스가 어떤 프로젝트에 활용되었는지 역참조 표시

3. **양방향 연결 시스템**
   - 프로젝트 → 사용한 리소스 연결
   - 리소스 → 활용된 프로젝트 연결
   - 학습 경로 추적 및 지식 그래프 형성

## 기술 스택

**Frontend**
- Next.js 15 (App Router)
- TypeScript
- TailwindCSS v4
- shadcn/ui 컴포넌트

**CMS & API**
- Notion API (@notionhq/client)
- notion-to-md (Notion 블록을 마크다운으로 변환)

**배포**
- Vercel (또는 선호하는 플랫폼)

**추가 라이브러리**
- react-markdown (마크다운 렌더링)
- lucide-react (아이콘)
- next-themes (다크모드)

## Notion 데이터베이스 구조

### Database 1: Projects (프로젝트)

- **Name**: Title - 프로젝트 제목
- **Description**: Rich Text - 프로젝트 설명 및 상세 내용
- **Tech Stack**: Multi-select - 사용한 기술 스택 (예: React, TypeScript, TailwindCSS)
- **Thumbnail**: Files & Media - 프로젝트 썸네일 이미지
- **GitHub URL**: URL - GitHub 저장소 링크
- **Demo URL**: URL - 라이브 데모 링크
- **Status**: Select - 프로젝트 상태 (완료/진행중/계획중)
- **Completed Date**: Date - 완료 날짜
- **Related Resources**: Relation to Resources - 사용한 리소스들
- **Slug**: Rich Text - URL 슬러그 (선택사항)

### Database 2: Resources (리소스)

- **Name**: Title - 리소스 제목
- **Description**: Rich Text - 리소스 설명
- **URL**: URL - 리소스 링크
- **Category**: Select - 카테고리 (Frontend/Backend/Design/DevOps/General)
- **Type**: Select - 타입 (문서/도구/튜토리얼/라이브러리)
- **Rating**: Select - 추천도 (⭐/⭐⭐/⭐⭐⭐)
- **Tags**: Multi-select - 태그 (React, TypeScript, CSS 등)
- **Used in Projects**: Relation to Projects - 이 리소스를 사용한 프로젝트들
- **Added Date**: Created time - 추가된 날짜

## 화면 구성

- **홈페이지 (`/`)**: Hero 섹션, 최근 프로젝트 하이라이트 (3-4개), 추천 리소스 섹션, 기술스택 통계
- **프로젝트 목록 (`/projects`)**: 모든 프로젝트를 카드 그리드로 표시, 기술스택 필터, 검색 기능
- **프로젝트 상세 (`/projects/[id]`)**: 프로젝트 전체 설명, 썸네일, 기술스택, GitHub/Demo 링크, 사용한 리소스 섹션
- **리소스 목록 (`/resources`)**: 카테고리별 탭/필터, 타입별 필터, 검색 기능, 리소스 카드
- **리소스 상세 (`/resources/[id]`)**: 리소스 설명, URL 링크, 카테고리/타입/추천도 표시, 사용된 프로젝트 섹션

## MVP 범위

**포함할 최소 기능**
- ✅ Notion API 연동 및 데이터 fetch
- ✅ 프로젝트 목록 페이지 (카드 그리드)
- ✅ 프로젝트 상세 페이지 (기본 정보 + 리치텍스트 렌더링)
- ✅ 리소스 목록 페이지 (카테고리별 필터)
- ✅ 리소스 상세 페이지
- ✅ 프로젝트-리소스 양방향 연결 표시
- ✅ 반응형 레이아웃
- ✅ 다크모드 지원

**MVP 이후 추가 가능 기능**
- 검색 기능 (프로젝트/리소스)
- 태그 기반 필터링
- ISR (Incremental Static Regeneration) 적용
- 홈페이지 통계 대시보드
- RSS 피드
- SEO 최적화 (메타태그, OG 이미지)

## 구현 단계

1. **환경 설정 및 Notion 연동**
   - Notion Integration 생성 및 API 키 발급
   - Notion 데이터베이스 생성 (Projects, Resources)
   - 샘플 데이터 입력
   - `.env.local` 파일에 환경변수 설정
   - `@notionhq/client` 설치 및 기본 연결 테스트

2. **API 라우트 및 데이터 fetch 로직 구현**
   - `lib/notion.ts`: Notion 클라이언트 초기화
   - `lib/notion-api.ts`: 프로젝트/리소스 목록 및 상세 가져오기 함수 구현
   - Notion 블록을 마크다운으로 변환하는 유틸리티

3. **프로젝트 페이지 구현**
   - `/app/projects/page.tsx`: 프로젝트 목록 페이지
   - `/app/projects/[id]/page.tsx`: 프로젝트 상세 페이지
   - `components/project-card.tsx`: 프로젝트 카드 컴포넌트

4. **리소스 페이지 구현**
   - `/app/resources/page.tsx`: 리소스 목록 페이지
   - `/app/resources/[id]/page.tsx`: 리소스 상세 페이지
   - `components/resource-card.tsx`: 리소스 카드 컴포넌트

5. **홈페이지 및 레이아웃 개선**
   - `/app/page.tsx`: 홈페이지 (Hero, 최근 프로젝트, 추천 리소스)
   - 네비게이션 업데이트 (`components/layout/navbar.tsx`)

6. **스타일링 및 반응형 최적화**
   - TailwindCSS로 전체 스타일링
   - 모바일/태블릿/데스크톱 반응형 테스트
   - 다크모드 색상 조정

7. **배포 및 테스트**
   - 환경변수 설정 (Vercel/배포 플랫폼)
   - 빌드 테스트
   - 배포 및 기능 검증

## 핵심 파일 구조

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
│   └── layout.tsx                  # 루트 레이아웃
├── components/
│   ├── project-card.tsx            # 프로젝트 카드
│   ├── resource-card.tsx           # 리소스 카드
│   ├── markdown-renderer.tsx       # 마크다운 렌더링
│   └── layout/
│       ├── navbar.tsx              # 네비게이션
│       └── footer.tsx              # 푸터
├── lib/
│   ├── notion.ts                   # Notion 클라이언트 설정
│   ├── notion-api.ts               # API 호출 함수들
│   └── types.ts                    # TypeScript 타입 정의
└── docs/
    └── PRD.md                      # 본 문서
```

## 환경변수 설정

```env
# Notion Integration Token
NOTION_API_KEY=secret_xxxxxxxxxxxxx

# Notion Database IDs
NOTION_PROJECTS_DATABASE_ID=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
NOTION_RESOURCES_DATABASE_ID=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

## 주의사항

1. **Notion API 제약사항**
   - Rate limit: 3 requests per second
   - 페이지네이션 처리 필요 (100개 이상 아이템 시)

2. **이미지 처리**
   - Notion에서 제공하는 이미지 URL은 만료됨 (1시간)
   - 해결책: Next.js Image 컴포넌트로 최적화 또는 외부 이미지 호스팅 사용

3. **빌드 타임 vs 런타임**
   - Static Generation (빌드 타임): 초기 MVP에서는 SSG 사용
   - ISR 적용 시 `revalidate` 옵션 추가하여 주기적으로 재생성

4. **타입 안정성**
   - Notion API 응답을 TypeScript 타입으로 정의하여 타입 안정성 확보

## 예상 작업 시간

- 1단계: 5분 (Notion 설정 및 연동)
- 2단계: 10분 (API 로직 구현)
- 3단계: 10분 (프로젝트 페이지)
- 4단계: 10분 (리소스 페이지)
- 5단계: 5분 (홈페이지)
- 6-7단계: 5분 (스타일링 및 배포)

**총 예상 시간: 약 45분** (버퍼 포함)
