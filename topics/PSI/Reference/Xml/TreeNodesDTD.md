[//]: # (title: Tree Nodes \(DTD\))



## A

### IAnyContent

<!-- Begin IAnyContent -->

```csharp
public interface IAnyContent :
  IElementContent,
  ITreeNode
{
}
```

* See also: [IElementContent](TreeNodesDTD.md#ielementcontent)

<!-- End IAnyContent -->

### IAttDef

<!-- Begin IAttDef -->

```csharp
public interface IAttDef :
  ITreeNode
{
  IAttType AttributeType { get; }
  string Name { get; }
}
```

* See also: [IAttType](TreeNodesDTD.md#iatttype)

<!-- End IAttDef -->

### IAttType

<!-- Begin IAttType -->

```csharp
public interface IAttType :
  ITreeNode
{
  AttType AttType { get; }
  IXmlLparenthTokenNode LPar { get; }
  IXmlRparenthTokenNode RPar { get; }
  IEnumerable<string> Types { get; }
}
```

* See also: [IXmlLparenthTokenNode](TreeNodes.md#ixmllparenthtokennode), [IXmlRparenthTokenNode](TreeNodes.md#ixmlrparenthtokennode)

<!-- End IAttType -->

## D

### IDocTypeDeclaration

<!-- Begin IDocTypeDeclaration -->

```csharp
public interface IDocTypeDeclaration :
  ITreeNode
{
  IXmlTagEndTokenNode End { get; }
  IExternalId ExternalId { get; }
  IDTDBody IntSubset { get; }
  IXmlDtdStartTokenNode Start { get; }
}
```

* See also: [IDTDBody](TreeNodesDTD.md#idtdbody), [IExternalId](TreeNodesDTD.md#iexternalid), [IXmlDtdStartTokenNode](TreeNodes.md#ixmldtdstarttokennode), [IXmlTagEndTokenNode](TreeNodes.md#ixmltagendtokennode)

<!-- End IDocTypeDeclaration -->

### IDTDAttListDecl

<!-- Begin IDTDAttListDecl -->

```csharp
public interface IDTDAttListDecl :
  ITreeNode
{
  TreeNodeCollection<IAttDef> AttDefs { get; }
  string ElementName { get; }
  IXmlTagEndTokenNode End { get; }
  IXmlAttlistStartTokenNode Start { get; }
}
```

* See also: [IAttDef](TreeNodesDTD.md#iattdef), [IXmlAttlistStartTokenNode](TreeNodesDTD.md#ixmlattliststarttokennode), [IXmlTagEndTokenNode](TreeNodes.md#ixmltagendtokennode)

<!-- End IDTDAttListDecl -->

### IDTDBody

<!-- Begin IDTDBody -->

```csharp
public interface IDTDBody :
  ITreeNode
{
  IXmlLbracketTokenNode LBracket { get; }
  IXmlRbracketTokenNode RBracket { get; }
}
```

* See also: [IXmlLbracketTokenNode](TreeNodes.md#ixmllbrackettokennode), [IXmlRbracketTokenNode](TreeNodes.md#ixmlrbrackettokennode)

<!-- End IDTDBody -->

### IDTDElementDecl

<!-- Begin IDTDElementDecl -->

```csharp
public interface IDTDElementDecl :
  ITreeNode
{
  IElementContent ContentSpec { get; }
  IXmlTagEndTokenNode End { get; }
  string Name { get; }
  IXmlElementStartTokenNode Start { get; }
}
```

* See also: [IElementContent](TreeNodesDTD.md#ielementcontent), [IXmlElementStartTokenNode](TreeNodesDTD.md#ixmlelementstarttokennode), [IXmlTagEndTokenNode](TreeNodes.md#ixmltagendtokennode)

<!-- End IDTDElementDecl -->

### IDTDEntityDecl

<!-- Begin IDTDEntityDecl -->

```csharp
public interface IDTDEntityDecl :
  ITreeNode
{
  IXmlTagEndTokenNode End { get; }
  string Name { get; }
  IXmlEntityStartTokenNode Start { get; }
  string Value { get; }
}
```

* See also: [IXmlEntityStartTokenNode](TreeNodesDTD.md#ixmlentitystarttokennode), [IXmlTagEndTokenNode](TreeNodes.md#ixmltagendtokennode)

<!-- End IDTDEntityDecl -->

### IDTDFile

<!-- Begin IDTDFile -->

```csharp
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

* See also: [IDTDBody](TreeNodesDTD.md#idtdbody), [IXmlDocumentNode](TreeNodes.md#ixmldocumentnode), [IXmlFile](TreeNodes.md#ixmlfile), [IXmlTagContainer](TreeNodes.md#ixmltagcontainer), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IDTDFile -->

Represents a Document Type Definition file, which is a means of describing the format of an XML file. It derives from `IFile`, indicating that it is a root of a PSI tree, and also inherits from `IXmlFile`, meaning it's also an XML file.

### IDTDNDataDecl

<!-- Begin IDTDNDataDecl -->

```csharp
public interface IDTDNDataDecl :
  ITreeNode
{
  string Name { get; }
}
```

<!-- End IDTDNDataDecl -->

### IDTDNotationDecl

<!-- Begin IDTDNotationDecl -->

```csharp
public interface IDTDNotationDecl :
  ITreeNode
{
  IXmlTagEndTokenNode End { get; }
  string Name { get; }
  IXmlNotationStartTokenNode Start { get; }
}
```

* See also: [IXmlNotationStartTokenNode](TreeNodesDTD.md#ixmlnotationstarttokennode), [IXmlTagEndTokenNode](TreeNodes.md#ixmltagendtokennode)

<!-- End IDTDNotationDecl -->

## E

### IElementContent

<!-- Begin IElementContent -->

```csharp
public interface IElementContent :
  ITreeNode
{
}
```

<!-- End IElementContent -->

### IEmptyContent

<!-- Begin IEmptyContent -->

```csharp
public interface IEmptyContent :
  IElementContent,
  ITreeNode
{
}
```

* See also: [IElementContent](TreeNodesDTD.md#ielementcontent)

<!-- End IEmptyContent -->

### IExternalId

<!-- Begin IExternalId -->

```csharp
public interface IExternalId :
  ITreeNode
{
}
```

<!-- End IExternalId -->

## G

### IGrouppedContent

<!-- Begin IGrouppedContent -->

```csharp
public interface IGrouppedContent :
  IElementContent,
  ITreeNode
{
  TreeNodeCollection<IElementContent> Items { get; }
  IXmlLparenthTokenNode LPar { get; }
  IXmlRparenthTokenNode RPar { get; }
}
```

* See also: [IElementContent](TreeNodesDTD.md#ielementcontent), [IXmlLparenthTokenNode](TreeNodes.md#ixmllparenthtokennode), [IXmlRparenthTokenNode](TreeNodes.md#ixmlrparenthtokennode)

<!-- End IGrouppedContent -->

## P

### IPublicExternalId

<!-- Begin IPublicExternalId -->

```csharp
public interface IPublicExternalId :
  IExternalId,
  ITreeNode
{
}
```

* See also: [IExternalId](TreeNodesDTD.md#iexternalid)

<!-- End IPublicExternalId -->

## R

### IRepetitionContent

<!-- Begin IRepetitionContent -->

```csharp
public interface IRepetitionContent :
  IElementContent,
  ITreeNode
{
  IElementContent Content { get; }
  RepetitionType RepetitionType { get; }
}
```

* See also: [IElementContent](TreeNodesDTD.md#ielementcontent)

<!-- End IRepetitionContent -->

## S

### ISimpleContent

<!-- Begin ISimpleContent -->

```csharp
public interface ISimpleContent :
  IElementContent,
  ITreeNode
{
}
```

* See also: [IElementContent](TreeNodesDTD.md#ielementcontent)

<!-- End ISimpleContent -->

### ISystemExternalId

<!-- Begin ISystemExternalId -->

```csharp
public interface ISystemExternalId :
  IExternalId,
  ITreeNode
{
}
```

* See also: [IExternalId](TreeNodesDTD.md#iexternalid)

<!-- End ISystemExternalId -->

## X

### IXmlAttlistStartTokenNode

<!-- Begin IXmlAttlistStartTokenNode -->

```csharp
public interface IXmlAttlistStartTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlAttlistStartTokenNode -->

### IXmlElementStartTokenNode

<!-- Begin IXmlElementStartTokenNode -->

```csharp
public interface IXmlElementStartTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlElementStartTokenNode -->

### IXmlEntityStartTokenNode

<!-- Begin IXmlEntityStartTokenNode -->

```csharp
public interface IXmlEntityStartTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlEntityStartTokenNode -->

### IXmlFixedTokenNode

<!-- Begin IXmlFixedTokenNode -->

```csharp
public interface IXmlFixedTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlFixedTokenNode -->

### IXmlImpliedTokenNode

<!-- Begin IXmlImpliedTokenNode -->

```csharp
public interface IXmlImpliedTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlImpliedTokenNode -->

### IXmlNotationStartTokenNode

<!-- Begin IXmlNotationStartTokenNode -->

```csharp
public interface IXmlNotationStartTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlNotationStartTokenNode -->

### IXmlRequiredTokenNode

<!-- Begin IXmlRequiredTokenNode -->

```csharp
public interface IXmlRequiredTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlRequiredTokenNode -->