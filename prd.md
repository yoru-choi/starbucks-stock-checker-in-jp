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

로그인 없이 공개 굿즈 교환 페이지 HTML을 파싱하여 재고 판단

**대상 페이지:** `https://www.starbucks.co.jp/mystarbucks/reward/exchange/original_goods/`

페이지 내 JAN 코드가 일치하는 버튼의 CSS 클래스로 판단:

| HTML | 의미 |
|---|---|
| `<button class="js-cartform-instock" data-jan="...">` | ✅ 재고 있음 (`hide` 없음) |
| `<button class="js-cartform-instock hide" data-jan="...">` | ❌ 품절 (`hide` 있음) |

로그인 불필요. Playwright 헤드리스 브라우저로 JS 완전 실행 후 DOM 확인

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
