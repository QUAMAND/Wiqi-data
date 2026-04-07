> 프로그래밍 입문자 및 마인크래프트 초심자에게는 친화적이지 **않을 수 있습니다**.

---

# 들어가며

`command`(명령어)는 마인크래프트 게임 내부의 기능을 직접 실행하기 위한 언어입니다. 즉, 플레이어가 게임에 어떠한 지시를 내리는 것과 같습니다.

명령어가 추가된 가장 최초의 버전은 [[`Classic 0.0.16a`](https://minecraft.wiki/w/Java_Edition_Classic_0.0.16a)]로 `op, deop, ban, unban, banip, kick, broadcast` 등의 명령어가 [[추가되었으며](https://web.archive.org/web/20111107011756/http://notch.tumblr.com/post/119434936/by-popular-demand-another-mp-test-today)], [[`26.1.1`](https://minecraft.wiki/w/Java_Edition_26.1.1)](최신 버전)에서의 명령어는 총 `91`개로 늘어났습니다.

마인크래프트 명령어는 일반적인 프로그래밍 언어와 다릅니다. 프로그래밍 경험이 있는 사람이라도 처음 보면 직관적이지 않을 수 있습니다. 따라서, 우리는 명령어의 근본부터, 차근차근 알아가는 접근이 필요합니다.

이 문서에서는 명령어를 처음부터 끝까지 학습할 수 있도록 구성했습니다.


# 명령어
마인크래프트에서 


마인크래프트에서 “명령어”의 정체 (엔진 기준)
1. 명령어는 문자열이 아니다

겉으로 보면:

/give @p diamond

그냥 텍스트처럼 보이지만, 내부에서는 이렇게 처리됨:

문자열 → 파싱 → 트리 구조 → 실행

2. 핵심 구조: CommandDispatcher

코드에서 가장 중요한 줄:

private final CommandDispatcher<CommandSourceStack> dispatcher = new CommandDispatcher();

👉 이게 바로 명령어 엔진의 핵심이다.

역할
명령어 등록
명령어 파싱
실행 관리

👉 즉:

마인크래프트의 “인터프리터”

3. 명령어는 “트리 구조”다

이 코드가 힌트:

RootCommandNode
CommandNode
ArgumentCommandNode

👉 명령어는 이렇게 구성됨:

give
 ├─ <target>
 │    └─ <item>
 │         └─ <count>
📌 예시 분석
/give @p diamond 1

파싱 결과:

단계	노드
1	give (리터럴)
2	@p (argument: target)
3	diamond (argument: item)
4	1 (argument: count)

👉 즉:

명령어는 “문자열”이 아니라 **문법 트리(AST)**다

4. 명령어 등록 방식

코드에서 계속 보이는 패턴:

GiveCommand.register(this.dispatcher, context);
TeleportCommand.register(this.dispatcher);
ExecuteCommand.register(this.dispatcher, context);

👉 의미:

각 명령어는 “클래스”로 정의됨
dispatcher에 등록됨
📌 개념적으로 보면
dispatcher.register(
  literal("give")
    .then(argument("target", ...))
    .then(argument("item", ...))
)

👉 이게 바로 문법 정의 (Grammar)

5. 실행 과정 (핵심 흐름)

이 코드가 실제 실행 루트:

this.performCommand(this.dispatcher.parse(command, sender), command);
🔁 전체 흐름
1. 입력
/give @p diamond
2. prefix 제거
trimOptionalPrefix()

👉 / 제거됨

3. 파싱
dispatcher.parse(command, sender)

👉 문자열 → 트리

4. 검증
validateParseResults()

👉 문법 오류 체크

5. 실행
ExecutionContext.queueInitialCommandExecution()

👉 실제 실행

6. 실행 컨텍스트 (중요)
CommandSourceStack sender

👉 이게 진짜 핵심 개념이다.

📌 의미

명령어는 항상 다음 정보와 함께 실행됨:

누가 실행했는가
어디에서 실행했는가
권한은 무엇인가
👉 그래서 이런 게 가능한 것
execute as @a at @s run say hi

👉 실행 주체/위치가 바뀌는 이유 = 컨텍스트

7. 명령어는 “큐 기반 실행”이다

이 코드:

executionContext.runCommandQueue();

👉 의미:

명령어는 즉시 실행이 아니라 큐에 들어가서 순차 실행됨

📌 추가 제한
GameRules.MAX_COMMAND_SEQUENCE_LENGTH
GameRules.MAX_COMMAND_FORKS

👉 무한 루프 방지 시스템

8. 에러 처리 시스템
CommandSyntaxException

👉 명령어 에러는 단순 실패가 아니라:

문법 오류
인자 오류
권한 오류

로 나뉨

📌 특징

에러 발생 시:

위치 표시
자동 하이라이트
추천 명령어 제공
9. 자동완성 시스템
SuggestionProvider

👉 탭 자동완성의 정체

🔥 최종 정리 (문서용 핵심 문장)

마인크래프트 명령어는 단순한 텍스트 입력이 아니라,
Brigadier 기반의 문법 트리 구조를 가진 인터프리터 언어 시스템이다.

💡 프로그래밍 언어 관점 비교
개념	마인크래프트
파서	CommandDispatcher
AST	CommandNode 트리
실행기	ExecutionContext
함수	mcfunction
런타임	게임 엔진
🚀 문서용 한 줄 정의 (강력 추천)

이건 그대로 써도 좋다:

명령어는 문자열 형태로 작성되지만, 내부적으로는 문법 트리로 파싱되어 실행되는 구조적 명령 실행 시스템이다.

원하면 다음 단계로:

👉 execute를 “언어 기능”처럼 완전 해부
👉 scoreboard를 “변수 시스템”으로 문서화
👉 Brigadier 기반 문법 정의까지 풀어서 설명