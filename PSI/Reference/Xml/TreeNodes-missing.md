# TreeNodes


<!-- toc -->
<!-- toc stop -->

## D

### IDocCommentBlockNodeWithPsi&lt;TXmlPsi, TCommentNode&gt;

<!-- Begin IDocCommentBlockNodeWithPsi`2 -->

```cs
public interface IDocCommentBlockNodeWithPsi<TXmlPsi, TCommentNode> :
  IDocCommentBlockNode,
  ITreeNode
{
  TreeNodeCollection<TCommentNode> DocComments { get; }

  TCommentNode AddDocCommentAfter(TCommentNode nodeToAdd, TCommentNode anchor);
  TCommentNode AddDocCommentBefore(TCommentNode nodeToAdd, TCommentNode anchor);

  TXmlPsi GetXmlPsi();

  void RemoveDocComment(TCommentNode docCommentNode);
}
```

<!-- End IDocCommentBlockNodeWithPsi`2 -->

