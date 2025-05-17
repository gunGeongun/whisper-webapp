# whisper-webapp
## Whisper 기반 음성 인식 웹 애플리케이션
### 1. 문제 정의

- **배경**: 사용자의 음성을 실시간으로 텍스트로 변환하는 서비스 수요 증가
- **문제점**: 일반적인 웹 환경에서 음성 인식 모델을 통합하고, 녹음-전송-처리-표시의 전 과정을 실시간으로 구현하기 어려움
- **해결 목표**: Whisper 모델과 FastAPI + React를 이용해 Colab에서 실행 가능한 실시간 음성 인식 앱 구축

---

### 2. 요구사항 분석

- 사용자는 웹에서 버튼 클릭으로 음성 녹음 시작 및 종료 가능해야 함
- 음성 데이터는 서버로 전송되고 Whisper로 처리되어 텍스트 결과 반환
- 결과는 프론트엔드에 자동으로 표시
- 외부에서 접근 가능한 ngrok 기반 HTTPS 주소 제공

---

### 3. 기술 스택 및 아키텍처 설명

| 구성 요소 | 사용 기술 |
| --- | --- |
| **음성 인식 모델** | OpenAI Whisper (base 모델) |
| **백엔드** | FastAPI, Uvicorn, Python |
| **프론트엔드** | React (CDN 방식), Material UI |
| **녹음 처리** | MediaRecorder API (WebM 형식) |
| **서버 노출** | ngrok |
| **오디오 전처리** | FFmpeg |
| **실행 환경** | Google Colab (Linux shell) |

**아키텍처 구조도**

```
브라우저 (React)
  ⬇ 음성 녹음, WebM Blob
Colab 서버 (FastAPI)
  ⬇ FormData 파일 업로드
Whisper 모델
  ⬇ 텍스트 결과
브라우저
```

---

### 4. 핵심 알고리즘 및 처리 메커니즘

- **녹음 처리 (브라우저)**
    - MediaRecorder API로 WebM 오디오 캡처
    - Blob → FormData 변환 → fetch로 `/transcribe` POST
- **FastAPI 처리 흐름**
    1. UploadFile로 웹에서 전송된 WebM 파일 수신
    2. `tempfile`을 사용해 임시 저장
    3. `whisper.transcribe(path)`로 음성 → 텍스트 변환
    4. 결과 텍스트 JSON 응답으로 반환
- **Whisper 알고리즘 요약**
    - 음성 신호 → Mel Spectrogram 변환
    - Transformer 기반 디코더로 텍스트 추출
    - base 모델은 소형이지만 성능/속도 균형이 우수

---

### 5. 구현 과정 및 주요 코드 설명

- `app.py`: FastAPI 서버 정의, `/transcribe` 엔드포인트 생성
- `www/index.html`: React 앱 포함, 녹음 UI 및 fetch 요청
- `run_server.py`: ngrok과 서버를 함께 실행하며 URL 자동 추출

예시 코드:

```python
@app.post("/transcribe")
async def transcribe(file: UploadFile = File(...)):
    with tempfile.NamedTemporaryFile(delete=False, suffix='.webm') as temp_file:
        temp_file.write(await file.read())
        path = temp_file.name
    result = model.transcribe(path)
    os.unlink(path)
    return JSONResponse(content={"text": result["text"]})
```

---

### 6. 발생한 문제점과 해결 방법

| 문제 | 해결 방법 |
| --- | --- |
| WebM 포맷 인식 오류 | Whisper가 `.webm` 인식 가능, 임시 파일로 저장하여 해결 |
| 마이크 오류 | ngrok HTTPS URL 사용, 브라우저 마이크 권한 허용 안내 |
| ngrok 인증 오류 | 토큰 발급 후 `config add-authtoken` 명령어로 등록 |
| 모델 로딩 지연 | base 모델 선택으로 초기 로딩 시간 단축 |

---

### 7. 습득한 기술 및 인사이트

- Whisper 구조와 음성 처리 방식에 대한 실전 이해
- FastAPI의 비동기 처리 흐름과 파일 처리 방식 습득
- React에서 MediaRecorder를 활용한 녹음 처리 기술
- ngrok을 통한 로컬 서버 외부 노출 자동화 경험

---

### 8. 결론 및 향후 개선 방향

- **결론**: Whisper 모델과 웹 기술을 연계하여 Colab 환경에서도 실시간 음성 인식 서비스를 구현할 수 있었음
- **향후 개선**:
    - Whisper 모델 크기 상향 (`small`, `medium`) 적용 및 성능 비교
    - 파일 다운로드 기능 추가 (텍스트 기록 저장)
    - 녹음 재생 및 다국어 지원 추가
    - STT 외에 TTS(Text-to-Speech) 기능 통합하여 양방향 음성 인터페이스 구현

---

### 9. 온라인 포트폴리오 링크

GitHub: https://github.com/gunGeongun/whisper-webapp

### 10. 실제 시연

![image](https://github.com/user-attachments/assets/b939ff15-1e1d-4465-89d9-d71a04f34bba)

