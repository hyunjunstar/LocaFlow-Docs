# LocaFlow 사용 설명서 (한국어)

LocaFlow는 언리얼 엔진 5.5·5.8용 에디터 도구입니다. 게임을 번역한 **뒤에** 깨지는 것들을 찾아 주고,
에디터 안에서 바로 고칠 수 있게 도와줍니다.

- **포맷 검사**: `{인자}` 누락·이름 변경, `|plural(...)` / `|gender(...)` / `|hpp(...)` 문법 오류,
  리치 텍스트 태그(`<b>…</>`, `<img/>`) 불일치를 찾습니다.
- **한국어 조사**: `{Name}을(를)` 같은 표기를 찾아 한 번에 `{Name}|hpp(을,를)`로 바꿉니다.
  엔진이 실행 중에 받침에 맞는 조사를 고릅니다.
- **빠진 글자(글리프)**: 폰트가 그리지 못하는 글자를 문화권별로 보여 주고, 시스템 폰트나 원하는
  .ttf / .otf / .ttc 파일을 대체(fallback) 서브 폰트로 등록합니다.
- **텍스트 넘침**: 위젯 블루프린트를 문화권마다 그려 보고, 칸을 넘는 텍스트를 썸네일과 함께 알려 줍니다.
- **번역 작업 흐름**: 문화권별 완료율, 바로 편집, 번역가와 CSV / PO 주고받기, 그리고 선택 사항으로
  DeepL 또는 OpenAI 호환 API를 쓰는 일괄 기계 번역(비용 미리 보기, 작업당 상한)을 제공합니다.

LocaFlow는 **Localization Dashboard**가 만드는 표준 파일(`Content/Localization/<Target>/...`)을 그대로 읽고 씁니다.
수집(Gather) 설정은 건드리지 않으며, Localization Dashboard도 그대로 쓸 수 있습니다.

---

## 1. 요구 사항

| | |
|---|---|
| 엔진 | 언리얼 엔진 5.5, 5.8 |
| 에디터 플랫폼 | Windows 64비트(테스트 완료). Mac·Linux는 같은 코드로 빌드되지만 아직 테스트하지 않았습니다. |
| 프로젝트 | 블루프린트·C++ 모두 가능. 현지화 타깃을 한 번 이상 **수집(Gather)**해 둬야 합니다. |
| 네트워크 | 기계 번역에만 필요합니다(선택). 검사 기능은 모두 오프라인으로 동작합니다. |

에디터 모듈(`LocaFlowCore`, `LocaFlowQA`, `LocaFlowTranslate`, `LocaFlowEditor`)은 게임 패키지에 들어가지 않습니다.
작은 `LocaFlowRuntime` 모듈은 게임 내 언어 메뉴용 블루프린트 노드 4개를 제공합니다(10장, 선택 사항).

## 2. 설치

1. Fab에서 LocaFlow를 엔진에 설치합니다(Fab / 에픽게임즈 런처 → *엔진에 설치*).
2. 프로젝트를 열고 **편집 → 플러그인**에서 **LocaFlow**를 찾아 켠 뒤 에디터를 다시 시작합니다.
3. **툴 → LocaFlow**를 엽니다.

## 3. 현지화 타깃 준비 (처음 한 번)

LocaFlow는 Localization Dashboard가 만든 데이터를 읽습니다. 현지화를 처음 한다면:

1. **툴 → Localization Dashboard**를 엽니다.
2. 기본 *Game* 타깃을 쓰거나 새로 추가합니다. *Gather Text*에서 *Gather from Text Files* 또는
   *Gather from Packages*를 켜고 Source / Content 폴더를 지정합니다.
3. 원하는 문화권(예: *ko*, *ja*, *zh-Hans*, *de*)을 추가하고 기본(native) 문화권(보통 *en*)을 고릅니다.
4. **Gather Text**를 누릅니다. `Content/Localization/Game/Game.manifest`와 문화권마다 `Game.archive`가 만들어집니다.

그 뒤로 LocaFlow에 타깃과 문화권이 보입니다. 게임에 새 텍스트를 넣으면 *Gather Text*를 다시 누르고
LocaFlow에서 **Reload**를 누르세요.

## 4. 대시보드

