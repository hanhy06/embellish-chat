# Embellish Chat 설정 작성 가이드

이 문서는 AI가 Embellish Chat 설정을 작성하고 수정할 때 사용하는 참고 자료다. 사용자는 이 파일과 원하는 동작을 AI에게 전달하면 된다. 기존 설정의 JSON 문법 오류와 정규식 오류를 수정할 때도 사용할 수 있다.

일반 설정, 스타일 규칙, 멘션 규칙, 프리셋을 다룬다.

## 1. 설정 파일 구성과 적용

설정 파일은 서버의 `config/embellish-chat/`에 있다.

| 파일 | 역할 | 최상위 필드 |
|---|---|---|
| `config.json` | 일반 설정과 저장된 플레이어 목록 | `version`, `delimiter`, `timestamp` 등 |
| `styles.json` | 텍스트를 꾸미거나 처리하는 규칙 | `style_rules` |
| `mentions.json` | 멘션 대상과 알림을 정하는 규칙 | `mention_rules` |
| `presets.json` | 메시지 머리말, URL 허용 목록, 아이콘, 아이템 스프라이트, 색상 | `message_header`, `whitelist`, `icon`, `item`, `color` |

서버가 시작되거나 리로드할 때 기본 설정을 만든 뒤 `config.json` → `styles.json` → `mentions.json` → `presets.json` 순서로 최상위 필드를 덮어쓴다. 같은 필드가 여러 파일에 있으면 나중에 읽은 값이 사용된다. 파일별 역할에 맞춰 필드를 배치한다.

빠진 최상위 필드는 기본값을 사용한다. 이미 제공한 객체의 내부까지 기본값과 재귀적으로 합치는 방식은 아니다. 예를 들어 `color`를 직접 작성하면 그 객체가 기본 색상 객체를 대체하므로, 유지할 기본 색상도 함께 넣어야 한다. 규칙 내부에서 생략한 필드도 기본 규칙의 값으로 채워지지 않는다.

기능을 끄려고 필드를 삭제하면 기본값이 다시 사용될 수 있다. 스타일 그룹을 비우려면 해당 그룹의 규칙 배열을 `[]`로 만들고, 전체 스타일을 끄려면 `"style_rules": {}`처럼 빈 객체를 명시한다. 멘션도 같은 방식으로 처리한다.

설정 수정 후 관리자 권한으로 `/embellish-chat reload`를 실행한다. 명령어 별칭의 기본값은 `/ec`다. 비어 있지 않은 별칭은 설정을 읽을 때 등록한다. `command_alias`가 `null`, 빈 문자열, 공백뿐인 문자열이면 추가 등록을 건너뛴다. 이미 등록한 별칭을 제거하는 처리는 없으므로 기존 별칭 제거까지 반영하려면 서버를 재시작한다.

`version`은 설치된 모드가 생성한 값을 유지한다. 임의의 버전이나 예시 문자열로 바꾸지 않는다. 저장된 버전이 실행 중인 모드 버전과 다르면 새 설정을 적용하지 않고 현재 메모리의 설정을 유지한다.

읽을 수 없는 파일이나 JSON 문법 오류가 있는 파일은 경고 로그를 남기고 건너뛴다. 나머지 파일을 기본값과 합쳐 적용할 수 있으므로 리로드 성공 메시지만으로 모든 파일이 정상적으로 읽혔다고 판단할 수는 없다. 합친 설정을 자료형으로 변환하거나 검증하는 단계에서 실패하면 새 설정 전체를 적용하지 않고 현재 메모리의 설정을 유지한다.

## 2. 일반 설정: config.json

| 필드 | 기본값 | 설명 |
|---|---|---|
| `version` | 실행 중인 모드 버전 | 직접 수정하지 않는다. |
| `delimiter` | `","` | 캡처한 옵션을 여러 동작에 나눠 전달할 때 사용하는 구분 정규식이다. |
| `timestamp` | `"yyyy-MM-dd HH:mm:ss"` | `METADATA`의 시간 표시 형식이다. Java 날짜·시간 패턴을 사용한다. |
| `command_alias` | `"ec"` | `/embellish-chat`과 같은 명령어 트리를 사용하는 추가 별칭이다. |
| `url_color` | `"#0000EE"` | `URL` 스타일의 색상이다. 명시적으로 `null`을 넣으면 설정 검증에서 거부한다. |
| `team_color` | `"#FF55FF"` | `TEAM`, `PLAYER` 멘션의 기본 색상이다. 대상의 팀·표시 이름 스타일이 있으면 함께 반영한다. `null`이면 이 자동 스타일을 적용하지 않는다. |
| `notify_command_enabled` | `true` | 개인별 멘션 알림 설정 기능을 사용한다. `false`이면 명령어를 막고 저장된 개인 알림 끄기 목록도 적용하지 않는다. |
| `notify_mention_enabled` | `true` | 멘션의 화면 알림과 소리를 사용한다. 멘션 대상을 정하는 기능 자체를 끄는 설정은 아니다. |
| `require_same_channel` | `true` | Advanced Chat 채널 멘션을 보낸 사람의 현재 채널로 제한한다. |
| `disable_vanilla_chat_format` | `false` | 기본 채팅 형식을 비활성화한다. 채팅 형식을 직접 구성할 때 사용한다. |
| `banned_players` | `[]` | 일반 스타일과 멘션 처리를 건너뛸 플레이어 UUID 목록이다. 메시지 머리말은 계속 적용된다. |
| `notify_off_players` | `[]` | 개인 멘션 알림을 끈 플레이어 UUID 목록이다. |

색상 값은 `"#RRGGBB"` 형태를 사용한다. `url_color`를 생략하면 기본값을 사용하지만, 명시적인 `null`은 `url_color is missing` 검증 오류가 되어 새 설정이 적용되지 않는다. `team_color: null`은 허용되며 팀·플레이어 멘션의 자동 스타일만 끈다. 규칙의 `styles`에 지정한 색상까지 끄지는 않는다.

플레이어 목록은 이름이 아니라 UUID를 저장한다.

`delimiter`도 Java 정규식이다. 예를 들어 문자 `|`로 나누려면 JSON에는 `"delimiter": "\\|"`로 작성한다. 따옴표 안에 `"|"`만 넣으면 문자 그대로의 구분자가 되지 않는다.

