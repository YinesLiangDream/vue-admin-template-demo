<template>
  <el-breadcrumb class="app-breadcrumb" separator="/">
    <transition-group name="breadcrumb">
      <el-breadcrumb-item v-for="(item, index) in levelList" :key="item.path">
        <span v-if="index === 0">当前位置：</span>
        <span
          v-if="item.redirect === 'noRedirect' || index == levelList.length - 1"
          class="no-redirect"
        >{{ item.meta.title }}</span>
        <a v-else @click.prevent="handleLink(item)">{{ item.meta.title }}</a>
      </el-breadcrumb-item>
    </transition-group>
  </el-breadcrumb>
</template>

<script>
import pathToRegexp from 'path-to-regexp'
import { constantRoutes } from '@/router'
export default {
  data() {
    return {
      levelList: null,
      levelListes: null,
      routes: constantRoutes
    }
  },
  watch: {
    $route() {
      this.getBreadcrumb()
    }
  },
  created() {
    this.getBreadcrumb()
  },
  methods: {
    getBreadcrumb() {
      // only show routes with meta.title
      let matched = this.$route.matched.filter(
        item => item.meta && item.meta.title
      )
      // console.log('this.$route.matched', this.$route.matched)
      const first = matched[0]

      if (!this.isDashboard(first)) {
        matched = [{ path: '/dashboard', meta: { title: 'Dashboard' }}].concat(
          matched
        )
      }

      this.levelListes = matched.filter(
        item => item.meta && item.meta.title && item.meta.breadcrumb !== false
      )
      // this.levelList
      // for (var i in this.levelListes) {
      //   if (this.levelListes[i].meta.title === 'Dashboard ') {
      //     this.levelListes[i].meta.title = '首页'
      //     this.levelList = this.levelListes
      //     console.log(this.levelListes, 'this.levelListes')
      //   }
      // }
      this.levelList = this.levelListes.map(item => {
        // return item.meta.title =
        if (item.meta.title === 'Dashboard') {
          item.meta.title = '首页'
        }
        return item
      })
      // console.log(this.levelList, 'arr')
    },
    isDashboard(route) {
      const name = route && route.name
      if (!name) {
        return false
      }
      return name.trim().toLocaleLowerCase() === 'Dashboard'.toLocaleLowerCase()
    },
    pathCompile(path) {
      // To solve this problem https://github.com/PanJiaChen/vue-element-admin/issues/561
      const { params } = this.$route
      var toPath = pathToRegexp.compile(path)
      return toPath(params)
    },
    handleLink(item) {
      const { redirect, path } = item
      if (redirect) {
        this.$router.push(redirect)
        return
      }
      this.handleSelect(item.path)
      // 点击头部的
      this.$router.push(this.pathCompile(path))
    },
    // 跳转对应的左边列表内容
    handleSelect(key, keyPath) {
      // 把选中路由的子路由保存store
      const route = this.routes.find(item => item.redirect === key)
      this.$store.commit('permission/SET_CURRENT_ROUTES', route)
      this.setSidebarHide(route)
    },
    // 设置侧边栏的显示和隐藏
    setSidebarHide(route) {
      if (!route.children || route.children.length === 1) {
        this.$store.dispatch('app/toggleSideBarHide', true)
      } else {
        this.$store.dispatch('app/toggleSideBarHide', false)
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.app-breadcrumb.el-breadcrumb {
  display: inline-block;
  font-size: 14px;
  line-height: 50px;
  margin-left: 8px;

  .no-redirect {
    color: #97a8be;
    cursor: text;
  }
}
</style>
