# Navigators

<!-- Index A - Z (auto-generated. Remove this line if manually adding/removing entries) -->

<!-- toc -->
<!-- toc stop -->

## C

### ConcatenationRegularExpressionNavigator

<!-- Begin ConcatenationRegularExpressionNavigator -->

```cs
public static class ConcatenationRegularExpressionNavigator
{
  public static IConcatenationRegularExpression GetByUnitRegularExpression(IUnitRegularExpression param);
}
```

<!-- End ConcatenationRegularExpressionNavigator -->

## G

### GroupNameNavigator

<!-- Begin GroupNameNavigator -->

```cs
public static class GroupNameNavigator
{
  public static IGroupName GetByName(IName param);
  public static IGroupName GetBySecondaryName(IName param);
}
```

<!-- End GroupNameNavigator -->

## N

### NamedBackreferenceNavigator

<!-- Begin NamedBackreferenceNavigator -->

```cs
public static class NamedBackreferenceNavigator
{
  public static INamedBackreference GetByName(IName param);
}
```

<!-- End NamedBackreferenceNavigator -->

### NamedBlockNavigator

<!-- Begin NamedBlockNavigator -->

```cs
public static class NamedBlockNavigator
{
  public static INamedBlock GetByName(IName param);
}
```

<!-- End NamedBlockNavigator -->

### NamedGroupNavigator

<!-- Begin NamedGroupNavigator -->

```cs
public static class NamedGroupNavigator
{
  public static INamedGroup GetByGroupName(IGroupName param);
  public static INamedGroup GetByRegularExpression(IRegularExpression param);
}
```

<!-- End NamedGroupNavigator -->

### NestedSetNavigator

<!-- Begin NestedSetNavigator -->

```cs
public static class NestedSetNavigator
{
  public static INestedSet GetByInnerSet(ISet param);
}
```

<!-- End NestedSetNavigator -->

### NumericQuantifierNavigator

<!-- Begin NumericQuantifierNavigator -->

```cs
public static class NumericQuantifierNavigator
{
  public static INumericQuantifier GetByNumber(INumber param);
  public static INumericQuantifier GetBySecondaryNumber(INumber param);
}
```

<!-- End NumericQuantifierNavigator -->

## O

### OptionGroupNavigator

<!-- Begin OptionGroupNavigator -->

```cs
public static class OptionGroupNavigator
{
  public static IOptionGroup GetByRegularExpression(IRegularExpression param);
}
```

<!-- End OptionGroupNavigator -->

## P

### PrefixGroupNavigator

<!-- Begin PrefixGroupNavigator -->

```cs
public static class PrefixGroupNavigator
{
  public static IPrefixGroup GetByPrefix(IGroupPrefix param);
  public static IPrefixGroup GetByRegularExpression(IRegularExpression param);
}
```

<!-- End PrefixGroupNavigator -->

## Q

### QuantifiableRegularExpressionNavigator

<!-- Begin QuantifiableRegularExpressionNavigator -->

```cs
public static class QuantifiableRegularExpressionNavigator
{
  public static IQuantifiableRegularExpression GetByOwner(IQuantifierOwner param);
  public static IQuantifiableRegularExpression GetByQuantifier(IQuantifier param);
}
```

<!-- End QuantifiableRegularExpressionNavigator -->

### QuantifierNavigator

<!-- Begin QuantifierNavigator -->

```cs
public static class QuantifierNavigator
{
  public static IQuantifier GetByNumeric(INumericQuantifier param);
}
```

<!-- End QuantifierNavigator -->

## R

### RegularExpressionFileNavigator

<!-- Begin RegularExpressionFileNavigator -->

```cs
public static class RegularExpressionFileNavigator
{
  public static IRegularExpressionFile GetByRegularExpression(IRegularExpression param);
}
```

<!-- End RegularExpressionFileNavigator -->

### RegularExpressionNavigator

<!-- Begin RegularExpressionNavigator -->

```cs
public static class RegularExpressionNavigator
{
  public static IRegularExpression GetByConcatenationRegularExpression(IConcatenationRegularExpression param);
}
```

<!-- End RegularExpressionNavigator -->

## S

### SetNavigator

<!-- Begin SetNavigator -->

```cs
public static class SetNavigator
{
  public static ISet GetByBody(ISetBody param);
}
```

<!-- End SetNavigator -->

### SetRangeNavigator

<!-- Begin SetRangeNavigator -->

```cs
public static class SetRangeNavigator
{
  public static ISetRange GetByMax(ISetRangeItem param);
  public static ISetRange GetByMin(ISetRangeItem param);
}
```

<!-- End SetRangeNavigator -->




