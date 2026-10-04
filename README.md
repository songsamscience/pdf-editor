# 송쌤과학 PDF 편집기

**브라우저 안에서만** 처리되는 PDF 도구 27개를 한 파일(`index.html`)로 만든 사이트. 파일은 서버로 가지 않는다.

## 1.3.1 (2026-10-05) 여러 결과는 zip 한 번에 (바꾸기 전 상태: `archive-v1.3.0/`)
- **결과 화면 「zip으로 모두 저장」 띠**(`50_app.js` `renderResult`, CSS `11_preview.css` `.pv-bulk`): 결과가 2개 이상이면 목록 **위**에 「그림 24장을 zip 파일 하나로 · 이름 · 약 크기」와 단추를 둔다. 전에는 아래 「모두 저장 (zip)」만 있어 그림이 많으면 격자 밑으로 밀려 안 보였고, 그림마다 「저장」을 눌러야 했다. 아래 단추·Ctrl+S 도 그대로.
- **한 개면 그 파일 그대로**: 결과가 하나면 zip 으로 묶지 않는다. 그림 한 장이면 아래 단추가 「그림 저장 (JPG/PNG)」.
- **zip 이름**: 도구가 `run` 에서 `ctx.zipName` 을 정하면 그 이름(→ `PE.state.resultMeta.zipName`). PDF → 그림은 `원본이름_그림.zip`(그림 추출은 `_추출그림`, PDF 여러 개면 `PDF_날짜_그림.zip`). 나머지 도구는 `zipNameOf('도구이름_날짜')` — 「PDF → 그림」의 화살표는 `_`로, 파일 이름에 못 쓰는 글자는 뺀다.
- **`makeZip`**(`30_core.js`): JPG·PNG·PDF·zip·docx 는 다시 압축하지 않고 담아(STORE) 수십 장도 금방 묶인다. 묶는 진행률 표시, 대소문자만 다른 이름(`A.jpg`·`a.JPG`)도 겹침 처리, zip 안 파일 시각을 현지 시각으로(JSZip 은 UTC 로 적어 9시간 어긋났다), 실패하면 알림.
- **빌드 정리**: OneDrive 동기화 충돌로 `src/`에 `…-송쌤과학의 MacBook Pro.js` 사본 10개와 옛 `10_style.css`·`32_ui.js`·`15_style_interaction.css`(10-03 접근성 다듬기 갈래, 배포본에 들어간 적 없음)가 섞여 있었다. 그대로 빌드하면 같은 `const`를 두 번 선언해 뒤 조각이 통째로 멈춘다. 그 파일들은 `onedrive-conflict-2026-10-04/`로 옮기고 두 파일은 배포본(1.3.0)과 같은 것으로 되돌렸다. `build.py`는 이제 `00_이름.css|js|html` 꼴이 아닌 파일을 건너뛰고 알린다.
- 시험: `tools/cases/zipsave.js`.

