# 虚拟DOM与DOM Diff算法

## 1.DOM



## 2. DOM Diff

### 2.1 Dom-Diff的原理

![react-dom-diff](https://raw.githubusercontent.com/cplasf-lixj/photo-album/main/react-dom-diff.png)

setState()更新状态 --> 重新创建虚拟DOM树 --> 新/旧树比较差异 --> 更新差异对应真实DOM --> 举办界面重绘

`diff`算法会比较前后虚拟`DOM`，从而得到`patches`(补丁)，然后与老`Virtual DOM`进行对比，将其应用在需要更新的地方，得到新的`Virtual DOM`

### 2.2 Diff的几种策略

#### 2.2.1 tree diff

​	该算法针对`Web UI`中`DOM`节点跨层级的移动操作非常少。

​	`react`只会对同一层级的节点进行比较，当发现节点不存在时，删除整个节点及其子节点，不会再进行必须，这样只需要遍历一次，就能完成整个DOM树的比较

​	![tree-diff-1](https://raw.githubusercontent.com/cplasf-lixj/photo-album/main/tree-diff-1.png)

​	如果出现DOM节点的跨层级的移动操作，React会简单的考虑同层级节点的位置变换，对于不同层级的节点，只有创建和删除操作。如，A节点整个被移动到D节点下，根节点发现子节点中A不见了，就会销毁A；然后D发现自己多了一个子节点，就会创建新的子节点及其子节点作为其子节点。`react diff`就会按照这样的次序执行: ` create a -> create b -> create c -> delete a`。这种跨层级的节点移动，并不会出现移动的情况，而是会有创建、删除操作。*这种操作会影响到React的性能，因此React官方不建议进行这种操作*。在开发组件时，保持稳定的dom结构会有助于性能的提升。

![tree-diff-2](https://raw.githubusercontent.com/cplasf-lixj/photo-album/main/tree-diff-2.png)

#### 2.2.2 component diff

拥有相同类的两个组件将会生成相似的树形结构，拥有不同类的两个组件将会生成不同的树形结构

`React`对于组件间的比较采取简洁高效的策略：

	* 同一类型的组件，按照原策略继续比较虚拟dom树
	* 不同类型的组件，则将该组件判断为`dirty component`，从而替换整个组件下的所有子节点。
	* 同一类型的组件，当`Virtual DOM`没有任何变化，如果能确切的知道这点就可以节省大量的`diff`运算的时间，因此`React`允许用户通过`shouldComponentUpdate()`判断该组件是否需要进行diff。

举例来说，当下图中`componet D`改变为`component G`时，即使这两个component结构相似，`react`会判断D和G并不是同类型组件，也就不会比较两者的结构了，而是直接删除D，重新创建G及其子节点。*这是会影响react的性能*。

​	![component-diff](https://raw.githubusercontent.com/cplasf-lixj/photo-album/main/component-diff.png)

