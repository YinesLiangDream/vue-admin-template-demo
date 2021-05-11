<template>
  <div class="top-nav">
    <div class="BossPi">
      <div class="log">搬家工具</div>
      <!-- 路由 -->
      <el-menu
        :active-text-color="variables.menuActiveText"
        :default-active="activeMenu"
        mode="horizontal"
        @select="handleSelect"
      >
        <div v-for="item in routes" :key="item.path" class="nav_item">
          <app-link :to="resolvePath(item)">
            <el-menu-item v-if="!item.hidden" :index="item.path">{{
              item.meta ? item.meta.title : item.children[0].meta.title
            }}</el-menu-item>
          </app-link>
        </div>
      </el-menu>
      <!-- 显示登录案例位置 -->
    </div>
    <div class="right-menu" @click="RightMenu">
      <!-- 消息 -->
      <div class="ElBadge">
        <el-badge
          :value="userinfor.solidValue"
          :hidden="userinfor.solidValue === 0 ? true : false"
          class="item"
        >
          <i
            class="el-icon-message-solid news"
            :class="{ zwyShake: userinfor.solidValue > 0 }"
          />
        </el-badge>
      </div>
      <!-- 登录 -->
      <div class="SignIn">
        <div class="headPortrait">
          <el-avatar :src="userinfor.avatar" />
        </div>
        <div class="designation">
          {{ name }}
          <i class="el-icon-caret-bottom" />
        </div>
      </div>
      <!-- 弹窗 -->
      <el-popover placement="bottom" width="400" trigger="click"> ></el-popover>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import AppLink from './Sidebar/Link'
import { constantRoutes } from '@/router'
import variables from '@/styles/variables.scss'
import { isExternal } from '@/utils/validate'

export default {
  name: 'Topbar',
  components: {
    AppLink
  },
  data() {
    return {
      routes: constantRoutes,
      userinfor: {
        solidValue: 10,
        avatar:
          'https://cube.elemecdn.com/0/88/03b0d39583f48206768a7534e55bcpng.png'
      }
    }
  },
  computed: {
    activeMenu() {
      const route = this.$route
      const { meta, path } = route
      // if set path, the sidebar will highlight the path you set
      if (meta.activeMenu) {
        return meta.activeMenu
      }
      // 如果是首页，首页高亮
      if (path === '/dashboard' || path === '/DataInformation') {
        return '/'
      }
      // 如果不是首页，高亮一级菜单
      const activeMenu = '/' + path.split('/')[1]
      return activeMenu
    },
    variables() {
      return variables
    },
    sidebar() {
      return this.$store.state.app.sidebar
    },
    ...mapGetters(['avatar', 'name'])
  },
  watch: {
    $route: {
      handler() {
        this.$store.commit(
          'permission/SET_CURRENT_ROUTES',
          JSON.parse(localStorage.getItem('route'))
        )
        this.setSidebarHide(JSON.parse(localStorage.getItem('route')))
      },
      deep: true
    }
  },
  mounted() {
    this.initCurrentRoutes()
  },
  methods: {
    // 通过当前路径找到二级菜单对应项，存到store，用来渲染左侧菜单
    initCurrentRoutes() {
      const { path } = this.$route
      let route = this.routes.find(
        item => item.path === '/' + path.split('/')[1]
      )
      // 如果找不到这个路由，说明是首页
      if (!route) {
        route = this.routes.find(item => item.path === '/')
      }
      this.$store.commit('permission/SET_CURRENT_ROUTES', route)
      this.setSidebarHide(route)
    },
    // 判断该路由是否只有一个子项或者没有子项，如果是，则在一级菜单添加跳转路由
    isOnlyOneChild(item) {
      if (item.children && item.children.length === 1) {
        return true
      }
      return false
    },
    resolvePath(item) {
      // 如果是个完成的url直接返回
      if (isExternal(item.path)) {
        return item.path
      }
      // 如果是首页，就返回重定向路由
      if (item.path === '/') {
        const path = item.redirect
        return path
      }

      // 如果有子项，默认跳转第一个子项路由
      let path = ''
      /**
       * item 路由子项
       * parent 路由父项
       */
      const getDefaultPath = (item, parent) => {
        // 如果path是个外部链接（不建议），直接返回链接，存在个问题：如果是外部链接点击跳转后当前页内容还是上一个路由内容
        if (isExternal(item.path)) {
          path = item.path
          return
        }
        // 第一次需要父项路由拼接，所以只是第一个传parent
        if (parent) {
          path += parent.path + '/' + item.path
        } else {
          path += '/' + item.path
        }
        // 如果还有子项，继续递归
        if (item.children) {
          getDefaultPath(item.children[0])
        }
      }

      if (item.children) {
        getDefaultPath(item.children[0], item)
        return path
      }

      return item.path
    },
    handleSelect(key, keyPath) {
      // console.log(key, 'key')
      // 把选中路由的子路由保存store
      const route = this.routes.find(item => item.path === key)
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
    },
    async logout() {
      await this.$store.dispatch('user/logout')
      this.$router.push(`/login?redirect=${this.$route.fullPath}`)
    },
    RightMenu() {
      // window.location.href = 'http://localhost:9528/#/example/table'
      // 把选中路由的子路由保存store
      this.handleSelect('/example')
      this.$router.push('/example/table')
    }
  }
}
</script>
