# 01. SSH 공개키 인증이란 무엇이고 왜 쓰는가
# 01. What is SSH Public-Key Authentication and Why Use It

## 1. SSH란? (What is SSH?)

SSH (**S**ecure **Sh**ell)는 원격 컴퓨터에 안전하게 접속해서 명령을 실행할 수 있게 해주는 표준 프로토콜이다.
SSH is the standard protocol for securely connecting to and running commands on a remote computer.

- 포트: 기본 22번 (변경 가능)
- 모든 통신은 암호화됨 (도청 방지)
- 서버를 운영·관리하는 가장 일반적인 방법

## 2. SSH 접속 방식 두 가지 (Two ways to authenticate)

| 방식<br/>Method | 어떻게 동작?<br/>How it works | 안전성<br/>Safety |
| :--- | :--- | :---: |
| **비밀번호 인증**<br/>Password auth | 매번 비밀번호 입력<br/>Type password every time | ⚠️ 낮음 (Low) |
| **공개키 인증**<br/>Public-key auth | 키 쌍으로 서로 검증, 비밀번호 입력 없음<br/>Mutual verification via key pair, no password | ✅ 높음 (High) |

오늘날 운영 서버들은 거의 모두 비밀번호 인증을 끄고 공개키 인증만 허용한다.
Modern production servers almost universally disable password auth and require public-key auth only.

## 3. 왜 공개키가 더 안전한가? (Why is public-key safer?)

### 3.1 비밀번호의 약점 (Weakness of passwords)

- **무차별 대입 공격(brute-force)에 취약** — 짧거나 흔한 비밀번호는 자동화 도구로 수시간 내 뚫림
  Vulnerable to brute-force — short/common passwords cracked in hours
- **재사용 위험** — 다른 사이트에서 유출된 비밀번호와 동일하면 즉시 침해
  Reuse risk — if leaked elsewhere, your server is exposed
- **인간이 기억하므로 복잡도에 한계** — `P@ssw0rd!` 정도가 한계
  Human memory limits complexity
- **전송 중 도청에 취약** — 약한 채널에서는 평문 노출 위험
  Network sniffing risk in weak channels

### 3.2 공개키의 강점 (Strength of public keys)

- **256비트 ED25519 키의 가능 경우의 수: 약 2²⁵⁶** — 우주 종말까지 brute-force 불가능
  256-bit ED25519 has ~2²⁵⁶ possibilities — uncrackable until heat death of the universe
- **비공개키는 절대 네트워크를 타지 않는다** — "챌린지-응답" 방식으로만 검증되므로 도청해도 키를 얻을 수 없음
  The private key never leaves your machine — verified via challenge/response, so sniffers gain nothing
- **공개키는 노출되어도 무해** — 자물쇠를 길에 떨어뜨려도 누가 열 수 없는 것과 같음
  The public key is harmless even if exposed — a lock dropped on the street can't be opened by passers-by
- **자동화 친화적** — 패스프레이즈 + ssh-agent 조합으로 매번 비밀번호 안 묻고도 안전 유지
  Automation-friendly — passphrase + ssh-agent makes it both seamless and secure

## 4. maple-staging 서버는 왜 비밀번호를 차단했나?
## 4. Why did the maple-staging server disable passwords?

운영 서버는 보안 강화 정책의 일환으로 `sshd_config`에서 `PasswordAuthentication no`를 설정하여 **비밀번호 접속을 의도적으로 차단**한다. 공개키만이 유일한 인증 수단이다.
As part of security hardening, production servers set `PasswordAuthentication no` in `sshd_config`, **intentionally disabling password login**. Public-key is the only allowed method.

이는 다음과 같은 공격을 원천 차단한다.
This blocks the following attacks at the root:

- ✅ 비밀번호 무차별 대입 (SSH brute-force)
- ✅ 약한 비밀번호 재사용으로 인한 측면 이동 (credential stuffing)
- ✅ 피싱으로 탈취된 비밀번호의 즉시 악용 (phished passwords are useless)

## 5. 핵심 포인트 (Key Takeaways)

- SSH 공개키 인증은 **표준이자 안전한 인증 방식**이다.
- "비밀번호로 접속 불가"는 **운영 서버의 정상적인 보안 설정**이며, 이상한 상황이 아니다.
- 공개키를 보내달라는 요청은 **표준 절차**이며, 의심할 필요 없다.
- 단, **반드시 `.pub` 파일만** 보내고, **`.pub` 안 붙은 파일은 절대 공유하지 말 것**. (자세한 내용은 `02_public-vs-private-key.md`ㅈ)