## 3. 권한과 처리 순서

`style_rules`와 `mention_rules` 바로 아래의 키는 규칙 그룹인 동시에 권한 노드다.

```json
{
  "style_rules": {
    "embellish-chat.chat": [],
    "embellish-chat.admin-style": []
  }
}
```

플레이어는 권한이 허용된 그룹의 규칙만 사용한다. 기본 그룹 `embellish-chat.chat`, `embellish-chat.mention`은 기본적으로 허용된다. 새 그룹을 설정에 추가하는 것과 플레이어에게 그 권한을 부여하는 것은 별개다. 서버의 권한 관리 방식에 맞춰 적용 대상을 확인한다.

한 메시지에서는 먼저 멘션을 찾고 멘션 스타일을 적용한 다음 일반 스타일 규칙을 순서대로 적용한다. 메시지 머리말은 그 뒤에 붙는다. 따라서 뒤의 일반 스타일이 멘션에 설정한 색상이나 클릭·호버 동작을 바꿀 수 있다.

일반 스타일 규칙은 앞선 규칙의 처리 결과에 다음 규칙이 적용된다. 첫 번째로 일치한 규칙 하나만 실행하는 방식이 아니다. 한 규칙도 메시지 안에서 여러 번 일치할 수 있다.

멘션 규칙은 같은 시작 위치에서 먼저 잡은 규칙이 우선한다. `@admin`, `@team(...)` 같은 구체적인 패턴은 일반 플레이어 패턴보다 앞에 둔다.

클릭·호버 동작을 직접 지정하는 규칙은 기본 `METADATA` 규칙 뒤에 넣는다. `METADATA`가 나중에 실행되면 사용자 정의 클릭·호버 동작을 수신 시각 표시와 복사 동작으로 덮어쓸 수 있다.

이때 새 규칙을 배열 맨 아래에 넣는 것이 아니라, `METADATA` 뒤이면서 `SHOW_ITEM`, `SHOW_INVENTORY`, `SHOW_ENDER_CHEST`, `ICON_PRESET` 규칙 앞에 넣는다. 기본 설정에서는 `METADATA`와 첫 번째 `SHOW_ITEM` 규칙 사이가 이 위치다. 아이템·인벤토리·아이콘을 표시용 컴포넌트로 바꾼 뒤 일반 텍스트 규칙을 적용하면 같은 메시지의 표시나 동작에 문제가 생길 수 있다.

차단 전에 실행해야 할 동작이 있다면 `BLOCK`보다 앞에 배치한다.

## 4. JSON과 정규식 작성 규칙

정규식은 Java `Pattern` 문법을 사용한다. JavaScript의 `/패턴/g`처럼 슬래시로 감싸거나 뒤에 플래그 문자를 붙이지 않는다. 필요하면 `(?i)` 같은 Java 인라인 플래그를 사용한다.

JSON 문자열에서는 정규식의 역슬래시를 한 번 더 이스케이프해야 한다.

| 의도 | 실제 정규식 | JSON의 문자열 값 |
|---|---|---|
| 숫자 한 개 | `\d` | `"\\d"` |
| 공백 | `\s` | `"\\s"` |
| 문자 `[` | `\[` | `"\\["` |
| 문자 `*` | `\*` | `"\\*"` |
| 단어 경계 | `\b` | `"\\b"` |

JSON에서 `"\b"`는 단어 경계 정규식이 아니라 백스페이스 문자다. `"\d"`, `"\s"`처럼 JSON에서 허용하지 않는 이스케이프도 쓰지 않는다.

캡처 그룹 번호는 여는 괄호 순서로 정해진다. 그룹을 늘리지 않으려면 `(?:...)`를 사용한다. 스타일은 그룹 1과 2, 멘션은 그룹 1을 읽으므로, 패턴을 고치면서 이 번호의 의미가 바뀌지 않게 한다.

`(.+)`는 넓은 범위를 잡는다. 닫는 구분자까지만 잡아야 한다면 `(.+?)`나 `[^]]+` 같은 패턴을 검토한다. 실제 구분자와 허용할 문자에 맞춰 선택하고 모든 패턴을 일괄적으로 바꾸지 않는다.

기본적으로 빈 텍스트까지 매치하는 패턴은 피한다. 문장 전체에 적용할 때는 부분 일치와 전체 일치를 구분하고, 필요한 경우 `^`, `$`를 사용한다.

JSON 문자열 안의 큰따옴표는 `\"`, 줄바꿈은 `\n`으로 작성한다. `JSON`, `DISCORD_JSON`처럼 문자열 안에 JSON을 넣을 때는 바깥 설정 JSON과 내부 JSON을 각각 확인한다. 정규식의 `$1`이나 `$2`가 `preset`에서 자동으로 치환된다고 가정하지 않는다.

## 5. 스타일 규칙: styles.json

다음은 굵게 표시하는 규칙 하나만 담은 파일 예시다. 전체 기본 설정이 아니다. 기존 설정을 수정할 때는 필요한 규칙을 기존 그룹의 배열에 추가하거나 해당 규칙만 교체한다.

```json
{
  "style_rules": {
    "embellish-chat.chat": [
      {
        "pattern": "\\*\\*(.+?)\\*\\*()",
        "comment": "**텍스트**: 굵게 표시",
        "styles": [
          {
            "styleType": "BOLD",
            "preset": ""
          }
        ]
      }
    ]
  }
}
```

| 필드 | 의미 |
|---|---|
| `pattern` | 그룹 1과 2가 있는 정규식이다. 그룹 1은 표시하고 처리할 텍스트, 그룹 2는 옵션이다. |
| `comment` | `/embellish-chat help style`에 표시할 설명이다. 채팅에 적용할 스타일 자체가 아니다. |
| `styles` | 실행할 스타일 동작 배열이다. 배열 순서대로 실행한다. `[]`이면 스타일 동작 없이 그룹 1만 남긴다. |
| `styleType` | 아래 목록의 타입 이름이다. 대소문자를 그대로 사용한다. |
| `preset` | 고정 옵션이다. 빈 문자열이면 캡처한 옵션 중 해당 동작 순서의 값을 사용한다. 생략하거나 `null`이면 설정 검증에서 거부한다. |

