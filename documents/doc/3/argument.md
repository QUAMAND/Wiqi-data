> 명령어에 사용되는 인수를 정리합니다.

---

# 기본 인수

## ColorArgument
명령어를 입력할 때의 색상을 값으로 받을 수 있도록 변환합니다.

> 사용 예시: `red`, `green`  
> 사용 위치: `/team`, `/waypoint`  
> 인수 추천 순서: 알파벳 순
> <details><summary>색상 목록(알파벳 순)</summary>
> 
> - `aqua`
> - `black`
> - `blue`
> - `dark_aqua`
> - `dark_blue`
> - `dark_gray`
> - `dark_green`
> - `dark_purple`
> - `dark_red`
> - `gold`
> - `gray`
> - `green`
> - `light_purple`
> - `red`
> - `reset`
> - `white`
> - `yellow`
> </details>

`parse`
```js
public ChatFormatting parse(final StringReader reader) throws CommandSyntaxException {
  String id = reader.readUnquotedString();
  ChatFormatting result = ChatFormatting.getByName(id);

  if (result != null && !result.isFormat()) {
    return result;

  } else {
    throw ERROR_INVALID_VALUE.createWithContext(reader, id);
  }
}
```

- `id`에 `""`로 감싸지 않은 문자열을 저장합니다.
  - `"red"`: X
  - `red`: O
- `result`에 `id`를 키로 색상 값을 가져옵니다.
  - 대문자를 신경쓰지 않습니다.
    - `aqua`: O
    - `AQUA`: O
    - `AqUa`: O
  - `_`를 신경쓰지 않습니다.
    - `__AQUA__`: O
    - `__RESET__`: O
- `result`가 존재하는 색상 `&&` (`obfuscated`, `bold`, `strikethrough`, `underlined`, `italic`)이 아니어야 합니다.

## ComponentArgument
## CompoundTagArgument
## DimensionArgument
## EntityAnchorArgument
## EntityArgument
## GameModeArgument
## GameProfileArgument
## HeightmapTypeArgument
## HexColorArgument
## IdentifierArgument
## MessageArgument
## NbtPathArgument
## NbtTagArgument
## ObjectiveArgument
## ObjectiveCriteriaArgument
## OperationArgument
## ParticleArgument
## RangeArgument
## ResourceArgument
## ResourceKeyArgument
## ResourceOrIdArgument
## ResourceOrTagArgument
## ResourceOrTagKeyArgument
## ResourceSelectorArgument
## ScoreboardSlotArgument
## ScoreHolderArgument
## SignedArgument
## SlotArgument
## SlotsArgument
## StringRepresentableArgument
## StyleArgument
## TeamArgument
## TemplateMirrorArgument
## TemplateRotationArgument
## TimeArgument
## UuidArgument
## WaypointArgument


# 블록 전용 인수
# 좌표 전용 인수
# 아이템 전용 인수
# 선택자 전용 인수