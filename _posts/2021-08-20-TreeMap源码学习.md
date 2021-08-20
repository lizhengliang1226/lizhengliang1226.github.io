# TreeMap源码中的红黑树

## 红黑树的添加

### put方法

```java
public V put(K key, V value) {
    //保存根节点
    Entry<K,V> t = root;
    if (t == null) {
        //根为空，说明插入的是第一个节点
        compare(key, key); // type (and possibly null) check
        //构建根节点，然后返回
        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    //比较结果
    int cmp;
    //父亲节点
    Entry<K,V> parent;
    // split comparator and comparable paths
    //比较器
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        do {
            //在循环中通过比较器对比key是否相同，
            // 直到找到了相同的key，则设置他的值为当前值，然后返回老的值
            parent = t;
            cmp = cpr.compare(key, t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    //没有比较器，key按自然排序
    else {
        //key不能为空，否则抛出异常
        if (key == null)
            throw new NullPointerException();
        //把kay强制转换成比较器
        @SuppressWarnings("unchecked")
        Comparable<? super K> k = (Comparable<? super K>) key;
        do {
            //同上
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    //执行到这里，cmp要么>0要么<0,没有等于0，
    // 说明没有找到符合的键去直接改变值返回，
    // 而是需要新建一个节点然后插入到树上
    Entry<K,V> e = new Entry<>(key, value, parent);
    //根据比较器的值判断放在父亲的左孩子还是右孩子
    if (cmp < 0)
        parent.left = e;
    else
        parent.right = e;
    fixAfterInsertion(e);
    size++;
    modCount++;
    return null;
}

```

### fixAfterInsertion方法

```java
private void fixAfterInsertion(Entry<K,V> x) {
    //插入节点永远是红色
    x.color = RED;
    //x.parent.color == RED，父亲的颜色是黑色，结束循环，对应case1
    //x != root,也就是x==root结束循环，也是对应case1，就是当一直递归，
    // 递归到根节点时，已经不能再往上走了，所以结束循环，然后把根节点变黑
    while (x != null && x != root && x.parent.color == RED) {
        //如果父亲节点是祖父的左孩子
        if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
            //保存叔叔
            Entry<K,V> y = rightOf(parentOf(parentOf(x)));
            //叔叔颜色是红色，说明是在234树中插入四节点，
            // 所以，需要父亲，叔叔变黑，祖父变红，然后往上递归，
            // 直至找到一个黑节点，然后结束循环
            //case1
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);
                setColor(y, BLACK);
                setColor(parentOf(parentOf(x)), RED);
                x = parentOf(parentOf(x));
            } else {
                /*
                    * 上面的colorOf方法，在参数不为空时返回节点的颜色，为空则返回黑色
                    * 所以，到了这里，说明叔叔节点是空的，因为叔叔颜色不为红色
                    * */
                //叔叔为空，x是右孩子，这又是一个特殊情况
                /*
                   pp             pp
                   /   左旋        /     右旋       x
                  xp    ===>     x      ==>       /\
                   \             /              xp pp
                    x            xp
                 */
                //x是父亲的右孩子，需要先左旋，再右旋，才能完成平衡
                if (x == rightOf(parentOf(x))) {
                    //x指向父亲
                    x = parentOf(x);
                    //左旋
                    rotateLeft(x);
                }
                //x父亲变黑，祖父变红，然后沿着祖父右旋，完成平衡
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                rotateRight(parentOf(parentOf(x)));
            }
        } else {
            //父亲是祖父的右孩子，与上面是一样的，对称操作
            Entry<K,V> y = leftOf(parentOf(parentOf(x)));
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);
                setColor(y, BLACK);
                setColor(parentOf(parentOf(x)), RED);
                x = parentOf(parentOf(x));
            } else {
                if (x == leftOf(parentOf(x))) {
                    x = parentOf(x);
                    rotateRight(x);
                }
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                rotateLeft(parentOf(parentOf(x)));
            }
        }
    }
    //循环结束，可能是x递归到了根节点，然后与根节点相同所以结束循环
    //所以要把root重新设置成黑色
    root.color = BLACK;
}

```

## 红黑树的删除

### remove方法

```java
 public V remove(Object key) {
     	//找到要删除的节点
        Entry<K,V> p = getEntry(key);
        if (p == null)
            return null;
		
        V oldValue = p.value;
       //删除节点，返回老的值 
       deleteEntry(p);
        return oldValue;
    }
```

