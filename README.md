# OpenAI Decisions API 참고자료

이미지를 입력하고, 정해진 선택지 중 하나를 고르는 AI 판단 흐름을 살펴보는 Jupyter Notebook 참고자료입니다.

사진의 외모상 연령대 추정과 Breakout 벽돌깨기 예제를 통해 API 호출, 결과 해석, 응답 시간·비용 측정 과정을 확인할 수 있습니다. OpenAI 공식 Python SDK를 직접 사용하며 LangChain은 사용하지 않습니다.

## 포함된 예제

| 노트북 | 살펴볼 내용 |
|---|---|
| [사진 연령대 추정](교안_OpenAI_Decisions_사진_연령대_추정.ipynb) | 사진 한 장의 핵심 호출, 샘플 6장 분류, 추정·참고 연령대 비교, 사진별·평균 응답 시간과 비용 |
| [Breakout 자동 플레이](교안_OpenAI_Decisions_Breakout_자동플레이.ipynb) | 연속 화면에서 공의 위치·방향 인식, 인식 결과를 활용한 행동 선택, 10초 분량 플레이와 리플레이 |

GitHub에서는 코드를 읽고 저장된 출력을 확인할 수 있습니다. 새로 실행하려면 로컬 환경과 본인의 OpenAI API 키가 필요합니다.

## 빠르게 시작하기

### 1. 저장소 받기

```bash
git clone https://github.com/paircodingofficial-cloud/openai-decisions-examples.git
cd openai-decisions-examples
```

Git을 사용하지 않는 경우 GitHub의 `Code → Download ZIP`으로 내려받고 압축을 풀어도 됩니다. 이후 명령은 모두 `pyproject.toml`과 노트북이 있는 저장소 최상위 폴더에서 실행합니다.

### 2. 환경 구성

