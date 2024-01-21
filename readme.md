# 文档

### 后端
1. 安装phpstudy, 启动mySql服务
2. 安装navicat, 连接mySql, 创建数据库, 运行sql，导入备用数据库数据
3. 修改zf_back-end数据库配置，npm run start 启动后台，连接数据库

### 前端
1. 安装antd-mobile UI组件，index.js导入全局样式
2. 安装react-router-dom 路由组件，App.js配置路由
3. 写home页面组件，以及页面内嵌的子页面路由。引用antd-mobile的TabBar组件，写home页底部的TabBar以及选中高亮、路由切换
4. 写index子页面组件，包括轮播图、导航栏、搜索框、小组、咨询。通过axios组件请求轮播图数据，并引用antd-mobile Carousel组件，渲染轮播图。引用antd-mobile Flex组件渲染导航栏，并添加页面条跳转。请求小组数据，并引用antd-mobile Grid组件渲染。请求咨询数据，并引用antd-mobile WinBlank组件渲染。引用antd-mobile Flex组件渲染顶部导航。通过百度地图API,获取当前城市的地理位置
5. 写CityList页面组件，通过react-virtualized 组件实现列表的“可视区域渲染”， 通过ref对象实现点击滚动
6. 写Map页面组件，包括地图初始化，添加覆盖物，点击下钻。
7. 写HouseList页面组件，封装顶部导航组件（NavHeader）、条件筛选栏组件(Filter)、react-vertualized实现房屋列表的懒加载
8. 写Login页面组件，使用formik实现表单校验，封装AuthRouter组件给路由添加登录查看权限
9. 写Search、Add页面组件，做搜索防抖和房源信息上传

### 案例
1. 配置基础路由
```javascript
// App.js
// 导入路由
// BrowserRouter 对照 HashRouter 来看
import { BrowserRouter as Router, Route, Redirect } from 'react-router-dom'

export default function App() {
  return (
    <Router>
      <div className="App">
        {/* 默认路由匹配时，跳转到 /home 实现路由重定向到首页 */}
        <Route path="/" exact render={() => <Redirect to="/home" />} />
        {/* 配置路由 */}
        {/* Home 组件是父路由的内容 */}
        <Route path="/home" component={Home} />
        <Route path="/citylist" component={CityList} />
        <Route path="/map" component={Map} />

        {/* 房源详情的路由规则： */}
        <Route path="/detail/:id" component={HouseDetail} />
        <Route path="/login" component={Login} />
        <Route path="/registe" component={Registe} />

        {/* 配置登录后，才能访问的页面 */}
        <AuthRoute exact path="/rent" component={Rent} />
        <AuthRoute path="/rent/add" component={RentAdd} />
        <AuthRoute path="/rent/search" component={RentSearch} />
      </div>
    </Router>
  )
}
```
2. 配置嵌套路由
```javascript
// home/index.js
// 导入路由
import { Route } from 'react-router-dom'
// 导入TabBar菜单的组件
import News from '../News'
import Index from '../Index'
import HouseList from '../HouseList'
import Profile from '../Profile'

export default class Home extends React.Component {
    render() {
    return (
      <div className="home">
        {/* 渲染子路由 */}
        <Route path="/home/news" component={News} />
        <Route exact path="/home" component={Index} />
        <Route path="/home/list" component={HouseList} />
        <Route path="/home/profile" component={Profile} />

        {/* TabBar... */}
      </div>
    )
  }
}
```
3. 全局样式配置
```css
/* index.css */
html,
body {
  margin: 0;
  padding: 0;
  font-family: 'Microsoft YaHei';
  color: #333;
  background-color: #fff;
}

* {
  box-sizing: border-box;
  /* 去掉 轮播图 Chrome浏览器的错误警告 */
  touch-action: pan-y;
}

h1,
h2,
h3,
p,
ul,
dd {
  margin: 0;
}

ul {
  padding: 0;
  list-style: none;
}

html,
body,
#root,
.App {
  height: 100%;
}

/* 解决 找房页面 展示遮罩层后，页面还会滚动的问题 */
.body-fixed {
  overflow: hidden;
}
```
4. 获取轮播图数据
```javascript
// 导入axios
import axios from 'axios'

// 获取轮播图数据的方法
  async getSwipers() {
    const res = await axios.get('http://localhost:8080/home/swiper')
    this.setState({
      swipers: res.data.body,
      isSwiperLoaded: true
    })
  }
```
5. antd-mobile 未加载完图片动态渲染不轮播
```javascript
{/* 轮播图 */}
render() {
    return (
        <div className="swiper">
            {this.state.isSwiperLoaded ? (
            <Carousel autoplay infinite autoplayInterval={5000}>
                {this.renderSwipers()}
            </Carousel>
            ) : (
            ''
            )}
        </div>
    )
}
```
6. index页导航栏跳转，home页没重新加载，TabBar高亮没切换
```javascript
// home/index.js
// 路由信息改变，导致props改变，触发home组件componentDidUpdate()
componentDidUpdate(prevProps) {
    // prevProps 上一次的props，此处也就是上一次的路由信息
    // this.props 当前最新的props，此处也就是最新的路由信息
    // 注意：在该钩子函数中更新状态时，一定要在 条件判断 中进行，否则会造成递归更新的问题
    if (prevProps.location.pathname !== this.props.location.pathname) {
      // 此时，就说明路由发生切换了
      this.setState({
        selectedTab: this.props.location.pathname
      })
    }
  }
```

