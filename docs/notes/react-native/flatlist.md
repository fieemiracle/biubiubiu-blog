---
title: FlatList&左滑删除组件
createTime: 2025/11/17 14:28:22
permalink: /article/b4m3rufl/
---

## index.tsx

```typescript
import { useEffect, useState, useRef } from 'react'
import {
  StyleSheet,
  View,
  Text,
  Image,
  FlatList,
  TouchableWithoutFeedback,
} from 'react-native'

type MemoryItemT = {
  k_1: string
  k_2: string
  line_1: string
  line_2: string
  create_at: number
}

const ITEM_HEIGHT = 60
const MAX_DISPLAY_COUNT = 10

type MemoryStatus = 'loading' | 'empty' | 'list'
enum MEMORY_STATUS {
  LOADING = 'loading',
  EMPTY = 'empty',
  LIST = 'list',
}

const App = () => {
  const [memoryList, setMemoryList] = useState<MemoryItemT[]>([])
  const [memoryStatus, setMemoryStatus] = useState<MemoryStatus>(
    MEMORY_STATUS.LOADING
  )
  const [activeIndex, setActiveIndex] = useState(-1)
  const [isHorizontalSwiping, setIsHorizontalSwiping] = useState(false)
  const [canResetSwiper, setCanResetSwiper] = useState(false)

  // 滑动删除相关事件
  const onSwiperStatusChange = (isMoving: boolean, itemIndex: number) => {
    if (isMoving) {
      setActiveIndex(itemIndex)
      setIsHorizontalSwiping(true)
    } else {
      setActiveIndex(-1)
      setIsHorizontalSwiping(false)
    }
  }
  const onSwiperTouch = (showDelIcon: boolean, itemIndex: number) => {
    if (activeIndex !== itemIndex) {
      setActiveIndex(-1)
      setCanResetSwiper(true)
    }
  }
  const onSwiperContainerPressOut = () => {
    if (activeIndex !== -1) {
      setActiveIndex(-1)
      setCanResetSwiper(true)
    }
  }

  return (
    <TouchableWithoutFeedback onPress={onSwiperContainerPressOut}>
      <View style={styles.memoryScrollView}>
        {memoryStatus == MEMORY_STATUS.LIST ? (
          <FlatList
            horizontal={false}
            style={{ flex: 1 }}
            data={memoryList}
            keyExtractor={(item) => item.k_1}
            renderItem={({ item, index }) => (
              <SwiperDelItem
                key={item.k_1}
                name={item.line_1}
                itemIndex={index}
                address={item.line_2}
                actived={activeIndex === index}
                canResetSwiper={canResetSwiper}
                onDelete={() => handleDeleteMemory(item)}
                onItemClick={(showDelIcon, itemIndex) => {
                  onSwiperTouch(showDelIcon, itemIndex)
                }}
                onSwipingChange={(isMoving) => {
                  onSwiperStatusChange(isMoving, index)
                }}
              ></SwiperDelItem>
            )}
            showsVerticalScrollIndicator={false}
            scrollEnabled={!isHorizontalSwiping && memoryList.length > MAX_DISPLAY_COUNT}
            overScrollMode='auto'
            removeClippedSubviews={true} // 移除不可见的子组件
            initialNumToRender={MAX_DISPLAY_COUNT} // 初始渲染数量
            maxToRenderPerBatch={MAX_DISPLAY_COUNT} // 每批渲染数量
            getItemLayout={(data, index) => (
              {length: ITEM_HEIGHT, offset: ITEM_HEIGHT * index, index}
            )}
          ></FlatList>
        ) : (
          memoryStatus == MEMORY_STATUS.EMPTY && (
            <View style={styles.emptyContainer}>
              <EmptyIcon />
              <Text style={styles.emptyText}>这里还没有内容</Text>
            </View>
          )
        )}
      </View>
    </TouchableWithoutFeedback>
  )
}

const EmptyIcon = () => (
  <View style={styles.emptyIcon}>
    <Image
      source={{
        uri: ICONS_LIST.SETTING_MEMORY_MANAGE_ICON,
      }}
      style={styles.historyIcon}
    ></Image>
  </View>
)

const styles = StyleSheet.create({
  memoryScrollView: {
    flex: 1,
    marginTop: 20,
  },
  // empty component
  emptyContainer: {
    flex: 1,
    alignItems: 'center',
  },
  emptyIcon: {
    width: 42,
    height: 42,
    borderRadius: 42,
    backgroundColor: '#E5EAF0',
    justifyContent: 'center',
    alignItems: 'center',
    marginBottom: 10,
    marginTop: 100,
  },
  historyIcon: {
    width: 18,
    height: 18,
  },
  emptyText: {
    fontSize: 13,
    color: '#89888F',
  },
})
```

## swiper.tsx

