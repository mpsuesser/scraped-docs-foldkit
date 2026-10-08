---
url: https://foldkit.dev/api-reference/ui-virtual-list
title: "Ui/VirtualList"
description: "API documentation for the Ui/VirtualList module."
access_date: 2026-10-08T02:46:24.261Z
current_date: 2026-10-08T02:46:24.261Z
---

# Ui/VirtualList

## Functions

### informItemsChanged

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/ui/src/virtualList/index.ts#L825)

```
/**
 * Notifies VirtualList that its parent-owned items changed. The next view
 *  resolves the stored stable-key anchor against the new items, and the
 *  Command restores that anchor after the DOM patch.
 */
(
  model: VirtualList.Model,
  itemKeys: readonly Array<string>
): ScrollReturn
```

### init

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/ui/src/virtualList/index.ts#L205)

```
/**
 * Creates an initial virtual list model from a config. The container starts
 *  in `Unmeasured` state until its Mount reports the first measurement.
 */
(config: InitConfig): VirtualList.Model
```

### scrollTo

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/ui/src/virtualList/index.ts#L775)

```
/** Programmatically scrolls to a logical target. */
(
  model: VirtualList.Model,
  target: {
    _tag: "Key"
    key: string
  } | {
    _tag: "Offset"
    offset: number
  } | {
    _tag: "End"
  } | {
    _tag: "Index"
    index: number
  },
  options: ScrollToOptions
): ScrollReturn
```

### scrollToEnd

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/ui/src/virtualList/index.ts#L819)

```
/** Programmatically scrolls to the end of the list. */
(model: VirtualList.Model): ScrollReturn
```

### scrollToIndex

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/ui/src/virtualList/index.ts#L799)

```
/**
 * Programmatically scrolls the container so the row at `index` is visible.
 *  The next view resolves the logical index with its current sizing mode, and
 *  the Command aligns the live rendered row.
 */
(
  model: VirtualList.Model,
  index: number,
  options: ScrollToOptions
): ScrollReturn
```

### scrollToKey

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/ui/src/virtualList/index.ts#L807)

```
/**
 * Programmatically scrolls to the row whose `itemToKey` result matches
 *  `key`. The next view resolves the key against its current items.
 */
(
  model: VirtualList.Model,
  key: string,
  options: ScrollToOptions
): ScrollReturn
```

### scrollToOffset

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/ui/src/virtualList/index.ts#L815)

```
/**
 * Programmatically scrolls to an exact pixel offset from the start of the
 *  list. Negative offsets clamp to zero.
 */
(
  model: VirtualList.Model,
  offset: number
): ScrollReturn
```

### update

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/ui/src/virtualList/index.ts#L717)

### view

function

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/ui/src/virtualList/index.ts#L1337)

```
<Item>(): ViewForItem<Item>
```

## Types

### InitConfig

type

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/ui/src/virtualList/index.ts#L170)

```
/** Configuration for creating a virtual list model with `init`. */
type InitConfig = Readonly<{
  followEnd: Readonly<{
    thresholdPx: number
  }>
  id: string
  initialScroll: Readonly<{
    alignment: ScrollAlignment
    target: ScrollTarget
  }>
  initialScrollTop: number
  rowHeightPx: number
}>
```

### RowHeightInputs

type

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/ui/src/virtualList/index.ts#L1308)

```
/**
 * Mutually exclusive configuration for fixed, exact variable, or measured
 *  dynamic row heights.
 */
type RowHeightInputs = Readonly<{
  dynamicRowHeights: undefined
  itemToEstimatedRowHeightPx: never
  itemToRowHeightPx: undefined
}> | Readonly<{
  dynamicRowHeights: true
  itemToEstimatedRowHeightPx: (item: Item, index: number) => number
  itemToRowHeightPx: never
}> | Readonly<{
  dynamicRowHeights: never
  itemToEstimatedRowHeightPx: never
  itemToRowHeightPx: (item: Item, index: number) => number
}>
```

### ScrollToOptions

type

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/ui/src/virtualList/index.ts#L530)

```
/** Options shared by row-targeted programmatic scrolling helpers. */
type ScrollToOptions = Readonly<{
  alignment: ScrollAlignment
}>
```