7. 使用less预编译
需要下载less less-loader, 并运行npm run enject 参照sass修webpack配置

8. 获取当前城市地理位置
```javascript
// utils/index.js
// 创建并导出获取定位城市的函数 getCurrentCity
export const getCurrentCity = () => {
  // 判断 localStorage 中是否有定位城市
  const localCity = JSON.parse(localStorage.getItem('hkzf_city'))
  if (!localCity) {
    // 如果没有，就使用首页中获取定位城市的代码来获取，并且存储到本地存储中，然后返回该城市数据
    return new Promise((resolve, reject) => {
      console.log("百度地图",window.BMapGL)
      const curCity = new window.BMapGLGL.LocalCity()
      curCity.get(async res => {
        try {
          // console.log('当前城市信息：', res)
          const result = await axios.get(
            `http://localhost:8080/area/info?name=${res.name}`
          )
          // result.data.body => { label: '上海', value: '' }

          // 存储到本地存储中
          localStorage.setItem('hkzf_city', JSON.stringify(result.data.body))
          // 返回该城市数据
          // return result.data.body
          resolve(result.data.body)
        } catch (e) {
          // 获取定位城市失败
          reject(e)
        }
      })
    })
  }

  // 如果有，直接返回本地存储中的城市数据
  // 注意：因为上面为了处理异步操作，使用了Promise，因此，为了该函数返回值的统一，此处，也应该使用Promise
  // 因为此处的 Promise 不会失败，所以，此处，只要返回一个成功的Promise即可
  return Promise.resolve(localCity)
}
```

9. 列表数据结构
```javascript
// pages/CityList/index.js
// 获取城市列表数据的方法
  async getCityList() {
    const res = await axios.get('http://localhost:8080/area/city?level=1')
    const { cityList, cityIndex } = formatCityData(res.data.body)

    // 获取热门城市数据
    const hotRes = await axios.get('http://localhost:8080/area/hot')
    cityList['hot'] = hotRes.data.body
    cityIndex.unshift('hot')

    // 获取当前定位城市
    const curCity = await getCurrentCity()
    cityList['#'] = [curCity]
    cityIndex.unshift('#')

    this.setState({
      cityList,
      cityIndex
    })
  }
// 数据格式化的方法
// list: [{}, {}]
const formatCityData = list => {
  const cityList = {}

  // 遍历list数组
  list.forEach(item => {
    // 获取每一个城市的首字母
    const first = item.short.substr(0, 1)
    // 判断 cityList 中是否有该分类
    if (cityList[first]) {
      // 如果有，直接往该分类中push数据
      // cityList[first] => [{}, {}]
      cityList[first].push(item)
    } else {
      // 如果没有，就先创建一个数组，然后，把当前城市信息添加到数组中
      cityList[first] = [item]
    }
  })
  // 获取索引数据
  const cityIndex = Object.keys(cityList).sort()

  return {
    cityIndex,
    cityList
  }
}
```

10. 大型列表（>1000）数据"可视区域"渲染
```javascript
// 只渲染可视区域，非可视区域“完全不渲染”
// 文档 https://github.com/bvaughn/react-virtualized
// 导入 List 组件
// 通过 render-props 模式，获取到 AutoSizer 组件暴露的 width 和 height 属性，设置 List 组件的 width 和 height 属性
import { List, AutoSizer } from 'react-virtualized'

