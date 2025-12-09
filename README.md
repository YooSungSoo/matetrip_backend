<h1 align="center">MateTrip Backend</h1>

<p align="center">
  AI 기반 여행 메이트 매칭 & 실시간 협업 플래너 백엔드<br/>
  <b>NestJS · TypeORM · PostgreSQL · Redis · WebSocket · RabbitMQ · AWS Bedrock</b>
</p>

<p align="center">
  <img width="800" alt="MateTrip Backend Poster" src="https://github.com/user-attachments/assets/610ca8b3-90d6-44f9-9329-38eb1431241b" />
</p>

---

## ✨ Introduction

**MateTrip Backend**는

- AI가 여행 일정을 함께 설계해 주고
- 여러 명이 실시간으로 같은 보드를 보면서 일정을 편집하고
- 프로필 기반으로 잘 맞는 여행 메이트를 추천해주는

소셜 여행 서비스 **MateTrip**의 백엔드 서버입니다.

`NestJS` + `PostgreSQL(PostGIS, pgvector)` + `Redis` + `RabbitMQ` + `AWS Bedrock`을 기반으로  
**AI 에이전트, 실시간 협업, 메이트 매칭**을 하나의 플랫폼으로 묶는 데 집중했습니다.

---

## 🗂 ERD Overview   
<a href="https://www.erdcloud.com/d/zisMDhbp4vftyMgZg" target="_blank" rel="noopener noreferrer">
  ERD 링크
</a>
<img width="1359" height="735" alt="cropped_erd" src="https://github.com/user-attachments/assets/631312a3-4df8-461a-90ac-65701ec795ac" />

---


## 📌 핵심 기능

### 🤖 AI 기반 여행 컨설팅
<img width="980" height="645" alt="image" src="https://github.com/user-attachments/assets/67ba737c-3efc-4d6a-88fd-4ce25b385f73" />

- FastAPI + LangGraph로 구현한 **MateTrip AI 서버**와 연동

```TEXT
사용자: "@AI 제주도 해녀촌 근처 맛집 추천해줘"
  ↓
Chat Gateway → AI Server /chat/v2
  ↓
LangGraph Agent (Claude 4.5 Haiku)
  ↓
recommend_nearby_places 도구 실행
  ↓
AI 응답 + 추천 장소 tool_data
  ↓
Socket.IO 브로드캐스트 → 모든 참여자 지도에 표시
```

