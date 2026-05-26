# 02. 공개키 vs 비공개키 — 자물쇠와 열쇠의 비유
# 02. Public vs Private Key — The Lock-and-Key Metaphor

## 1. 한눈에 보기 (At a glance)

| 구분<br/>Aspect | 공개키 (Public Key)<br/>`*.pub` | 비공개키 (Private Key)<br/>확장자 없음 / no extension |
| :--- | :--- | :--- |
| 비유<br/>Metaphor | 🔒 **자물쇠** (Lock) | 🗝️ **열쇠** (Key) |
| 역할<br/>Role | 서버에 등록되어 본인을 식별<br/>Registered on server to identify you | 본인 PC에 보관, 챌린지에 서명<br/>Kept on your PC, signs challenges |
| 공유 가능?<br/>Shareable? | ✅ **예 — 누구에게 줘도 안전**<br/>Yes — safe to share with anyone | ❌ **아니오 — 절대 공유 금지**<br/>No — never share |
| 유출 시<br/>If leaked | 무해<br/>Harmless | 🚨 즉시 모든 서버 권한 탈취<br/>Immediate full takeover |
| 파일 권한<br/>File permissions | `644` (모두 읽기 가능)<br/>`644` (world-readable) | `600` (본인만 읽기/쓰기)<br/>`600` (owner-only) |
| 파일 크기<br/>File size | ~100 bytes | ~400 bytes |
| 어디에 보관?<br/>Where stored? | ① 내 PC `~/.ssh/`<br/>② 접속할 서버의 `~/.ssh/authorized_keys`<br/>① Local `~/.ssh/`<br/>② Target server's `~/.ssh/authorized_keys` | 내 PC `~/.ssh/` 만<br/>Local `~/.ssh/` only |

## 2. 비유로 이해하기 (Understanding via metaphor)

```
[내 PC]                        [서버]
┌─────────────────┐            ┌─────────────────┐
│  🗝️ id_ed25519  │ ← 절대 안 나감 │   🔒 자물쇠 등록 │
│   (비공개키)     │  never leaves│  (공개키 등록됨) │
│                 │              │                 │
│  📄 id_..pub    │ → 안전하게 전송→│  ~/.ssh/        │
│   (공개키 사본)  │  safely sent  │  authorized_keys│
└─────────────────┘              └─────────────────┘
        │                                ▲
        │    1) "들어가도 돼?"            │
        │       "Can I come in?"          │
        ├────────────────────────────────►│
        │                                 │
        │    2) "이 챌린지에 서명해봐"     │
        │       "Sign this challenge"     │
        │◄────────────────────────────────┤
        │                                 │
        │    3) 🗝️로 서명해서 전송         │
        │       Signs with private key    │
        ├────────────────────────────────►│
        │                                 │
        │    4) 🔒로 검증 → ✅ 통과       │
        │       Verifies with public key  │
        │◄────────────────────────────────┤
```

핵심: **비공개키는 절대 네트워크를 타지 않는다.** 서명만 보낸다. 도청자는 챌린지/응답을 가로채도 비공개키를 역산할 수 없다.
The private key never travels the network — only signatures do. Eavesdroppers cannot reverse-engineer the private key from intercepted traffic.

## 3. macOS 기준 파일 위치 (File locations on macOS)

표준 SSH 키 저장 위치:
Standard SSH key location:

```
~/.ssh/
├── id_ed25519        ← 🗝️ 비공개키 (private) — NEVER share
├── id_ed25519.pub    ← 🔒 공개키 (public)  — safe to share
├── known_hosts       ← 접속해본 서버들의 지문 모음 (visited servers' fingerprints)
├── authorized_keys   ← 내 PC를 서버로 쓸 때, 접속 허용할 외부 공개키 모음
│                      (when your PC acts as a server, list of trusted public keys)
└── config            ← (선택) SSH 별칭/옵션 설정 (optional shortcuts/options)
```

> 참고: ED25519는 2024년 이후 가장 권장되는 알고리즘이다. 과거에는 `id_rsa` / `id_rsa.pub` (RSA 2048/4096) 가 표준이었다. 둘 다 안전하지만 ED25519가 더 짧고 빠르다.
> Note: ED25519 is the recommended modern algorithm. RSA (2048/4096) is the older standard — both are secure, but ED25519 is shorter and faster.

## 4. 공개키 파일의 실제 구조 (Anatomy of a `.pub` file)

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIExAmpl...eKy...Data user@example.com
└────┬────┘ └──────────────┬────────────────┘ └──────┬─────┘
   algorithm           base64 key data            comment (보통 이메일)
   알고리즘            base64로 인코딩된 키 데이터    주석 (보통 이메일)
```

- **algorithm**: `ssh-ed25519`, `ssh-rsa`, `ecdsa-sha2-nistp256` 등
- **base64 key data**: 실제 공개키 바이트열의 base64 인코딩
- **comment**: 이 키의 식별용 메모 (보통 `사용자명@호스트` 또는 이메일)

한 줄로 한 키를 표현하며, **줄바꿈 없이 정확히 한 줄** 그대로 전송·등록한다.
One key = exactly one line, no linebreaks — copy/paste verbatim.

## 5. 자주 헷갈리는 점 (Common confusion)

### 5.1 "id_25519"인가 "id_ed25519"인가?

maple 개발팀이 보낸 메시지에는 `id_255*.pub` 라고 쓰여 있었다. 이는 `id_ed25519.pub` 의 줄임/약어로 보면 된다. **ED25519** 알고리즘 이름에서 `25519`만 따온 비공식 약어.
The developer's message wrote `id_255*.pub` — an informal shorthand for `id_ed25519.pub` (just the `25519` part of the algorithm name).

| 명명<br/>Naming | 의미<br/>Meaning |
| :--- | :--- |
| `id_ed25519` | 정식 파일명 (official filename) |
| `id_25519` | 비공식 줄임 (informal shorthand) |
| `id_255*` | 와일드카드 표기 (wildcard, "the one starting with id_255...") |

→ 모두 같은 파일을 지칭한다. **정확한 표준 파일명은 `~/.ssh/id_ed25519.pub`**.

### 5.2 "공개키"인데 왜 비밀처럼 다뤄야 하지 않나?

공개키는 정말 공개해도 된다. 길에 떨어뜨려도 무해하다.
The public key is genuinely public. Dropping it in the street is harmless.

다만 **공개키 ≠ 비공개키** 임을 헷갈리지 말 것. 답장 보낼 때 `.pub` 만 첨부했는지 한 번 더 확인하는 습관이 중요하다.
But **never confuse public with private**. Always double-check that only the `.pub` file is attached in your reply.

### 5.3 한 PC에서 여러 키를 쓸 수 있나?

가능하다. 일반적으로:
- 개인용 GitHub: `~/.ssh/id_ed25519`
- 회사 서버용: `~/.ssh/id_ed25519_company`
- 클라우드 인프라용: `~/.ssh/id_ed25519_aws`

`~/.ssh/config` 로 호스트별 키를 매핑한다.
Use `~/.ssh/config` to map hosts to keys.

```
Host maple-staging
    HostName staging.example.org
    User <username>
    IdentityFile ~/.ssh/id_ed25519
    Port 22
```