// ...class
render() {
    return (
        <div className="citylist">
            {/* 城市列表 */}
            <AutoSizer>
            {({ width, height }) => (
                <List
                ref={this.cityListComponent}
                width={width}
                height={height}
                rowCount={this.state.cityIndex.length}
                rowHeight={this.getRowHeight}
                rowRenderer={this.rowRenderer}
                onRowsRendered={this.onRowsRendered}
                scrollToAlignment="start"
                />
            )}
            </AutoSizer>
        </div>
    )
}
```

11. 右侧索引高亮
```javascript
// List组件属性，用于获取List组件中渲染行的信息
// startIndex是列表可视区域最顶部一行的索引
// index和activeIndex 相等的右侧索引高亮
  onRowsRendered = ({ startIndex }) => {
    // console.log('startIndex：', startIndex)
    if (this.state.activeIndex !== startIndex) {
      this.setState({
        activeIndex: startIndex
      })
    }
  }
```

12. 索引点击跳转
```javascript
// 创建列表的ref，索引点击事件里通过ref current的scrollToRow方法 指定目标列表项

import React from 'react'
import { List, AutoSizer } from 'react-virtualized'

class CityList extends React.Component {
    construct(props) {
        super(props)
        // 创建ref对象
        this.cityListComponent = React.createRef()
    }

    renderCityIndex() {
        // 获取到 cityIndex，并遍历其，实现渲染
        const { cityIndex, activeIndex } = this.state
        return cityIndex.map((item, index) => (
        <li
            className="city-index-item"
            key={item}
            onClick={() => {
                this.cityListComponent.current.scrollToRow(index)
            }}
        >
            <span className={activeIndex === index ? 'index-active' : ''}>
                {item === 'hot' ? '热' : item.toUpperCase()}
            </span>
        </li>
        ))
    }

    render() {
        return (
            <div className="citylist">

                {/* 城市列表 */}
                <AutoSizer>
                {({ width, height }) => (
                    <List
                    ref={this.cityListComponent}
                    width={width}
                    height={height}
                    rowCount={this.state.cityIndex.length}
                    rowHeight={this.getRowHeight}
                    rowRenderer={this.rowRenderer}
                    onRowsRendered={this.onRowsRendered}
                    scrollToAlignment="start"
                    />
                )}
                </AutoSizer>

                {/* 右侧索引列表 */}
                <ul className="city-index">{this.renderCityIndex()}</ul>
            </div>
        )
    }
}
```

13. 添加props校验
```javascript
// components/NavHeader/index.js

function NavHeader({
  children,
  history,
  onLeftClick,
  className,
  rightContent
}) {
// .......
}

// 添加props校验
NavHeader.propTypes = {
  children: PropTypes.string.isRequired,
  onLeftClick: PropTypes.func,
  className: PropTypes.string,
  rightContent: PropTypes.array
}

export default withRouter(NavHeader)
```

14. 组件获取当前路由信息
```javascript
// 导入 withRouter 高阶组件包装，获取history属性
import { withRouter } from 'react-router-dom'

function NavHeader({
  children,
  history,
  onLeftClick,
  className,
  rightContent
}) {

  // 默认点击行为
  const defaultHandler = () => history.go(-1)

  return (
    <NavBar
      className={[styles.navBar, className || ''].join(' ')}
      mode="light"
      icon={<i className="iconfont icon-back" />}
      onLeftClick={onLeftClick || defaultHandler}
      rightContent={rightContent}
    >
      {children}
    </NavBar>
  )
}

export default withRouter(NavHeader)
```

15. CSS IN JS
```javascript
// React已经集成css_module
// 自动生成类名，{filename}_{classname}_{hash}
import styles from './index.module.css'

