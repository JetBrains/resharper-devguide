[//]: # (title: Text Buffers)

Lexers process a block of text to produce a sequence of tokens. ReSharper provides the `IBuffer` interface to abstract accessing this text block:

```csharp
public interface IBuffer
{
  int Length { get; }
  char this[int index] { get; }
  string GetText();
  string GetText(TextRange range);
  void AppendTextTo(StringBuilder builder, TextRange range);
  void CopyTo(int sourceIndex, char[] destinationArray, int destinationIndex, int length);
}
```

The `GetText(TextRange)` method will return a string from a range within the buffer. See below for details on `TextRange`.

The `AppendTextTo` and `CopyTo` will append the text in the buffer to the given `StringBuilder` and copy to the given char array, respectively.

## Implementations

ReSharper provides several implementations of `IBuffer`. The most interesting are:

* `StringBuffer` - a simple implementation backed by a `string`. Also provides `StringBuffer.Empty` to represent a buffer over an empty string.
* `StringBuilderBuffer` - a simple implementation backed by a `StringBuilder`. Also provides `StringBuilderBuffer.Empty` to represent a buffer over an empty `StringBuilder`.
* `ProjectedBuffer` - projects an implementation of `IBuffer` onto a `TextRange` of an existing buffer.
* `AggregatedBuffer` - implements `IBuffer` on top of an array of other `IBuffer` implementations. The buffers are assumed to be in the correct order, with no gaps between the end of one buffer and the start of another.
* `ArrayBuffer` - similar to `StringBuffer`, in that the data comes from a `string`, either passed directly to the constructor, or via `IBuffer`, where the constructor calls `IBuffer.GetText()`. This class also implements the `IEditableBuffer` interface, which allows modifying the text buffer, internally using a `char[]` to hold the new buffer.

## Text range

`IBuffer` uses `TextRange` to specify the start and end offset of a range within a text document. The start offset is inclusive, while the end offset is not. For example, given the string "Hello world", the range `new TextRange(0, 5)` would give the text "Hello". The character at the start index is included, but the character at the end index isn't.

The `TextRange` struct is immutable, and has many useful methods for creating a new `TextRange`, such as `ExtendLeft` and `ExtendRight` to increase the range before the start offset and after the start offset respectively. It also implements `IEquatable<TextRange>` and the `==` and `!=` operators, so it can be compared to other `TextRange` instances.

```csharp
public struct TextRange : IEquatable<TextRange>
{
  // Should be replaced with Nullable<TextRange> wherever possible
  public static readonly TextRange InvalidRange = new TextRange(-1, -1);

  public TextRange(int startOffset, int endOffset);
  public TextRange(int offset);

  // Based on a string such as (0, 10)
  public static TextRange Parse(string s);
  public static TextRange FromLength(int offset, int length);
  public static TextRange FromLength(int length);
  public static TextRange FromUnorderedOffsets(int one, int onemore);

  public int StartOffset { get; }
  public int EndOffset { get; }
  public int Length { get; }
  public bool IsEmpty { get; }

  // Should be replaced with Nullable<TextRange> wherever possible.
  public bool IsValid { get; }

  public int GetMinOffset();
  public int GetMaxOffset();
  public bool ContainedIn(TextRange textRange);
  public bool StrictContainedIn(TextRange textRange);
  public bool Contains(TextRange textRange);
  public bool Contains(int offset);
  public bool ContainsCharIndex(int charindex);

  // Return new instances
  public TextRange SetStartTo(int offset);
  public TextRange SetEndTo(int offset);

  // Get the leftmost or rightmost portion of the range
  public TextRange Left(int length);
  public TextRange Right(int length);

  // Remove the length from the left (start) or right (end) of the range
  public TextRange TrimLeft(int length);
  public TextRange TrimRight(int length);

  // Extend the range to the left (decrease start offset) or right (increase end offset)
  public TextRange ExtendLeft(int offset);
  public TextRange ExtendRight(int length);

  // Move the range to the right (positive delta) or the left (negative delta)
  public TextRange Shift(int delta);

  // Create a new range that covers both ranges
  public TextRange Join(TextRange textRange);

  // Extend the range to cover the left (start) or right (end) of both ranges
  public TextRange JoinLeft(TextRange textRange);
  public TextRange JoinRight(TextRange textRange);

  public bool Intersects(TextRange textRange);
  public bool StrictIntersects(TextRange textRange);
  public TextRange Intersect(TextRange textRange);

  [Conditional("JET_MODE_ASSERT")] public void AssertValid();
  [Conditional("JET_MODE_ASSERT")] public void AssertNormalized();
  [Conditional("JET_MODE_ASSERT")] public void AssertContainedIn(TextRange rangeContainer);

  public bool IsNormalized { get; }

  // Returns a normalized range
  public TextRange Normalized();
  {
    return IsValid ? new TextRange(GetMinOffset(), GetMaxOffset()) : InvalidRange;
  }

  public int DistanceTo(int offset);
  public bool IsLeftTo(int offset);

  public TextRange UpdateRange(int changeStartOffset, int oldLength, int newLength, bool greedyToLeft, bool greedyToRight);
}
```
