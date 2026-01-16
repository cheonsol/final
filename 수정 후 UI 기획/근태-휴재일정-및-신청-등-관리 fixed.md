# 근태 휴재일정 및 신청 등 관리

**작성일**: 2026-01-13  
**프로젝트**: Serialization Break Management System (JavaScript)  
**현재 상태**: 백엔드 연동 준비 완료, UI 구현 필요

---

## 📋 프로젝트 개요

웹소설/웹툰 작가 에이전시 HR SaaS의 **휴재 관리 모듈**이다.

### 🔄 개념 전환
**연차 중심 → 휴재 중심**

| 기존 (일반 회사) | 개선 (작가 에이전시) |
|---|---|
| 연차(휴가) 중심 근태 관리 | 마감 중심 휴재 관리 |
| 잔여 연차 일수 관리 | 마감 일정 기반 관리 |
| 사전 승인 필수 | 자동 승인 가능 (설정 시) |
| 일반 사무직 관점 | 작가/프리랜서 관점 |
| 획일적 휴가 정책 | 유연한 휴재 정책 |

### 핵심 기능
1. **휴재 신청** - 정기/건강/긴급/시즌/일상 휴재 신청
2. **마감 캘린더** - 작가별 연재 일정 및 휴재 계획 확인
3. **재택 근무** - 출퇴근 기록, 업무 보고, 11시간 자동 퇴근
4. **휴재 허용** - 마감일에 지장 없는 휴식 자동 허용

---

## 📝 휴재 유형 세분화

### 5가지 휴재 유형

| 유형 | 설명 | 사전 승인 | 마감 영향 | 예시 |
|------|------|-----------|-----------|------|
| **정기 휴재** | 미리 계획된 휴재 | 필요 | 미리 조정 | 명절, 여름휴가 |
| **건강 휴재** | 건강 문제로 인한 휴재 | 필요 | 조정 필요 | 손목 통증, 눈 피로 |
| **긴급 휴재** | 갑작스러운 사정 | 사후 보고 | 즉시 조정 | 가족 경조사, 응급 상황 |
| **시즌 휴재** | 시즌 종료 후 장기 휴재 | 필요 | 영향 없음 | 시즌 1 종료 후 2주 휴재 |
| **일상 휴식** | 짧은 휴식 (1-2일) | 자동 승인 | 지장 없음 | 너무 힘든 날, 번아웃 방지 |

### 마감 내역 지정

**휴재 신청 시 필수 정보**
```javascript
{
  break_type: 'health',        // 휴재 유형
  reason: '손목 통증 치료',
  start_date: '2026-01-20',
  end_date: '2026-01-22',
  
  // 🔴 핵심: 마감 정보
  affected_deadlines: [
    {
      serial_id: 'novel_123',
      serial_title: '환생 검신',
      original_deadline: '2026-01-21',
      adjusted_deadline: '2026-01-24',  // 3일 연장
      adjustment_reason: '건강 휴재로 인한 마감 연장'
    }
  ],
  
  // 자동 승인 설정 (관리자가 미리 설정)
  auto_approve: false  // 건강 휴재는 승인 필요
}
```

---

## 🎯 개발 방향