// class
render() {
    return (
        <div className={style.test}></div>
    )
}
```

16. 初始化地图
```javascript
// 百度地图开发平台注册应用，获取密钥，引入js的SDK

// pages/Map/index.js
// 创建Map组件，组件内创建地图
initMap() {
    // 获取当前定位城市
    const { label, value } = JSON.parse(localStorage.getItem('hkzf_city'))
    // 初始化地图实例
    const map = new BMapGL.Map('container')
    // 作用：能够在其他方法中通过 this 来获取到地图对象
    this.map = map
    // 创建地址解析器实例
    const myGeo = new BMapGL.Geocoder()
    // 将地址解析结果显示在地图上，并调整地图视野
    myGeo.getPoint(
      label,
      async point => {
        if (point) {
          // 初始化地图, 缩放级别11
          map.centerAndZoom(point, 11)
          // 添加常用（比例尺、平移）控件
          map.addControl(new BMapGL.ScaleControl())
          map.addControl(new BMapGL.NavigationControl())
        }
      },
      label
    )
}
```
 
17. 地图覆盖物点击嵌套

```javascript
// pages/Map/index.js
/**
  1. renderOverlays(): 接收区域id,获取房源数据。 getTypeAndZoom(): 获取覆盖物类型， 获取下级地图覆盖级别。
  2. createOverlays(): 根据覆盖物类型、坐标、级别等，调用创建覆盖物方法
  3. createCircle()/createRect(): 绑定事件：放大地图、清除覆盖物、渲染下一级覆盖物，调用renderOverlays（）递归
 */
```

18. 移动隐藏房源列表
```javascript
// pages/Map/index.js
// 初始化地图实例
const map = new BMapGL.Map('container')

// 给地图绑定移动事件
map.addEventListener('movestart', () => {
    if (this.state.isShowList) {
    this.setState({
        isShowList: false
    })
    }
})
```

19. axios域名配置
```javascript
// 通过脚手架提供环境变量，在开发文件 .env.development 中， 配置
REACT_APP_URL=http://localhost:8080/

// utils/url.js
export const BASE_URL = process.env.REACT_APP_URL

// utils/api.js
import axios from 'axios'
import { BASE_URL } from './url'

// 创建axios示例
const API = axios.create({
  baseURL: BASE_URL
})

export { API }
```

20. 条件筛选栏组件
```javascript
// pages/HouseList/components/Filter/index.js
/**
 * 父组件：Filter
 * 子组件：FilterTitle、FilterPiker、FilterMore
 */
// Filter
import React, { Component } from 'react'

// 筛选栏
import FilterTitle from '../FilterTitle'

// 弹窗
import FilterPicker from '../FilterPicker'
import FilterMore from '../FilterMore'

import styles from './index.module.css'

// 标题高亮状态
// true 表示高亮； false 表示不高亮
const titleSelectedStatus = {
  area: false,
  mode: false,
  price: false,
  more: false
}

// FilterPicker 和 FilterMore 组件的选中值
const selectedValues = {
  area: ['area', 'null'],
  mode: ['null'],
  price: ['null'],
  more: []
}

export default class Filter extends Component {
  state = {
    titleSelectedStatus,
    // 控制 FilterPicker 或 FilterMore 组件的展示或隐藏
    openType: '',
    // 所有筛选条件数据
    filtersData: {
      // FilterMore
      roomType: [],
      oriented: [],
      floor: [],
      characteristic: [],
      // FilterPicker
      area: {},
      subway: {},
      rentType: [],
      price: []
    },
    // 筛选条件的选中值
    selectedValues
  }

