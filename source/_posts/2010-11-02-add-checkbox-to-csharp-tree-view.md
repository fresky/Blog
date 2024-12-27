---
layout: post
title: "带checkbox的C# TreeView处理父子节点选择"
date: 2010-11-02
comments: true
tags: Programming
---
今天写了一个C#的TreeView，需要带checkbox，msdn上<a href="http://msdn.microsoft.com/en-us/library/system.windows.forms.treeview.aftercheck.aspx">TreeView.AfterCheck Event (System.Windows.Forms)</a>之处理了子节点的递归选择问题，贴一下我写的父子节点递归选择。



```csharp
void tree_AfterCheck(object sender, TreeViewEventArgs e)
{
    if (e.Action != TreeViewAction.Unknown)
    {
        UpdateCheckStatus(e);
    }
}


// update check status for parent and child
private void UpdateCheckStatus(TreeViewEventArgs e)
{
    CheckAllChildNodes(e.Node);
    UpdateAllParentNodes(e.Node);         
}


// updates all parent nodes recursively.

private void UpdateAllParentNodes(TreeNode treeNode)
{
    TreeNode parent = treeNode.Parent;
    if (parent != null)
    {
        if (parent.Checked && !treeNode.Checked)
        {
            parent.Checked = false;
            UpdateAllParentNodes(parent);
        }

        else if (!parent.Checked && treeNode.Checked)
        {
            bool all = true;
            foreach (TreeNode node in parent.Nodes)
            {
                if (!node.Checked)
                {
                    all = false;
                    break;
                }
            }
            if (all)
            {
                parent.Checked = true;
                UpdateAllParentNodes(parent);
            }
        }
    }
}


// updates all child tree nodes recursively.
private void CheckAllChildNodes(TreeNode treeNode)
{
    foreach (TreeNode node in treeNode.Nodes)
    {
        node.Checked = treeNode.Checked;
        if (node.Nodes.Count > 0)
        {
            // If the current node has child nodes, call the CheckAllChildsNodes method recursively.
            this.CheckAllChildNodes(node);
        }
    }
}
```