정규식 전체가 일치한 부분 중 그룹 1만 결과 텍스트로 남는다. `**텍스트**`에서 별표가 사라지는 이유도 별표가 그룹 1 바깥에 있기 때문이다. 문법 표식을 남겨야 한다면 그룹 1에 포함해야 한다.

옵션을 쓰지 않는 스타일도 그룹 2는 있어야 한다. 위 예시의 마지막 `()`가 빈 옵션 그룹이다. 캡처 그룹을 더 만들 수는 있지만 특별한 이유가 없으면 두 개로 유지한다.

`comment`는 생략하거나 `null`로 둘 수 있다. 이 경우 도움말에서 해당 규칙의 설명을 건너뛰며 규칙 자체는 계속 적용된다.

### 여러 스타일과 옵션

그룹 2를 `delimiter`로 나눈 뒤 첫 번째 값은 첫 번째 스타일, 두 번째 값은 두 번째 스타일에 대응한다. 고정 `preset`이 있는 스타일도 배열의 자리 하나를 차지한다. 빈 `preset`인 스타일끼리만 순서를 세는 방식이 아니다.

다음 객체는 기존 스타일 그룹의 배열에 넣을 규칙 예시다.

```json
{
  "pattern": "\\[([^\\]]+?)]<([^>]+)>",
  "comment": "[텍스트]<색상,설명>: 색상과 호버 설명",
  "styles": [
    { "styleType": "COLOR_HEX", "preset": "" },
    { "styleType": "HOVER_TEXT", "preset": "" }
  ]
}
```

기본 구분자 `,`에서 `[안내]<#00AAFF,서버 안내입니다>`를 입력하면 그룹 1은 `안내`, 그룹 2는 `#00AAFF,서버 안내입니다`다. 첫 번째 스타일에는 `#00AAFF`, 두 번째 스타일에는 `서버 안내입니다`가 전달된다.

이 패턴은 기본 색상 문법과 겹친다. 실제 설정에 추가할 때는 기존 색상 규칙보다 앞에 두거나 별도의 문법을 정한다. 옵션 자체에 쉼표가 필요한 경우에도 캡처 옵션 분할을 고려한다. JSON이나 명령어처럼 쉼표를 포함할 수 있는 고정 값은 `preset`에 넣으면 이 분할을 거치지 않는다.

### 스타일 타입 전체

옵션이 없는 타입은 `preset`에 빈 문자열을 쓸 수 있다. 표의 옵션은 최종적으로 각 스타일에 전달되는 값이다.

| 타입 | 옵션 | 동작 |
|---|---|---|
| `COLOR_HEX` | `#00AAFF` 같은 색상 | 텍스트 색상을 적용한다. |
| `COLOR_RAINBOW` | 채도, 예: `0.7` | 글자별 무지개 색상을 적용한다. 숫자로 해석할 수 없으면 `0.7`을 사용한다. 채도는 보통 0~1 범위로 작성한다. |
| `COLOR_GRADIENT` | `#FF5555#5555FF`처럼 두 개 이상의 색상 | 글자에 그라데이션을 적용한다. 유효한 색상이 없으면 변경하지 않는다. 색상이 한 개이거나 텍스트 길이가 색상 개수보다 짧으면 첫 색상으로 표시한다. |
| `COLOR_PRESET` | `color`의 키, 예: `light purple` | 저장된 색상을 적용한다. 키가 없으면 흰색을 사용한다. |
| `COLOR_SHADOW` | `#RRGGBB` | 지원하는 버전에서 텍스트 그림자 색상을 적용한다. |
| `COLOR_TEAM` | 없음 | 보낸 사람의 스코어보드 팀 색상을 적용한다. |
| `BOLD` | 없음 | 굵게 표시한다. |
| `ITALIC` | 없음 | 기울임꼴로 표시한다. |
| `UNDERLINE` | 없음 | 밑줄을 긋는다. |
| `STRIKETHROUGH` | 없음 | 취소선을 긋는다. |
| `OBFUSCATED` | 없음 | 난독화하고 호버 시 원래 텍스트를 보여 준다. |
| `FONT` | 폰트 ID, 예: `minecraft:alt` | 지정한 폰트를 적용한다. ID 문법이 잘못되면 변경하지 않는다. |
| `CLEAR` | 없음 | 현재 텍스트의 스타일과 상호작용을 제거하고 일반 텍스트로 만든다. |
| `CLICK_COMMAND_RUN` | 명령어 | 클릭할 때 명령어를 실행한다. |
| `CLICK_COMMAND_SUGGEST` | 명령어 | 클릭할 때 채팅 입력창에 명령어를 넣는다. |
| `CLICK_COPY` | 복사할 문자열 | 클릭할 때 클립보드에 복사한다. |
| `HOVER_TEXT` | 설명 텍스트 | 마우스를 올릴 때 설명을 표시한다. |
| `HOVER_ITEM` | 인벤토리 슬롯 번호 또는 빈 문자열 | 해당 아이템 툴팁을 표시한다. 빈 값이면 주로 든 아이템을 사용한다. 번호 해석에 실패하거나 0~42 범위를 벗어나면 변경하지 않고, 선택한 슬롯이 비어 있으면 메시지를 차단한다. |
| `URL` | URL | 허용 목록을 통과한 URL을 열고 `url_color`를 적용한다. 호스트가 없거나 허용되지 않으면 변경하지 않는다. |
| `METADATA` | 없음 | 수신 시각을 호버에 표시하고 클릭하면 시각과 텍스트를 복사한다. |
| `UPPER` | 없음 | 대문자로 바꾼다. |
| `LOWER` | 없음 | 소문자로 바꾼다. |
| `CAPITALIZE` | 없음 | 첫 글자를 대문자로 바꾼다. |
| `REPLACE` | 바꿀 텍스트 | 선택한 텍스트를 교체한다. |
| `MASK` | 반복할 문자열 | 원래 텍스트 길이만큼 옵션 문자열을 반복한다. |
| `PREFIX` | 앞에 붙일 텍스트 | 선택한 텍스트 앞에 붙인다. |
| `SUFFIX` | 뒤에 붙일 텍스트 | 선택한 텍스트 뒤에 붙인다. |
| `SHOW_ITEM` | 빈 문자열 또는 `only_name` | 주로 든 아이템을 공유한다. `only_name`이면 아이콘 대신 이름을 표시한다. 클릭하면 조회 전용 화면을 연다. 손에 아이템이 없으면 메시지를 차단한다. |
| `SHOW_INVENTORY` | 없음 | 보낸 사람의 인벤토리를 공유하는 조회 전용 화면을 연결한다. |
| `SHOW_ENDER_CHEST` | 없음 | 보낸 사람의 엔더 상자를 공유하는 조회 전용 화면을 연결한다. |
| `ICON_PRESET` | `icon`의 키, 예: `heart` | 등록된 아틀라스 스프라이트를 표시한다. 키가 없거나 값이 `null`이면 변경하지 않는다. |
| `JSON` | Minecraft 텍스트 컴포넌트 JSON 문자열 | 텍스트 컴포넌트를 직접 구성한다. 대상 Minecraft 버전에 맞는 형식을 사용한다. |
| `DISCORD_JSON` | `웹훅 URL;JSON 본문` | Discord 웹훅에 JSON을 비동기로 보낸다. 첫 번째 세미콜론이 URL과 본문을 나눈다. 세미콜론이 없으면 전송하지 않는다. 전송 실패는 로그에 기록한다. |
| `COMMAND_RUN` | 명령어 | 메시지를 처리할 때 보낸 사람의 명령어 소스로 실행한다. |
| `LOG` | 선택적인 로그 옵션 | 선택한 텍스트, 옵션, 보낸 사람을 서버 로그에 기록한다. |
| `BUBBLE` | 없음 | 보낸 사람 머리 위에 말풍선을 표시한다. 원래 채팅도 숨기려면 뒤에 `BLOCK`을 사용한다. |
| `BLOCK` | 차단 안내 또는 빈 문자열 | 메시지 전송을 중단한다. 값이 있으면 보낸 사람에게 안내한다. |