  // 点击标题菜单实现高亮
  // 注意：this指向的问题！！！
  onTitleClick = type => {
    // 给 body 添加样式
    this.htmlBody.className = 'body-fixed'

    const { titleSelectedStatus, selectedValues } = this.state
    // 创建新的标题选中状态对象
    const newTitleSelectedStatus = { ...titleSelectedStatus }

    // 遍历标题选中状态对象
    // Object.keys() => ['area', 'mode', 'price', 'more']
    Object.keys(titleSelectedStatus).forEach(key => {
      // key 表示数组中的每一项，此处，就是每个标题的 type 值。
      if (key === type) {
        // 当前标题
        newTitleSelectedStatus[type] = true
        return
      }

      // 其他标题：
      const selectedVal = selectedValues[key]
      if (
        key === 'area' &&
        (selectedVal.length !== 2 || selectedVal[0] !== 'area')
      ) {
        // 高亮
        newTitleSelectedStatus[key] = true
      } else if (key === 'mode' && selectedVal[0] !== 'null') {
        // 高亮
        newTitleSelectedStatus[key] = true
      } else if (key === 'price' && selectedVal[0] !== 'null') {
        // 高亮
        newTitleSelectedStatus[key] = true
      } else if (key === 'more' && selectedVal.length !== 0) {
        // 更多选择项 FilterMore 组件
        newTitleSelectedStatus[key] = true
      } else {
        newTitleSelectedStatus[key] = false
      }
    })

    this.setState({
      // 展示对话框
      openType: type,
      // 使用新的标题选中状态对象来更新
      titleSelectedStatus: newTitleSelectedStatus
    })
  }

  // 渲染 FilterPicker 组件的方法
  renderFilterPicker() {
    // 解构
    const {
      openType,
      filtersData: { area, subway, rentType, price },
      selectedValues
    } = this.state

    if (openType !== 'area' && openType !== 'mode' && openType !== 'price') {
      return null
    }

    // 根据 openType 来拿到当前筛选条件数据
    let data = []
    let cols = 3
    let defaultValue = selectedValues[openType]
    switch (openType) {
      case 'area':
        // 获取到区域数据
        data = [area, subway]
        cols = 3
        break
      case 'mode':
        data = rentType
        cols = 1
        break
      case 'price':
        data = price
        cols = 1
        break
      default:
        break
    }

    return (
      <FilterPicker
        // 添加key，不同标题切换时，key值不同，React会重新创建组件，重新赋值state
        key={openType}
        // 父子组件，回调函数传参
        onCancel={this.onCancel}
        onSave={this.onSave}

        data={data}
        cols={cols}
        type={openType}
        defaultValue={defaultValue}
      />
    )
  }

   // 渲染 FilterMore 组件
  renderFilterMore() {
    const {
      openType,
      selectedValues,
      filtersData: { roomType, oriented, floor, characteristic }
    } = this.state

    const data = {
      roomType,
      oriented,
      floor,
      characteristic
    }

    const defaultValue = selectedValues.more

    return (
      <FilterMore
        data={data}
        type={openType}

        onSave={this.onSave}
        onCancel={this.onCancel}

        defaultValue={defaultValue}
      />
    )
  }

  onSave(type, value) {
    // ...组装筛选条件

    // 调用父组件中的方法，来将筛选数据传递给父组件
    this.props.onFilter(filters)

    // 隐藏对话框
    this.setState({
      openType: '',

      // 更新菜单高亮状态数据
      titleSelectedStatus: newTitleSelectedStatus,

      // 更新筛选值
      selectedValues: newSelectedValues
    })
  }

  render() {
    const { titleSelectedStatus } = this.state

    return (
      <div className={styles.root}>

        <div className={styles.content}>
          {/* 标题栏 */}
          <FilterTitle
            titleSelectedStatus={titleSelectedStatus}
            onClick={this.onTitleClick}
          />

          {/* 前三个菜单对应的内容： */}
          {this.renderFilterPicker()}

          {/* 最后一个菜单对应的内容： */}
          {this.renderFilterMore()}
        </div>
      </div>
    )
  }
}
```

21. key不同，重新创建组件，赋值state
```javascript
<FilterPicker
  // 添加key，不同标题切换时，key值不同，React会重新创建组件，重新赋值state
  key={openType}
  // 父子组件，回调函数传参
  onCancel={this.onCancel}
  onSave={this.onSave}
  data={data}
  cols={cols}
  type={openType}
  defaultValue={defaultValue}
/>
```

22. List组件实现房屋列表懒加载
```javascript
// pages/HouseList/index.js
import {
  List,
  AutoSizer,
  WindowScroller,
  InfiniteLoader
} from 'react-virtualized'

