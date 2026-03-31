# 쓰레드3 (BiniLab on Paperclip) Handoff — 세션 3 (2026-03-31)

## 현재 상태: 홈피드 수집 강화 + 글자수 시스템 + 파이프라인 재설계

---

## 세션 3 완료 작업

### 홈피드 셀프리플라이 + 제휴링크 수집 추가
- `scripts/collect-homefeed.ts`에 `extractSelfReply()` 함수 추가
- 상세 페이지 방문 시 조회수뿐 아니라 셀프리플라이 텍스트 + 쿠팡 제휴링크도 추출
- collect.ts의 DOM 추출 로직 재사용 (block identification, affiliate detection, clean text)
- DB insert에 `comments`, `link_url`, `link_domain`, `link_location` 필드 추가
- PR: chltlgns/oh-my-claudecode#1

### 글자수 검증 시스템 (gate9 + Jihyun bash)
- `gate9_preciseCharCount`: Unicode-aware `[...text].length` 카운트
- 동적 기준: `data/analysis/char_count_config.json`에서 읽음 (Seoyeon이 매일 업데이트)
- 기본값: optimal 80-120, hard_max 200 (config 없으면 기본값)
- Jihyun content.md: bash로 글자수 검증 필수화 (`node -e "console.log([...'텍스트'].length)"`)
- AI 추정치 금지 — char_count 필드에 bash 결과만 기입

### 글자수-조회수 분석 (DB 1,709 포스트)
- 81-120자: 중간값 조회수 1,350~1,600 (최적)
- 300자+: 중간값 324 (최악)
- Seoyeon performance-analyzer.md에 글자수 분석 쿼리 + config 출력 형식 추가

### 파이프라인 재설계
- **Phase 0 콜드스타트**: 게시 이력 없으면 성과 리뷰 스킵, 초기 파라미터 설정 후 바로 진행
- **Phase 0/5 역할 분리**: Phase 5 = 데이터 수집만, Phase 0 = 전략 조정만 (중복 제거)
- **글자수 2단계 접근**: 지금 = 분석 기반 동적, 추후 500개+ = COMPANY.md 고정 규칙

### Paperclip 서버
- 기존 default + binilab 인스턴스 종료 → binilab만 재시작
- URL: `http://127.0.0.1:3200`
- Junho(Researcher) error 상태 — 미해결

---

## Paperclip 서버
- URL: `http://127.0.0.1:3200`
- 인스턴스: binilab
- config: `쓰레드3/.paperclip/config.json` → `~/.paperclip/instances/binilab/config.json`

## 에이전트 (8명 활성)

| 이름 | ID | urlKey | 역할 | 상태 |
|------|-----|--------|------|------|
| Minjun(CEO) | 6e5e2def | minjun-ceo | 큰 방향, 주 1회 | idle |
| Yuna(COO) | 6d1eb85d | yuna-coo | 일일 운영, 파이프라인 트리거 | idle |
| Jihyun(CMO) | f3c1cbe5 | jihyun-cmo | 콘텐츠 기획+작성 | idle |
| Seoyeon(Analyst) | 86b0d549 | seoyeon-analyst | 데이터 분석 | idle |
| Junho(Researcher) | e5be940e | junho-researcher | 수집+게시 | **error** |
| Doyun(QA) | 6abc7359 | doyun-qa | 품질검수 | idle |
| Taeho(Engineer) | 62ed7b35 | taeho-engineer | 시스템 개발 | idle |
| Riri(Reviewer) | 9b4d5e0b | riri-reviewer | 코드 리뷰 | idle |

## 일일 파이프라인 (순차 실행, COMPANY.md 참조)

```
Phase 0: COO 전략 조정 (콜드스타트 or Phase 5 데이터 기반)
Phase 1: 수집 (Junho) → 홈피드만, --search 금지
Phase 2: 분석 (Seoyeon) → blockedBy Phase 1 → 글자수별 조회수 분석 포함 → config 업데이트
Phase 3: 기획+작성 (Jihyun) → blockedBy Phase 2 → bash 글자수 검증 필수
Phase 4: 게시 (Junho) → blockedBy Phase 3
Phase 5: 성과 수집 (Seoyeon) → 12시간 후 → 스냅샷 DB 저장 + 리포스트 태그
```

## DB 현황 (2026-03-31)
- thread_posts: 1,840개 (homefeed 38, keyword 360, benchmark 1,218, legacy 224)
- aff_contents: draft 14, ready 12, published 2
- 게시 완료: 2개 (워밍업 포스트)

---

## 다음 세션 TODO

### P0
1. Junho(Researcher) error 상태 해결
2. COO(Yuna) instructions 파일 작성 (binilab 인스턴스용)
3. 남은 draft 게시 (BIN-25)
4. schema.ts 정리 — 삭제된 테이블/enum 정의 제거

### P1
5. COO 파이프라인 전체 테스트 (콜드스타트 → Phase 0~4)
6. 성과 추적 자동화 — 12h postSnapshots 수집 구현 (Phase 5)
7. 리포스트 자동화

### P2
8. terminated 에디터 .claude/agents/ 파일 아카이브
9. Phase B 심층 분석 (thread_posts 500개+ 후)
10. 글자수 고정 규칙 등록 (데이터 안정화 후)

---

## 주요 파일 변경점 (이번 세션)

| 파일 | 변경 |
|------|------|
| `scripts/collect-homefeed.ts` | extractSelfReply() 추가, DB insert에 comments/link 필드 |
| `src/safety/gates.ts` | gate9_preciseCharCount 추가 (동적 config 기반) |
| `data/analysis/char_count_config.json` | 글자수 분석 config (Seoyeon → gate9 피드) |
| `COMPANY.md` | Phase 0 콜드스타트, Phase 0/5 분리, COO 루프 업데이트 |
| Jihyun content.md (Paperclip) | bash 글자수 검증, 2단계 접근, 유연한 가이드 |
| Seoyeon performance-analyzer.md (Paperclip) | 글자수별 조회수 분석 쿼리 + config 출력 |

---

## FAILED_APPROACHES
- 한글 에이전트 이름 @멘션 → regex 공백 매칭 실패 → 영문 이름 전환
- CEO에게 코딩 태스크 할당 → stale context → 태호 직접 assign
- 에디터 4명 분리 → 중복 태스크 + 비용 → Jihyun 통합
- Board가 직접 코드 수정 → 에이전트 워크플로우 깨짐 → 태스크 위임으로 전환
- community_posts 분석 → 잘못된 데이터 소스 → thread_posts만 사용
- AI 글자수 직접 카운트 → 부정확 → bash 검증 필수화
- Phase 0/5 성과 리뷰 중복 → Phase 5=수집만, Phase 0=전략만으로 분리
- PR을 원본 리포(Yeachan-Heo)에 생성 → fork(chltlgns)에 생성하도록 수정