```typescript
import { useEffect, useState, useRef } from 'react'
import {
  StyleSheet,
  View,
  Text,
  Image,
  TouchableWithoutFeedback,
  PanResponder,
  Animated,
} from 'react-native'
import { ICONS_LIST } from '@/common/constant'

const SWIPE_THRESHOLD = 10 // 滑动阈值
// 限制左滑最大距离 DeleteIcon的width + DeleteIcon的marginLeft
const TRANSLATE_X = 60
const SHOW_DELETE_X = 40

type SwiperDelItemPropsT = {
  name: string
  address: string
  itemIndex: number
  actived: boolean
  canResetSwiper: boolean
  onDelete?: () => void
  onItemClick?: (showDelIcon: boolean, itemIndex: number) => void
  onSwipingChange?: (isMoving: boolean) => void
  getItemHeight?: (itemHeight: number) => void
}

export const SwiperDelItem = (props: SwiperDelItemPropsT) => {
  const {
    name, address, itemIndex, actived, canResetSwiper,
    onDelete, onItemClick, onSwipingChange, getItemHeight,
  } = props
  const [showDelIcon, setShowDelIcon] = useState(false)
  const translateX = useRef(new Animated.Value(0)).current
  const isHorizontalSwiping = useRef(false)

  // 删除
  const onDeleteMemory = () => {
    onDelete?.()
    translateX.setValue(0)
    setShowDelIcon(false)
    onSwipingChange?.(false)
  }

  // 手势
  const panResponder = useRef(
    PanResponder.create({
      // 决定是否在触摸开始时（手指按下）就捕获这个手势
      onStartShouldSetPanResponder: () => false,
      // 决定是否在手指移动时捕获手势
      onMoveShouldSetPanResponder: (_, gesture) => {
        if (isHorizontalSwiping.current) return true
        const { dx, dy } = gesture
        const isHorizontal = Math.abs(dx) > Math.abs(dy)
        const hasMinimumDistance = Math.abs(dx) > SWIPE_THRESHOLD
        const shouldRespond = isHorizontal && hasMinimumDistance
        if (shouldRespond) {
          isHorizontalSwiping.current = true
        } else {
          isHorizontalSwiping.current = false
        }
        return shouldRespond
      },
      // 手势被授权给当前组件时调用（相当于触摸开始）
      onPanResponderGrant() {
        onItemClick?.(showDelIcon, itemIndex)
      },
      // 处理手指移动事件
      onPanResponderMove(e, gesture) {
        const { dx } = gesture
        const constrainedDx = Math.max(dx, -TRANSLATE_X)
        
        // 左滑
        if (dx < 0) {
          if (!showDelIcon) {
            setShowDelIcon(true)
            onSwipingChange?.(true)
          }
          const clampedDx = Math.max(constrainedDx, Math.min(0, gesture.dx))
          translateX.setValue(clampedDx)
        } else {
          if (showDelIcon) {
            const clampedDx = Math.min(0, Math.max(constrainedDx, gesture.dx))
            translateX.setValue(clampedDx)
          }
        }
      },
      // 处理手指抬起事件（触摸结束）
      onPanResponderRelease(e, gesture) {
        const { dx, vx } = gesture
        
        // 根据滑动距离和速度决定最终行为
        const shouldDelete = dx < -SHOW_DELETE_X / 2 || vx < -0.5
        const toValue = shouldDelete ? -TRANSLATE_X : 0
        
        Animated.timing(translateX, {
          toValue,
          duration: 200,
          useNativeDriver: true,
        }).start(() => {
          if (toValue === 0) {
            setShowDelIcon(false)
            onSwipingChange?.(false)
            isHorizontalSwiping.current = false
          }
        })
      },
      // 手势被系统或其他组件中断时调用
      onPanResponderTerminate() {
        translateX.setValue(0)
        setShowDelIcon(false)
        onSwipingChange?.(false)
        isHorizontalSwiping.current = false
      },
      // 决定是否阻止原生组件响应这个手势
      onShouldBlockNativeResponder: () => isHorizontalSwiping.current,
    })
  ).current

  const onItemPress = () => {
    onItemClick?.(showDelIcon, itemIndex)
  }

  useEffect(() => {
    if (!actived && showDelIcon) {
      translateX.setValue(0)
      setShowDelIcon(false)
      onSwipingChange?.(false)
    }
  }, [actived])

  useEffect(() => {
    if (canResetSwiper && showDelIcon) {
      translateX.setValue(0)
      setShowDelIcon(false)
      onSwipingChange?.(false)
      isHorizontalSwiping.current = false
    }
  }, [canResetSwiper])

  return (
    <Animated.View
      {...panResponder.panHandlers}
      style={[styles.swiperDelContainer, { transform: [{ translateX }] }]}
      onLayout={(e) => {
        const { height } = e.nativeEvent.layout
        getItemHeight?.(height)
      }}
    >
      <TouchableWithoutFeedback onPress={onItemPress}>
      <View style={[styles.swiperTextWrap]}>
        <View >
          <Text style={styles.lineName}>{name}</Text>
        </View>
        <View>
          <Text
            style={styles.lineAddress}
            numberOfLines={1}
            ellipsizeMode='tail'
          >{address}</Text>
        </View>
      </View>
      </TouchableWithoutFeedback>
      {
        showDelIcon && (
          <TouchableWithoutFeedback
            onPress={onDeleteMemory}
          >
            <View>
              <DeleteIcon></DeleteIcon>
            </View>
          </TouchableWithoutFeedback>
        )
      }
    </Animated.View>
  )
}

const styles = StyleSheet.create({
  swiperDelContainer: {
    flexDirection: 'row',
    width: 'auto',
    marginBottom: 10,
    alignItems: 'center',
    marginHorizontal: 20,
  },
  swiperTextWrap: {
    width: '100%',
    height: 60,
    paddingHorizontal: 16,
    paddingTop: 14,
    paddingBottom: 13.5,
    borderRadius: 10,
    boxSizing: 'border-box',
    backgroundColor: '#ffffff'
  },
  lineName: {
    fontSize: 14,
    fontFamily: 'PingFang SC',
    color: '#000000',
    height: 19.5,
  },
  lineAddress: {
    fontSize: 11,
    fontFamily: 'PingFang SC',
    color: '#999999',
    marginTop: 1.5,
    height: 15.5,
  },
})

const DeleteIcon = () => (
  <View style={deleteStyles.deleteContainer}>
    <Image
      source={{
        uri: ICONS_LIST.SETTING_DELETE_MEMORY,
      }}
      style={deleteStyles.deleteIcon}
    ></Image>
  </View>
)

const deleteStyles = StyleSheet.create({
    deleteContainer: {
    width: 40,
    height: 40,
    borderRadius: 42,
    marginLeft: 20,
    backgroundColor: '#FFF',
    justifyContent: 'center',
    alignItems: 'center',
  },
  deleteIcon: {
    width: 36,
    height: 36,
  },
})
```