| 영역 | 하는 일 |
|---|---|
| **Target / Culture** | 작업할 현지화 타깃과 문화권을 고릅니다. |
| **Reload** | 타깃과 현재 문화권을 디스크에서 다시 읽습니다(저장 안 한 편집이 있으면 먼저 물어봅니다). |
| **Save** | 편집 내용을 문화권의 `.archive`에 씁니다. 세션마다 처음 저장하기 전에 `.bak` 백업을 만듭니다. |
| **Run All QA** | 현재 문화권의 포맷·리치 텍스트·한국어 조사(ko)·빈 항목·오래된 항목을 검사합니다. |
| **Translate Missing** | *Missing*·*Stale* 항목을 기계 번역합니다(8장). |
| **Export / Import CSV, PO** | 번역가와 번역을 주고받습니다(7장). |
| **완료율 막대** | 타깃의 모든 문화권별 번역 완료 비율입니다. |
| **필터** | 키·원문·번역으로 검색하고 상태(*Translated, Missing, Stale, Same as source, Has issues, Human edited*)로 거릅니다. |
| **항목 목록** | 키, 원문, 번역(클릭해서 편집), 상태, 문제 개수. 직접 고친 항목은 *(human)*으로 표시됩니다. |
| **QA Issues / Font Wizard / Overflow** | 아래쪽 도구 탭 세 개입니다. |

디스크에 아직 쓰지 않은 편집이 있으면 툴바에 `* Unsaved changes`가 보입니다.

### 항목 상태

| 상태 | 뜻 |
|---|---|
| Translated | 번역이 있습니다. |
| Missing | 아직 번역이 없습니다. |
| Stale | 번역을 쓴 뒤 원문이 바뀌었습니다. 언리얼은 이런 번역을 컴파일할 때 쓰지 않으므로, 다시 번역하거나 승인할 때까지 게임에는 원문이 나옵니다. |
| Same as source | 기본 문화권이 아닌데 번역이 원문과 똑같습니다(대개 번역 안 됨). |

## 5. QA 검사

**Run All QA**를 누르면 **QA Issues** 탭에 종류별로 결과가 모입니다. 문제를 클릭하면 해당 항목이 선택되고,
고칠 방법이 있는 문제는 **Apply fix**로 번역을 제안된 문장으로 바꿉니다.

| 검사 | 심각도 | 예 |
|---|---|---|
| 포맷 인자 | 오류 | 원문 `Welcome, {Name}!` → 번역 `환영합니다!` (`{Name}` 빠짐) |
| plural / gender / hpp 문법 | 오류 | `{Count}|plural(one=item,other=items` (`)` 빠짐) |
| 리치 텍스트 태그 | 오류 | 원문 `Press <b>Start</> to begin.` → 번역에 `<b>` 없음 |
| 한국어 조사 | 정보(자동 수정 가능) | `{Name}을(를) 초대했습니다.` → `{Name}|hpp(을,를) 초대했습니다.` |
| 빈 번역 | 경고 | Missing 항목 |
| 오래된 번역 | 경고 | 번역 뒤 원문이 바뀐 항목 |

참고:

- `|plural(...)` 안에만 나오는 인자도 있는 것으로 칩니다. 그래서 `{Count} {Count}|plural(...)`을
  `{Count}개`처럼 바꿔 써도 오류가 아닙니다.