### 현재까지 작업
- API 연동 함수 17개 작성 (근태 7개, 재택근무 6개, 날짜 유틸 4개)
- 날짜 처리 함수 준비
- 개발 서버 실행 중 (http://localhost:3000)

### 다음 작업 흐름
```
[지금] 백엔드 연동 준비됨
  ↓
1단계: 휴재 신청 폼 만들기 (1-2주)
  ├─ 5가지 휴재 유형 선택 UI (카드 형태)
  ├─ 날짜 선택기 (MUI DatePicker)
  ├─ 마감 일정 선택 및 조정
  ├─ 사유 입력
  └─ 검증 로직 (마감 충돌 감지, 중복 방지)
  ↓
2단계: 마감 캘린더 화면 (1주)
  ├─ FullCalendar 연동
  ├─ 연재 마감일 표시 (빨간색)
  ├─ 휴재 계획 표시 (파란색)
  └─ 충돌 감지 및 알림
  ↓
3단계: 재택근무 + 야근 관리 (1주)
  ├─ 출퇴근 시간 기록
  ├─ 11시간 자동 퇴근 시스템 ⭐
  ├─ 업무 보고 작성
  └─ 근무 시간 집계
  ↓
4단계: 알림 시스템 (1주)
  ├─ 마감 임박 알림 (D-3, D-1)
  ├─ 자동 퇴근 알림 (11시간 경과 시)
  ├─ 휴재 승인/반려 알림
  └─ Slack 연동
  ↓
5단계: 관리자 기능 (2주)
  ├─ 작가별 휴재 현황
  ├─ 마감 준수율 통계
  └─ 자동 승인 규칙 설정
```

---

## 💡 이렇게 구현하면 좋겠다

### 1. 휴재 신청 흐름

**작가 관점의 사용자 경험**
1. 화면 진입 → **다음 마감일** 크게 표시 (D-3 같은 형식)
2. **카드 형태로 휴재 유형 선택** (클릭 한 번)
3. 날짜 선택 시 **마감일 자동 감지 및 알림**
4. 실시간 검증 (마감 충돌, 중복, 조정 필요 여부)
5. 마감 조정 계획 입력 (필요시)
6. 미리보기 후 제출

#### 카드 형태 - 휴재 유형

**5가지 휴재 유형을 독립된 박스(카드)로 표시**

```
┌─────────────────────────────────────────────┐
│  다음 마감: 환생 검신 123화 (D-5)            │
│  진행 중인 연재: 3개                         │
├─────────────────────────────────────────────┤
│                                             │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐   │
│  │   📅     │ │   ❤️     │ │   ⚡     │   │
│  │ 정기휴재 │ │ 건강휴재 │ │ 긴급휴재 │   │
│  │ 사전계획 │ │ 손목통증 │ │ 즉시처리 │   │
│  │ 승인필요 │ │ 승인필요 │ │ 사후보고 │   │
│  │ [선택]   │ │ [선택]   │ │ [선택]   │   │
│  └──────────┘ └──────────┘ └──────────┘   │
│                                             │
│  ┌──────────┐ ┌──────────┐                │
│  │   🏖️     │ │   ☕     │                │
│  │ 시즌휴재 │ │ 일상휴식 │                │
│  │ 장기휴식 │ │ 1-2일    │                │
│  │ 승인필요 │ │ 자동승인 │                │
│  │ [선택]   │ │ [선택]   │                │
│  └──────────┘ └──────────┘                │
│                                             │
└─────────────────────────────────────────────┘
```

**카드 특징:**
- 테두리가 있는 박스 형태
- 아이콘 + 제목 + 설명 + 승인 방식
- 마우스 올리면(hover) 살짝 확대 또는 색상 변경
- 클릭하면 선택 효과 (파란 테두리)
- **"자동승인" 배지** 표시 (일상 휴식)
- 모바일에서 터치하기 쉬운 크기

**Material-UI 코드 예시:**
```javascript
import { Card, CardContent, Typography, Chip } from '@mui/material';

<Card 
  onClick={() => selectBreakType('casual')}
  sx={{ 
    cursor: 'pointer',
    position: 'relative',
    '&:hover': { 
      transform: 'scale(1.05)',
      boxShadow: 3 
    }
  }}
>
  <CardContent>
    <Typography variant="h4">☕</Typography>
    <Typography variant="h6">일상 휴식</Typography>
    <Typography variant="body2">1-2일</Typography>
    <Chip 
      label="자동승인" 
      color="success" 
      size="small"
      sx={{ mt: 1 }}
    />
  </CardContent>
</Card>
```

**이렇게 하면:**
- 신청이 3분 안에 끝남
- 마감일 충돌을 사전에 방지
- 자동 승인으로 빠른 휴식 가능

```javascript
// 예시: 마감일 충돌 검증
const affectedDeadlines = await getDeadlines({
  author_id: userId,
  start_date: startDate,
  end_date: endDate
});

if (affectedDeadlines.length > 0 && breakType !== 'casual') {
  showWarning(
    `마감일이 ${affectedDeadlines.length}개 있습니다.\n` +
    `마감 조정 계획을 입력해주세요.`
  );
}

// 일상 휴식 (1-2일) → 자동 승인
if (breakType === 'casual' && days <= 2 && affectedDeadlines.length === 0) {
  showSuccess('자동 승인되었습니다! 편히 쉬세요 😊');
}
```

---

### 2. 마감 캘린더는 직관적으로

**작가 관점의 캘린더**
- 색상으로 구분:
  - **빨간색**: 마감일 (진하기로 임박도 표시)
  - **파란색**: 휴재 계획
  - **초록색**: 여유 기간
  - **주황색**: 마감 충돌 경고
- 작가 이름 + 연재물 제목 표시
- 클릭하면 상세 팝업 (마감일, 조정 내역, 휴재 사유)
- **마감일 충돌 감지**: 휴재 날짜와 마감일이 겹치면 경고

**추가하면 좋을 기능:**
- 필터링 (내 연재물만, 팀 전체, 특정 작가)
- 월간/주간/일간 전환
- 마감 D-day 표시
- 일정 공유 URL
- 마감 준수율 표시 (작가별)

```javascript
// 마감 충돌 감지 예시
const deadlines = await getDeadlines({ 
  author_id: authorId,
  start_date: startDate,
  end_date: endDate
});

if (deadlines.length > 0) {
  const conflictMsg = deadlines.map(d => 
    `- ${d.title} ${d.episode}화 (${d.deadline})`
  ).join('\n');
  
  alert(
    `⚠️ 마감일 충돌이 감지되었습니다:\n${conflictMsg}\n\n` +
    '마감 조정 계획을 입력해주세요.'
  );
}

// 여유 기간 계산
const daysBeforeDeadline = getDaysDiff(new Date(), deadline);
if (daysBeforeDeadline < 3) {
  showWarning('⏰ 마감 3일 전입니다. 휴재 신중히 고려하세요.');
}
```

---

### 3. 재택근무는 간편하게 + 과로 방지

**출퇴근 버튼으로 시간 기록**
1. 출근 버튼 클릭 → 시간 자동 기록
2. 퇴근 버튼 클릭 → 근무 시간 자동 계산
3. 8시간 미만이면 알림
4. ⭐ **11시간 자동 퇴근 시스템** (과로 방지)

#### 🔴 11시간 자동 퇴근 시스템

**작동 방식**
```
출근 시각: 09:00
  ↓
11시간 경과 (20:00)
  ↓
야근 신청 안 했나? → 확인
  ↓
[YES] 자동 퇴근 처리 (20:00)
  └─ 화면 알림: "11시간 경과로 자동 퇴근 처리되었습니다"
  └─ 실제 퇴근 시간 수정 가능
  
[NO] 정상 근무 (야근 신청함)
  └─ 퇴근 버튼 활성 상태 유지
```

**화면 표시**
```
┌─────────────────────────────────────┐
│  ⚠️ 자동 퇴근 처리                   │
│                                     │
│  출근: 09:00                        │
│  자동 퇴근: 20:00 (11시간 경과)      │
│                                     │
│  야근 신청을 하지 않아               │
│  자동으로 퇴근 처리되었습니다.       │
│                                     │
│  실제 퇴근 시간이 다른가요?          │
│  [시간 수정하기]                    │
│                                     │
│  💡 과로 방지를 위해                │
│     11시간 후 자동 퇴근됩니다        │
└─────────────────────────────────────┘
```

**목적**
- 과로 방지 및 건강 보호
- 근무 시간 정확한 기록
- 야근 신청 누락 방지
- 실제 퇴근 시간과 기록 시간 불일치 해소

**일일 보고는 간단하게**
- 오늘 한 일 (텍스트)
- 진행률 (슬라이더 0-100%)
- 완료 작업 (체크리스트)
- 내일 할 일

```javascript
// 출근 기록
const checkInResult = await checkIn(requestId);
console.log('출근 시각:', checkInResult.check_in_time);

// 자동 퇴근 체크 (11시간 경과 시)
const autoCheckOutTime = new Date(checkInResult.check_in_time);
autoCheckOutTime.setHours(autoCheckOutTime.getHours() + 11);

setTimeout(() => {
  const hasOvertimeRequest = await checkOvertimeRequest(requestId);
  
  if (!hasOvertimeRequest) {
    // 자동 퇴근 처리
    await autoCheckOut(requestId, {
      reason: 'auto_11h',
      message: '11시간 경과로 자동 퇴근 처리'
    });
    
    showNotification({
      title: '⚠️ 자동 퇴근 처리',
      body: '11시간 경과로 자동 퇴근되었습니다.\n실제 퇴근 시간이 다르다면 수정해주세요.',
      action: '시간 수정하기'
    });
  }
}, autoCheckOutTime - Date.now());

// 수동 퇴근 기록
await checkOut(requestId);
// 근무 시간 자동 계산

// 업무 보고
await submitReport(requestId, {
  work_content: '123화 스토리보드 완성, 라인 작업 50%',
  progress_rate: 70,
  completed_tasks: ['스토리보드', '러프 스케치'],
  next_day_plan: '라인 마무리, 채색 시작'
});
```

---

### 4. 알림은 적절하게

**필요한 순간에만, 작가 맞춤형**

#### 발송 시점
- **마감 알림** → D-7, D-3, D-1 (작가에게)
- **마감 당일** → 오전 9시 알림 (작가 + 담당 에디터)
- **휴재 신청** → 승인자에게
- **휴재 승인/반려** → 신청자에게
- **자동 퇴근** → 11시간 경과 시 본인에게 ⭐
- **10시까지 미출근** → 본인 + 관리자 (재택근무 시)
- **23시까지 미보고** → 본인 (재택근무 시)
- **마감 연장** → 휴재로 인한 마감 조정 시 에디터에게

#### 발송 채널
- **긴급** (마감 당일, 자동 퇴근): 푸시 + 이메일 + Slack
- **중요** (마감 D-3, 휴재 승인): 푸시 + 이메일
- **일반** (마감 D-7): 이메일
- **참고** (휴재 신청): Slack 메시지

#### 알림 메시지 예시
```javascript
// 마감 임박 알림 (D-3)
{
  title: '⏰ 마감 3일 전입니다',
  body: '환생 검신 125화 마감일이 3일 남았습니다.\n현재 진행률: 60%',
  action: '진행 상황 확인',
  priority: 'high'
}

// 자동 퇴근 알림
{
  title: '⚠️ 자동 퇴근 처리',
  body: '11시간 경과로 자동 퇴근되었습니다.\n고생하셨습니다! 편히 쉬세요 😊',
  action: '시간 수정하기',
  priority: 'urgent'
}

// 휴재 자동 승인
{
  title: '✅ 휴재 자동 승인',
  body: '일상 휴식 신청이 자동 승인되었습니다.\n마감일에 지장이 없어 즉시 승인됩니다.',
  action: '내 일정 보기',
  priority: 'normal'
}
```

---

### 5. 관리자 화면은 한눈에

**대시보드 구성 (작가 에이전시 맞춤)**
```
┌─────────────────────────────────────┐
│  오늘 휴재 작가: 5명                 │
│  임박한 마감 (D-3 이내): 12건        │
│  대기 중인 휴재 승인: 3건            │
│  마감 충돌 경고: 2건 ⚠️              │
├─────────────────────────────────────┤
│  [작가별 마감 준수율 차트]           │
│  [월별 휴재 현황 (유형별)]           │
│  [재택근무 시간 통계]                │
│  [자동 퇴근 발생 건수]               │
│  [건강 휴재 추이] (손목 통증 등)     │
└─────────────────────────────────────┘
```

**빠른 액션**
- 일괄 승인 버튼
- 마감 충돌 해결 지원
- 자동 승인 규칙 설정 (작가별)
- 이상 징후 알림 (미출근, 미보고, 과로)
- Excel 다운로드 (마감 일정표)

**자동 승인 규칙 설정 예시**
```javascript
// 작가별 자동 승인 규칙
{
  author_id: 'author_123',
  author_name: '김작가',
  rules: [
    {
      break_type: 'casual',        // 일상 휴식
      max_days: 2,                 // 최대 2일
      auto_approve: true,          // 자동 승인
      condition: 'no_deadline_conflict'  // 마감 충돌 없을 때만
    },
    {
      break_type: 'regular',       // 정기 휴재
      auto_approve: false,         // 승인 필요
      notify_to: ['editor_456']    // 담당 에디터에게 알림
    }
  ]
}
```

---

## 🛠️ 기술 구현 방향

### UI 프레임워크
**Material-UI를 쓰면 좋겠다**
- 이미 설치됨
- 컴포넌트가 풍부함
- DatePicker 내장
- 반응형 자동 지원

### 상태 관리
**React Query + Zustand 조합**
- React Query: 서버 데이터 (근태 목록, 연차 현황)
- Zustand: UI 상태 (모달 열림/닫힘, 필터)

### 폼 관리
**React Hook Form + Yup**
- 이미 설치됨
- 유효성 검증 간편
- 성능 우수

```javascript
// 폼 검증 예시
const schema = yup.object({
  start_date: yup.date().required(),
  end_date: yup.date().min(yup.ref('start_date')),
  reason: yup.string().required().min(10)
});
```

---

## 📊 우선순위별 작업

### 🔴 1순위 (지금 당장)
**이것부터 만들자 - 작가 맞춤 핵심 기능**
1. 휴재 신청 폼 (5가지 유형 카드)
2. 마감일 충돌 검증
3. 자동 승인 로직 (일상 휴식)
4. 마감 캘린더 (월간 뷰 + 마감일 표시)
5. 휴재 신청 목록 조회
6. 반응형 레이아웃

**예상 기간**: 2주  
**팀원**: 프론트 1명 + 백엔드 1명

---

### 🟡 2순위 (다음 달)
**작가 건강 및 과로 방지**
1. ⭐ **11시간 자동 퇴근 시스템**
2. 재택근무 출퇴근 기록
3. 업무 보고 작성 (연재물별)
4. 마감 알림 (D-7, D-3, D-1)
5. 자동 퇴근 알림
6. 휴재 승인/반려 처리
7. 건강 휴재 통계 (손목 통증, 눈 피로 등)

**예상 기간**: 2주  
**팀원**: 프론트 1명 + 백엔드 1명

---

### 🟢 3순위 (여유 있을 때)
**완성도를 높이는 기능**
- 작가별 자동 승인 규칙 설정
- 마감 준수율 통계 대시보드
- 연재물별 진행률 추적
- 마감 일정 공유 URL
- 휴재 패턴 분석 (과로 징후 감지)
- 임시 저장
- 다크 모드
- Slack 연동 (마감 알림, 휴재 공지)

**예상 기간**: 3주  
**팀원**: 프론트 1명 + 백엔드 1명

---

## 🎨 화면 구성 제안

### 메인 레이아웃 (작가 대시보드)
```
┌─────────────────────────────────────┐
│  [로고]  휴재관리    [알림] [프로필] │
├────┬────────────────────────────────┤
│ 📅 │  다음 마감: 환생 검신 125화     │
│휴재│  D-5 (2026.01.18)              │
│신청│                                │
│────│  [휴재 신청하기] 버튼           │
│ 📆 │                                │
│마감│  진행 중인 연재:                │
│캘린│  - 환생 검신 (125화, 60%)       │
│더  │  - 마법학교 일상 (87화, 80%)    │
│────│                                │
│ 🏠 │  최근 휴재 이력:                │
│재택│  - 2026.01.10-11 일상휴식 ✓    │
│근무│  - 2025.12.30-01.05 시즌휴재 ✓ │
│    │                                │
└────┴────────────────────────────────┘
```

### 모바일 화면
```
┌─────────────────┐
│  [≡] 휴재관리    │
├─────────────────┤
│                 │
│ 다음 마감 D-5    │
│ 환생 검신 125화  │
│                 │
│ [휴재 신청하기]  │
│                 │
│ 진행 중인 연재   │
│ 환생 검신 60%    │
│ 마법학교 80%     │
│                 │
│ 최근 휴재        │
│ 01/10-11 일상✓  │
│                 │
│ [하단 탭 바]     │
│ 휴재|마감|재택   │
└─────────────────┘
```

---

## 🚀 빠른 시작 가이드

### API 사용 예시

```javascript
// 1. 휴재 신청 (마감일 충돌 검증 포함)
import { createBreak, getDeadlines } from '@/api/leave';

// 마감일 확인
const deadlines = await getDeadlines({
  author_id: userId,
  start_date: '2026-01-20',
  end_date: '2026-01-22'
});

// 휴재 신청
await createBreak({
  break_type: 'health',           // 건강 휴재
  start_date: '2026-01-20',
  end_date: '2026-01-22',
  reason: '손목 통증 치료',
  
  // 마감 조정 계획
  affected_deadlines: deadlines.map(d => ({
    serial_id: d.serial_id,
    original_deadline: d.deadline,
    adjusted_deadline: addDays(d.deadline, 3),
    adjustment_reason: '건강 휴재로 인한 연장'
  })),
  
  auto_approve: false  // 승인 필요
});

// 2. 재택근무 출퇴근 기록 (11시간 자동 퇴근 포함)
import { checkIn, checkOut, autoCheckOut } from '@/api/remote';

// 출근
const checkInResult = await checkIn('remote_123');

// 11시간 후 자동 퇴근 스케줄링
const auto_checkout_time = new Date(checkInResult.check_in_time);
auto_checkout_time.setHours(auto_checkout_time.getHours() + 11);

setTimeout(async () => {
  const hasOvertime = await checkOvertimeRequest('remote_123');
  if (!hasOvertime) {
    await autoCheckOut('remote_123', {
      reason: 'auto_11h',
      message: '11시간 경과로 자동 퇴근'
    });
  }
}, auto_checkout_time - Date.now());

// 수동 퇴근
await checkOut('remote_123');

// 3. 마감일 계산
import { calculateWorkDays, getDaysDiff } from '@/utils/date';

const deadline = new Date('2026-01-25');
const today = new Date();
const daysLeft = getDaysDiff(today, deadline); // D-12

// 여유 기간 확인
if (daysLeft < 3) {
  alert('⚠️ 마감 3일 전입니다!');
}
```

---

## 📁 프로젝트 구조

```
leave-management/
├── src/
│   ├── api/              ← 준비됨 (13개 함수)
│   │   ├── client.js     (axios 설정)
│   │   ├── leave.js      (휴재 신청, 조회)
│   │   └── remote.js     (재택근무, 자동퇴근)
│   │
│   ├── utils/            ← 준비됨 (4개 함수)
│   │   └── date.js       (날짜 계산, 포맷팅)
│   │
│   ├── components/       ← 이제 만들 차례
│   │   ├── BreakForm.jsx         (휴재 신청 폼)
│   │   ├── BreakTypeCard.jsx     (휴재 유형 카드)
│   │   ├── DeadlineCalendar.jsx  (마감 캘린더)
│   │   ├── CheckInButton.jsx     (출근 버튼)
│   │   ├── AutoCheckOutAlert.jsx (자동 퇴근 알림)
│   │   └── DeadlineAlert.jsx     (마감 알림)
│   │
│   ├── pages/            ← 이제 만들 차례
│   │   ├── BreakPage.jsx         (휴재 신청/조회)
│   │   ├── DeadlineCalendarPage.jsx  (마감 캘린더)
│   │   ├── RemoteWorkPage.jsx    (재택근무)
│   │   └── DashboardPage.jsx     (작가 대시보드)
│   │
│   ├── hooks/            ← 필요하면 추가
│   │   ├── useDeadlineCheck.js   (마감일 충돌 검증)
│   │   └── useAutoCheckOut.js    (11시간 자동 퇴근)
│   │
│   └── stores/           ← 필요하면 추가
│       └── breakStore.js         (휴재 상태 관리)
│
├── docs/                 ← 참고 문서
└── package.json          ← 패키지 설치 완료
```

---

## ✅ 체크리스트

### 인프라
- [x] 프로젝트 생성
- [x] 패키지 설치
- [x] 개발 서버 실행
- [x] API 함수 작성 (휴재, 재택근무)
- [x] 유틸리티 함수 작성 (날짜 계산)

### 다음 작업 (휴재 중심)
- [ ] 라우팅 설정 (React Router)
- [ ] 휴재 신청 폼 UI (5가지 유형 카드)
- [ ] 마감 캘린더 화면 (충돌 감지)
- [ ] 11시간 자동 퇴근 시스템 ⭐
- [ ] 상태 관리 (React Query 설정)
- [ ] 자동 승인 로직

---

## 💭 개발 시 고려사항

### 성능
- 마감 캘린더 데이터는 캐싱 (1분)
- 무한 스크롤로 휴재 이력 표시
- 작가별 연재물 목록 최적화

### 보안
- API 호출마다 토큰 검증
- XSS 방지 (휴재 사유 입력 검증)
- HTTPS 필수
- 마감 조정 권한 확인

### 사용성 (작가 맞춤)
- 로딩 상태 표시 필수
- 마감 충돌 시 명확한 안내
- "D-5" 같은 직관적 표시
- 모바일에서도 편하게
- **자동 퇴근 알림은 친절하게** (과로 방지 목적 명시)

### 작가 건강 보호
- 11시간 자동 퇴근 강제 적용
- 건강 휴재 패턴 분석 (손목 통증 빈도)
- 과로 징후 감지 (연속 12시간 이상 근무)
- 적절한 휴식 권장 알림

---

**다음 작업**: 휴재 신청 폼 UI 구현 (5가지 유형 카드)  
**예상 소요**: 3-4일  
**필요한 것**: 
- react-router-dom 설치
- MUI 컴포넌트 활용 (Card, Chip)
- 마감일 API 연동
- 자동 승인 로직 백엔드 협의

---

## 🔌 추가로 필요한 백엔드 API

### 1. 휴재 관리 API

```javascript
// 휴재 신청 (마감 정보 포함)
POST /api/breaks/request
{
  break_type: 'health',
  start_date: '2026-01-20',
  end_date: '2026-01-22',
  reason: '손목 통증 치료',
  affected_deadlines: [...],
  auto_approve: false
}

// 휴재 목록 조회
GET /api/breaks/requests?author_id={id}&status={status}

// 휴재 상세 조회
GET /api/breaks/requests/{id}

// 휴재 승인/반려
PATCH /api/breaks/requests/{id}/approve
PATCH /api/breaks/requests/{id}/reject
```

### 2. 마감일 관리 API

```javascript
// 작가의 마감일 조회
GET /api/deadlines?author_id={id}&start_date={date}&end_date={date}

// 마감일 충돌 검사
POST /api/deadlines/check-conflict
{
  author_id: 'author_123',
  start_date: '2026-01-20',
  end_date: '2026-01-22'
}
Response: {
  has_conflict: true,
  conflicts: [
    {
      serial_id: 'novel_123',
      title: '환생 검신',
      episode: 125,
      deadline: '2026-01-21',
      days_before_deadline: 1
    }
  ]
}

// 마감일 조정
PATCH /api/deadlines/{id}/adjust
{
  new_deadline: '2026-01-24',
  reason: '건강 휴재로 인한 연장',
  break_request_id: 'break_456'
}
```

### 3. 자동 퇴근 관리 API

```javascript
// 야근 신청 여부 확인
GET /api/overtime/check?remote_id={id}&date={date}

// 자동 퇴근 처리
POST /api/remote-work/{id}/auto-checkout
{
  reason: 'auto_11h',
  message: '11시간 경과로 자동 퇴근',
  original_check_in: '2026-01-13 09:00:00'
}

// 퇴근 시간 수정
PATCH /api/remote-work/{id}/checkout-time
{
  checkout_time: '2026-01-13 21:30:00',
  reason: '실제 퇴근 시간 수정'
}
```

### 4. 자동 승인 규칙 API

```javascript
// 작가별 자동 승인 규칙 조회
GET /api/authors/{id}/auto-approve-rules

// 자동 승인 규칙 설정 (관리자)
POST /api/authors/{id}/auto-approve-rules
{
  break_type: 'casual',
  max_days: 2,
  auto_approve: true,
  condition: 'no_deadline_conflict'
}

// 자동 승인 가능 여부 확인
POST /api/breaks/check-auto-approve
{
  author_id: 'author_123',
  break_type: 'casual',
  start_date: '2026-01-15',
  end_date: '2026-01-16'
}
Response: {
  auto_approve: true,
  reason: '마감 충돌 없음, 2일 이내 일상 휴식'
}
```

### 5. 통계 및 분석 API

```javascript
// 작가별 휴재 패턴 분석
GET /api/analytics/break-pattern?author_id={id}

// 건강 휴재 추이
GET /api/analytics/health-breaks?author_id={id}

// 마감 준수율
GET /api/analytics/deadline-compliance?author_id={id}

// 과로 징후 감지
GET /api/analytics/overwork-detection?author_id={id}
Response: {
  is_overworked: true,
  indicators: [
    '최근 7일간 평균 근무 시간 12시간',
    '건강 휴재 빈도 증가 (월 2회 → 4회)',
    '마감 지연 2건 발생'
  ],
  recommendation: '적절한 휴식을 권장합니다'
}
```

---

## 📝 용어 정리

### 기존 vs 변경
| 기존 용어 | 변경 용어 | 설명 |
|-----------|-----------|------|
| 연차 (Annual Leave) | 휴재 (Break) | 작가가 연재를 쉬는 것 |
| 잔여 연차 | 다음 마감일 | 관리의 중심 변경 |
| 휴가 신청 | 휴재 신청 | 작가 용어로 변경 |
| 팀원 일정 | 작가별 마감 일정 | 작가 중심 |
| 출장 | 외부 작업 | (필요시) |

---

**🎯 핵심 차별점**

이 시스템은 일반 근태 관리와 다르다:

1. **마감 중심**: 잔여 연차가 아닌 마감일 기준으로 관리
2. **유연한 승인**: 마감에 지장 없으면 자동 승인
3. **작가 건강**: 11시간 자동 퇴근, 건강 휴재 추이 분석
4. **프리랜서 친화**: 획일적 정책 대신 개인별 맞춤 규칙
5. **과로 방지**: 시스템이 적극적으로 휴식 권장

---

**다음 단계**: 백엔드 팀과 협의 → API 명세 확정 → 프론트 UI 개발 시작