[uv](https://docs.astral.sh/uv/getting-started/installation/)가 설치되어 있어야 합니다. 프로젝트는 Python 3.12를 사용합니다.

```bash
uv sync --locked
```

`pyproject.toml`과 `uv.lock`을 기준으로 `.venv`에 필요한 패키지를 설치합니다. 노트북 안에서 별도로 `pip install`을 실행할 필요는 없습니다.

### 3. API 키 설정

`.env.example`을 같은 폴더의 `.env`로 복사합니다. 기존 `.env`가 있다면 덮어쓰지 말고 내용을 확인하세요.

Windows PowerShell:

```powershell
Copy-Item .env.example .env
```

macOS / Linux:

```bash
cp .env.example .env
```

생성한 `.env`를 편집해 예시 문구를 본인의 API 키로 바꿉니다.

```dotenv
OPENAI_API_KEY=본인의_API_키
```

`.env`는 Git에서 제외됩니다. API 키를 노트북 코드나 출력에 직접 적지 말고, GitHub·디스코드·스크린샷에 노출하지 마세요.

### 4. 노트북 열기

```bash
uv run jupyter lab
```

원하는 노트북을 열어 위에서 아래로 실행합니다. VS Code에서 여는 경우 저장소의 `.venv` Python을 노트북 커널로 선택하세요.

API 호출 셀을 실행하거나 전체 셀을 다시 실행하면 비용이 발생합니다. 코드를 읽기만 할 때는 실행하지 않아도 됩니다.

## 예제의 핵심 흐름

### 사진 연령대 추정

사진 한 장을 보내는 핵심 호출을 먼저 확인한 뒤, 같은 처리를 함수로 묶어 샘플 6장을 분류합니다.

- 어린이부터 10대·20대·30대·40대·50대까지의 참고 사진이 `data/age_faces/`에 포함되어 있습니다.
- 선택지에는 `10세 미만`, `10대`~`50대`, `60대 이상`, `판단 어려움`을 둡니다.
- 각 사진과 함께 추정 연령대·참고 연령대·응답 시간·달러/원화 비용을 표시합니다.
- 사진별 결과와 상세 응답은 실행 후 `output/age_faces/`에 저장됩니다.

외모상 연령대 추정은 실제 나이 확인이 아닙니다. 성인 인증이나 이용 자격 판단에 사용하지 마세요. 공개 인물 사진에 대한 모델의 기존 지식이 영향을 줄 수 있으므로 이 샘플을 정확도 벤치마크로 해석하지 않습니다.

### Breakout 자동 플레이

공이 어디로 움직이는지 살펴보기 위해 이전 화면과 현재 화면을 함께 보냅니다. 현재 예제는 인식과 행동을 순차적으로 판단합니다.

```text
최근 게임 화면 2장
    → 첫 API 호출: 공의 상대 위치·가로 방향·세로 방향 인식
    → 두 번째 API 호출: 인식 결과를 받아 left / right / stay 선택
    → 게임에 행동 적용 → 다음 화면으로 반복
```

- 한 번 선택한 행동은 3프레임 동안 적용합니다.
- 공 발사와 목숨을 잃은 뒤의 재발사만 코드에서 자동 처리합니다.
- 패들 이동은 AI가 결정하며, 숨겨진 공 좌표나 자동 조준 로직은 사용하지 않습니다.
- 모델을 새로 훈련하는 강화학습 예제가 아니라, 이미지 기반 판단으로 게임을 제어하는 예제입니다.

기본 게임 시간은 60fps 기준 600프레임, 약 10초입니다. API 응답을 기다리는 동안 게임은 멈춰 있으므로 실제 실행 시간은 더 길 수 있습니다. 게임 종료나 행동 판단 거절 시에는 일찍 멈춥니다.

실행 후 `output/breakout/<실행 시각>/`에 `play.gif`, `replay.html`, `actions.csv`, `summary.json`을 저장합니다. 리플레이는 노트북에서 확인하거나 HTML 파일을 브라우저로 열어 볼 수 있습니다. `output/`은 저장소에 포함하지 않으므로 새 실행으로 생성합니다.

## 시간과 비용을 읽는 방법

이 저장소는 `client.decisions.create()`와 `gpt-6-luna`를 사용합니다. 공통적으로 `input`에 판단할 내용을, `questions`에 선택지와 기준을 전달하고 `answers`에서 결과를 읽습니다.

2026-10-08 확인 기준 Decisions API의 기본 요금은 입력 100만 토큰당 0.10달러이며 출력 토큰 요금은 없습니다. 다른 API의 같은 모델 요금과 구분해야 하며, 실행 전 [공식 Decisions 안내](https://developers.openai.com/api/docs/guides/decisions#pricing-and-availability)에서 최신 요금을 확인하세요.

```text
예상 달러 비용 = 응답의 입력 토큰 수 ÷ 1,000,000 × 0.10
예상 원화 비용 = 예상 달러 비용 × 1,500
```

- 원화 환산은 1달러=1,500원 가정이며 실시간 환율이 아닙니다. 세금·지역 처리 할증 등은 제외합니다.
- API 시간은 요청부터 응답까지의 시간이며, 이미지 준비 시간은 제외합니다. Breakout의 판단 시간은 두 요청 시간의 합입니다.
- 사진 예제는 핵심 호출 1회와 일괄 분류 6회를 실행합니다. 결과표의 비용 합계는 일괄 분류 6장만 포함합니다.
- Breakout은 핵심 예제 2회와 자동 플레이 요청을 별도로 실행합니다. 기본 자동 플레이는 최대 약 400회 요청하며, 비용 합계에는 핵심 예제를 포함하지 않습니다.
- 선택 확률과 `confidence`는 실제 정답률이나 성공 보장이 아닙니다. 모델은 작은 공의 위치·방향을 잘못 읽을 수 있습니다.

## 데이터와 보안

사진과 게임 화면은 API 판단을 위해 OpenAI로 전송됩니다. 개인 사진으로 바꾸는 경우 본인 사진 또는 사용 동의를 받은 사진을 사용하세요. 실행 결과를 공유할 때도 노트북 출력에 개인정보가 없는지 확인하세요.

샘플 사진의 저작자·촬영일·참고 나이·라이선스는 [사진 출처 문서](data/age_faces/출처.md)에 정리되어 있습니다. 사진별 라이선스를 준수해야 하며, 저장소 공개 자체가 별도의 이용 허락을 뜻하지는 않습니다. 게임은 설치된 ALE 패키지의 자산을 사용하며 ROM 파일을 이 저장소에 별도로 배포하지 않습니다.

실제 키가 들어 있는 `.env`, 가상환경 `.venv/`, 실행 산출물 `output/`은 Git에서 제외합니다. `.env.example`은 키 입력 형식만 안내하는 파일입니다.

## 실행이 안 될 때

- 패키지를 찾지 못한다면 `uv sync --locked`를 실행하고 노트북 커널이 이 저장소의 `.venv`인지 확인하세요.
- `.env`나 사진을 찾지 못한다면 저장소 최상위 폴더에서 Jupyter를 시작했는지 확인하세요.
- 인증·권한·사용량 관련 API 오류가 나면 키와 해당 OpenAI 프로젝트의 접근 권한·결제/사용량 설정을 확인하세요. 키를 공유해 해결하려 하지 마세요.
- 노트북에서 설명이 편집 상태로 보이면 해당 마크다운 셀을 실행해 렌더링합니다. API 호출이 있는 코드 셀과 구분하세요.

## 공식 참고 문서

- [OpenAI Decisions API](https://developers.openai.com/api/docs/guides/decisions)
- [Decisions Python API Reference](https://developers.openai.com/api/reference/python/resources/decisions/methods/create)
- [uv와 Jupyter 사용하기](https://docs.astral.sh/uv/guides/integration/jupyter/)
- [ALE Breakout 환경](https://ale.farama.org/environments/breakout/)
