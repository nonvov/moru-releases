# moru

moru(모루)는 여러 컴퓨터에서 실행하는 Claude Code·Codex 세션을 한 터미널 화면에서 함께 쓰고 관리하는 TUI입니다. 이 저장소에는 서명된 배포 파일과 설치 스크립트만 있습니다(소스는 비공개).

- 지원: macOS(Apple silicon), Windows 11 x64
- 필요한 것: Claude Code 또는 Codex CLI 설치·로그인(moru는 기존 계정을 그대로 씀). Node.js·Zig는 필요 없습니다.
- 원격 연결은 두 컴퓨터가 같은 moru 버전이어야 합니다.

## 설치

macOS 터미널:

```sh
curl -fsSL https://github.com/nonvov/moru-releases/releases/latest/download/install.sh | sh
```

Windows PowerShell(5.1 또는 7):

```powershell
irm https://github.com/nonvov/moru-releases/releases/latest/download/install.ps1 | iex
```

설치 스크립트는 배포물의 RSA 서명을 스크립트에 들어 있는 공개키(SPKI SHA-256 `ed13d996dde7…`)로 확인한 뒤에만 실행합니다. 설치가 끝나면 새 터미널을 열고 `moru --version`으로 확인합니다.

| | macOS | Windows |
| --- | --- | --- |
| 프로그램 | `~/.local/share/moru`, 실행 파일 `~/.local/bin/moru` | `%LOCALAPPDATA%\Programs\moru`, 사용자 `PATH`에 `bin` 추가 |
| 데이터 | `~/Library/Application Support/moru` | `%LOCALAPPDATA%\moru` |

## 사용

작업 폴더에서 `moru`를 실행합니다.

```sh
cd <작업 폴더>
moru
```

- `n` 새 세션: 폴더를 고르고 Claude Code 또는 Codex를 선택합니다.
- `s` 세션 열기, `c` 저장된 대화 이어 하기, `?` 전체 키, `q` 나가기
- 세션 안에서는 `Ctrl+B`를 먼저 누릅니다(`Ctrl+B ?` 전체 키, `Ctrl+B Tab` 다음 Pane, `Ctrl+B z` 분할·한 화면 전환, `Ctrl+B q` 나가기).
- 한글 입력 상태에서는 두 번째 키를 Ctrl과 함께 누르면 바로 동작합니다(`Ctrl+B Ctrl+Q`).
- 나가도 세션은 계속 실행되며, 다시 `moru`를 실행하면 이어서 봅니다.
- `moru claude` / `moru codex`는 그 에이전트를 바로 띄우고, 에이전트가 끝나면 moru도 닫힙니다.

## 업데이트

```sh
moru update --check   # 새 버전 확인만
moru update           # 받아서 서명 확인 뒤 교체(실행 중 세션이 있으면 보류)
```

업데이트 뒤에는 현재 버전과 직전 버전만 남깁니다. 등록한 원격 컴퓨터는 관리자 장비에서 `moru --node <이름> update`로 올립니다(버전이 달라도 이 요청은 통과합니다).

## 다른 컴퓨터 연결

세션이 실행되는 컴퓨터(Node)에서 연결을 열고 일회용 코드를 만든 뒤, 보는 컴퓨터(클라이언트)에서 등록합니다. 주소는 사설망이나 Tailscale 같은 VPN 주소를 씁니다. 공개 인터넷에 포트를 열지 마세요.

```sh
# Node
moru remote enable --listen <Node 주소>:7700
moru remote pair --name <클라이언트 이름> [--admin]   # 일회용 코드와 Node 지문 출력

# 클라이언트(코드는 입력할 때 물어봄, 지문은 Node 화면과 직접 비교)
moru remote add <Node 이름> <Node 주소>:7700 --fingerprint <지문>
moru remote list
moru            # 왼쪽 목록에 이 컴퓨터와 등록한 Node의 세션이 함께 보임
```

- 연결은 TLS 1.3이며 Node 인증서 지문을 고정합니다. 일회용 코드는 채팅·메일로 보내지 말고 직접 입력하세요.
- `--admin`은 원격 업데이트 같은 관리 권한을 줍니다. 필요할 때만 씁니다.
- 장비 해제는 Node에서 `moru remote devices`, `moru remote revoke <deviceId>`로 합니다.
- Windows는 처음 연결을 열 때 방화벽 창이 뜨면 개인 네트워크만 허용합니다.
- Node가 재부팅 직후 VPN 주소보다 먼저 시작하면 연결이 열리지 않을 수 있습니다. `moru remote status`로 확인하고 `moru remote enable --listen …`을 다시 실행합니다.

## 제거

```sh
moru uninstall
```

프로그램만 지우고 데이터 폴더는 남깁니다. 자동 시작은 `moru autostart --enable|--disable`로 바꿉니다.