## 1.3.0 (2026-10-04) 결과 비교·미리 확인 (바꾸기 전 상태: `archive-v1.2.1/`)
- **원본과 비교**(`35_viewer.js`): 원본 한 파일 → 결과 한 파일이고 쪽이 그대로인 도구(`PE.viewer.SAME_PAGES`: 압축·복구·OCR·회전·쪽 번호·워터마크·자르기·양식·서명·가리기·잠금 해제·암호·QR·쪽 크기, 편집은 쪽 판을 안 바꿨을 때)는 실행 뒤 결과마다 원본을 숨김 속성 `o.src`로 붙인다(`linkSources`, 결과 이름 = 원본 이름_꼬리). 결과 줄 「원본과 비교」 또는 큰 보기의 단추·글쇠 C → 같은 쪽·같은 확대로 **밀어 보기**(막대를 끌어 겹쳐 봄, 쪽 크기가 같을 때) / **나란히**(두 창 스크롤 맞춤, 휴대폰은 위아래). 라벨에 원본·결과 크기와 증감.
- **OCR**: 결과(`o.ocr.pages` = 인식한 쪽) 큰 보기 아래 「인식한 글자」 끔·상자·읽은 글 — 글자 층을 쪽 위에 겹쳐 어디를 무엇으로 읽었는지 확인(인식한 쪽/원래 글자 쪽 표시). 도구 화면 「한 쪽 시험 인식」 탭: 지금 언어·해상도로 고른 쪽만 인식해 낱말 상자(확신도 색: 높음·보통·낮음, 30% 미만은 점선=글자 층에서 뺌)·읽은 글·걸린 시간. 일꾼은 `PE.ocrWorker`, 낱말은 `PE.ocrWordsOf`(실행과 같음), 도구를 떠나면 정리.
- **그림→PDF·스캔 「쪽 미리 보기」**: 쪽 크기·방향·여백을 바꾸면(또는 그림 카드를 누르면) 그 그림이 놓이는 쪽을 그린다. 실행과 같은 계산 `PE.imagePageLayout`.
- **PDF→그림 「미리 보기」**(기본 탭): 고른 쪽을 지금 해상도·형식으로 실제로 만들어 픽셀 크기·용량·장수 합계를 보여 주고 「실제 크기」로 선명도 확인. 그림 추출은 그 쪽에 든 그림들(`PE.pageImages`, 실행과 같음).
- **합치기 「합친 쪽 미리 보기」**: 합칠 순서대로 모든 쪽(파일 돌리기·양면 빈 쪽·책갈피 표시, `PE.mergePages`), 쪽을 누르면 큰 보기(`PE.viewer.openPages`, 입력 쪽 목록 — 저장·인쇄 없음).
- CSS: `11_preview.css`(pv-cmp…), `11_ocr.css`(ocr-), `11_viewtools.css`(ipv-·jpv-·mg-). 「PDF 비교」 도구의 `.cmp`·`.col` 과 겹치지 않게 상태 클래스는 `pv-cmpon`·`pv-cmp-col`.
- 시험: `tools/cases/viewcheck.js`(실제 마우스: 비교 막대 끌기·「원본과 비교」 단추·합친 쪽 카드).

## 1.2.1 (2026-10-04) 고친 것 (고치기 전 상태: `archive-v1.2.0/`)
- **편집 「글」(T)·「글 고치기」(E)가 실제 마우스로 안 되던 문제**: 쪽을 누르면 pointerdown 에서 연 입력 칸을 뒤따르는 mousedown 의 기본 동작(초점 옮기기)이 바로 닫아, 새 글 상자는 빈 채로 지워지고 고칠 줄은 입력 칸이 열리자마자 닫혔다. 무대 캔버스(`canvas.ov`)의 mousedown 기본 동작을 막는다(`31_stage.js`). 시험은 합성 PointerEvent 라 mousedown 이 없어 못 잡았다 → 「실제 마우스」 사례 추가.
- 쪽 오른쪽 끝에 글 상자를 넣으면 폭이 몇 pt 로 줄어 「10점」이 한 글자씩 줄 바뀜 → 글자 다섯 개(또는 넣을 이름·날짜) 폭은 남기고 모자라면 왼쪽으로 당긴다.
- 쪽 범위를 「2 - 5」·「3 ~ end」처럼 띄어 쓰면 2·5쪽만 골라지던 문제(`parseRanges`, 쪽 삭제·추출·분할·쪽 번호·워터마크·OCR 등 모든 범위 칸).
- 쪽 번호 「시작 번호」에 0을 넣으면 1부터 찍히던 문제.
- 분할 「쪽」에서 고른 뒤 파일을 바꾸면 「고른 2쪽」이 남아 실행해도 결과가 없던 문제(`PE.pickPages` 가 없는 파일의 쪽을 고름에서 뺀다).
- 편집·서명·자르기·가리기·양식 무대 「맞춤」에서 쪽이 몇 px 넘쳐 윈도우에서 가로 스크롤바가 생기던 문제(무대 여백은 좌우 40px인데 36px만 뺐다 → 실제 padding 을 읽는다).
- 시험: `tools/cases/fixes.js`(위 버그들, 무대는 실제 마우스 클릭).

