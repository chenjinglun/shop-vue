/<template>
  <el-container class="home-container">
      <!--头部区域-->
    <el-header>
        <div>
            <img src="../assets/vue.png" alt="">
            <span>电商管理系统</span>
        </div>
        <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!--页面主体区域-->
    <el-container>
      <!--侧边栏-->
      <el-aside :width="isCollapse ? '64px' : '200px'">
          <div class="toggle-button" @click="toggleCollapse">|||</div>
          <!--侧边栏菜单区域-->
          <el-menu background-color="#333744" text-color="#fff" active-text-color="#ffd04b" unique-opened :collapse="isCollapse" :collapse-transition="false" :router="true" :default-active="activePath">
              <!--一级菜单-->
            <el-submenu :index="item.ckCode" v-for="item in menulist" :key="item.ckCode">
                <!--一级菜单的模板区域-->
               <template slot="title">
                   <!--图标-->
                  <i :class="isconsObj[item.ckCode]"></i>
                  <span>{{item.ckName}}</span>
               </template>
               <!--二级菜单-->
                <el-menu-item :index="'/zl' + subItem.ckCode" v-for="subItem in item.ejCkList" :key="subItem.ckCode" @click="saveNavState('/zl' + subItem.ckCode)">
                <template slot="title">
                   <!--图标-->
                  <i class="el-icon-help"></i>
                  <span>{{subItem.ckName}}</span>
               </template>
                </el-menu-item>
            </el-submenu>
          </el-menu>
      </el-aside>
      <!--右侧内容主体-->
      <el-main><router-view></router-view></el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
    data() {
        return {
            menulist: [],
            isconsObj: {
                ck03: 'el-icon-user',
                ck06: 'el-icon-warning',
                ck109: 'el-icon-success'
            },
            isCollapse: false,
            activePath: ''
        }
    },
    created() {
        this.getBmList()
        this.activePath = window.sessionStorage.getItem('activePath')
    },
    methods: {
        logout() {
            window.sessionStorage.clear();
            this.$router.push('/login')
        },
        async getBmList() {
        const { data: res } = await this.$http.get('zl/YjAndEjCk');
        if (res.code !== 200) return this.$message.error('部门列表获取失败');
        this.menulist = res.data;
        console.log(res)
    },
    toggleCollapse() {
        this.isCollapse = !this.isCollapse
    },
    saveNavState(activePath) {
        window.sessionStorage.setItem('activePath', activePath)
        this.activePath = activePath
    }
    }

}
</script>

<style lang="less" scoped>
.home-container{
    height: 100%;
}
.el-header {
    background-color: #373D41;
    display: flex;
    justify-content: space-between;
    padding-left: 0;
    align-items: center;
    color: #fff;
    font-size: 20px;
    > div {
        display: flex;
        align-items: center;
    }
}
.el-aside {
    background-color: #333744;
    .el-menu {
        border-right: none;
    }
}
.el-main {
    background-color: #EAEDF1;
}
.toggle-button{
    background-color: #4A5064;
    font-size: 15px;
    text-align: center;
    line-height: 24px;
    color: #fff;
    letter-spacing: 0.2em;
    cursor: pointer;
}
</style>
