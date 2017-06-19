---
title: RN 中使用mobx来进行状态管理
date: 2017-06-13 11:25:12
tags: react-native
---

在使用RN开发app过程中需要状态管理，也就是state 这个概念，由此衍生出来的两个状态管理的框架，一个是redux，另一个就是mobx了。这两个我都用过，今天就先来介绍下mobx 在开发RN中的使用。实际上不使用状态管理的框架，你也是能够进行开发的，但是随着项目的开发以及项目的壮大，会发现没有状态管理，这个工程越来越难维护。当然redux和mobx也都是可以使用在react的web项目中。

mobx核心的理念是**动作**改变**状态**，而状态的改变会更新所有受影响的**视图**，通俗来讲，一个应用可以拥有一个大的state，而每一个页面都和这个这个state相关连，通过改变这个state来更新页面。当然一个应用可以拥有多个state不一定是一个，这个开每个开发者对项目的规划。举个例子：用户的state,在用户未登录时，页面的显示时登录界面，当用户点击登录时，通过更新这个state，来更新这个view。更多的mobx概念请移步[mobx中文文档](http://cn.mobx.js.org/)

接下来就是在实际开发中使用mobx了

<!-- more -->

- 在package.json 引入

  ```js
  "dependencies": {
      "mobx": "^2.6.1",
      "mobx-react": "^3.5.8",
      //其他库
  }
  //不加这个装饰器使用不了，如@action   
  "devDependencies": {
    "babel-plugin-transform-decorators-legacy": "^1.3.4"
  }
  ```

- .babelrc 中添加

  ```js
  {
    "presets": ["react-native"],
    "plugins": ["transform-decorators-legacy"] //在babel转义的时候能够将装饰器转义
  }
  ```


### 创建一个简易的计数器

- 创建一个一个store

  ```js
  import {observable, action, computed, toJS} from 'mobx'

  class mobxStore {
      @observable count = 0

      @action addCount() {
          this.count = this.count + 1;
      }

      @action reduceCount() {
          this.count = this.count - 1;
      }
  }

  const store = new mobxStore()
  export default store
  ```

- 在界面中和store 关联

  ```javascript
  import React, {Component} from 'react';
  import {
      Text,
      StyleSheet,
      TouchableHighlight,
      View
  } from 'react-native';
  import {observer} from 'mobx-react/custom'
  import store from './mobxStore'

  @observer
  class Root extends Component {

      render() {
          return (
              <View style={styles.container}>
                  <Text style={styles.welcome}>
                      Mobx 的简易使用demo
                  </Text>
                  <Text style={styles.instructions}>
                      store中的计数器：{store.count}
                  </Text>
                  <TouchableHighlight style={{marginTop: 8}} onPress={() => store.addCount()}>
                      <Text style={styles.instructions}>
                          点击计数+1
                      </Text>
                  </TouchableHighlight>
                  <TouchableHighlight style={{marginTop: 8}} onPress={() => store.reduceCount()}>
                      <Text style={styles.instructions}>
                          点击计数-1
                      </Text>
                  </TouchableHighlight>
              </View>
          );
      }
  }

  const styles = StyleSheet.create({
      container: {
          flex: 1,
          justifyContent: 'center',
          alignItems: 'center',
          backgroundColor: '#F5FCFF',
      },
      welcome: {
          fontSize: 20,
          textAlign: 'center',
          margin: 10,
      },
      instructions: {
          textAlign: 'center',
          color: 'red',
          marginBottom: 5,
      },
  });

  export default Root
  ```

导入`import {observer} from 'mobx-react/custom'` 和刚才创建的store，`import store from './mobxStore'`

demo 的源码已经放到github上 [mobxDemo](https://github.com/Cocoon-break/mobxDemo)，如有疑问请在该仓库中提issue


