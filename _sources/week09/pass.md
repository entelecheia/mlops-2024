# Unix 패스워드 관리자

**`pass`와 `passage` - Unix 패스워드 관리자**

## 소개

`pass`는 Unix 기반 시스템을 위한 간단하면서도 강력한 패스워드 관리자로, GnuPG(GNU Privacy Guard)의 견고함과 보안성을 활용하여 패스워드를 암호화하고 관리합니다. `passage`는 `pass`의 포크(fork) 버전으로, 백엔드 암호화 도구로 GnuPG 대신 `age`([https://age-encryption.org](https://age-encryption.org))를 사용합니다. 이 강의 노트에서는 `pass`와 `passage`를 모두 소개하고, 설치 및 사용 방법을 설명하며, 도구에 대한 이해를 돕기 위한 예제를 제공하고, 두 도구 간의 차이점을 설명합니다.

## 설치

### `pass` 설치

Debian/Ubuntu의 경우:

```bash
sudo apt-get install pass
```

MacOS(Homebrew 사용)의 경우:

```bash
brew install pass
```

### `age`와 `rage` 설치

`age`:

```bash
go get -u github.com/FiloSottile/age/cmd/age
```

`rage`:

```bash
cargo install rage
```

### `age-plugin-yubikey` 설치

```bash
cargo install age-plugin-yubikey
```

## 초기화

### `pass` 초기화

GPG 키 생성(아직 없는 경우):

```bash
gpg --gen-key
```

키 ID로 `pass` 초기화:

```bash
pass init <GPG-key-ID>
```

### `passage` 초기화

#### 간단한 설정

디스크에 키 저장:

```bash
age-keygen >> $HOME/.passage/identities
```

#### 비밀번호로 보호된 키를 사용한 설정

저장소를 잠금 해제하기 위한 기본 비밀번호로 신원 파일 비밀번호 사용:

```bash
KEY="$(age-keygen)"
echo "$KEY" | age -p -a >> $HOME/.passage/identities
echo "$KEY" | age-keygen -y >> $HOME/.passage/store/.age-recipients
```

#### age-plugin-yubikey를 사용한 설정

age v1.1.0 또는 rage([https://github.com/str4d/rage](https://github.com/str4d/rage))와 PIV 플러그인 age-plugin-yubikey([https://github.com/str4d/age-plugin-yubikey](https://github.com/str4d/age-plugin-yubikey))가 필요합니다. `.age-recipients` 파일에 복구 옵션으로 더 많은 YubiKey 또는 age 키를 추가하세요.

```bash
age-plugin-yubikey # 대화형 설정 실행
age-plugin-yubikey --identity >> $HOME/.passage/identities
age-plugin-yubikey --list >> $HOME/.passage/store/.age-recipients
```

## 사용법

### 패스워드 추가

`pass`:

```bash
pass insert <entry-name>
```

`passage`:

```bash
passage insert <entry-name>
```

### 패스워드 생성

`pass`:

```bash
pass generate <entry-name> <password-length>
```

`passage`:

```bash
passage generate <entry-name> <password-length>
```

### 패스워드 검색

`pass`:

```bash
pass show <entry-name>
```

`passage`:

```bash
passage show <entry-name>
```

### 패스워드 업데이트

`pass`:

```bash
pass edit <entry-name>
```

`passage`:

```bash
passage edit <entry-name>
```

### 패스워드 삭제

`pass`:

```bash
pass rm <entry-name>
```

`passage`:

```bash
passage rm <entry-name>
```

## `pass`와 `passage`의 차이점

- 기본 패스워드 저장소 위치: `passage`는 `$HOME/.passage/store`, `pass`는 `$HOME/.password-store`입니다.
- 암호화 백엔드: `passage`는 `age` 암호화를 사용하고, `pass`는 GnuPG를 사용합니다.
- 복호화: `passage`는 `-i age` CLI 옵션과 함께 `$HOME/.passage/identities`에 있는 Age 신원을 사용하고, `pass`는 GPG 키를 사용합니다.
- 암호화: `passage`는 `-R age` CLI 옵션과 함께 가장 가까운 `.age-recipients` 파일을 사용하거나 `-i` 옵션과 함께 신원 파일을 사용합니다. `pass`는 GPG 수신자 키를 사용합니다.
- 확장: `passage`는 `$HOME/.passage/extensions`에, `pass`는 `$HOME/.password-store/.extensions`에 저장됩니다. 두 도구와 호환되는 확장은 `PASSAGE` 변수에 따라 전환할 수 있습니다.
- Init 명령: `passage`에는 없지만 `pass`에는 있습니다.
- 비밀 이동/복사: `passage`에서는 항상 비밀을 재암호화하지만, `pass`는 원래 암호화를 유지합니다.

## 추가 기능

### fzf와 통합

이 스크립트는 fzf([https://github.com/junegunn/fzf](https://github.com/junegunn/fzf))를 사용하여 비밀을 선택하기 위한 퍼지 검색 대화 상자를 생성하고 모든(또는 없는) 플래그와 함께 `passage`를 호출합니다.

```bash
#! /usr/bin/env bash
set -eou pipefail
PREFIX="${PASSAGE_DIR:-$HOME/.passage/store}"
FZF_DEFAULT_OPTS=""
name="$(find "$PREFIX" -type f -name '*.age' | \
  sed -e "s|$PREFIX/||" -e 's|\.age$||' | \
  fzf --height 40% --reverse --no-multi)"
passage "${@}" "$name"
```

### `pass`에서 `passage`로 마이그레이션

이 스크립트는 `pass`에서 `passage`로 비밀을 마이그레이션합니다.

```bash
#! /usr/bin/env bash
set -eou pipefail
cd "${PASSWORD_STORE_DIR:-$HOME/.password-store}"
while read -r -d "" passfile; do
  name="${passfile#./}"; name="${name%.gpg}"
  [[ -f "${PASSAGE_DIR:-$HOME/.passage/store}/$name.age" ]] && continue
  pass "$name" | passage insert -m "$name" || { passage rm "$name"; break; }
done < <(find . -path '*/.git' -prune -o -iname '*.gpg' -print0)
```

## 결론

`pass`와 `passage`는 Unix 기반 시스템을 위한 안전한 패스워드 관리자로, `pass`는 GnuPG 암호화를, `passage`는 `age` 암호화를 사용합니다. 두 도구의 차이점을 이해하고 제공된 예제를 따르면 선택한 도구로 패스워드 저장소를 설정하고 관리할 수 있습니다. fzf와의 통합과 `pass`에서 `passage`로의 마이그레이션 스크립트를 통해 쉽게 전환하고 패스워드 관리 경험을 향상시킬 수 있습니다.

이 강의 노트에서는 `pass`와 `passage`의 기본적인 사용법과 기능에 대해 다루었지만, 이들은 더 많은 고급 기능과 사용 사례를 제공합니다. 예를 들어, 두 도구 모두 Git과 통합되어 패스워드 저장소의 버전 관리와 협업을 가능하게 합니다. 또한 다양한 플랫폼 및 장치에서 액세스할 수 있도록 패스워드 저장소를 클라우드 스토리지와 동기화할 수도 있습니다.

보안은 패스워드 관리에 있어 가장 중요한 측면 중 하나입니다. `pass`와 `passage`는 검증된 암호화 알고리즘과 best practice를 사용하여 패스워드를 안전하게 보호합니다. 그러나 사용자는 강력한 마스터 패스워드를 선택하고 정기적으로 변경하며, 패스워드 저장소에 대한 액세스를 제한하는 등 보안을 유지하기 위한 적절한 조치를 취해야 합니다.

요약하면 `pass`와 `passage`는 Unix 사용자를 위한 강력하고 사용하기 쉬운 패스워드 관리 솔루션입니다. 이들은 안전하고 효율적인 방식으로 패스워드를 저장, 생성, 검색할 수 있는 기능을 제공합니다. 개인 사용자와 팀 모두 이러한 도구를 활용하여 패스워드 관리 프로세스를 간소화하고 보안을 강화할 수 있습니다. 디지털 자산을 보호하는 것이 그 어느 때보다 중요해진 만큼, `pass`와 `passage`는 모든 Unix 사용자의 도구 모음에 포함되어야 할 필수 도구입니다.

Unix 시스템 관리자로서 패스워드 관리에 대한 조직 차원의 접근 방식을 고려해 볼 수 있습니다. `pass`나 `passage`를 표준 패스워드 관리 솔루션으로 채택하고 모든 사용자가 이를 사용하도록 권장하거나 요구하는 정책을 수립할 수 있습니다. 또한 팀 구성원과 패스워드를 안전하게 공유할 수 있는 방법을 제공하여 협업을 촉진할 수 있습니다.

또한 `pass`와 `passage`를 다른 보안 도구 및 프로세스와 통합하는 방법을 모색할 수 있습니다. 예를 들어, 이러한 도구를 ID 관리 시스템, 2FA(2-factor authentication) 솔루션, 로그 모니터링 도구와 통합하여 포괄적인 보안 체계를 구축할 수 있습니다.

마지막으로 직원 교육과 인식 제고가 성공적인 패스워드 관리 전략의 핵심입니다. 사용자가 강력한 패스워드를 만들고 안전하게 관리하는 방법을 이해할 수 있도록 정기적인 교육과 워크숍을 실시하는 것이 좋습니다. 또한 패스워드 관리의 중요성과 조직의 보안에 미치는 영향을 강조하는 인식 제고 캠페인을 고려해 볼 수 있습니다.

결론적으로 Unix 시스템에서 효과적인 패스워드 관리는 기술적 솔루션(`pass`, `passage`와 같은 도구)과 인적 요소(사용자 교육 및 인식)의 조합을 통해 달성할 수 있습니다. 시스템 관리자는 이러한 측면을 모두 고려하여 조직의 특정 요구 사항에 맞는 포괄적인 패스워드 관리 전략을 수립해야 합니다. 이를 통해 디지털 자산을 보호하고, 규정을 준수하며, 사용자의 생산성과 편의성을 향상시킬 수 있을 것입니다.