// ...class
renderList() {
  return (
    // 懒加载，实现无限滚动
    // isRowLoaded 表示是否加载完成
    // loadMoreRows 加载更多数据的方法
    // rowCount 列表数据总条数
    <InfiniteLoader
        isRowLoaded={this.isRowLoaded}
        loadMoreRows={this.loadMoreRows}
        rowCount={count}
      >
      {({ onRowsRendered, registerChild }) => {
        // 提供高度，让List租件随页面滚动
        <WindowScroller>
          {({ height, isScrolling, scrollTop }) => (
            // AutoSizer提高宽度
            <AutoSizer>
              {({ width }) => (
                <List
                  onRowsRendered={onRowsRendered}
                  ref={registerChild}
                  autoHeight // 设置高度为 WindowScroller 最终渲染的列表高度
                  width={width} // 视口的宽度
                  height={height} // 视口的高度
                  rowCount={count} // List列表项的行数
                  rowHeight={120} // 每一行的高度
                  rowRenderer={this.renderHouseList} // 渲染列表项中的每一行
                  isScrolling={isScrolling}
                  scrollTop={scrollTop}
                />
              )}
            </AutoSizer>
          )}
        </WindowScroller>
      }}
    </InfiniteLoader>
  )
}
```

22. 吸顶高阶组件
```javascript
// componets/Sticky/index.js
import React, { Component, createRef } from 'react'

import PropTypes from 'prop-types'

import styles from './index.module.css'

class Sticky extends Component {
  // 创建ref对象
  placeholder = createRef()
  content = createRef()

  // scroll 事件的事件处理程序
  handleScroll = () => {
    const { height } = this.props
    // 获取DOM对象
    const placeholderEl = this.placeholder.current
    const contentEl = this.content.current

    const { top } = placeholderEl.getBoundingClientRect()
    if (top < 0) {
      // 吸顶
      contentEl.classList.add(styles.fixed)
      placeholderEl.style.height = `${height}px`
    } else {
      // 取消吸顶
      contentEl.classList.remove(styles.fixed)
      placeholderEl.style.height = '0px'
    }
  }

  // 监听 scroll 事件
  componentDidMount() {
    window.addEventListener('scroll', this.handleScroll)
  }

  componentWillUnmount() {
    window.removeEventListener('scroll', this.handleScroll)
  }

  render() {
    return (
      <div>
        {/* 占位元素 */}
        <div ref={this.placeholder} />
        {/* 内容元素 */}
        <div ref={this.content}>{this.props.children}</div>
      </div>
    )
  }
}

Sticky.propTypes = {
  height: PropTypes.number.isRequired
}

export default Sticky

// 导入吸顶组件
import Sticky from '../../components/Sticky'

//...class
render() {
  return (
    {/* 条件筛选栏 */}
    <Sticky height={40}>
      <Filter onFilter={this.onFilter} />
    </Sticky>
  )
}
```

23. 遮罩层动画
```javascript
// pages/ouseList/components/Filter/index.js
// 导入 Spring 组件
import { Spring } from 'react-spring/renderprops'

// 渲染遮罩层div
  renderMask() {
    const { openType } = this.state

    return (
      <Spring from={{ opacity: 0 }} to={{ opacity: 1 }}>
        {props => {
          return (
            <div
              style={props}
              className={styles.mask}
              onClick={() => this.onCancel(openType)}
            />
          )
        }}
      </Spring>
    )
  }
```

23. 路由传参
```javascript
// App.js
function App() {
  return (
    <Router>
      <div>
        <Route path="/detail/:id" component={HouseDetail} />
      </div>
    </Router>
  ) 
}

// pages/HouseList/index.js
// ...class
renderHouseList() {
  // ...
  return (
    <HouseItem
        key={key}
        onClick={() => this.props.history.push(`/detail/${house.houseCode}`)}
        // ....
      />
  )
}

// pages/HouseDetail/index.js
// ...class
const { id } = this.props.match.params
```

23. 表单校验formik
```javascript
// pages/login/index.js
import React, { Component } from 'react'
// 导入withFormik
import { withFormik, Form, Field, ErrorMessage } from 'formik'
// 导入Yup
import * as Yup from 'yup'

