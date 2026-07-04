# FL Press

FreeLang fx2로 만든 WordPress급 CMS.

- 게시물 / 페이지 / 댓글 / 미디어 / 사용자 / 카테고리·태그 / SEO / 플러그인
- 포트: 40814 · 저장소: SQLite (`flpress.db`)
- 관리자: `/admin` · API: `/api/posts`, `/api/media`, `/api/auth/login` 등

## 실행
```bash
pm2 start fl-press        # 또는 fx2 빌드 후 ./fl-press
```

## 빌드 (fx2)
`server.fl` → fx2 컴파일 → `fl-press` 바이너리 (gitignore됨).
