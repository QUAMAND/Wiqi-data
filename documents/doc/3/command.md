> 프로그래밍 입문자 및 마인크래프트 초심자에게는 친화적이지 **않을 수 있습니다**.

---

# 들어가며

`command`(명령어)는 마인크래프트 게임 내부의 기능을 직접 실행하기 위한 언어입니다. 즉, 플레이어가 게임에 어떠한 지시를 내리는 것과 같습니다.

명령어가 추가된 가장 최초의 버전은 [[`Classic 0.0.16a`](https://minecraft.wiki/w/Java_Edition_Classic_0.0.16a)]로 `op, deop, ban, unban, banip, kick, broadcast` 등의 명령어가 [[추가되었으며](https://web.archive.org/web/20111107011756/http://notch.tumblr.com/post/119434936/by-popular-demand-another-mp-test-today)], [[`26.1.1`](https://minecraft.wiki/w/Java_Edition_26.1.1)](최신 버전)에서의 명령어는 총 `91`개로 늘어났습니다.

마인크래프트 명령어는 일반적인 프로그래밍 언어와 다릅니다. 프로그래밍 경험이 있는 사람이라도 처음 보면 직관적이지 않을 수 있습니다. 따라서, 우리는 명령어의 근본부터, 차근차근 알아가는 접근이 필요합니다.

이 문서에서는 명령어를 처음부터 끝까지 학습할 수 있도록 구성했습니다.

<br/>

# 명령어
마인크래프트에서 명령어는 단순 텍스트 입력으로 실행되는 것이 아닙니다. 내부적으로는 명령어를 해석하고 실행하는 하나의 시스템으로 존재합니다.

## 명령어의 등록
모든 명령어는 고유한 값을 가진 데이터로 저장되어 있는데, 이러한 데이터를 실제로 사용할 수 있도록 게임 내에 등록하는 과정이 필요합니다.

코드에서는 `명령어.register`의 형태로 이루어져 있으며, 명령어 `Class`(클래스)내에 정의되어 있습니다.

예를 들어, `/gamemode` 명령어의 경우 다음과 같이 이루어져 있습니다.

```js
public static void register(final CommandDispatcher<CommandSourceStack> dispatcher) {
  dispatcher.register(
    Commands.literal("gamemode")
      .requires(Commands.hasPermission(PERMISSION_CHECK))

      .then(Commands.argument("gamemode", GameModeArgument.gameMode())
        .executes(c -> setMode(c, Collections.singleton(c.getSource().getPlayerOrException()), GameModeArgument.getGameMode(c, "gamemode")))

        .then(Commands.argument("target", EntityArgument.players())
          .executes(c -> setMode(c, EntityArgument.getPlayers(c, "target"), GameModeArgument.getGameMode(c, "gamemode")))
        )
      )
  );
}
```

- 명령어 인수에 대한 내용은 [[:markdown: Argument](#/doc/3/command/argument)]를 참고하세요. (`targets`, `pos` 등)
- 명령어 구문 분석에 대한 내용은 [[:markdown: Brigadier](#/doc/3/command/brigadier)]를 참고하세요.

<br/>

이러한 명령어 등록은 어떠한 환경인지에 따라 여부가 달라집니다.

<details>
  <summary>환경에 따른 명령어 목록</summary>

  ```js
  if (JFR을 설치 가능한 환경이라면) {
    /jfr 명령어를 등록합니다.
  }

  if (CHASE_COMMAND JVM 인수가 true로 설정되어 있다면){
    /chase 명령어를 등록합니다.
  }

  if (DEV_COMMANDS JVM 인수가 true로 설정되어 있다면 || 마인크래프트가 IDE에서 실행된다면) {
    /raid 명령어를 등록합니다.
    /debugpath 명령어를 등록합니다.
    /debugmobspawning 명령어를 등록합니다.
    /warden_spawn_tracker 명령어를 등록합니다.
    /spawn_armor_trims 명령어를 등록합니다.
    /serverpack 명령어를 등록합니다.
    if (전용 서버(server.jar)라면) {
      /debugconfig 명령어를 등록합니다.
    }
  }

  if (전용 서버라면) {
    /ban-ip 명령어를 등록합니다.
    /banlist 명령어를 등록합니다.
    /ban 명령어를 등록합니다.
    /deop 명령어를 등록합니다.
    /op 명령어를 등록합니다.
    /pardon 명령어를 등록합니다.
    /pardon-ip 명령어를 등록합니다.
    /perf 명령어를 등록합니다.
    /save-all 명령어를 등록합니다.
    /save-off 명령어를 등록합니다.
    /save-on 명령어를 등록합니다.
    /setidletimeout 명령어를 등록합니다.
    /stop 명령어를 등록합니다.
    /transfer 명령어를 등록합니다.
    /whitelist 명령어를 등록합니다.
  }

  if (싱글 서버라면) {
    /publish 명령어를 등록합니다.
  }

  else {
    기본 명령어는 다른 조건 없이 모두 등록됩니다.
  }
  ```
</details>

모든 명령어는 사용할 문자열 입력 -> 권한 확인 -> 하위 인수 입력 및 실행으로 구현되어 있습니다.

## 명령어의 구조
명령어는 정해진 규칙에 따라 해석되는데, 기본적인 형태는 다음과 같습니다.

```js
/command <required> [optional]
```