- 중괄호를 글자로 쓰려면 언리얼 방식대로 `{`·`}` 앞에 백틱(`)을 붙입니다.
- QA Issues 탭의 **Convert all 을(를) to hpp**(ko 문화권)는 모든 조사 표기를 한 번에 바꿉니다.
  `{인자}` 바로 뒤에 오는 을(를), 이(가), 은(는), 와(과), 으로(로), 이라(라)를 지원합니다.

## 6. 폰트: 빠진 글자와 대체 폰트

**Font Wizard** 탭을 엽니다.

1. **Font** 에셋을 고릅니다(*Runtime* 컴포지트 폰트, UE5에서 만드는 폰트의 기본값). 기본값은
   *에디터 환경설정 → 플러그인 → LocaFlow → Reference Font*에서 정할 수 있습니다.
2. **Check glyphs**를 누릅니다. 현재 문화권 번역에 쓰인 모든 글자를 폰트의 기본 서체·폴백 서체·서브 폰트
   (각 서브 폰트의 글자 범위 포함)로 그릴 수 있는지 검사하고, 그릴 수 없는 글자를 보여 줍니다.
3. 고치려면 **Use system font**(Windows: 맑은 고딕 / Yu Gothic / Microsoft YaHei·JhengHei, Mac: Apple SD Gothic Neo /
   Hiragino / PingFang, Linux: Noto CJK)를 누르거나, **Browse...**로 .ttf / .otf / .ttc 파일을 고른 뒤
   **Register fallback**을 누릅니다. 폰트 파일이 `/Game/LocaFlow/Fonts`에 Font Face 에셋으로 들어가고,
   그 문화권의 문자(한글, 가나+한자, 중국어 한자) 범위를 맡는 서브 폰트로 추가됩니다.
4. 폰트 에셋을 저장하고(변경 표시가 됩니다) 다시 검사하면 목록이 비어 있어야 합니다.

> **라이선스 주의:** 시스템 폰트는 내 컴퓨터에서 쓰는 권리일 뿐, 게임에 넣어 배포할 권리가 아닐 수 있습니다.
> 출시 전에는 배포 가능한 폰트(예: SIL Open Font License인 Noto Sans CJK)를 등록하세요.

*Offline* 캐시 방식(비트맵) 폰트는 검사할 수 없습니다.

## 7. 번역가와 작업하기: CSV와 PO

- **Export CSV**: `Namespace,Key,Source,Translation,Comment` 열로 저장합니다(BOM 있는 UTF-8이라 엑셀에서 바로 열립니다).
- **Export PO**: GNU gettext 형식(`msgctxt` = `Namespace,Key`)으로 저장합니다. Poedit, Crowdin, Weblate 등에서 씁니다.
- **Import CSV / Import PO**: 네임스페이스와 키 기준으로 번역을 합칩니다. 가져온 항목은 사람이 고친 것으로 표시되어
  기계 번역이 덮어쓰지 않습니다. 가져온 뒤 **Save**를 누르세요.
  파일의 *Source*(원문)가 현재 원문과 다른 줄은 예전 원문으로 번역된 것이라 건너뛰고 개수를 알려 줍니다.
  원문 열이 없는 줄은 그대로 적용합니다.

## 8. 기계 번역 (선택)

### 설정

**편집 → 에디터 환경설정 → 플러그인 → LocaFlow**를 엽니다.

| 설정 | 설명 |
|---|---|
| Provider | *DeepL* 또는 *OpenAI-compatible* |
| DeepL API Key | DeepL 계정의 키. 무료 키는 `:fx`로 끝나며 무료 주소를 자동으로 씁니다. |
| OpenAI Base URL / API Key / Model | OpenAI 호환 Chat Completions 주소(OpenAI, 프록시, 로컬 서버 등). 기본값 `https://api.openai.com/v1`, 모델 `gpt-4o-mini`. |
| Price Per Million Chars | 비용 계산용 단가(DeepL Pro는 약 25달러). |
| Max Cost Per Job USD | 예상 비용이 이 값을 넘으면 작업이 시작되지 않습니다. 0이면 제한 없음. |
| Glossary CSV | 선택 사항. `원문,번역` 한 줄에 한 쌍, 머리글 줄 선택, `#`은 주석. |
| Batch Size / Max Requests Per Second | 한 번에 보내는 개수와 초당 요청 수 제한. |

이 설정은 사용자별로 `Saved/Config/.../EditorPerProjectUserSettings.ini`에 저장됩니다. 이 파일은 보통 버전 관리에
올리지 않으므로 API 키가 저장소에 들어가지 않습니다.

### 번역 실행

1. 기본 문화권이 아닌 문화권을 고릅니다.
2. **Translate Missing**을 누르면 보낼 항목 수·글자 수·예상 비용이 나옵니다.
3. 확인을 누르면 툴바에 진행률이 나옵니다. **Cancel**로 멈출 수 있습니다(이미 끝난 항목은 남습니다).
4. 결과를 확인하고 QA를 돌린 뒤 **Save**를 누릅니다.

보호되는 것:

- *Missing*·*Stale* 항목만 보냅니다. 직접 고치거나 가져오거나 승인한(*human*) 항목은 절대 덮어쓰지 않습니다.
- `{인자}`, plural / gender / hpp 블록, 리치 텍스트 태그는 보내기 전에 토큰으로 바꾸고 받은 뒤 되돌립니다.
  토큰이 빠지거나 바뀐 결과는 버리고 실패로 셉니다.
- 요청 실패(속도 제한, 서버 오류)는 간격을 늘려 가며 다시 시도합니다. 키가 틀리면 분명한 메시지와 함께 멈춥니다.

**개인정보:** 선택한 항목의 원문이 설정한 번역 서비스로 전송됩니다. *Translate Missing*을 누르고 확인하기 전에는
아무것도 보내지 않습니다.

## 9. 텍스트 넘침

**Overflow** 탭을 엽니다.

1. 검사할 화면마다 **Add Widget Blueprint**로 위젯을 추가합니다(메뉴, 버튼, HUD 등).
2. 검사할 문화권을 체크합니다.
3. **Run overflow check**를 누릅니다.

LocaFlow는 문화권마다 에디터의 *게임 언어 미리보기*를 그 문화권으로 바꾸고, 위젯을 만들어 배치한 뒤 각 Text Block /
Rich Text Block에 필요한 크기와 실제 크기를 비교합니다. 결과에는 문화권·위젯·텍스트와 위젯 썸네일
(`Saved/LocaFlow/OverflowThumbs`에 저장)이 나옵니다. 에디터 자체의 메뉴 언어는 바뀌지 않으며, 끝나면 이전 미리보기
언어로 되돌립니다.

조건: 번역이 반영되려면 그 문화권이 **컴파일**되어 있어야 합니다(Localization Dashboard → *Compile Text*).
또 일반 에디터 렌더러가 필요합니다(`-nullrhi`로 실행하면 건너뜁니다).

## 10. 런타임 블루프린트 노드 (선택)

카테고리 **LocaFlow**, 게임 어디서나 쓸 수 있습니다.

| 노드 | 설명 |
|---|---|
| Get Available Languages | 게임에 함께 패키징된 현지화 데이터가 있는 문화권 목록 |
| Get Language Native Name | `ko`면 "한국어", `ja`면 "日本語" 같은 이름(언어 메뉴용) |
| Get Current Language | 현재 문화권 코드 |
| Set Language | 언어를 바꿉니다. *Save*를 켜면 GameUserSettings.ini에 기억합니다. |

## 11. 팁과 알려진 제한

- LocaFlow에서 저장한 뒤에는 Localization Dashboard에서 **Compile Text**를 누르거나 게임을 패키징해야
  게임이 쓰는 `.locres` 파일이 만들어집니다.
- **Gather Text**를 다시 해도 번역은 모두 남습니다. LocaFlow의 *사람이 고침* 표시는 archive 옆의
  `<문화권>/<타깃>.locaflow.json`에 저장되므로(archive와 함께 버전 관리에 올리세요) 유지됩니다. LocaFlow 밖에서
  번역문이 바뀐 항목은 표시가 해제됩니다.
- 원문이 바뀌면 엔진 수집 과정에서 새 항목이 생겨 *Missing*으로 보이고, 예전 번역은 Localization Dashboard와
  마찬가지로 엔진이 지웁니다.
- 비트맵 폰트(*Offline* 캐시 방식)는 글리프 검사를 지원하지 않습니다.

## 12. 문제 해결

| 문제 | 해결 |
|---|---|
| 타깃 목록이 비어 있음 | Localization Dashboard에서 타깃을 한 번 수집한 뒤 **Reload**를 누릅니다. |
| "API key is missing" | *에디터 환경설정 → 플러그인 → LocaFlow*에 키를 넣습니다. |
| "Estimated cost exceeds the cap" | *Max Cost Per Job USD*를 올리거나 한 번에 적게 번역합니다. |
| 넘침 검사에 원문만 나옴 | 현지화를 컴파일(Localization Dashboard → *Compile Text*)하고 다시 실행합니다. |
| 글리프 검사: "uses the Offline cache type" | Runtime(컴포지트) 폰트 에셋을 쓰세요. |
| 도움이 필요할 때 | khj476611@naver.com 으로 문의해 주세요. 엔진 버전과 Output Log의 `LogLocaFlow`로 시작하는 줄을 함께 보내 주세요. |
