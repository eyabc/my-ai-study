# 04. 자주 저지르는 실수와 안전 체크리스트
# 04. Common Pitfalls and Safety Checklist

---

## 1. Top 5 위험 실수 (Top 5 dangerous mistakes)

### 🚨 ① 비공개키를 채팅·이메일·Slack에 붙여넣음
### 🚨 ① Pasting the private key in chat/email/Slack

**시나리오:** "공개키 보내달라"는 요청에 `cat ~/.ssh/id_ed25519` (`.pub` 없음) 의 결과를 복사해서 답장.
**Scenario:** Reply with the output of `cat ~/.ssh/id_ed25519` (no `.pub`) instead of the `.pub` file.

**영향:** 이 키가 등록된 **모든 서버**에 즉시 침해. 본인 PC 권한 그대로 누가 사용 가능.
**Impact:** Immediate compromise of **every** server where this key is registered.

**대응:**
- 보내기 직전 파일명에 `.pub` 가 있는지 **한 번 더 눈으로 확인**
- 비공개키 내용은 보통 **여러 줄에 걸친 PEM 블록**(`-----BEGIN OPENSSH PRIVATE KEY-----`)이므로 한눈에 다름
  Private key content is a multi-line PEM block, visually distinct
- 사고 발생 시 **즉시 키 폐기**: `~/.ssh/id_ed25519*` 삭제 + 새 키 생성 + 등록된 모든 서버에서 해당 공개키 제거
  If leaked, immediately rotate: delete keys + generate new ones + remove from `authorized_keys` everywhere

---

### 🚨 ② 비공개키를 GitHub/공개 레포에 커밋
### 🚨 ② Committing the private key to GitHub or any public repo

자동 봇들이 GitHub를 24시간 스캔하며 SSH 키, AWS 키 등을 찾는다. **1분 안에 발견되고 1시간 안에 악용된다.**
Bots scan GitHub 24/7 for keys. Discovery happens within minutes, exploitation within an hour.