### ViewInputs

type

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/ui/src/virtualList/index.ts#L1325)

```
type ViewInputs = BaseViewInputs<Item> & RowHeightInputs<Item>
```

## Constants

### ContentAlignment

const

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/ui/src/virtualList/index.ts#L874)

```
/** Alignment of content when its total height is shorter than the viewport. */
const ContentAlignment: Literals<readonly ["Start", "End"]>
```

### Message

const

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/ui/src/virtualList/index.ts#L139)

```
/** Union of all messages the virtual list component can produce. */
const Message: MessageUnion<{
  CompletedApplyScroll: {
    outcome: TaggedUnion<{
      Applied: {
        anchor: TaggedUnion<{
          None: {}
          Row: {
            index: Number
            key: String
            viewportOffset: Number
          }
        }>
        containerHeight: Number
        scrollHeight: Number
        scrollTop: Number
      }
      Skipped: {}
    }>
    version: Number
  }
  MeasuredRows: {
    measurements: $Array<Struct<{
      height: Number
      key: String
      layoutVersion: Number
    }>>
  }
  ObservedContainerScroll: {
    anchor: TaggedUnion<{
      None: {}
      Row: {
        index: Number
        key: String
        viewportOffset: Number
      }
    }>
    containerHeight: Number
    scrollHeight: Number
    scrollTop: Number
  }
  ResizedContainer: {
    containerHeight: Number
    containerWidth: Number
  }
}>
```

### Model

const

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/ui/src/virtualList/index.ts#L120)

```
/**
 * Schema for the virtual list's state. Tracks scroll position, container
 *  measurement, and any in-flight programmatic scroll.
 */
const Model: Struct<{
  endBehavior: TaggedUnion<{
    Follow: {
      thresholdPx: Number
    }
    PreserveAnchor: {}
  }>
  id: String
  initialScroll: TaggedUnion<{
    Applied: {}
    Pending: {
      alignment: Literals<readonly ["Start", "Center", "End", "Nearest"]>
      target: TaggedUnion<{
        End: {}
        Index: {
          index: Number
        }
        Key: {
          key: String
        }
        Offset: {
          offset: Number
        }
      }>
    }
  }>
  layoutVersion: Number
  measuredRowHeights: $Record<String, Number>
  measurement: TaggedUnion<{
    Measured: {
      containerHeight: Number
      containerWidth: Number
    }
    Unmeasured: {}
  }>
  pendingScroll: TaggedUnion<{
    Idle: {}
    Pending: {
      request: TaggedUnion<{
        Anchor: {
          anchor: TaggedUnion<{
            End: {}
            Offset: {
              scrollTop: unknown
            }
            Row: {
              index: unknown
              key: unknown
              viewportOffset: unknown
            }
          }>
        }
        Target: {
          alignment: Literals<readonly ["Start", "Center", "End", "Nearest"]>
          target: TaggedUnion<{
            End: {}
            Index: {
              index: unknown
            }
            Key: {
              key: unknown
            }
            Offset: {
              offset: unknown
            }
          }>
        }
      }>
      version: Number
    }
  }>
  pendingScrollVersion: Number
  rowHeightPx: Number
  scrollTop: Number
  viewportAnchor: TaggedUnion<{
    End: {}
    Offset: {
      scrollTop: Number
    }
    Row: {
      index: Number
      key: String
      viewportOffset: Number
    }
  }>
}>
```

### ScrollAlignment

const

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/ui/src/virtualList/index.ts#L46)

```
/** Alignment of a row within the viewport after a programmatic scroll. */
const ScrollAlignment: Literals<readonly ["Start", "Center", "End", "Nearest"]>
```

### ScrollTarget

const

[source](https://github.com/foldkit/foldkit/blob/74071173b1253e9efeec050a31ca86df1931ce5a/packages/ui/src/virtualList/index.ts#L56)

```
/** Logical destination for initial and programmatic scrolling. */
const ScrollTarget: TaggedUnion<{
  End: {}
  Index: {
    index: Number
  }
  Key: {
    key: String
  }
  Offset: {
    offset: Number
  }
}>
```
