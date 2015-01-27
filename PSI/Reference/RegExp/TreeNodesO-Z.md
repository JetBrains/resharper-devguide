# TreeNodes O - Z

<!-- Index O - Z (auto-generated. Remove this line if manually adding/removing entries) -->

<!-- toc -->
<!-- toc stop -->

## O

### IOptionGroup

<!-- Begin IOptionGroup -->

```cs
public interface IOptionGroup :
  IGroup,
  IQuantifierOwner,
  IRegExpTreeNode,
  ITreeNode
{
  ITokenNode Colon { get; }
  ITokenNode Dash { get; }
  TreeNodeCollection<ITokenNode> Options { get; }
  TreeNodeEnumerable<ITokenNode> OptionsEnumerable { get; }
  ITokenNode QuestionSign { get; }
  IRegularExpression RegularExpression { get; }

  IRegularExpression SetRegularExpression(IRegularExpression param);
}
```

* See also: [IGroup](TreeNodesA-N.md#igroup), [IQuantifierOwner](TreeNodesO-Z.md#iquantifierowner), [IRegExpTreeNode](TreeNodesO-Z.md#iregexptreenode), [IRegularExpression](TreeNodesO-Z.md#iregularexpression)

<!-- End IOptionGroup -->

## Q

### IPrefixGroup

<!-- Begin IPrefixGroup -->

```cs
public interface IPrefixGroup :
  IGroup,
  IQuantifierOwner,
  IRegExpTreeNode,
  ITreeNode
{
  IGroupPrefix Prefix { get; }
  ITokenNode QuestionSign { get; }
  IRegularExpression RegularExpression { get; }

  IGroupPrefix SetPrefix(IGroupPrefix param);
  IRegularExpression SetRegularExpression(IRegularExpression param);
}
```

* See also: [IGroup](TreeNodesA-N.md#igroup), [IGroupPrefix](TreeNodesA-N.md#igroupprefix), [IQuantifierOwner](TreeNodesO-Z.md#iquantifierowner), [IRegExpTreeNode](TreeNodesO-Z.md#iregexptreenode), [IRegularExpression](TreeNodesO-Z.md#iregularexpression)

<!-- End IPrefixGroup -->

### IQuantifiableRegularExpression

<!-- Begin IQuantifiableRegularExpression -->

```cs
public interface IQuantifiableRegularExpression :
  IUnitRegularExpression,
  IRegExpTreeNode,
  ITreeNode
{
  IQuantifierOwner Owner { get; }
  IQuantifier Quantifier { get; }

  IQuantifierOwner SetOwner(IQuantifierOwner param);
  IQuantifier SetQuantifier(IQuantifier param);
}
```

* See also: [IQuantifier](TreeNodesO-Z.md#iquantifier), [IQuantifierOwner](TreeNodesO-Z.md#iquantifierowner), [IRegExpTreeNode](TreeNodesO-Z.md#iregexptreenode), [IUnitRegularExpression](TreeNodesO-Z.md#iunitregularexpression)

<!-- End IQuantifiableRegularExpression -->

### IQuantifier

<!-- Begin IQuantifier -->

```cs
public interface IQuantifier :
  IRegExpTreeNode,
  ITreeNode
{
  INumericQuantifier Numeric { get; }
  ITokenNode OptionalQuestion { get; }
  ITokenNode Plus { get; }
  ITokenNode Question { get; }
  ITokenNode Star { get; }

  INumericQuantifier SetNumeric(INumericQuantifier param);
}
```

* See also: [INumericQuantifier](TreeNodesA-N.md#inumericquantifier), [IRegExpTreeNode](TreeNodesO-Z.md#iregexptreenode)

<!-- End IQuantifier -->

### IQuantifierOwner

<!-- Begin IQuantifierOwner -->

```cs
public interface IQuantifierOwner :
  IRegExpTreeNode,
  ITreeNode
{
}
```

* See also: [IRegExpTreeNode](TreeNodesO-Z.md#iregexptreenode)

<!-- End IQuantifierOwner -->

## R

### IRegExpTokenNode

<!-- Begin IRegExpTokenNode -->

```cs
public interface IRegExpTokenNode :
  IRegExpTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IRegExpTreeNode](TreeNodesO-Z.md#iregexptreenode)

<!-- End IRegExpTokenNode -->

### IRegExpTreeNode

<!-- Begin IRegExpTreeNode -->

```cs
public interface IRegExpTreeNode :
  ITreeNode
{
  void Accept(TreeNodeVisitor visitor);
  void Accept<TContext>(TreeNodeVisitor<TContext> visitor, TContext context);
  TReturn Accept<TContext, TReturn>(TreeNodeVisitor<TContext, TReturn> visitor, TContext context);
}
```

<!-- End IRegExpTreeNode -->

### IRegularCharacter

<!-- Begin IRegularCharacter -->

```cs
public interface IRegularCharacter :
  ICharacter,
  IQuantifierOwner,
  ISetBodyCharacter,
  ISetBodyItem,
  IRegExpTreeNode,
  ITreeNode
{
}
```

* See also: [ICharacter](TreeNodesA-N.md#icharacter), [IQuantifierOwner](TreeNodesO-Z.md#iquantifierowner), [IRegExpTreeNode](TreeNodesO-Z.md#iregexptreenode), [ISetBodyCharacter](TreeNodesO-Z.md#isetbodycharacter), [ISetBodyItem](TreeNodesO-Z.md#isetbodyitem)

<!-- End IRegularCharacter -->

### IRegularExpression

<!-- Begin IRegularExpression -->

```cs
public interface IRegularExpression :
  IRegExpTreeNode,
  ITreeNode
{
  TreeNodeCollection<IConcatenationRegularExpression> ConcatenationRegularExpressions { get; }
  TreeNodeEnumerable<IConcatenationRegularExpression> ConcatenationRegularExpressionsEnumerable { get; }
  TreeNodeCollection<ITokenNode> Pipes { get; }
  TreeNodeEnumerable<ITokenNode> PipesEnumerable { get; }
}
```

* See also: [IConcatenationRegularExpression](TreeNodesA-N.md#iconcatenationregularexpression), [IRegExpTreeNode](TreeNodesO-Z.md#iregexptreenode)

<!-- End IRegularExpression -->

### IRegularExpressionFile

<!-- Begin IRegularExpressionFile -->

```cs
public interface IRegularExpressionFile :
  IFile,
  IRegExpTreeNode,
  ITreeNode
{
  IRegularExpression RegularExpression { get; }

  IRegularExpression SetRegularExpression(IRegularExpression param);
}
```

* See also: [IRegExpTreeNode](TreeNodesO-Z.md#iregexptreenode), [IRegularExpression](TreeNodesO-Z.md#iregularexpression)

<!-- End IRegularExpressionFile -->

## S

### ISet

<!-- Begin ISet -->

```cs
public interface ISet :
  IQuantifierOwner,
  IRegExpTreeNode,
  ITreeNode
{
  ISetBody Body { get; }
  ITokenNode LBracket { get; }
  ITokenNode NegativeMark { get; }
  ITokenNode RBracket { get; }

  ISetBody SetBody(ISetBody param);
}
```

* See also: [IQuantifierOwner](TreeNodesO-Z.md#iquantifierowner), [IRegExpTreeNode](TreeNodesO-Z.md#iregexptreenode), [ISetBody](TreeNodesO-Z.md#isetbody)

<!-- End ISet -->

### ISetBody

<!-- Begin ISetBody -->

```cs
public interface ISetBody :
  IRegExpTreeNode,
  ITreeNode
{
}
```

* See also: [IRegExpTreeNode](TreeNodesO-Z.md#iregexptreenode)

<!-- End ISetBody -->

### ISetBodyCharacter

<!-- Begin ISetBodyCharacter -->

```cs
public interface ISetBodyCharacter :
  ISetBodyItem,
  IRegExpTreeNode,
  ITreeNode
{
}
```

* See also: [IRegExpTreeNode](TreeNodesO-Z.md#iregexptreenode), [ISetBodyItem](TreeNodesO-Z.md#isetbodyitem)

<!-- End ISetBodyCharacter -->

### ISetBodyItem

<!-- Begin ISetBodyItem -->

```cs
public interface ISetBodyItem :
  IRegExpTreeNode,
  ITreeNode
{
}
```

* See also: [IRegExpTreeNode](TreeNodesO-Z.md#iregexptreenode)

<!-- End ISetBodyItem -->

### ISetCharacter

<!-- Begin ISetCharacter -->

```cs
public interface ISetCharacter :
  ISetBodyCharacter,
  ISetBodyItem,
  IRegExpTreeNode,
  ITreeNode
{
}
```

* See also: [IRegExpTreeNode](TreeNodesO-Z.md#iregexptreenode), [ISetBodyCharacter](TreeNodesO-Z.md#isetbodycharacter), [ISetBodyItem](TreeNodesO-Z.md#isetbodyitem)

<!-- End ISetCharacter -->

### ISetRange

<!-- Begin ISetRange -->

```cs
public interface ISetRange :
  ISetBodyItem,
  IRegExpTreeNode,
  ITreeNode
{
  ITokenNode Dash { get; }
  ISetRangeItem Max { get; }
  ISetRangeItem Min { get; }

  ISetRangeItem SetMax(ISetRangeItem param);
  ISetRangeItem SetMin(ISetRangeItem param);
}
```

* See also: [IRegExpTreeNode](TreeNodesO-Z.md#iregexptreenode), [ISetBodyItem](TreeNodesO-Z.md#isetbodyitem), [ISetRangeItem](TreeNodesO-Z.md#isetrangeitem)

<!-- End ISetRange -->

### ISetRangeItem

<!-- Begin ISetRangeItem -->

```cs
public interface ISetRangeItem :
  IRegExpTreeNode,
  ITreeNode
{
}
```

* See also: [IRegExpTreeNode](TreeNodesO-Z.md#iregexptreenode)

<!-- End ISetRangeItem -->

### ISimpleGroup

<!-- Begin ISimpleGroup -->

```cs
public interface ISimpleGroup :
  IGroup,
  IQuantifierOwner,
  IRegExpTreeNode,
  ITreeNode
{
  IRegularExpression RegularExpression { get; }

  IRegularExpression SetRegularExpression(IRegularExpression param);
}
```

* See also: [IGroup](TreeNodesA-N.md#igroup), [IQuantifierOwner](TreeNodesO-Z.md#iquantifierowner), [IRegExpTreeNode](TreeNodesO-Z.md#iregexptreenode), [IRegularExpression](TreeNodesO-Z.md#iregularexpression)

<!-- End ISimpleGroup -->

### IStartAnchor

<!-- Begin IStartAnchor -->

```cs
public interface IStartAnchor :
  IAnchor,
  IUnitRegularExpression,
  IRegExpTreeNode,
  ITreeNode
{
}
```

* See also: [IAnchor](TreeNodesA-N.md#ianchor), [IRegExpTreeNode](TreeNodesO-Z.md#iregexptreenode), [IUnitRegularExpression](TreeNodesO-Z.md#iunitregularexpression)

<!-- End IStartAnchor -->

### ISymbolCharacter

<!-- Begin ISymbolCharacter -->

```cs
public interface ISymbolCharacter :
  ICharacter,
  IQuantifierOwner,
  ISetBodyCharacter,
  ISetBodyItem,
  IRegExpTreeNode,
  ITreeNode
{
}
```

* See also: [ICharacter](TreeNodesA-N.md#icharacter), [IQuantifierOwner](TreeNodesO-Z.md#iquantifierowner), [IRegExpTreeNode](TreeNodesO-Z.md#iregexptreenode), [ISetBodyCharacter](TreeNodesO-Z.md#isetbodycharacter), [ISetBodyItem](TreeNodesO-Z.md#isetbodyitem)

<!-- End ISymbolCharacter -->