### getEntry方法

```java
 final Entry<K,V> getEntry(Object key) {
        // Offload comparator-based version for sake of performance
     	//比较器不为空，使用比较器找到节点返回
        if (comparator != null)
            return getEntryUsingComparator(key);
        if (key == null)
            throw new NullPointerException();
     	//为空将k强制转换为比较器
        @SuppressWarnings("unchecked")
        Comparable<? super K> k = (Comparable<? super K>) key;
        Entry<K,V> p = root;
        while (p != null) {
            int cmp = k.compareTo(p.key);
            if (cmp < 0)
                p = p.left;
            else if (cmp > 0)
                p = p.right;
            else
                return p;
        }
        return null;
    }
```

### getEntryUsingComparator方法

```java
 final Entry<K,V> getEntryUsingComparator(Object key) {
        @SuppressWarnings("unchecked")
        K k = (K) key;
        Comparator<? super K> cpr = comparator;
        if (cpr != null) {
            Entry<K,V> p = root;
            while (p != null) {
                int cmp = cpr.compare(k, p.key);
                if (cmp < 0)
                    p = p.left;
                else if (cmp > 0)
                    p = p.right;
                else
                    return p;
            }
        }
        return null;
    }

```

### deleteEntry方法

```java
private void deleteEntry(Entry<K,V> p) {
    //修改次数加一。大小减一
    modCount++;
    size--;
    /*
	* 删除节点的步骤
    *   1. 删除的是叶子节点，直接删除，不影响
    *   2. 删除的是有一个孩子的节点，删掉节点，孩子顶上
    *   3. 删除的是有左右子树的节点，找到后继或者前驱，
    *   然后替换两个节点的值，然后删除后继或者前驱，删除时按照1和2来删除就行
    * */
    // If strictly internal, copy successor's element to p and then make p
    // point to successor.
    //删除节点的左右子树不为空，要去找后继或者前驱节点
    if (p.left != null && p.right != null) {
        //寻找后继节点
        Entry<K,V> s = successor(p);
        //找到后，替换两个节点的值，后继节点到达p的位置，
        // p指向后继节点，也就是要删除的节点
        p.key = s.key;
        p.value = s.value;
        p = s;
    } // p has 2 children

    // Start fixup at replacement node, if it exists.
    //替换节点，因为要删除p，在上面已经找到后继节点了，但是，
    // 删除后继，后继也有可能有子孩子来替换她，所以还要去找后继的替换节点
    Entry<K,V> replacement = (p.left != null ? p.left : p.right);
    if (replacement != null) {
        // Link replacement to parent
        //开始替换
        replacement.parent = p.parent;
        //p的父亲是空，说明替换节点是根节点，将跟节点重新赋值
        if (p.parent == null)
            root = replacement;
        //判断放在父亲的左边还是右边
        else if (p == p.parent.left)
            p.parent.left  = replacement;
        else
            p.parent.right = replacement;
        //然后将p真正的删除
        p.left = p.right = p.parent = null;
        // Fix replacement
        //判断，如果删除的是黑节点，则不平衡了，需要重新平衡
        if (p.color == BLACK)
            fixAfterDeletion(replacement);
    } else if (p.parent == null) { // return if we are the only node.
         //到这里替换节点为空，也就是要删除的是叶子节点
        // ，然后他又没有父亲，说明要删除的是根节点，所以，
        //直接把根节点置空
        root = null;
    } else { //  No children. Use self as phantom replacement and unlink.
        //此时说明删除的是叶子节点，一样，判断删除的是黑色还是红色
        if (p.color == BLACK)
            //删除的是黑色，需要平衡
            fixAfterDeletion(p);
        //由于没有替换节点，删除的是她本身，所以，
        // 在平衡后，需要将p真正的删除
        if (p.parent != null) {
            if (p == p.parent.left)
                p.parent.left = null;
            else if (p == p.parent.right)
                p.parent.right = null;
            p.parent = null;
        }
    }
}

```

### successorf方法

