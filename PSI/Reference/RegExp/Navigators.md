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

* See also: [IConcatenationRegularExpression](TreeNodesA-N.md#iconcatenationregularexpression), [IUnitRegularExpression](TreeNodesO-Z.md#iunitregularexpression)

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

* See also: [IGroupName](TreeNodesA-N.md#igroupname), [IName](TreeNodesA-N.md#iname)

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

* See also: [IName](TreeNodesA-N.md#iname), [INamedBackreference](TreeNodesA-N.md#inamedbackreference)

<!-- End NamedBackreferenceNavigator -->

### NamedBlockNavigator

<!-- Begin NamedBlockNavigator -->

```cs
public static class NamedBlockNavigator
{
  public static INamedBlock GetByName(IName param);
}
```

* See also: [IName](TreeNodesA-N.md#iname), [INamedBlock](TreeNodesA-N.md#inamedblock)

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

* See also: [IGroupName](TreeNodesA-N.md#igroupname), [INamedGroup](TreeNodesA-N.md#inamedgroup), [IRegularExpression](TreeNodesO-Z.md#iregularexpression)

<!-- End NamedGroupNavigator -->

### NestedSetNavigator

<!-- Begin NestedSetNavigator -->

```cs
public static class NestedSetNavigator
{
  public static INestedSet GetByInnerSet(ISet param);
}
```

* See also: [INestedSet](TreeNodesA-N.md#inestedset), [ISet](TreeNodesO-Z.md#iset)

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

* See also: [INumber](TreeNodesA-N.md#inumber), [INumericQuantifier](TreeNodesA-N.md#inumericquantifier)

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

* See also: [IOptionGroup](TreeNodesO-Z.md#ioptiongroup), [IRegularExpression](TreeNodesO-Z.md#iregularexpression)

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

* See also: [IGroupPrefix](TreeNodesA-N.md#igroupprefix), [IPrefixGroup](TreeNodesO-Z.md#iprefixgroup), [IRegularExpression](TreeNodesO-Z.md#iregularexpression)

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

* See also: [IQuantifiableRegularExpression](TreeNodesO-Z.md#iquantifiableregularexpression), [IQuantifier](TreeNodesO-Z.md#iquantifier), [IQuantifierOwner](TreeNodesO-Z.md#iquantifierowner)

<!-- End QuantifiableRegularExpressionNavigator -->

### QuantifierNavigator

<!-- Begin QuantifierNavigator -->

```cs
public static class QuantifierNavigator
{
  public static IQuantifier GetByNumeric(INumericQuantifier param);
}
```

* See also: [INumericQuantifier](TreeNodesA-N.md#inumericquantifier), [IQuantifier](TreeNodesO-Z.md#iquantifier)

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

* See also: [IRegularExpression](TreeNodesO-Z.md#iregularexpression), [IRegularExpressionFile](TreeNodesO-Z.md#iregularexpressionfile)

<!-- End RegularExpressionFileNavigator -->

### RegularExpressionNavigator

<!-- Begin RegularExpressionNavigator -->

```cs
public static class RegularExpressionNavigator
{
  public static IRegularExpression GetByConcatenationRegularExpression(IConcatenationRegularExpression param);
}
```

* See also: [IConcatenationRegularExpression](TreeNodesA-N.md#iconcatenationregularexpression), [IRegularExpression](TreeNodesO-Z.md#iregularexpression)

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

* See also: [ISet](TreeNodesO-Z.md#iset), [ISetBody](TreeNodesO-Z.md#isetbody)

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

* See also: [ISetRange](TreeNodesO-Z.md#isetrange), [ISetRangeItem](TreeNodesO-Z.md#isetrangeitem)

<!-- End SetRangeNavigator -->





