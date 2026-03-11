> 📂 **관광 식당 메뉴판 번역 프로젝트** — [📱 App](https://github.com/Chun-Bae/restaurant-menu-translation-app) · [🖥️ Web](https://github.com/Chun-Bae/restaurant-menu-translation-web) · **🔧 API** · [🔍 OCR Model](https://github.com/Chun-Bae/restaurant-menu-translation-ocr-model) · [🌐 Translate Model](https://github.com/Chun-Bae/restaurant-menu-translation-translate-model)

# 🔧 API Server — 파이프라인 오케스트레이터

> **OCR · 번역 컨테이너를 비동기로 제어하는 FastAPI 미들웨어 서버**

| 기술 스택 | 역할 |
|-----------|------|
| **FastAPI** | 비동기 REST API · WebSocket |
| **asyncio / subprocess** | Docker CLI 논블로킹 호출 |
| **Docker** | OCR · Translate 모델 컨테이너 오케스트레이션 |
| **Surya (SegFormer)** | 텍스트 영역 검출 · Bounding Box 크롭 |
| **pandas** | 엑셀 결과 파싱 · 데이터 매핑 |

---

## 📁 프로젝트 구조

```
restaurant-menu-translation-api/
├── main.py              # FastAPI 엔드포인트, 10단계 파이프라인, WebSocket 진행률
├── docker_utils.py      # docker cp / docker exec 비동기 래핑 유틸리티
├── ocr_crop.py          # Surya 모델 기반 텍스트 검출 및 PIL 크롭
├── requirements.txt     # 의존성 목록
└── analysis.md          # 전체 프로젝트 기술 분석 문서 (포트폴리오)
```

## ⚙️ 핵심 파이프라인 흐름

```
① POST /upload (이미지 수신)
  ↓
② Surya 모델로 텍스트 영역 검출 · Bounding Box 크롭
  ↓
③ 크롭 이미지를 ZIP 압축
  ↓
④ docker cp → OCR 컨테이너에 전송
  ↓
⑤ docker exec → OCR 추론 실행 (TPS-VGG-BiLSTM-Attn)
  ↓
⑥ 결과 엑셀(.xlsx) → 호스트로 회수
  ↓
⑦ docker cp → 번역 컨테이너에 전달
  ↓
⑧ docker exec → OpenNMT 번역 실행 (ko→en)
  ↓
⑨ 최종 결과 수합 → Base64 인코딩
  ↓
⑩ POST → Next.js /api/receive로 JSON 전송
```

## 🔌 WebSocket 진행률

각 단계마다 `/ws/progress`를 통해 클라이언트에 실시간 퍼센티지를 푸시합니다.

```
10% → 30% → 50% → 70% → 90% → 100%
(크롭)  (OCR전송) (OCR완료) (번역전송) (번역완료) (결과전송)
```

## 📄 관련 문서

- **[analysis.md](./analysis.md)** — 전체 프로젝트 기술 분석 (도입 배경 · 문제 해결 · 성과)
