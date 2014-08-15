# Tree Nodes (DTD)

<!-- toc -->
<!-- toc stop -->

## A

### IAnyContent

<!-- Begin IAnyContent -->

```cs
public interface IAnyContent :
  IElementContent,
  ITreeNode
{
}
```

<!-- End IAnyContent -->

### IAttDef

<!-- Begin IAttDef -->

```cs
public interface IAttDef :
  ITreeNode
{
  IAttType AttributeType { get; }
  string Name { get; }
}
```

<!-- End IAttDef -->

### IAttType

<!-- Begin IAttType -->

```cs
public interface IAttType :
  ITreeNode
{
  AttType AttType { get; }
  IXmlLparenthTokenNode LPar { get; }
  IXmlRparenthTokenNode RPar { get; }
  IEnumerable<string> Types { get; }
}
```

<!-- End IAttType -->

## D

### IDocTypeDeclaration

<!-- Begin IDocTypeDeclaration -->

```cs
public interface IDocTypeDeclaration :
  ITreeNode
{
  IXmlTagEndTokenNode End { get; }
  IExternalId ExternalId { get; }
  IDTDBody IntSubset { get; }
  IXmlDtdStartTokenNode Start { get; }
}
```

<!-- End IDocTypeDeclaration -->

### IDTDAttListDecl

<!-- Begin IDTDAttListDecl -->

```cs
public interface IDTDAttListDecl :
  ITreeNode
{
  TreeNodeCollection<IAttDef> AttDefs { get; }
  string ElementName { get; }
  IXmlTagEndTokenNode End { get; }
  IXmlAttlistStartTokenNode Start { get; }
}
```

<!-- End IDTDAttListDecl -->

### IDTDBody

<!-- Begin IDTDBody -->

```cs
public interface IDTDBody :
  ITreeNode
{
  IXmlLbracketTokenNode LBracket { get; }
  IXmlRbracketTokenNode RBracket { get; }
}
```

<!-- End IDTDBody -->

### IDTDElementDecl

<!-- Begin IDTDElementDecl -->

```cs
public interface IDTDElementDecl :
  ITreeNode
{
  IElementContent ContentSpec { get; }
  IXmlTagEndTokenNode End { get; }
  string Name { get; }
  IXmlElementStartTokenNode Start { get; }
}
```

<!-- End IDTDElementDecl -->

### IDTDEntityDecl

<!-- Begin IDTDEntityDecl -->

```cs
public interface IDTDEntityDecl :
  ITreeNode
{
  IXmlTagEndTokenNode End { get; }
  string Name { get; }
  IXmlEntityStartTokenNode Start { get; }
  string Value { get; }
}
```

<!-- End IDTDEntityDecl -->

### IDTDFile

<!-- Begin IDTDFile -->

```cs
public interface IDTDFile :
  IXmlFile,
  IFile,
  IXmlTagContainer,
  IXmlDocumentNode,
  IXmlTreeNode,
  IDTDBody,
  ITreeNode
{
}
```

<!-- End IDTDFile -->

Represents a Document Type Definition file, which is a means of describing the format of an XML file. It derives from `IFile`, indicating that it is a root of a PSI tree, and also inherits from `IXmlFile`, meaning it's also an XML file.

### IDTDNDataDecl

<!-- Begin IDTDNDataDecl -->

```cs
public interface IDTDNDataDecl :
  ITreeNode
{
  string Name { get; }
}
```

<!-- End IDTDNDataDecl -->

### IDTDNotationDecl

<!-- Begin IDTDNotationDecl -->

```cs
public interface IDTDNotationDecl :
  ITreeNode
{
  IXmlTagEndTokenNode End { get; }
  string Name { get; }
  IXmlNotationStartTokenNode Start { get; }
}
```

<!-- End IDTDNotationDecl -->

## E

### IElementContent

<!-- Begin IElementContent -->

```cs
public interface IElementContent :
  ITreeNode
{
}
```

<!-- End IElementContent -->

### IEmptyContent

<!-- Begin IEmptyContent -->

```cs
public interface IEmptyContent :
  IElementContent,
  ITreeNode
{
}
```

<!-- End IEmptyContent -->

### IExternalId

<!-- Begin IExternalId -->

```cs
public interface IExternalId :
  ITreeNode
{
}
```

<!-- End IExternalId -->

## G

### IGrouppedContent

<!-- Begin IGrouppedContent -->

```cs
public interface IGrouppedContent :
  IElementContent,
  ITreeNode
{
  TreeNodeCollection<IElementContent> Items { get; }
  IXmlLparenthTokenNode LPar { get; }
  IXmlRparenthTokenNode RPar { get; }
}
```

<!-- End IGrouppedContent -->

## P

### IPublicExternalId

<!-- Begin IPublicExternalId -->

```cs
public interface IPublicExternalId :
  IExternalId,
  ITreeNode
{
}
```

<!-- End IPublicExternalId -->

## R

### IRepetitionContent

<!-- Begin IRepetitionContent -->

```cs
public interface IRepetitionContent :
  IElementContent,
  ITreeNode
{
  IElementContent Content { get; }
  RepetitionType RepetitionType { get; }
}
```

<!-- End IRepetitionContent -->

## S

### ISimpleContent

<!-- Begin ISimpleContent -->

```cs
public interface ISimpleContent :
  IElementContent,
  ITreeNode
{
}
```

<!-- End ISimpleContent -->

### ISystemExternalId

<!-- Begin ISystemExternalId -->

```cs
public interface ISystemExternalId :
  IExternalId,
  ITreeNode
{
}
```

<!-- End ISystemExternalId -->

## X

### IXmlAttlistStartTokenNode

<!-- Begin IXmlAttlistStartTokenNode -->

```cs
public interface IXmlAttlistStartTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

<!-- End IXmlAttlistStartTokenNode -->

### IXmlElementStartTokenNode

<!-- Begin IXmlElementStartTokenNode -->

```cs
public interface IXmlElementStartTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

<!-- End IXmlElementStartTokenNode -->

### IXmlEntityStartTokenNode

<!-- Begin IXmlEntityStartTokenNode -->

```cs
public interface IXmlEntityStartTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

<!-- End IXmlEntityStartTokenNode -->

### IXmlFixedTokenNode

<!-- Begin IXmlFixedTokenNode -->

```cs
public interface IXmlFixedTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

<!-- End IXmlFixedTokenNode -->

### IXmlImpliedTokenNode

<!-- Begin IXmlImpliedTokenNode -->

```cs
public interface IXmlImpliedTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

<!-- End IXmlImpliedTokenNode -->

### IXmlNotationStartTokenNode

<!-- Begin IXmlNotationStartTokenNode -->

```cs
public interface IXmlNotationStartTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

<!-- End IXmlNotationStartTokenNode -->

### IXmlRequiredTokenNode

<!-- Begin IXmlRequiredTokenNode -->

```cs
public interface IXmlRequiredTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

<!-- End IXmlRequiredTokenNode -->