## mock.ts

```typescript
export const memoryMockData = [
  {
    "k_1": "school",
    "k_2": "0",
    "line_1": "学校地址：清华大学",
    "line_2": "北京市海淀区清华园街道双清路30号清华大学",
    "create_at": 1761740123.789125
  },
  {
    "k_1": "gym",
    "k_2": "0",
    "line_1": "健身地址：力健健身俱乐部",
    "line_2": "上海市浦东新区张江高科技园区博云路2号智慧大厦B座5层",
    "create_at": 1761741345.234567
  },
  {
    "k_1": "hospital",
    "k_2": "0",
    "line_1": "医院地址：协和医院",
    "line_2": "北京市东城区帅府园1号北京协和医院东单院区",
    "create_at": 1761742567.890123
  },
  {
    "k_1": "mall",
    "k_2": "0",
    "line_1": "商场地址：国贸商城",
    "line_2": "北京市朝阳区建国门外大街1号国贸中心三期B1-B3层",
    "create_at": 1761743789.345678
  },
  {
    "k_1": "restaurant",
    "k_2": "0",
    "line_1": "餐厅地址：海底捞火锅",
    "line_2": "广州市天河区珠江新城冼村路3号利通广场4层401",
    "create_at": 1761744911.567890
  },
  {
    "k_1": "station",
    "k_2": "0",
    "line_1": "车站地址：上海虹桥站",
    "line_2": "上海市闵行区申贵路1500号上海虹桥火车站",
    "create_at": 1761746133.890124
  },
  {
    "k_1": "park",
    "k_2": "0",
    "line_1": "公园地址：颐和园",
    "line_2": "北京市海淀区新建宫门路19号颐和园景区",
    "create_at": 1761747355.123456
  },
  {
    "k_1": "library",
    "k_2": "0",
    "line_1": "图书馆地址：国家图书馆",
    "line_2": "北京市海淀区中关村南大街33号中国国家图书馆总馆",
    "create_at": 1761748577.345679
  },
  {
    "k_1": "airport",
    "k_2": "0",
    "line_1": "机场地址：北京首都机场",
    "line_2": "北京市顺义区首都国际机场3号航站楼",
    "create_at": 1761749799.678901
  },
  {
    "k_1": "relative",
    "k_2": "0",
    "line_1": "亲友地址：朝阳公园南门",
    "line_2": "北京市朝阳区朝阳公园南路1号朝阳公园南门小区3号楼",
    "create_at": 1761751021.890125
  },
  {
    "k_1": "coffee",
    "k_2": "0",
    "line_1": "咖啡馆地址：星巴克",
    "line_2": "深圳市南山区科技园高新南一道1号创维大厦1层102",
    "create_at": 1761752243.234567
  },
  {
    "k_1": "bookstore",
    "k_2": "0",
    "line_1": "书店地址：三联韬奋书店",
    "line_2": "北京市东城区美术馆东街22号三联韬奋书店总店",
    "create_at": 1761753465.789012
  },
  {
    "k_1": "cinema",
    "k_2": "0",
    "line_1": "影院地址：万达影城",
    "line_2": "成都市锦江区东大街99号IFS国际金融中心7层万达影城",
    "create_at": 1761754687.345678
  },
  {
    "k_1": "market",
    "k_2": "0",
    "line_1": "菜市场地址：新发地便民市场",
    "line_2": "北京市丰台区花乡新发地村新发地便民菜市场A区15号摊位",
    "create_at": 1761755909.890123
  }
]
```