명령어나 외부 전송을 구성할 때는 사용자가 의도한 값을 `preset`에 고정한다. 특히 `COMMAND_RUN`, `DISCORD_JSON`, `JSON`을 채팅에서 캡처한 임의의 값으로 구성하지 않는다. 고정 차단 안내가 필요할 때도 `BLOCK`의 `preset`에 직접 쓴다. 안내 없이 차단할 때는 빈 문자열을 사용한다.

다음은 `JSON` 동작 하나의 예시다. 안쪽 컴포넌트 JSON을 문자열로 넣으므로 큰따옴표를 이스케이프한다.

```json
{
  "styleType": "JSON",
  "preset": "{\"text\":\"서버 안내\",\"color\":\"gold\"}"
}
```

이 객체는 규칙 전체가 아니라 규칙의 `styles` 배열에 들어갈 동작이다. `DISCORD_JSON`도 같은 방식으로 JSON 본문을 문자열 안에 넣고, 그 앞에 실제 웹훅 URL과 세미콜론을 붙인다.

## 6. 멘션 규칙: mentions.json

다음은 `@here` 규칙 하나만 담은 파일 예시다. 기존 멘션을 유지하려면 기존 파일의 해당 배열에 규칙을 넣는다.

```json
{
  "mention_rules": {
    "embellish-chat.mention": [
      {
        "pattern": "@here()",
        "comment": "@here: 같은 월드의 반경 64블록 안에 있는 플레이어에게 알림",
        "title": "%player:displayname%님이 멘션했습니다",
        "sound": {
          "id": "minecraft:entity.experience_orb.pickup",
          "category": "UI",
          "volume": 1.0,
          "pitch": 1.75
        },
        "cooldown": 0,
        "onlyTarget": false,
        "mentions": [
          { "mentionType": "INSIDE", "preset": "64" }
        ],
        "styles": [
          { "styleType": "BOLD", "preset": "" },
          { "styleType": "COLOR_PRESET", "preset": "light purple" }
        ]
      }
    ]
  }
}
```

| 필드 | 의미 |
|---|---|
| `pattern` | 그룹 1을 옵션으로 사용하는 정규식이다. 캡처 그룹이 최소 한 개 있어야 한다. |
| `comment` | `/embellish-chat help mention`에 표시할 설명이다. |
| `title` | 대상의 화면에 표시할 알림이다. 빈 문자열이면 표시할 텍스트가 없다. 생략하거나 `null`이면 알림 처리 중 오류가 날 수 있다. |
| `sound` | `id`, `category`, `volume`, `pitch`를 가진 객체다. 소리를 없애려면 `null`을 사용한다. |
| `cooldown` | 보낸 사람별로 이 규칙에 적용할 대기 시간(초)이다. `0`이면 사용하지 않는다. |
| `onlyTarget` | `true`이면 메시지를 멘션 대상에게만 전달한다. `false`이면 일반 전송을 유지한다. |
| `mentions` | 대상을 정하는 동작 배열이다. 비어 있으면 안 된다. |
| `styles` | 멘션 전체에 적용할 스타일 배열이다. 추가 스타일이 필요 없으면 `[]`를 사용한다. |

멘션은 그룹 1을 옵션으로 사용한다. 예를 들어 JSON 패턴 `"@team\\(([^)]+)\\)"`에 `@team(red)`가 일치하면 그룹 1은 `red`다. 대상 이름을 고정하려면 `preset`에 값을 넣고, 캡처 옵션이 필요 없다면 `@admin()`처럼 빈 그룹을 둔다.

`mentions`도 캡처 옵션을 `delimiter`로 나눈 뒤 동작의 배열 순서에 맞춰 사용한다. 여러 대상 동작을 넣으면 각 동작의 대상 중 공통으로 포함된 플레이어, 즉 교집합을 선택한다. 예를 들어 `INSIDE`와 `TEAM`을 함께 쓰면 반경 안에 있는 해당 팀 플레이어만 선택한다.

여러 대상 동작을 조합할 때 자동 스타일은 첫 번째 대상 동작의 결과를 사용한다. `INSIDE` 다음에 `TEAM`을 넣으면 팀으로 대상을 제한하지만, 두 번째 동작의 팀 스타일이 자동으로 추가되는 것은 아니다. 필요한 스타일은 규칙의 `styles`에 지정한다.