## 1.2.0 (2026-10-04) 바뀐 것
- **학생별로 나누기**(수행평가 왕복): 합치기 「파일마다 책갈피 넣기」(기본 켬, `PE.writeOutline`)·「양면 인쇄용 빈 쪽」. 분할에 「학생별」 — 책갈피마다(`PE.readOutline`) / 빈 종이(구분지)마다(`PE.findBlankPages`) / N쪽씩, 명렬표 붙여넣기로 `03_홍길동.pdf` 이름(`PE.parseRoster`). 만들 파일 목록을 먼저 보여 준다. 쪽 번호·워터마크·편집(쪽 패널 포함)은 같은 문서를 고쳐 저장하므로 책갈피가 남는다.
- **설정 기억**(`37_remember.js`, `PE.prefs`): 도구별 마지막 실행 옵션을 `localStorage pe.opts.<id>`에 두고 init 직후 불러온다. 옵션 판 「기본값으로」. 암호·그림·쪽 선택·범위는 저장하지 않는다. 기본 회귀 사례는 `pe.prefs.off`로 끈다(`regress_cases.js` 첫 줄).
- **PDF → 글 뽑기**(`46_tools_text.js`, id `pdf2text`): 줄·두 단·문단·제목·머리글을 다시 묶어 TXT·마크다운·Word(.docx, JSZip으로 직접 만듦). 작업 영역에서 쪽마다 미리 보기·복사, 스캔본은 「OCR 먼저 하기」.
- **QR 코드 넣기**(`47_tools_qr.js`, id `qr`): 벡터 QR + 누르는 링크(Link 주석). qrcode-generator 1.4.4는 도구를 열 때만 cdnjs에서 받는다. 실시간 미리보기에서 끌어 놓기, 회전·원점 이동 쪽도 맞음.
- **OCR**: 이미 글자가 있는 쪽은 건너뛰기(기본 켬), 「정밀」 300 dpi, 쪽 범위 오류 안내.
- 편집기 쪽 패널로 쪽을 바꿔도 원래 문서를 고쳐 쓰므로 양식 칸·책갈피·문서 정보가 남는다(지운 쪽의 글자는 파일에서도 빠짐).

## 1.1.0 (2026-10-04) 바뀐 것
- **PDF 열어서 바로 편집**(PDF 편집 도구): 「글 고치기」(E) — 원래 글 줄을 눌러 그 자리·크기·색으로 고치고 옮긴다. 원래 글리프는 내용 흐름에서 같은 폭의 `[-n] TJ`로 바꿔 데이터에서도 지운다(양식 XObject 속 글·잴 수 없는 글꼴은 배경색 덮개로 대신하고 안내). 쪽 패널 — 쪽 삭제·끌어서 위치 옮기기·돌리기·복제·빈 쪽·다른 PDF 끼워 넣기, 요소는 자기 쪽을 따라감, 모두 같은 Ctrl+Z. 요소 Ctrl+C/V/D, 「모든 쪽 같은 자리에」.
- **용량 예측**: 실행 단추 위 「예상 결과 ≈ …」(도구의 `estimate(ctx)` 또는 `PE.estimators[id]`, `36_estimate.js`). 압축은 단계별 예상 크기와 「목표 용량 이하로」 모드.
- **결과 미리보기**: 결과 줄 썸네일, 큰 보기(`35_viewer.js`), 인쇄, 이름 바꾸기, 「설정 고쳐 다시 하기」, 나가기 경고.
- **실시간 미리보기**: 쪽 번호·워터마크(실제 결과 한 쪽을 그려 보여 줌), 쪽 번호 도구에 머리글·바닥글 6칸.
- **새 도구**(`45_tools_layout.js`): 모아 찍기·소책자, 쪽 나누기(A3 펼침면 → A4), 쪽 크기 맞추기.
- **정리**: 빈 쪽 자동 찾기·홀수/짝수/반전 고르기, 양면 스캔 순서 맞추기, 이름 자연 정렬(1, 2, 10).
- **고친 버그**: 가리기 결과에 원래 글자가 남던 문제, 압축이 투명 마스크·Predictor 그림을 깨뜨리던 문제, 되돌리기 뒤 도장·그림이 깨져 저장 실패, 투명 PNG가 검게 되던 문제, 분할 범위 오류, 자르기 「모든 쪽」 자동 여백 오류, 채운 양식 값이 그림 변환·미리보기에서 빠지던 문제.

