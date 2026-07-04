# FL Press 📝

> **FreeLang fx2로 처음부터 만든 WordPress급 CMS.**
> PHP도, MySQL도, 외부 npm도 없이 — C 네이티브 바이너리 하나로 도는 자체 제작 콘텐츠 관리 시스템.

[![FreeLang fx2](https://img.shields.io/badge/FreeLang-fx2-blue)](https://gogs.dclub.kr/kim/freelang-v11-fx)
[![Port](https://img.shields.io/badge/port-40814-green)]()
[![Storage](https://img.shields.io/badge/db-SQLite-lightgrey)]()

---

## 왜 만들었나

워드프레스를 쓰려면 PHP + MySQL + 웹서버 + 수십 개 플러그인이 필요하다. FL Press는 그 전부를
**FreeLang fx2로 컴파일된 단일 실행 바이너리 + SQLite 파일 하나**로 대체한다.

- **설치 지옥 없음** — 바이너리 실행하면 끝. DB도 자동 생성.
- **빠름** — fx2 = C 네이티브. 인터프리터 오버헤드 0.
- **완전 소유** — 남의 호스팅·구독 없이 내 서버에서 100% 로컬 운영.
- **워드프레스급 기능** — 게시물/페이지/블록 에디터/댓글/미디어/사용자·권한/SEO/카테고리·태그.

---

## 기능 한눈에

| 영역 | 기능 |
|------|------|
| **콘텐츠** | 게시물(post) · 페이지(page) — `post_type`으로 구분. 발행/초안 상태(`status`). |
| **블록 에디터** | 글 본문을 블록(`blocks` 컬럼, JSON)으로 저장. 마크다운 렌더 지원. |
| **퍼머링크** | `/:slug` — 워드프레스식 슬러그 URL로 글 직접 접근. |
| **분류** | 카테고리(1:N) + 태그(N:M, `post_tags` 조인 테이블). |
| **댓글** | 작성 · **승인 모더레이션**(`/approve`) · 삭제. |
| **미디어** | 업로드 + `/uploads/:filename` 서빙 + 목록/삭제. |
| **사용자·권한** | `role` 기반 · bcrypt 비밀번호 해시 · **세션 인증**(`sessions` 테이블). |
| **SEO** | `/sitemap.xml` · `/robots.txt` · 슬러그 · featured image · excerpt. |
| **관리자** | `/admin` — 1,779줄 단일 페이지 관리 UI (대시보드/글/댓글/미디어/설정). |
| **설정** | `options` 테이블 기반 사이트 설정(`/api/settings`). |

---

## 아키텍처

```
┌───────────────────────────────────────────┐
│  fl-press (fx2 C 네이티브 바이너리, :40814) │
│                                             │
│  프론트    /              홈 (글 목록)      │
│            /:slug         글/페이지 상세     │
│            /admin         관리자 SPA         │
│            /uploads/:f    미디어 서빙         │
│            /sitemap.xml   SEO                │
│                                             │
│  REST API  /api/posts   /api/pages          │
│            /api/comments /api/media          │
│            /api/users    /api/categories     │
│            /api/tags     /api/settings       │
│            /api/auth/login  /api/dashboard   │
│                                             │
│  저장소    flpress.db (SQLite, WAL 모드)    │
└───────────────────────────────────────────┘
```

### DB 스키마 (9 테이블)

| 테이블 | 역할 |
|--------|------|
| `users` | id · username · email · **password_hash** · **role** · display_name · bio |
| `posts` | id · title · **slug** · content · **blocks** · excerpt · status · **post_type** · author_id · category_id · featured_image · view_count · created_at · updated_at |
| `categories` / `tags` / `post_tags` | 분류 (태그는 N:M 조인) |
| `comments` | 댓글 (승인 상태 포함) |
| `media` | 업로드 파일 메타 |
| `options` | 사이트 설정 (key-value) |
| `sessions` | 세션 인증 토큰 |

---

## REST API

### 인증
```bash
# 로그인 → 세션
curl -X POST http://localhost:40814/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"..."}'
```

### 게시물
| 메서드 | 경로 | 설명 |
|--------|------|------|
| GET | `/api/posts?status=published` | 목록 (상태 필터) |
| GET | `/api/posts/:id` | 상세 |
| POST | `/api/posts` | 작성 (title/slug/content/blocks/status/category_id/featured_image) |
| POST | `/api/posts/:id` | 수정 |
| POST | `/api/posts/:id/delete` | 삭제 |
| POST | `/api/posts/:id/tags` | 태그 연결 |

### 그 외
- **페이지**: `/api/pages` (CRUD)
- **댓글**: `/api/comments` · `/api/comments/:id/approve` · `/delete`
- **미디어**: `GET /api/media` · `POST /api/media/:id/delete`
- **분류**: `/api/categories` · `/api/tags`
- **사용자**: `/api/users` (CRUD)
- **대시보드**: `GET /api/dashboard` (통계)
- **설정**: `/api/settings`

---

## 실행

```bash
# PM2 (권장)
pm2 start fl-press

# 또는 직접
./fl-press          # fx2 컴파일 바이너리
```

- 웹: http://localhost:40814/
- 관리자: http://localhost:40814/admin
- 최초 실행 시 DB(`flpress.db`)와 기본 게시물 자동 생성.

### 빌드 (fx2)

`server.fl` (41KB, FreeLang fx2 소스) → fx2 컴파일 → `fl-press` C 바이너리.
> 바이너리와 `flpress.db*`(런타임 데이터)는 `.gitignore` 처리 — 소스만 저장소에 포함.

---

## 파일 구조

```
fl-press/
├── server.fl          # 전체 서버 로직 (라우트/DB/인증/렌더) — 41KB
├── fx-std.fl          # fx2 표준 라이브러리
├── app/
│   └── admin.html     # 관리자 SPA (1,779줄)
├── .projectrc.json    # 메타데이터 (포트 40814)
└── flpress.db         # SQLite (gitignore)
```

---

## 콘텐츠 자동 발행 (선택)

FL Press는 API 발행을 지원하므로 자동화 파이프라인의 발행 대상으로 쓸 수 있다:

```
키워드 리서치  → agent-reach
글 초안        → 로컬 LLM (무료)
이미지         → 로컬 FLUX → POST /api/media
발행           → POST /api/posts
```

---

*FreeLang fx2 — C 네이티브, 셀프호스팅, 외부 의존성 0.*