멘션의 `styles`에는 멘션 그룹 1의 옵션이 전달되지 않는다. 색상, 호버 설명, 명령어 등 값이 필요한 스타일은 `preset`에 직접 넣는다. `BOLD`처럼 값이 필요 없는 스타일은 빈 문자열을 사용한다. 추가 스타일이 없어도 `TEAM`, `PLAYER` 대상 처리에서 자동 스타일이 붙을 수 있다.

알림을 꺼도 대상 지정과 `onlyTarget` 동작까지 꺼지는 것은 아니다. `onlyTarget`이 활성화된 멘션이 여러 개 있으면 각 멘션의 수신 대상을 합쳐서 메시지를 전달한다. 보낸 사람도 대상에 포함되어야 메시지를 받으므로, 비공개 채팅을 만들 때 송신자 포함 여부를 확인한다.

쿨다운 중에 해당 패턴이 다시 일치하면 메시지 자체가 차단될 수 있다. 단순히 알림 소리만 생략하는 설정으로 설명하지 않는다.

`sound: null`은 소리만 끈다. `sound` 객체를 유지하면서 `id`만 `null`로 넣으면 사운드 ID를 읽는 단계에서 거부한다. `title`, `sound.category`처럼 설정 검증에서 별도로 확인하지 않는 필드도 있어, 로드 성공과 알림 처리 성공은 구분해야 한다.

### 멘션 타입 전체

| 타입 | 옵션 | 대상 |
|---|---|---|
| `PLAYER` | 플레이어 이름 | 해당 플레이어다. Styled Nicknames 연동이 있으면 닉네임도 조회한다. 실제로 닉네임이 패턴에 잡히는지는 별도로 확인한다. |
| `TEAM` | 스코어보드 팀 이름 | 해당 팀의 온라인 플레이어다. |
| `INSIDE` | 반경, 예: `64` | 보낸 사람과 같은 월드에서 반경 안에 있는 플레이어다. |
| `EVERYONE` | 없음 | 온라인 플레이어 전체다. |
| `WORLD` | 월드 ID, 예: `minecraft:overworld` | 해당 월드에 있는 플레이어다. |
| `LUCK_PERMS_GROUP` | LuckPerms 그룹 이름 | 해당 그룹의 플레이어다. LuckPerms가 필요하다. |
| `ADVANCED_CHAT_CHANNEL` | Advanced Chat 채널 이름 | 해당 채널의 플레이어다. Advanced Chat이 필요하며 `require_same_channel` 설정도 적용된다. |
| `PERMISSION` | 권한 노드 | 그 권한이 있는 온라인 플레이어다. |
| `CUSTOM` | 플레이어 선택자, 예: `@a[distance=..32]` | 보낸 사람의 명령어 컨텍스트를 기준으로 선택한 플레이어다. |

연동 모드가 없으면 `LUCK_PERMS_GROUP`, `ADVANCED_CHAT_CHANNEL` 대상은 비어 있다.

### 응용 예시: 관리자 호출과 Discord 알림

`embellish-chat.example`의 `@admin`은 멘션과 여러 스타일을 조합하는 방법을 보여 주는 예제다. LuckPerms의 `admin` 그룹을 알림 대상으로 정하고, 호출 문구에 그라데이션과 이동 명령을 붙이며, 같은 호출을 Discord에도 보낸다.

아래는 이 조합을 단순화한 규칙 객체다. `WEBHOOK_ID`, `WEBHOOK_TOKEN`은 실제 웹훅 URL에 맞게 바꿀 자리다.

```json
{
  "pattern": "@admin()",
  "comment": "@admin: 관리자 호출, 클릭 이동, Discord 알림",
  "title": "%player:displayname%님의 관리자 호출",
  "sound": {
    "id": "minecraft:entity.experience_orb.pickup",
    "category": "UI",
    "volume": 1.0,
    "pitch": 1.75
  },
  "cooldown": 30,
  "onlyTarget": false,
  "mentions": [
    { "mentionType": "LUCK_PERMS_GROUP", "preset": "admin" }
  ],
  "styles": [
    { "styleType": "BOLD", "preset": "" },
    { "styleType": "COLOR_GRADIENT", "preset": "#FF5555#C77DFF" },
    {
      "styleType": "CLICK_COMMAND_RUN",
      "preset": "execute in %world:id% run tp %player:pos_x% %player:pos_y% %player:pos_z%"
    },
    {
      "styleType": "DISCORD_JSON",
      "preset": "https://discord.com/api/webhooks/WEBHOOK_ID/WEBHOOK_TOKEN;{\"content\":\"%player:name_unformatted% 관리자 호출: %embellish-chat:content%\\n위치: %world:id% %player:pos_x% %player:pos_y% %player:pos_z%\"}"
    }
  ]
}
```

`@admin()`의 빈 캡처 그룹은 채팅에서 추가 옵션을 받지 않는다는 뜻이다. 대상 그룹과 각 스타일 옵션은 `preset`으로 고정한다. 이동 명령의 월드·좌표 플레이스홀더는 호출한 사람을 기준으로 채워지고, 클릭한 플레이어가 자신의 권한으로 그 명령을 실행한다. Discord 전송은 클릭할 때가 아니라 메시지를 처리할 때 실행된다. `onlyTarget`이 `false`이므로 채팅 메시지 자체는 일반 방식으로 전송된다.

| 바꿀 부분 | 응용 방법 |
|---|---|
| `pattern` | `@support()`처럼 다른 호출 문구로 바꾼다. |
| `mentions` | 지원 담당 그룹으로 바꾸거나, `PERMISSION`과 권한 노드를 사용해 권한을 가진 플레이어를 호출한다. |
| `title`, `sound` | 호출 목적에 맞는 화면 알림과 소리를 정한다. |
| `CLICK_COMMAND_RUN` | 이동 대신 호출자에게 귓속말을 보내는 명령 등으로 바꾼다. |
| `DISCORD_JSON` | 원본 채팅·좌표 외에 필요한 플레이스홀더를 넣거나 Discord embed 형식으로 구성한다. 게임 안에서만 알리려면 이 동작을 뺀다. |
| `cooldown` | 같은 플레이어의 반복 호출 간격을 조절한다. |