**대응:**
- `.gitignore` 에 처음부터 `id_*` 패턴 등록
- 커밋 전 `git diff --cached` 로 항상 한 번 더 검토
- `gitleaks` 같은 사전 스캔 도구 사용 (maple 프로젝트는 #561로 이미 도입됨)
  Use pre-commit scanners like `gitleaks` (maple already uses it per #561)
- 사고 발생 시 **단순 커밋 삭제는 무효** — git 히스토리에서 영구 제거(`filter-repo` 등)하고 새 키로 교체

---

### ⚠️ ③ 패스프레이즈 없이 키 생성
### ⚠️ ③ Generating keys without a passphrase

`ssh-keygen` 실행 시 Enter만 누르면 패스프레이즈 없이 키가 생성된다. 편하지만 **노트북 분실·도난 시 즉시 무방비**.
Skipping passphrase makes the key usable by anyone who steals your laptop.

**대응:**
- 처음부터 강한 패스프레이즈를 설정
- 매번 입력하기 귀찮다면 macOS `ssh-add --apple-use-keychain` 으로 Keychain 통합
- 이미 패스프레이즈 없이 만들어졌다면 다음 명령으로 사후 추가 가능
  Add a passphrase to an existing key:

```bash
ssh-keygen -p -f ~/.ssh/id_ed25519
```

---

### ⚠️ ④ 파일 권한이 너무 열려 있음
### ⚠️ ④ Overly permissive file permissions

```bash
ls -la ~/.ssh/id_ed25519
# 잘못된 예: -rw-r--r--  (644) — 누구나 읽기 가능
```

SSH는 이런 키를 **사용 거부**한다 (정상 작동을 안 한다).
SSH **refuses** to use such a key — it just won't work.

**대응:**

```bash
chmod 600 ~/.ssh/id_ed25519       # 비공개키
chmod 644 ~/.ssh/id_ed25519.pub   # 공개키
chmod 700 ~/.ssh                  # 디렉터리
chmod 600 ~/.ssh/authorized_keys  # (있다면)
chmod 600 ~/.ssh/config           # (있다면)
chmod 644 ~/.ssh/known_hosts      # (보통 자동 관리)
```

---

### ⚠️ ⑤ `known_hosts` 경고를 무시하고 계속 접속
### ⚠️ ⑤ Ignoring `known_hosts` warnings

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
```

이 경고는 두 가지 가능성을 알린다.
This warning means one of two things:

1. ✅ **정상:** 서버가 재설치되었거나 OS 재구성으로 호스트 키가 바뀜
   Server was reinstalled or reconfigured
2. 🚨 **MITM 공격:** 중간자(공격자)가 자신의 서버로 트래픽을 라우팅 중
   Man-in-the-middle attack — an attacker is intercepting

**대응:**
- 절대 무조건 `yes` 누르지 말 것
- 서버 관리자에게 새 지문(fingerprint)을 확인받고 일치하면 `ssh-keygen -R <host>` 로 옛 지문 제거 후 재접속
  Verify the new fingerprint with the admin before clearing with `ssh-keygen -R <host>`

---

## 2. 안전 체크리스트 (Safety Checklist)

### 2.1 키 생성·관리 (Key generation & management)

- [ ] ED25519 알고리즘 사용 (`ssh-keygen -t ed25519`)
- [ ] 강한 패스프레이즈 설정
- [ ] 비공개키 권한 `600`, 디렉터리 권한 `700` 확인
- [ ] 키마다 의미 있는 주석(`-C`) 부여 (예: 이메일, "personal-mac")

### 2.2 키 공유 (Key sharing)

- [ ] **반드시 `.pub` 파일만** 공유 (한 번 더 눈으로 확인)
- [ ] `pbcopy < ~/.ssh/id_ed25519.pub` 같이 명시적 파일 지정으로 복사
- [ ] 공유 채널이 안전한지 확인 (사내 채팅, 회사 이메일 OK; 공개 트위터 NG는 아니지만 무의미)

### 2.3 키 저장 (Key storage)

- [ ] `id_*` 패턴을 `.gitignore` 에 등록
- [ ] 백업 시에는 비공개키를 **별도의 암호화 볼륨**(예: Disk Utility 암호화 dmg, 1Password Secrets)에 저장
- [ ] 클라우드 동기화(iCloud, Dropbox)에 `~/.ssh/` 자체를 통째로 동기화하지 말 것 — 동기화 충돌 시 권한이 깨질 수 있고, 공급사가 키에 접근 가능

### 2.4 키 폐기·교체 (Rotation & revocation)

- [ ] 분기 또는 반기마다 키 교체 권장 (강제 아님)
- [ ] PC 폐기/교체 시 새 PC에서 새 키 생성 → 기존 키는 서버 `authorized_keys` 에서 제거
- [ ] 노트북 분실 시 즉시 모든 서버의 `authorized_keys` 에서 해당 공개키 줄 삭제
- [ ] 사고 시 명령:

```bash
# 새 키 생성
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_new -C "rotated-2026-05-24"

# 서버에서 옛 키 제거 (서버에 ssh-copy-id 또는 수동 편집)
ssh <server> 'sed -i.bak "/old_key_fingerprint_or_comment/d" ~/.ssh/authorized_keys'

# 새 키 등록 후 옛 키 삭제 확인 후 로컬에서도 옛 키 제거
rm ~/.ssh/id_ed25519 ~/.ssh/id_ed25519.pub
mv ~/.ssh/id_ed25519_new ~/.ssh/id_ed25519
mv ~/.ssh/id_ed25519_new.pub ~/.ssh/id_ed25519.pub
```

---

## 3. 실전 점검 — 지금 내 PC 상태 (Live audit of current PC)

이번 학습 시점(2026-05-24) 내 PC 점검 결과:
Audit result at the time of this study (2026-05-24):

| 점검 항목<br/>Check | 결과<br/>Result | 평가<br/>Verdict |
| :--- | :---: | :---: |
| `~/.ssh/id_ed25519` 존재 | ✅ 있음 (464 bytes) | OK |
| `~/.ssh/id_ed25519.pub` 존재 | ✅ 있음 (99 bytes) | OK |
| 비공개키 권한 | `-rw-------` (600) | ✅ 안전 |
| 공개키 권한 | `-rw-r--r--` (644) | ✅ 정상 |
| 알고리즘 | ED25519 | ✅ 현재 표준 |
| `.ssh` 디렉터리 권한 | `drwx------` (700) | ✅ 안전 |
| 패스프레이즈 설정 여부 | (확인 필요)<br/>(verify) | ❓ — `ssh-keygen -y -f ~/.ssh/id_ed25519` 실행 시 패스프레이즈 묻는지로 판별 |

→ 전반적 위생 양호. 패스프레이즈 설정 여부만 확인하면 완성.
Overall hygiene is solid. Only the passphrase status needs confirmation.

---

## 4. 한 줄 결론 (Bottom line)

> 공개키는 **자유롭게 공유**. 비공개키는 **목숨처럼 보관**. 답장 보내기 전 **`.pub` 한 글자만 확인**하면 거의 모든 사고는 방지된다.
> **Share** public keys freely. **Guard** private keys with your life. Verifying the `.pub` suffix before pressing send prevents almost every accident.