## 구조
- `src/` 조각 → `python build.py` → `index.html` (배포본, 직접 고치지 말 것. 윈도우에서도 LF로 씀)
  - `00_head.html` CDN 라이브러리 · `10_style.css` 공통 스타일 · `11_*.css` 기능별 스타일 · `20_shell.html` 머리·대화상자
  - `30_core.js` 공통(파일 저장소, pdf.js·pdf-lib 열기, 한글 글꼴, 진행 표시)
  - `31_stage.js` 한 쪽 편집 무대(편집·서명·자르기·가리기·양식 만들기가 공유). 그림 바이트는 `st.assets`에 두고 되돌리기 스냅숏에는 넣지 않는다
  - `32_ui.js` 옵션 위젯·파일 격자·쪽 격자·끌어서 정렬
  - `33_textedit.js` 원래 글 고치기(줄 찾기, 내용 흐름 해석·글리프 지우기) · `34_pagepanel.js` 편집 도구 쪽 패널
  - `35_viewer.js` 결과 큰 보기 · `36_estimate.js` 용량 예측 · `37_remember.js` 설정 기억
  - `40_tools_organize.js` 합치기·분할·쪽 삭제·쪽 추출·정리
  - `41_tools_optimize.js` 압축·복구·OCR
  - `42_tools_convert.js` 그림→PDF·스캔→PDF·PDF→그림
  - `43_tools_edit.js` 편집·회전·쪽 번호(머리글·바닥글)·워터마크·자르기·양식·서명
  - `44_tools_security.js` 잠금 해제·암호 걸기·가리기·비교
  - `45_tools_layout.js` 모아 찍기·소책자·쪽 나누기·쪽 크기 맞추기
  - `46_tools_text.js` PDF → 글 뽑기 · `47_tools_qr.js` QR 코드 넣기
  - `50_app.js` 홈 → 도구 → 결과 화면 흐름
- 기능별 CSS 클래스는 접두어를 나눠 쓴다(결과 보기 `pv-`, 쪽 번호 미리보기 `lp-`, 학생별 `stu-`, 설정 기억 `rem-`, 글 뽑기 `tex-`, QR `qr-`, OCR 시험 인식 `ocr-`, 그림→PDF 미리 보기 `ipv-`, PDF→그림 미리 보기 `jpv-`, 합치기 미리 보기 `mg-`). 같은 이름을 두 파일에서 정의하면 서로 섞인다 — `.cmp`·`.col`(PDF 비교 도구)처럼 짧은 이름은 특히 조심.

