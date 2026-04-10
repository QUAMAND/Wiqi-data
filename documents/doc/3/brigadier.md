- [[:github: Brigadier](https://github.com/Mojang/brigadier)]
  - `2017. 07. 27`에 공개: [[x.com/Dinnerbone](https://x.com/Dinnerbone/status/890548451372564480)]

---

# 들어가며
`Brigadier`는 `Mojang`에서 직접 관리하는 명령어 `syntax`(구문) 해석 라이브러리입니다. 플레이어가 입력한 명령어를 분석 및 문법을 확인하며, 게임 내에서 실행할 수 있도록 변환하는 역할까지 수행합니다.

<br/>

# 구조
모든 명령어는 `tree`(트리) 형태로 관리합니다. 모든 명령어는 하나의 `root`(뿌리)에서 시작하여, 인수에 따라 자식 계층으로 확장됩니다.

```js
root
  - gamemode
    - <gamemode>
      - [<target>]
```

이는 플레이어가 입력하는 형태와 직접적으로 대응됩니다.
```js
/gamemode creative @a
```

1. `/gamemode`: root
2. `creative`: children
3. `@a`: children

<br/>

실제 코드에서는 `CommandNode`를 중심으로 구성됩니다.

```js
Map<String, CommandNode<S>> children;
Map<String, LiteralCommandNode<S>> literals;
Map<String, ArgumentCommandNode<S, ?>> arguments;

Command<S> command;
Predicate<S> requirement;

CommandNode<S> redirect;
RedirectModifier<S> modifier;
boolean forks;
```

1. `children`: 모든 자식 계층
2. `literals`: 상수 문자열 기반 자식 계층
3. `arguments`: 인수 기반 자식 계층

명령어 입력 시, `literal`을 먼저 확인하고 없다면 `argument`를 확인합니다.

- `argument`는 동적인 값을 받기에 어떤 값이 올지 모릅니다. 따라서, 모든 경우를 확인해야 하는 동작을 수행해야 하므로(`O(N)`) 고정 값인 `literal`을 우선적으로 검사하는 것(`O(1)`)이 더 적은 비용을 소모합니다.

- 단, 이러한 구조 때문에 같은 인수를 넘길 때는 `literal`이 무조건 우선적으로 반환하는 문제가 있습니다.
  ```js
  literal("test")
    .then(argument("number", integer())).executes(c -> {
      return 0;
    })
    .then(literal("123")).executes(c -> {
      return 1;
    })
  ```
  > `/test 123` 입력 시, `literal("123")`이 `argument("number", integer())` 뒤에 있더라도 `1`이 반환됩니다.

<br/>

## LiteralCommandNode
상수 문자열을 표현합니다. 예를 들어, `/gamemode creative`에서 `creative`가 해당됩니다.

단어 단위로 끊어서 문자열을 확인하는데, 구문을 분석하는 로직은 다음과 같습니다.

<details>
<summary>LiteralCommandNode 중</summary>

```java
@Override
public void parse(final StringReader reader, final CommandContextBuilder<S> contextBuilder) throws CommandSyntaxException {
  final int start = reader.getCursor();
  final int end = parse(reader);

  if (end > -1) {
    contextBuilder.withNode(this, StringRange.between(start, end));
    return;
  }

  throw CommandSyntaxException.BUILT_IN_EXCEPTIONS.literalIncorrect().createWithContext(reader, literal);
}


private int parse(final StringReader reader) {
  final int start = reader.getCursor();

  if (reader.canRead(literal.length())) {
    final int end = start + literal.length();

    if (reader.getString().substring(start, end).equals(literal)) {
      reader.setCursor(end);

      if (!reader.canRead() || reader.peek() == ' ') {
        return end;
      } else {
          reader.setCursor(start);
        }
      }
  }
  return -1;
}
```
</details>

1. 현재 커서 위치(`start`)를 기준으로, `literal` 길이만큼 읽을 수 있는지 확인합니다.
2. 해당 구간의 문자열이 `literal`과 완전히 일치하는지 확인합니다.  
  2-1. 일치하면 커서를 `literal` 끝 위치(`end`)로 이동시킵니다.
3. 이동한 뒤, 문자열이 끝났거나 다음 문자가 공백인지 확인합니다.  
  3-1. 조건을 만족하면 성공 → `end` 반환  
  3-2. 조건을 만족하지 않으면 실패 → `-1` 반환

## ArgumentCommandNode
동적 자료형 값을 표현합니다. 예를 들어, `/random value 0..1`에서 `0..1`이 해당됩니다.

구문을 분석하는 로직은 다음과 같습니다.

<details>
<summary>ArgumentCommandNode 중</summary>

```java
@Override
public void parse(final StringReader reader, final CommandContextBuilder<S> contextBuilder) throws CommandSyntaxException {
  final int start = reader.getCursor();
  final T result = type.parse(reader, contextBuilder.getSource());
  final ParsedArgument<S, T> parsed = new ParsedArgument<>(start, reader.getCursor(), result);

  contextBuilder.withArgument(name, parsed);
  contextBuilder.withNode(this, parsed.getRange());
}
```
</details>

1. 현재 커서 위치(`start`)를 기준으로 `ArgumentType`을 통해 입력 값을 분석합니다.
2. 각 자료형에 맞는 값으로 변경됩니다.
    <details>
    <summary>자료형변환</summary>

    `readBoolean()`
    ```js
    if (value.equals("true")) {
      return true;
    } else if (value.equals("false")) {
      return false;
    }
    ```

    `readDouble()`
    ```js
    return Double.parseDouble(number);
    ```

    `readFloat()`
    ```js
    return Float.parseFloat(number);
    ```

    `readInt()`
    ```js
    return Integer.parseInt(number);
    ```

    `readLong()`
    ```js
    return Long.parseLong(number);
    ```

    `parse()`
    ```js
    if (type == StringType.GREEDY_PHRASE) {
      final String text = reader.getRemaining();
      reader.setCursor(reader.getTotalLength());
      return text;

    } else if (type == StringType.SINGLE_WORD) {
      return reader.readUnquotedString();

    } else {
      return reader.readString();
    }
    ```
    </details>