- 채팅 한 줄로:
  - 일정 초안 생성
  - 주변 장소 추천
  - 일정 수정
  - 등 다양한 기능 탑재 (자세한 내용 : [matetrip-ai README.md](https://github.com/YooSungSoo/matetrip_ai))
- **사용자 행동 추적 및 개인화**
  - RabbitMQ를 통한 비동기 이벤트 처리
  - POI 북마크, 일정 추가 등 행동 데이터 수집
  - AI 서버로 전송하여 개인화 추천 학습

---

### 🤝 실시간 협업 워크스페이스 (Socket.IO)

<img width="975" height="553" alt="image" src="https://github.com/user-attachments/assets/8d073ffd-7033-453a-b85e-ae210765a745" />

> `Socket.IO` + `Redis Adapter`로 **다중 서버 환경에서도 실시간 동기화**


- **Socket.IO 기반 실시간 동기화**
  - Socket.IO + Redis Adapter를 통한 다중 서버 환경 지원
  - 여러 사용자가 동시에 여행 계획 수립 및 편집
  - 실시간 채팅 및 POI(관심 지점) 관리
  - 모든 참여자에게 변경사항 즉시 브로드캐스트

- **실시간 채팅**
  - 워크스페이스별 채팅 룸
  - AI 멘션 자동 감지 및 처리
  - 채팅 히스토리 관리
  - Redis Pub/Sub을 통한 다중 서버 동기화

- **POI 실시간 관리**
  - `MARKED`: 보관함 추가
  - `SCHEDULED`: 일정에 추가됨
  - POI 추가/삭제/상태 변경 즉시 브로드캐스트
  - 드래그 앤 드롭 순서 변경 동기화

- **일별 일정 관리**
  - N박 M일 형식으로 자동 일정 생성
  - 일자별 POI 목록 관리
  - 숙박, 식사, 관광지 등 카테고리별 분류
  - 모든 참여자에게 변경사항 실시간 반영
  
---

### 🎯 지능형 여행 메이트 매칭
<img width="1158" height="902" alt="image" src="https://github.com/user-attachments/assets/5a83c355-9a9b-43be-b987-c045c1966b11" />


- **모집글 기반 여행 메이트 찾기**
  - 여행 목적지, 일정, 선호 성별/나이대 필터링
  - 참여 신청 및 승인/거절 관리
  - 승인 시 자동으로 워크스페이스 생성

- **임베딩 기반 하이브리드 프로필 임베딩**
  - 프로필 기반 벡터 임베딩 (`AWS Titan Embeddings v2` + `pgvector`)
  - 여행 스타일(16가지), 여행 성향(80+), MBTI 등을 함께 고려하는 **하이브리드 점수**
  ```text
  매칭 점수 = (벡터 유사도 × 0.7)
            + (여행 스타일 겹침도 × 0.15)
            + (여행 성향 겹침도 × 0.1)
            + (MBTI 궁합 × 0.05)
  ```

## 📂 프로젝트 구조

```
matetrip-back/
├── src/
│   ├── domain/                    # 도메인별 모듈 (DDD)
│   │   │
│   │   ├── profile/               # 👤 프로필 & 매칭
│   │   │   ├── matching.service.ts       #  매칭 알고리즘 핵심
│   │   │   ├── profile.service.ts
│   │   │   ├── profile.controller.ts
│   │   │   ├── entities/
│   │   │   │   ├── profile.entity.ts
│   │   │   │   ├── travel-style-type.enum.ts   # 여행 스타일 (16가지)
│   │   │   │   ├── tendency-type.enum.ts       # 여행 성향 (80+ 가지)
│   │   │   │   └── mbti.enum.ts
│   │   │   ├── dto/
│   │   │   │   ├── match-request.dto.ts
│   │   │   │   ├── match-response.dto.ts
│   │   │   │   └── embedding-matching-profile.dto.ts
│   │   │   └── utils/
│   │   │       └── embedding-payload.util.ts
│   │   │
│   │   ├── post/                  # 📝 모집글
│   │   │   ├── post.service.ts
│   │   │   ├── post.controller.ts
│   │   │   ├── entities/
│   │   │   │   ├── post.entity.ts
│   │   │   │   └── post-status.enum.ts  # 모집중 | 완료
│   │   │   └── dto/
│   │   │       ├── create-post.dto.ts
│   │   │       ├── search-post.dto.ts
│   │   │       └── post-response.dto.ts
│   │   │
│   │   ├── post-participation/    # 🤝 참여 관리
│   │   │   ├── post-participation.service.ts
│   │   │   ├── post-participation.controller.ts
│   │   │   ├── entities/
│   │   │   │   ├── post-participation.entity.ts
│   │   │   │   └── post-participation-status.enum.ts  # 대기중 | 승인 | 거절
│   │   │   └── dto/
│   │   │
│   │   ├── workspace/             # 🏢 워크스페이스 (핵심)
│   │   │   ├── workspace.controller.ts
│   │   │   ├── workspace.service.ts
│   │   │   ├── gateway/           # ⭐ WebSocket 게이트웨이 
│   │   │   │   ├── chat.gateway.ts       # 💬 채팅 & AI 연동
│   │   │   │   └── poi.gateway.ts        # 📍 POI 실시간 동기화
│   │   │   ├── service/           # 비즈니스 로직
│   │   │   │   ├── workspace-ai.service.ts        # AI 서버 호출
│   │   │   │   ├── workspace-poi.service.ts       # POI 관리
│   │   │   │   ├── workspace-planday.service.ts   # 일정 관리
│   │   │   │   └── workspace-chime.service.ts     # Chime 화상 회의
│   │   │   ├── entities/
│   │   │   │   ├── workspace.entity.ts
│   │   │   │   ├── plan-day.entity.ts
│   │   │   │   ├── poi.entity.ts
│   │   │   │   └── chat-message.entity.ts
│   │   │   └── dto/
│   │   │       ├── chat/
│   │   │       ├── poi/
│   │   │       ├── planday/
│   │   │       └── chime/
│   │   │
│   │   ├── place/                 # 🗺️ 장소
│   │   │   ├── place.service.ts
│   │   │   ├── place.controller.ts
│   │   │   ├── entities/
│   │   │   │   └── place.entity.ts
│   │   │   └── dto/
│   │   │       ├── get-behavior-based-recommendation-req.dto.ts
│   │   │       └── recommendation-reason.dto.ts
│   │   │
│   │   ├── user_behavior/         # 👁️ 사용자 행동 추적
│   │   │   ├── user_behavior.service.ts
│   │   │   ├── user_behavior.controller.ts
│   │   │   ├── entities/
│   │   │   │   ├── user-behavior-event.entity.ts
│   │   │   │   └── user-behavior-embedding.entity.ts
│   │   │   └── dto/
│   │   │
│   │   ├── place_user_review/     # 💬 장소 리뷰
│   │   │   ├── place_user_review.service.ts
│   │   │   ├── place_user_review.controller.ts
│   │   │   ├── entities/
│   │   │   └── dto/
│   │   │
│   │   ├── auth/                  # 🔐 인증
│   │   │   ├── auth.service.ts
│   │   │   ├── auth.controller.ts
│   │   │   ├── strategies/
│   │   │   │   ├── jwt.strategy.ts
│   │   │   │   └── local.strategy.ts
│   │   │   └── dto/
│   │   │
│   │   ├── users/                 # 👥 사용자
│   │   │   ├── users.service.ts
│   │   │   ├── users.controller.ts
│   │   │   ├── entities/
│   │   │   │   └── user.entity.ts
│   │   │   └── dto/
│   │   │
│   │   ├── binary-content/        # 📸 이미지 (S3)
│   │   │   ├── binary-content.service.ts
│   │   │   ├── binary-content.controller.ts
│   │   │   ├── entities/
│   │   │   └── dto/
│   │   │
│   │   ├── notifications/         # 🔔 알림
│   │   │   ├── notifications.service.ts
│   │   │   ├── notifications.controller.ts
│   │   │   ├── entities/
│   │   │   └── dto/
│   │   │
│   │   ├── review/                # 🍟워크스페이스 리뷰
│   │   │   ├── review.service.ts
│   │   │   ├── review.controller.ts
│   │   │   ├── entities/
│   │   │   └── dto/
│   │   │
│   │   ├── behavior/              # 📊 행동 처리 (RabbitMQ 발행)
│   │   │   └── behavior.service.ts
│   │   │
│   │   └── proxy/                 # 🔄 프록시 (외부 API)
│   │       └── proxy.controller.ts
│   │
│   ├── infra/                     # 인프라 레이어
│   │   ├── redis/                 # Redis 연동
│   │   │   ├── redis.service.ts
│   │   │   ├── redis.module.ts
│   │   │   └── redis-io.adapter.ts    # Socket.IO Redis Adapter
│   │   │
│   │   └── rabbitmq/              # RabbitMQ 연동
│   │       ├── rabbitmq.module.ts
│   │       ├── rabbitmq.service.ts
│   │       └── rabbitmq.producer.ts
│   │
│   ├── ai/                        # 프로필 관련 AI 서버 연동
│   │   ├── ai.service.ts          
│   │   ├── summaryLLM.service.ts  
│   │   ├── titan-embedding.service.ts  
│   │   └── dto/
│   │
│   ├── aws/                       # AWS 서비스
│   │   ├── s3.service.ts          # S3 업로드
│   │   └── chime.service.ts       # Chime SDK
│   │
│   ├── common/                    # 공통 컴포넌트
│   │   ├── enum/                  # 열거형 타입
│   │   └── transformers/          # 타입 변환기
│   │       └── geography.transformer.ts  # PostGIS Geography
│   │
│   ├── config/                    # 설정
│   │   ├── database.config.ts
│   │   └── redis.config.ts
│   │
│   ├── scripts/                   # SEED 관련 유틸리티 스크립트
│   │
│   ├── base.entity.ts             # 공통 엔티티 베이스
│   ├── app.module.ts              # 루트 모듈
│   └── main.ts                    # 애플리케이션 진입점
│
├── test/                          # 테스트
├── .github/workflows/             # CI/CD
│   └── deploy-ecs.yml
├── schema.sql                     # DB 스키마
├── Dockerfile
├── docker-compose.yml
└── package.json
```

---



## 🧱 기술 스택

- **Language & Framework** : NestJS, TypeScript, Node.js
- **Database & ORM** : PostgreSQL, PostGIS (공간 쿼리), pgvector (벡터 검색), TypeORM
- **Real-time** : Socket.IO, redis-adapter, ioredis
- **Message Queue** : RabbitMQ (AWS MQ)
- **Auth** : JWT, bcrypt
- **AWS** : S3 (이미지 저장), Chime SDK (화상 회의), Bedrock

## 📡 핵심 API (일부)

### 🤖 AI 기반 여행 컨설팅
**WebSocket Gateway:** `chat.gateway.ts:54`
- `message` - 채팅으로 "@AI 멘션" 시 AI 에이전트 호출


**핵심 서비스**
- `workspace-ai.service.ts` - AI 서버 연동
- `poi.service.ts` - POI 생성/일정 추가


**REST API** `workspace.controller.ts`
- AI 에이전트 연동용 내부 API
- AI 서버(에이전트)가 채팅 도중 일정 생성/수정 작업이 필요할 때 워크스페이스 백엔드와 통신하기 위해 호출하는 **서버-투-서버 전용 API*
- `POST /workspace/:id/ai/schedule-batch` - AI가 생성한 여행 코스 초안을 **해당 워크스페이스 일정에 일괄 반영**하는 엔드포인트 
- `POST /workspace/:id/ai/replace-schedule` - 기존 일정 중 일부 장소를, AI가 제안한 **대체 장소로 교체**하는 엔드포인트  

---

### 🤝 실시간 협업 워크스페이스
**WebSocket Gateway:** `chat.gateway.ts`, `poi.gateway.ts`
- `sync` - 전체 POI/일정 실시간 동기화
- `mark/addSchedule/removeSchedule` - POI 상태 변경 브로드캐스트
- `reorder` - 드래그앤드롭 순서 변경
- `cursorMove/poi:hover` - 협업 커서 공유

**REST API:** `workspace.controller.ts`
- `GET /workspace/:id/scheduled-pois` - 일정 POI 조회
- `POST /workspace/:id/reviews` - 여행 후기 작성

**핵심 서비스:**
- `poi-cache.service.ts` - Redis 기반 POI 캐싱
- `plan-day.service.ts` - 일별 일정 관리
- `chat-message.service.ts` - 채팅 저장/조회

---

### 🎯 지능형 메이트 매칭
**REST API:** `profile.controller.ts`, `post.controller.ts`
- `POST /profile/matching/search` - 벡터 유사도 기반 메이트 추천
- `GET /profile/matching/detailsearch` - 모집글 + 매칭 점수 통합 검색
- `PATCH /profile/my` - 프로필 수정 시 임베딩 자동 갱신
- `PATCH /post-participation/:id/accept` - 참여 승인 → 워크스페이스 자동 생성

**핵심 서비스:**
- `matching.service.ts` - 하이브리드 매칭 알고리즘
  - 벡터 유사도 70% + 여행 스타일 15% + 성향 10% + MBTI 5%
- `profile.service.ts` - AWS Titan Embeddings 기반 프로필 임베딩

---

## 🔗 Related
1. [MateTrip AI Server](https://github.com/YooSungSoo/matetrip_ai)
  - AI 에이전트 & 워커 서버

2. [MateTrip Front](https://github.com/YooSungSoo/matetrip_frontend)
