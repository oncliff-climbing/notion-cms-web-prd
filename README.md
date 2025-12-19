# DevHub - 개발자 지식 허브

Notion을 CMS로 활용하여 개발 프로젝트와 학습 리소스를 통합 관리하고 공유하는 개인 지식 플랫폼입니다.

## 🎯 프로젝트 개요

DevHub는 개발자들이 자신의 프로젝트 포트폴리오와 학습 리소스를 한 곳에서 관리할 수 있도록 돕는 웹 애플리케이션입니다. Notion을 CMS로 사용하여 비개발자도 쉽게 콘텐츠를 관리할 수 있으며, 프로젝트와 리소스 간의 양방향 연결을 통해 학습 경로를 추적할 수 있습니다.

### 주요 기능

- 💼 **프로젝트 포트폴리오 쇼케이스**: 완성한 프로젝트를 카드 형태로 표시하고 기술스택별로 필터링
- 📚 **학습 리소스 라이브러리**: 유용한 도구, 문서, 튜토리얼을 카테고리별로 정리
- 🔗 **양방향 연결 시스템**: 프로젝트와 사용한 리소스를 서로 연결하여 지식 그래프 형성
- 🌙 **다크모드 지원**: 라이트/다크 테마 자동 전환
- 📱 **반응형 디자인**: 모든 디바이스에서 최적화된 경험 제공

### CMS로 Notion을 선택한 이유

- 직관적인 UI로 비개발자도 쉽게 콘텐츠 관리 가능
- 데이터베이스 관계 설정으로 프로젝트-리소스 연결 자동화
- 실시간 콘텐츠 업데이트 및 마크다운 기반 리치 텍스트 지원
- 별도의 관리자 페이지 구축 불필요

📄 **상세 문서**: [PRD 문서](./docs/PRD.md)를 참조하세요.

## ✨ 기술 스택

**Frontend**
- **[Next.js v15](https://nextjs.org)** - React 기반 풀스택 프레임워크
- **[TypeScript](https://www.typescriptlang.org)** - 타입 안전성을 위한 언어
- **[TailwindCSS v4](https://tailwindcss.com)** - 유틸리티 기반 CSS 프레임워크
- **[shadcn/ui](https://ui.shadcn.com)** - 재사용 가능한 UI 컴포넌트
- **[Lucide React](https://lucide.dev)** - 아름다운 아이콘 라이브러리
- **[next-themes](https://github.com/pacocoursey/next-themes)** - 다크모드 지원

**CMS & API**
- **[Notion API](https://developers.notion.com)** - 콘텐츠 관리 시스템
- **[@notionhq/client](https://github.com/makenotion/notion-sdk-js)** - Notion JavaScript SDK
- **[notion-to-md](https://github.com/souvikinator/notion-to-md)** - Notion 블록을 마크다운으로 변환

## 🚀 빠른 시작

### 1. 의존성 설치

```bash
npm install
# 또는
yarn install
# 또는
pnpm install
```

### 2. Notion 설정

1. [Notion Developers](https://www.notion.so/my-integrations)에서 새로운 Integration을 생성합니다
2. Integration Token을 복사합니다
3. Notion에서 두 개의 데이터베이스를 생성합니다:
   - **Projects 데이터베이스**: 프로젝트 정보 저장
   - **Resources 데이터베이스**: 학습 리소스 정보 저장
4. 각 데이터베이스를 Integration에 연결합니다 (Share → Invite)
5. 데이터베이스 ID를 복사합니다 (URL에서 확인 가능)

자세한 데이터베이스 구조는 [PRD 문서](./docs/PRD.md#notion-데이터베이스-구조)를 참조하세요.

### 3. 환경변수 설정

```bash
cp .env.example .env.local
```

`.env.local` 파일을 편집하여 Notion API 정보를 설정하세요:

```env
NOTION_API_KEY=your_notion_integration_token
NOTION_PROJECTS_DATABASE_ID=your_projects_database_id
NOTION_RESOURCES_DATABASE_ID=your_resources_database_id
```

### 4. 개발 서버 실행

```bash
npm run dev
# 또는
yarn dev
# 또는
pnpm dev
```

[http://localhost:3000](http://localhost:3000)에서 결과를 확인하세요.

## 📁 프로젝트 구조

```
├── app/                    # Next.js 앱 라우터
│   ├── globals.css        # 전역 스타일
│   ├── layout.tsx         # 루트 레이아웃
│   └── page.tsx           # 홈페이지
├── components/            # React 컴포넌트
│   ├── ui/               # shadcn/ui 컴포넌트
│   ├── layout/           # 레이아웃 컴포넌트
│   └── providers/        # Context 프로바이더
├── lib/                  # 유틸리티 함수
└── public/               # 정적 파일
```

## 🎨 UI 컴포넌트

이 프로젝트는 shadcn/ui를 사용합니다. 새로운 컴포넌트를 추가하려면:

```bash
npx shadcn@latest add [component-name]
```

사용 가능한 컴포넌트:
- Button, Card, Input, Label
- Navigation Menu, Dropdown Menu
- Badge 등

## 🌙 다크모드

다크모드가 기본적으로 설정되어 있습니다:
- 시스템 테마 자동 감지
- 라이트/다크/시스템 모드 전환
- 테마 상태 유지

## 📝 스크립트

- `npm run dev` - 개발 서버 실행
- `npm run build` - 프로덕션 빌드
- `npm run start` - 프로덕션 서버 실행
- `npm run lint` - ESLint 실행

## 🔧 사용자 정의

### 테마 색상 변경

`app/globals.css`에서 CSS 변수를 수정하여 테마를 사용자 정의할 수 있습니다.

### 컴포넌트 추가

`components/` 디렉토리에 새로운 컴포넌트를 추가하세요. shadcn/ui 컴포넌트는 `components/ui/`에 자동으로 설치됩니다.

## 🚀 배포

### Vercel (권장)

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new)

### 기타 플랫폼

- **Netlify**: `npm run build && npm run export`
- **Docker**: Dockerfile 포함
- **정적 호스팅**: `npm run build && npm run export`

## 📚 학습 자료

- [Next.js 문서](https://nextjs.org/docs)
- [TailwindCSS 문서](https://tailwindcss.com/docs)
- [shadcn/ui 문서](https://ui.shadcn.com)
- [TypeScript 문서](https://www.typescriptlang.org/docs)

## 🤝 기여하기

1. 이 저장소를 포크합니다
2. 기능 브랜치를 생성합니다 (`git checkout -b feature/amazing-feature`)
3. 변경사항을 커밋합니다 (`git commit -m '멋진 기능 추가'`)
4. 브랜치에 푸시합니다 (`git push origin feature/amazing-feature`)
5. Pull Request를 생성합니다

## 📄 라이선스

MIT 라이선스 하에 배포됩니다. 자세한 내용은 `LICENSE` 파일을 참조하세요.