```java
  static <K,V> Entry<K,V> successor(Entry<K,V> t) {
        if (t == null)
            return null;
        else if (t.right != null) {
            Entry<K,V> p = t.right;
            while (p.left != null)
                p = p.left;
            return p;
        } else {
            //这步几乎不可能会做，相当于没有左右子树，
            // 往上遍历去找后继，其实就是删除子节点，直接删除就行，根本不需要去找
            //但是在treemap的其他地方会用到，比如遍历时查找后继节点
            Entry<K,V> p = t.parent;
            Entry<K,V> ch = t;
            while (p != null && ch == p.right) {
                ch = p;
                p = p.parent;
            }
            return p;
        }
    }

```



### fixAfterDeletion方法

```java
    private void fixAfterDeletion(Entry<K,V> x) {
        //这两个条件都是递归上来的，要一直递归找到一个红节点
        // 然后把她染黑才行，或者递归到根节点，才结束循环
        while (x != root && colorOf(x) == BLACK) {
            //对称操作，x是父亲的左孩子
            if (x == leftOf(parentOf(x))) {
                //保存兄弟
                Entry<K,V> sib = rightOf(parentOf(x));
                //若兄弟是红色，则不是真兄弟，需要左旋找到真兄弟才能借节点
                if (colorOf(sib) == RED) {
                    //兄弟变黑，父亲变红
                    setColor(sib, BLACK);
                    setColor(parentOf(x), RED);
                    //左旋
                    rotateLeft(parentOf(x));
                    //重新赋值兄弟，因为左旋导致兄弟变了，现在才是真兄弟
                    sib = rightOf(parentOf(x));
                }
                //兄弟是黑色的，但是他的左右子树都是黑色
                // 根据colorOf方法源码，其实就是左右子树为空，
                // 说明没有节点可以借，一样需要递归，因为兄弟存在，
                //所以发生自损，兄弟变红，然后往上递归
                if (colorOf(leftOf(sib))  == BLACK &&
                        colorOf(rightOf(sib)) == BLACK) {
                    setColor(sib, RED);
                    x = parentOf(x);
                } else {
                    //说明兄弟有孩子节点是红色，可以借来用
                    if (colorOf(rightOf(sib)) == BLACK) {
                        //此时，左孩子是红色，但是没办法直接借，
                        // 需要先右旋，再左旋，才能借到
                        /*
                         *            xp(r)                  xp(b)                   sl
                         *              /\      右旋         /\       左旋            /\
                         *             x  s    ==>          x  sl(xp.col)  ==>     xp  s
                         *               /\                      \                 /
                         *           sl(r) nil                    s(b)            x
                         * */
                        //把左孩子变黑
                        setColor(leftOf(sib), BLACK);
                        //兄弟变红
                        setColor(sib, RED);
                        //右旋
                        rotateRight(sib);
                        //重新赋值兄弟
                        sib = rightOf(parentOf(x));
                    }
                    //若不是上面这种特殊情况，兄弟的右孩子已经是红色，
                    // 可以拿来替换，则左旋，完成平衡
                    //设置兄弟为父亲的颜色，然后父亲，孩子变黑，沿着父亲左旋
                    setColor(sib, colorOf(parentOf(x)));
                    setColor(parentOf(x), BLACK);
                    setColor(rightOf(sib), BLACK);
                    rotateLeft(parentOf(x));
                    //平衡完成，重新赋值根节点，使得循环跳出
                    x = root;
                }
            } else { // 对称操作
                Entry<K,V> sib = leftOf(parentOf(x));

                if (colorOf(sib) == RED) {
                    setColor(sib, BLACK);
                    setColor(parentOf(x), RED);
                    rotateRight(parentOf(x));
                    sib = leftOf(parentOf(x));
                }

                if (colorOf(rightOf(sib)) == BLACK &&
                        colorOf(leftOf(sib)) == BLACK) {
                    setColor(sib, RED);
                    x = parentOf(x);
                } else {
                    if (colorOf(leftOf(sib)) == BLACK) {
                        setColor(rightOf(sib), BLACK);
                        setColor(sib, RED);
                        rotateLeft(sib);
                        sib = leftOf(parentOf(x));
                    }
                    setColor(sib, colorOf(parentOf(x)));
                    setColor(parentOf(x), BLACK);
                    setColor(leftOf(sib), BLACK);
                    rotateRight(parentOf(x));
                    x = root;
                }
            }
        }
        //可能是因为找红节点退出的循环，所以最后要置红节点为黑色
        setColor(x, BLACK);
    }

```