## 화면 체계 (2026-10 개편)
- 푸른색 단일 체계: 토큰은 `10_style.css` 맨 위 `:root`(파랑 명도 단계·역할 색·갈래 색·형태). 교실 도우미의 「진한 소개 판 + 흰 작업 판」을 재해석했다.
- 홈: `.hero` = `.story`(남색 소개 판, 비스듬한 `.preview` 카드) + `.setup`(흰 판: 빠른 칩·큰 검색·화살표 CTA). 아래 `#tools`(갈래 `.cat-nav` 고정 띠 + `.cat` 격자). 검색은 입력 밑 `.search-results`와 격자 흐리기(`#tools` 안만)를 함께 한다.
- 도구 화면: 머리에 같은 갈래 전환 칩(`.tool-switch` → `App.switchTo`, 호환 파일은 가져감). 옵션 판 머리는 eyebrow `OPTIONS` + `b` 제목(`ctx.setPanelTitle`). 실행 단추는 `span.label` + 화살표(`ctx.setRunLabel`). `ctx.setRunEnabled(on, why)`의 둘째 인자는 실행 단추 위 `.run-hint`에, `ctx.showError(html)`은 `.note.err`에 보인다. 예상 용량은 그 위 따로 둔 칸(`setMeta`로 지워지지 않음). ≤900px 에서는 `.panel`이 화면 아래 고정 독(`.panel.open` 이 몸통 펼침, 높이는 `--dock-h`).
- 옵션 2단: 자주 쓰는 것은 위, 나머지는 `PE.ui.adv('제목', ...)`(details) 안에.
- 무대: 되돌리기·비우기는 `st.navExtra`(아래 줄 오른쪽), 편집 도구의 새 요소 스타일은 `.stage-sub`(도구 줄 아래), 도장은 도구 줄 팝오버 `.stamp-pop`. 글쇠 `st.keyModes`(V·T·E·H·P·R·O·L·A·W…), `+`/`-`/`0` 확대, Ctrl+휠.
- 전역 글쇠: `/` 도구 찾기(`#dlg-finder`), `Ctrl+O` 파일 열기, `Ctrl+V` 붙여넣기, `Ctrl+Enter` 실행, 결과 화면 `Ctrl+S` 저장·`Ctrl+P` 인쇄. 페이지 어디에 끌어 놓아도 `.drop-veil`이 받는다.
- 결과 화면: 남색 띠(`.result-top`) + 흰 몸통, 결과 줄마다 썸네일·미리 보기·인쇄·이름 고치기. 「이어서 하기」는 `NEXT_OF` 추천 4개 + 접힌 나머지. 실행한 도구는 `localStorage pe.recent`, 머리글·바닥글은 `pe.hf`에 기억한다.
- 개발 서버: `.claude/launch.json`의 `pdf-editor`(맥, 8832) · `pdf-editor-win`(윈도우, 8834)

## 라이브러리 (CDN)
- `@cantoo/pdf-lib` 2.11.1 — 만들기·고치기·암호. **`@cantoo/fontkit` 2.0.12 필수** (옛 `@pdf-lib/fontkit`은 한글 글꼴 부분 집합 실패)
- `pdf.js` 3.11.174 — 그리기·글자 읽기 (`intent:'print'`로 그려야 숨긴 탭에서도 멈추지 않음)
- `Tesseract.js` 5.1.1 — OCR (처음 쓸 때만 내려받음)
- `JSZip`, `jsdiff`, Pretendard(PDF 글꼴, 쓰인 글자만 담음)

## 검사
- `tests/fixtures/` 시험용 PDF·그림 (PyMuPDF로 만든 것, `locked.pdf` 암호 1234). 기능별로 만든 것은 `editor_*`, `layout_*` 같은 접두어.
- `tests/preview.html?tool=<id>&files=a.pdf,b.pdf&act=stage|run` — 도구 화면을 자동으로 채우는 하네스
- **윈도우(노드 없음)**: `python tools/regress.py --serve` — 서버와 헤드리스 크롬을 스스로 띄워 기본 51개를 돈다.
  - 기능별 사례: `--cases tools/cases/editor.js` (여러 개는 쉼표, `"tools/cases/*.js"`도 됨). 사례 파일은 `REG.h` 도우미와 `REG.add(cases, drags)`를 쓴다.
  - 사례 파일을 한꺼번에 이어 돌리면 앞 사례가 남긴 상태가 뒤를 막을 수 있으니 **파일마다 따로**(동시에 여러 개) 돌린다.
  - 화면 찍기: `python tools/regress.py --serve --shot out.png --tool merge --files doc5.pdf [--js "식"] [--size 390x844]` (Git Bash에서는 `--hash "#/…"`가 경로로 바뀌므로 `--tool`을 쓴다)
- 맥: `tools/regress.mjs` + `tools/regress_cases.js` (StreamDeck에 딸린 node), `tools/cdp.mjs` 스크린샷
- 브라우저 콘솔에서 `App.ctx`, `PE.state`, `PE.tools`로 상태를 볼 수 있다.

## 새 도구 추가
```js
PE.registerTool({ id, cat, name, desc, icon, accept: 'pdf'|'image'|'any', multi, min,
  init(ctx) {}, render(ctx) { /* ctx.work, ctx.panel.body 채우기 */ },
  async estimate(ctx) { return { bytes, files, note }; },   // 선택: 예상 용량
  async run(ctx) { return [{ name, bytes, mime }]; } });
```
