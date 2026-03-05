# Starbucks Japan Stock Checker

## 개요

스타벅스 재팬 한정 굿즈의 재고를 자동으로 감지하여, 재입고 시 이메일로 알림을 보내는 GitHub Actions 봇

---

## 배경

스타벅스 재팬 한정 컵 등 굿즈는 재입고 시점을 알기 어렵고, 수동으로 계속 확인하기 번거럽다.  
재입고를 빠르게 포착하여 구매 기회를 놓치지 않는 것이 목적이다.

---

## 동작 방식

### 재고 감지 원리

스타벅스 재팬 계정으로 로그인 후 최종 도달 URL로 재고 판단

**전체 플로우:**
1. `api/star-login?jan_code=...` 요청 → 로그인 페이지로 리다이렉트
2. 로그인 페이지에서 세션 쿠키 + CSRF 토큰 추출
3. 계정 이메일/패스워드 + CSRF 토큰 POST 로그인
4. OAuth 콜백 거쳐 최종 URL로 재고 판단

| 최종 도달 URL | 의미 |
|---|---|
| `star_cart?error_code=110` | ❌ 품절 |
| `star_cart` (error 없음) | ✅ 재고 있음 |
| `login` 페이지 | ⚠️ 로그인 실패 (자격증명 오류) |

### 실행 주기

- **자동:** 4시간마다 cron 실행 (UTC 0, 4, 8, 12, 16, 20시 / KST 9, 13, 17, 21, 1, 5시)
- **수동:** GitHub Actions UI에서 `Run workflow` 버튼으로 즉시 실행 가능

### 알림

- 재고 감지 시 Gmail SMTP를 통해 이메일 발송
- `force_notify: true` 옵션으로 재고 여부 무관하게 테스트 메일 발송 가능

---

## 설정값

### GitHub Repository Secrets

| 이름 | 설명 |
|---|---|
| `GMAIL_USER` | 발송용 Gmail 주소 |
| `GMAIL_APP_PASSWORD` | Gmail 앱 비밀번호 (16자리) |
| `NOTIFY_EMAIL` | 알림 받을 이메일 주소 |
| `SBUX_EMAIL` | 스타벅스 재팬 계정 이메일 |
| `SBUX_PASSWORD` | 스타벅스 재팬 계정 패스워드 |

### GitHub Repository Variables

| 이름 | 설명 | 기본값 |
|---|---|---|
| `JAN_CODE` | 감시할 상품 JAN 코드 | `4524785613492` |

---

## 감시 상품

| JAN 코드 | 비고 |
|---|---|
| `4524785613492` | 감시 대상 한정 컵 |
| `4524785531086` | 테스트용 (재고 있는 굿즈) |

---

## 파일 구조

```
.github/
  workflows/
    check-stock.yml   # GitHub Actions 워크플로우
```
