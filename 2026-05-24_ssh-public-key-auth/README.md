# SSH 공개키 인증 (Public Key Authentication)

2026-05-24, maple-staging 서버 접속을 위해 개발팀으로부터 **"SSH 공개키(`id_25519.pub`)를 보내달라. 우리 서버는 비밀번호 접속이 차단되어 있다"** 라는 요청을 받았던 실제 상황을 바탕으로 정리한 학습 노트.

Real-world study notes triggered by a request to register an SSH public key with the `maple-staging` server, which has password authentication disabled.

## 개요 (Overview)

- **트리거 메시지 (Trigger):** "테스트 서버 접근을 위해서 사용하는 `id_255*.pub` 을 알려주세요. 암호로는 접근 불가 설정이어요. 제가 등록 후 알려드릴게요."
- **핵심 키 알고리즘 (Algorithm):** ED25519 (`id_ed25519` / `id_ed25519.pub`)
- **사용자 OS:** macOS (`~/.ssh/` 기반)
- **목표 (Goal):** 공개키 인증의 본질을 이해하고, 안전하게 공개키만 공유한 뒤 비밀번호 없이 서버에 접속할 수 있게 한다.
  Understand the essence of SSH public-key auth, safely share only the public half, and log in without a password.

## 파일 목록 (Files)

| 파일<br/>File | 내용<br/>Content |
| :--- | :--- |
| `01_what-and-why.md` | 공개키 인증이 무엇이고 왜 비밀번호보다 안전한가<br/>What public-key auth is and why it's safer than passwords |
| `02_public-vs-private-key.md` | 공개키 vs 비공개키 — 자물쇠와 열쇠의 비유<br/>Public vs private key — the lock-and-key metaphor |
| `03_register-and-connect.md` | 공개키 등록·전달·접속 실전 절차<br/>Hands-on: generate, share, register, connect |
| `04_security-pitfalls.md` | 자주 저지르는 실수와 안전 체크리스트<br/>Common mistakes and a safety checklist |

## 한 줄 요약 (TL;DR)

> 공개키(`.pub`)는 **자물쇠** — 누구에게 줘도 안전. 비공개키(`.pub` 없음)는 **열쇠** — 절대 공유하지 말 것. 답장에는 반드시 `.pub` 파일의 한 줄만 붙여넣는다.
> The `.pub` file is a **lock** — safe to share. The non-`.pub` file is the **key** — never share. Paste only the single `.pub` line in your reply.