### 멘션 예시: 주변의 같은 팀에 알림

아래 객체는 `@red-here`를 입력하면 반경 64블록 안에 있는 `red` 팀에만 알림을 보낸다. `onlyTarget`이 `false`이므로 채팅 메시지 자체는 일반 방식으로 전송된다.

```json
{
  "pattern": "@red-here()",
  "comment": "@red-here: 근처 red 팀에 알림",
  "title": "%player:displayname%님의 팀 호출",
  "sound": null,
  "cooldown": 10,
  "onlyTarget": false,
  "mentions": [
    { "mentionType": "INSIDE", "preset": "64" },
    { "mentionType": "TEAM", "preset": "red" }
  ],
  "styles": [
    { "styleType": "BOLD", "preset": "" },
    { "styleType": "COLOR_HEX", "preset": "#FF5555" }
  ]
}
```

이 규칙은 일반 플레이어 멘션 규칙보다 앞에 배치한다.

## 7. 프리셋: presets.json

### 메시지 머리말

`message_header`의 키는 권한 노드, 값은 머리말 텍스트다. 여러 키의 권한이 일치하면 설정 순서에서 마지막으로 일치한 머리말을 사용한다. 모두 이어 붙이는 방식이 아니다.

```json
{
  "message_header": {
    "chat.header.default": "<b>[%player:displayname%]</b> "
  }
}
```

위 코드는 `presets.json`의 머리말 부분 예시다. 기본 채팅 형식과 머리말이 중복되지 않게 확인한다. 채팅 형식을 직접 구성할 때는 `disable_vanilla_chat_format`을 함께 검토한다.

텍스트는 `<b>텍스트</b>`, `<red>텍스트</red>` 같은 태그와 지원되는 플레이스홀더를 사용할 수 있다. 머리말 값에 텍스트 컴포넌트 JSON을 쓸 때도 바깥 값은 JSON 문자열로 작성한다. 객체를 직접 넣지 않는다.

### URL 허용 목록

`whitelist`는 URL의 호스트 이름을 검사한다. 빈 배열이면 모든 호스트를 허용한다. `"example.com"`을 넣으면 해당 도메인과 그 하위 도메인을 허용한다. 전체 URL이나 경로 대신 도메인을 적는다.

```json
{
  "whitelist": ["example.com", "modrinth.com"]
}
```

### 색상

`color`는 이름에서 색상으로 연결되는 객체다. `COLOR_PRESET`의 옵션과 키가 일치해야 한다. 기본 색상 이름에는 공백이 들어가는 경우가 있으므로 임의로 밑줄로 바꾸지 않는다.

| 기본 키 | 색상 | 기본 키 | 색상 |
|---|---|---|---|
| `black` | `#000000` | `dark gray` | `#555555` |
| `dark blue` | `#0000AA` | `blue` | `#5555FF` |
| `dark green` | `#00AA00` | `green` | `#55FF55` |
| `dark aqua` | `#00AAAA` | `aqua` | `#55FFFF` |
| `dark red` | `#AA0000` | `red` | `#FF5555` |
| `dark purple` | `#AA00AA` | `light purple` | `#FF55FF` |
| `gold` | `#FFAA00` | `yellow` | `#FFFF55` |
| `gray` | `#AAAAAA` | `white` | `#FFFFFF` |

### 아이콘과 아이템 스프라이트

`icon`은 `ICON_PRESET`과 `/embellish-chat help icon`에 사용한다. 각 값은 `atlas`, `sprite`를 가진 객체다. 다음은 두 프리셋의 부분 예시다.

```json
{
  "icon": {
    "heart": {
      "atlas": "minecraft:gui",
      "sprite": "minecraft:hud/heart/full"
    }
  },
  "item": {
    "minecraft:clock": {
      "atlas": "minecraft:items",
      "sprite": "minecraft:item/clock_00"
    }
  }
}
```

`item`은 `SHOW_ITEM`의 스프라이트를 지정할 때 사용한다. 현재 구현은 아이템의 모델 ID를 키로 조회한다. 기본 아이템에서는 아래처럼 아이템 ID와 같은 형태의 키를 사용한다. 모드 아이템에서 모델 ID가 다르면 실제 모델 ID를 확인한다.

기본 아이콘은 다음과 같다.

| 키 | atlas | sprite |
|---|---|---|
| `fire` | `minecraft:blocks` | `minecraft:block/campfire_fire` |
| `lava` | `minecraft:blocks` | `minecraft:block/lava_still` |
| `food` | `minecraft:items` | `minecraft:item/cooked_beef` |
| `hunger` | `minecraft:gui` | `minecraft:hud/food_half` |
| `heart` | `minecraft:gui` | `minecraft:hud/heart/full` |
| `love` | `minecraft:gui` | `minecraft:mob_effect/health_boost` |
| `flower` | `minecraft:blocks` | `minecraft:block/poppy` |
| `tree` | `minecraft:gui` | `minecraft:toast/tree` |
| `yes` | `minecraft:gui` | `minecraft:container/beacon/confirm` |
| `no` | `minecraft:gui` | `minecraft:container/beacon/cancel` |
| `move` | `minecraft:gui` | `minecraft:mob_effect/wind_charged` |
| `emerald` | `minecraft:items` | `minecraft:item/emerald` |
| `diamond` | `minecraft:items` | `minecraft:item/diamond` |
| `star` | `minecraft:items` | `minecraft:item/nether_star` |
| `time` | `minecraft:items` | `minecraft:item/clock_54` |
| `note` | `minecraft:items` | `minecraft:item/paper` |
| `totem` | `minecraft:items` | `minecraft:item/totem_of_undying` |
| `music` | `minecraft:particles` | `minecraft:note` |
| `mine` | `minecraft:items` | `minecraft:item/diamond_pickaxe` |
| `luck` | `minecraft:gui` | `minecraft:mob_effect/luck` |
| `poison` | `minecraft:gui` | `minecraft:mob_effect/poison` |
| `fight` | `minecraft:gui` | `minecraft:mob_effect/raid_omen` |
| `world` | `minecraft:gui` | `minecraft:icon/link` |
| `news` | `minecraft:gui` | `minecraft:icon/news` |
| `search` | `minecraft:gui` | `minecraft:icon/search` |
| `tv` | `minecraft:gui` | `minecraft:icon/video_link` |

