# Security Notes

정보보안과 모의해킹 학습 내용을 기록하는 Quartz 기반 기술 블로그입니다.

- Site: <https://Alt030.github.io>
- Framework: Quartz 5
- Content: Markdown / Obsidian
- Hosting: GitHub Pages

## 로컬 실행

Node.js 22 이상과 npm 10.9.2 이상이 필요합니다.

```bash
npm ci
npx quartz build --serve
```

브라우저에서 <http://localhost:8080>으로 접속합니다.

## 글 작성

게시할 Markdown 파일은 `content/` 아래에 저장합니다. 새 글은 `content/templates/post-template.md`를 복사해 작성할 수 있습니다.

```text
content/
├── posts/
│   ├── web-security/
│   ├── mobile-security/
│   ├── system-security/
│   ├── browser-security/
│   └── study/
├── projects/
└── assets/
```

이미지는 `content/assets/`에 저장하고 Obsidian 문법으로 첨부합니다.

```markdown
![[assets/example.png|이미지 설명]]
```

## 빌드 확인

```bash
npx quartz build
```

정적 빌드 결과는 `public/`에 생성됩니다.