class Login extends Component {
  // ....
  render() {
    return(
      // ....
      // Form, Field, ErrorMessage
    )
  }
}

Login = withFormik({
   // 提供状态：
  mapPropsToValues: () => ({ username: '', password: '' }),

  // 添加表单校验规则
  validationSchema: Yup.object().shape({
    username: Yup.string()
      .required('账号为必填项')
      .matches(REG_UNAME, '长度为5到8位，只能出现数字、字母、下划线'),
    password: Yup.string()
      .required('密码为必填项')
      .matches(REG_PWD, '长度为5到12位，只能出现数字、字母、下划线')
  }),

  // 表单的提交事件
  handleSubmit: async (values, { props }) => {
    // 获取账号和密码
    const { username, password } = values

    // ...
  }
})(Login)

export default Login
```

24. axios拦截
```javascript
import axios from 'axios'
import { BASE_URL } from './url'

import { getToken, removeToken } from './auth'

// 创建axios示例
const API = axios.create({
  baseURL: BASE_URL
})

// 添加请求拦截器
API.interceptors.request.use(config => {
  const { url } = config
  if (
    url.startsWith('/user') &&
    !url.startsWith('/user/login') &&
    !url.startsWith('/user/registered')
  ) {
    // 添加请求头
    config.headers.Authorization = getToken()
  }
  return config
})

// 添加响应拦截器
API.interceptors.response.use(response => {
  const { status } = response.data
  if (status === 400) {
    // 此时，说明 token 失效，直接移除 token 即可
    removeToken()
  }
  return response
})

export { API }
```

25. 路由鉴定
```javascript
// App.js
{/* 配置登录后，才能访问的页面 */}
<AuthRoute exact path="/rent" component={Rent} />
<AuthRoute path="/rent/add" component={RentAdd} />
<AuthRoute path="/rent/search" component={RentSearch} />

// components/AuthRoute/index.js
import React from 'react'
import { Route, Redirect } from 'react-router-dom'
import { isAuth } from '../../utils'

const AuthRoute = ({ component: Component, ...rest }) => {
  return (
    <Route {...rest} 
      render={props => {
        const isLogin = isAuth()

        if (isLogin) {
          // 已登录
          // 将 props 传递给组件，组件中才能获取到路由相关信息
          return <Component {...props} />
        } else {
          // 未登录
          return (
            <Redirect
              to={{
                pathname: '/login',
                state: {
                  from: props.location
                }
              }}
            />
          )
        }
      }}
    />
  )
}

export default AuthRoute
```

26. 输入搜索防抖
```javascript
// pages/Rent/Search/index.js
/* 
  关键词搜索小区信息,监听输入框change事件
*/
handleSearchTxt = value => {
  this.setState({ searchTxt: value })

  if (!value) {
    // 文本框的值为空
    return this.setState({
      tipsList: []
    })
  }

  // 清除上一次的定时器
  clearTimeout(this.timerId)

  this.timerId = setTimeout(async () => {
    // 获取小区数据
    const res = await API.get('/area/community', {
      params: {
        name: value,
        id: this.cityId
      }
    })

    this.setState({
      tipsList: res.data.body
    })
  }, 500)
}
```

27. 上传图片
```javascript
// pages/Rent/Add/index.js
// 将图片对象 file 添加到 form中，传递form参数，调用图片上传接口，获取线上路径
// ...render
<ImagePicker
  files={tempSlides}
  onChange={this.handleHouseImg}
  multiple={true}
  className={styles.imgpicker}
/>

handleHouseImg = (files, type, index) => {
  this.setState({
    tempSlides: files
  })
}

// ...addHose
if (tempSlides.length > 0) {
  // 已经有上传的图片了
  const form = new FormData()
  tempSlides.forEach(item => form.append('file', item.file))

  const res = await API.post('/houses/image', form, {
    headers: {
      'Content-Type': 'multipart/form-data'
    }
  })

  houseImg = res.data.body.join('|')
}
```

28. antD-mobile css 按需加载

29. react 的 lazy Suspense 组件实现基于路由的代码分割，实现页面动态加载

30. react-vertualized只加载用到的组件

31. 脚手架配置代理解决跨域问题
