기본 `item` 프리셋은 모두 `minecraft:items` 아틀라스를 사용한다.

| 키 | sprite |
|---|---|
| `minecraft:clock` | `minecraft:item/clock_00` |
| `minecraft:compass` | `minecraft:item/compass_20` |
| `minecraft:recovery_compass` | `minecraft:item/recovery_compass_20` |
| `minecraft:crossbow` | `minecraft:item/crossbow_arrow` |
| `minecraft:tipped_arrow` | `minecraft:item/arrow` |
| `minecraft:enchanted_golden_apple` | `minecraft:item/golden_apple` |

## 8. 기본으로 제공되는 채팅 문법

기본 스타일 그룹은 `embellish-chat.chat`이다. 아래는 `Config.java`의 기본 규칙 순서다.

| 순서 | 입력 문법 | 스타일 | 옵션 |
|---|---|---|---|
| 1 | `[텍스트](https://example.com)` | `URL` | URL |
| 2 | `https://example.com` | `URL` | URL 자체 |
| 3 | `[텍스트]{minecraft:alt}` | `FONT` | 폰트 ID |
| 4 | `[텍스트]<#00AAFF>` | `COLOR_HEX` | 색상 |
| 5 | `[텍스트]<#FF5555#5555FF>` | `COLOR_GRADIENT` | 색상 목록 |
| 6 | `[텍스트]<light purple>` | `COLOR_PRESET` | 색상 이름 |
| 7 | `**텍스트**` | `BOLD` | 없음 |
| 8 | `__텍스트__` | `UNDERLINE` | 없음 |
| 9 | `_텍스트_` | `ITALIC` | 없음 |
| 10 | `~~텍스트~~` | `STRIKETHROUGH` | 없음 |
| 11 | `\|\|텍스트\|\|` | `OBFUSCATED` | 없음 |
| 12 | `[텍스트]<RAINBOW>` | `COLOR_RAINBOW` | 고정 채도 `0.7` |
| 13 | 전체 텍스트 | `METADATA` | 없음 |
| 14 | `[i]` | `SHOW_ITEM` | 빈 문자열 |
| 15 | `[inv]` | `SHOW_INVENTORY` | 없음 |
| 16 | `[end]` | `SHOW_ENDER_CHEST` | 없음 |
| 17 | `:heart:` 등 `:이름:` | `ICON_PRESET` | 아이콘 이름 |

기본 링크 정규식은 `https://`만 잡으며 `http://`는 자동 감지하지 않는다. 색상 이름의 기본 패턴은 영문 소문자와 공백을 받으므로, 한글·숫자·대문자가 포함된 프리셋 이름을 새로 만들면 패턴도 함께 검토한다.

기본 멘션 그룹은 `embellish-chat.mention`이다. 기본 순서는 아래와 같다.

| 순서 | 문법 | 타입 | 옵션 |
|---|---|---|---|
| 1 | `@here` | `INSIDE` | 고정 반경 `64` |
| 2 | `@everyone` | `EVERYONE` | 없음 |
| 3 | `@team(red)` | `TEAM` | `red` |
| 4 | `@group(admin)` | `LUCK_PERMS_GROUP` | `admin` |
| 5 | `@world(minecraft:overworld)` | `WORLD` | `minecraft:overworld` |
| 6 | `@Steve` | `PLAYER` | `Steve` |

기본 플레이어 패턴의 JSON 문자열은 `"@([A-Za-z0-9_]{1,16})(?=\\b|\\s|$)"`다. Styled Nicknames가 있어도 이 패턴 밖의 닉네임이 자동으로 매치되는 것은 아니다.

기본 멘션은 쿨다운 `0`, `onlyTarget: false`, 알림 `%player:displayname% mentioned you`를 사용한다. 소리는 `minecraft:entity.experience_orb.pickup`, 카테고리 `UI`, 볼륨 `1.0`, 피치 `1.75`다.

기본 멘션 스타일은 모두 `BOLD`를 포함한다. `@here`, `@everyone`, `@group`, `@world`에는 `COLOR_PRESET`의 `light purple`도 지정된다. `@team`, `@Player`에는 대상 처리에서 얻은 자동 스타일을 사용한다.

`ADVANCED_CHAT_CHANNEL`, `PERMISSION`, `CUSTOM` 타입이 존재해도 해당 문법의 규칙이 기본 설정에 등록된 것은 아니다. 예를 들어 `@channel(staff)`를 쓰려면 그 패턴과 동작을 가진 규칙을 추가해야 한다.

기본 `message_header`, `whitelist`, `banned_players`, `notify_off_players`는 비어 있다. 기본 `icon`, `item`, `color`는 앞의 표에 있는 값을 가진다.

## 9. 플레이스홀더와 연동

텍스트 설정과 스타일 옵션은 지원되는 위치에서 Text Placeholder API의 플레이스홀더와 태그를 사용할 수 있다. 멘션 알림의 플레이어 컨텍스트는 알림을 받는 사람이 아니라 보낸 사람이다.

| 플레이스홀더 | 용도 |
|---|---|
| `%player:displayname%` | 보낸 사람의 표시 이름 |
| `%player:name_unformatted%` | 서식 없는 플레이어 이름 |
| `%player:uuid%` | 플레이어 UUID |
| `%player:pos_x%`, `%player:pos_y%`, `%player:pos_z%` | 보낸 사람의 좌표 |
| `%world:id%` | 보낸 사람의 월드 ID |
| `%embellish-chat:content%` | 파싱 전 원본 채팅 내용 |
| `%embellish-chat:prefix%` | LuckPerms 접두사. LuckPerms가 필요하다. |
| `%embellish-chat:suffix%` | LuckPerms 접미사. LuckPerms가 필요하다. |

플레이스홀더는 값이 사용되는 시점과 위치에 따라 컨텍스트가 다를 수 있다. `%embellish-chat:content%`는 메시지 처리 중 원본 채팅을 참조한다.

