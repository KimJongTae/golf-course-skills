# Golf Course Skills

골프장 URL로부터 홀별 공략 가이드를 생성하는 Claude Code 스킬 모음입니다.

## 스킬 목록

### golf-course-guide

골프장 웹사이트 URL을 입력하면 전 코스 18홀 공략 가이드를 HTML 파일로 자동 생성합니다.

**주요 기능:**
- 골프장 사이트 크롤링으로 홀별 PAR, HDCP, 티별 거리 자동 추출
- 각 홀의 코스맵 이미지 자동 포함
- 홀별 코스 공략 & 그린 공략 설명 생성
- 탭 네비게이션 (전체 / OUT / IN / 스코어카드)
- 브라우저 인쇄로 PDF 저장 (홀당 1페이지, A4)

**사용 예시:**
```
https://www.somegolfclub.com 이 골프장 18홀 공략 가이드 만들어줘
```

## 설치

```bash
npx skills add blackkjt/golf-course-skills@golf-course-guide
```

## 요구사항

- Claude Code CLI
- WebFetch 도구 접근 권한

## 라이선스

MIT
