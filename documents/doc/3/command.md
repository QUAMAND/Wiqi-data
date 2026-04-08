> 프로그래밍 입문자 및 마인크래프트 초심자에게는 친화적이지 **않을 수 있습니다**.

---

# 들어가며

`command`(명령어)는 마인크래프트 게임 내부의 기능을 직접 실행하기 위한 언어입니다. 즉, 플레이어가 게임에 어떠한 지시를 내리는 것과 같습니다.

명령어가 추가된 가장 최초의 버전은 [[`Classic 0.0.16a`](https://minecraft.wiki/w/Java_Edition_Classic_0.0.16a)]로 `op, deop, ban, unban, banip, kick, broadcast` 등의 명령어가 [[추가되었으며](https://web.archive.org/web/20111107011756/http://notch.tumblr.com/post/119434936/by-popular-demand-another-mp-test-today)], [[`26.1.1`](https://minecraft.wiki/w/Java_Edition_26.1.1)](최신 버전)에서의 명령어는 총 `91`개로 늘어났습니다.

마인크래프트 명령어는 일반적인 프로그래밍 언어와 다릅니다. 프로그래밍 경험이 있는 사람이라도 처음 보면 직관적이지 않을 수 있습니다. 따라서, 우리는 명령어의 근본부터, 차근차근 알아가는 접근이 필요합니다.

이 문서에서는 명령어를 처음부터 끝까지 학습할 수 있도록 구성했습니다.


# 명령어
마인크래프트에서 명령어는 단순 텍스트 입력으로 실행되는 것이 아닙니다. 내부적으로는 명령어를 해석하고 실행하는 하나의 시스템으로 존재합니다.

## 명령어의 등록
모든 명령어는 고유한 값을 가진 데이터로 저장되어 있는데, 이러한 데이터를 실제로 사용할 수 있도록 게임 내에 등록하는 과정이 필요합니다.

예를 들어, `/spawnpoint` 명령어는 다음 `Tree`(트리) 구조로 게임에 등록됩니다.

```js
spawnpoint
spawnpoint targets
spawnpoint targets pos
spawnpoint targets pos angle
```

1. 인수를 받지 않습니다.
2. `targets` 인수만 받습니다.
3. `targets`, `pos` 인수만 받습니다.
4.  `targets`, `pos`, `angle` 인수만 받습니다.

- 명령어 인수에 대한 내용은 [[명령어 인수]]를 참고하세요.

<br/>

이러한 명령어 등록은 어떠한 환경인지에 따라 여부가 달라집니다.

```js
if (JFR을 설치 가능한 환경이라면) {
  /jfr 명령어를 등록합니다.
}

if (CHASE_COMMAND JVM 인수가 true로 설정되어 있다면){
  /chase 명령어를 등록합니다.
}

if (DEV_COMMANDS JVM 인수가 true로 설정되어 있다면 || 마인크래프트가 IDE에서 실행되어 있다면) {
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

if (전용 서버(server.jar)라면) {
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

if (싱글(클라이언트) 서버라면) {
  /publish 명령어를 등록합니다.
}

else {
  기본 명령어는 다른 조건 없이 모두 등록됩니다.
}
```