- Fabric Permissions API와 Text Placeholder API는 내장되어 있다.
- LuckPerms 그룹 멘션은 LuckPerms가 필요하다.
- Advanced Chat 채널 멘션은 Advanced Chat이 필요하다.
- Styled Chat이 설치되어 있으면 채팅 스타일 처리에서 우선할 수 있다. 멘션 알림은 계속 사용할 수 있지만 스타일이 기대대로 보이는지는 구분해서 확인한다.
- Geyser로 접속하는 Bedrock 클라이언트는 일부 클릭·호버 효과를 Java 클라이언트와 다르게 표시할 수 있다.
- 핵심 기능은 서버에서 동작한다. 선택적인 클라이언트 멘션 자동완성을 위해서는 클라이언트 설치가 필요하다.

## 10. 오류 수정과 검증

### 설정 로드 시 거부되는 값

다음 최상위 필드는 명시적인 `null`을 허용하지 않는다.

- `version`, `style_rules`, `mention_rules`
- `message_header`, `whitelist`, `icon`, `item`, `color`
- `delimiter`, `timestamp`, `url_color`
- `banned_players`, `notify_off_players`

`delimiter`가 유효한 정규식이 아니거나 `timestamp`가 유효한 Java 날짜·시간 패턴이 아니어도 설정 검증에서 거부한다.

스타일·멘션 그룹의 규칙 배열, 각 규칙, `pattern`, `styles`, 각 동작과 타입, 각 동작의 `preset`도 `null`을 허용하지 않는다. 스타일은 최소 두 개, 멘션은 최소 한 개의 캡처 그룹이 필요하다. 멘션의 `mentions` 배열은 `null`과 빈 배열을 모두 거부한다. `message_header`의 각 값도 `null`을 거부한다.

이 조건을 통과해도 모든 옵션의 의미까지 검증된 것은 아니다. 예를 들어 존재하지 않는 색상 프리셋, 잘못된 스타일 옵션, 존재하지 않는 멘션 대상은 실제 메시지 처리 때 확인된다.

### 메시지 처리 중 오류가 났을 때

| 오류가 발생한 위치·종류 | 처리 결과 |
|---|---|
| 스타일 동작 함수 내부의 일반 예외 | 경고 로그를 남기고 해당 동작이 받은 텍스트 객체를 반환한다. 이후 스타일 동작은 계속 실행한다. |
| 멘션 대상 동작 함수 내부의 일반 예외 | 경고 로그를 남기고 빈 대상과 자동 스타일 없음으로 처리한다. 이후 처리는 계속한다. |
| `MessageBlockedException` | 남은 메시지 처리를 중단하고 일반 전송을 막는다. 예외에 안내 문구가 있으면 보낸 사람에게 표시한다. |
| 위 동작별 처리 바깥으로 전파된 일반 예외 | 메시지 처리 경고를 남기고 처리 전 메시지를 반환한다. |

스타일의 플레이스홀더·태그 해석 등은 동작 함수를 호출하기 전에 실행되는 부분도 있다. 따라서 모든 스타일 관련 오류가 동작 하나만 건너뛰는 것은 아니다. 앞에서 이미 실행한 명령어, 멘션 알림, 외부 전송은 뒤의 오류로 자동 취소되지 않는다.

스타일 동작이 실패해 입력 텍스트를 반환하더라도, 일반 스타일 규칙에서 그룹 1 바깥의 문법 표식은 이미 제외된 상태다. 예를 들어 `[텍스트]<옵션>`에서 그룹 1을 `텍스트`로 잡았다면 동작 실패 후에도 `[텍스트]<옵션>` 전체가 복원되지는 않는다.

존재하지 않는 팀, 잘못되었거나 존재하지 않는 월드, 잘못된 선택자, 존재하지 않는 Advanced Chat 채널과 채널 제한 위반은 메시지를 차단한다. `PLAYER`는 이름·닉네임으로 플레이어를 찾지 못하면 스코어보드의 해당 이름에 팀 기록이 있는지도 확인하며, 그 기록마저 없으면 차단한다. 연동 모드가 없는 그룹·채널 멘션처럼 빈 대상을 반환하는 경우와 구분한다.

`onlyTarget: true`인 멘션의 대상이 비어 있으면 수신자가 없고 일반 전송도 차단된다. 대상 동작에서 오류를 복구했다는 사실이 메시지의 일반 전송을 보장하지는 않는다.

### 확인 순서

기존 설정을 고칠 때는 다음 순서로 확인한다.

1. **JSON 문법**: 따옴표, 쉼표, 괄호, 문자열 이스케이프와 값의 자료형을 확인한다.
2. **파일 구조**: `style_rules`, `mention_rules` 등의 위치와 그룹·배열 구조를 확인한다.
3. **정규식 문법**: Java `Pattern`으로 해석할 수 있는지 확인한다.
4. **캡처 그룹**: 스타일의 그룹 1·2와 멘션의 그룹 1이 올바른 값을 잡는지 확인한다.
5. **옵션 전달**: `preset`, `delimiter`, 동작 순서를 확인한다. 멘션 스타일의 값은 `preset`에 있는지 확인한다.
6. **규칙 순서와 권한**: 앞선 규칙이 문법을 소비하거나 수정하는지, 플레이어가 해당 그룹 권한을 가졌는지 확인한다.
7. **동작 조건**: 연동 모드, 팀·월드·플레이어 존재 여부, 색상·아이콘 키, 알림 설정과 쿨다운을 확인한다.
8. **적용 결과**: 리로드 성공 여부와 서버 로그를 보고, 대표 입력과 비일치 입력을 확인한다.

정규식은 Java `Pattern` 기준으로 검증한다. 다른 언어의 정규식 엔진과 문법·동작이 다를 수 있다.

`/embellish-chat regex_test <regex> <text>`는 스타일 규칙의 두 캡처 그룹을 확인할 때 사용한다. JSON 문자열에서 이스케이프를 해제한 정규식과 명령어 인자용 표기를 구분한다. 멘션 규칙을 이 명령어로 검증하려고 실제 설정에 불필요한 캡처 그룹을 추가하지 않는다.

리로드 후 동작이 바뀌지 않았다면 버전 불일치나 설정 검증 실패로 현재 설정이 유지되었는지 확인한다. JSON 파싱 성공과 권한·연동·규칙 순서에 따른 실제 동작 성공은 별개다.
