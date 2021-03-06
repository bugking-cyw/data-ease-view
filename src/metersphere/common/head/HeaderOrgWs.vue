<template>
  <el-menu
    :unique-opened="true"
    mode="horizontal"
    router
    class="header-user-menu align-right"
    background-color="#2c2a48"
    active-text-color="#fff"
    default-active="1"
    text-color="#fff"
  >
    <el-menu-item v-show="false" index="1">Placeholder</el-menu-item>
    <el-submenu
      v-roles="['org_admin', 'test_manager', 'test_user', 'test_viewer']"
      index="1"
      popper-class="org-ws-submenu"
    >
      <template v-slot:title>{{ $t('commons.organization') }}: {{ currentOrganizationName }}</template>
      <el-input
        v-model="searchOrg"
        :placeholder="$t('project.search_by_name')"
        prefix-icon="el-icon-search"
        clearable
        class="search-input"
        size="small"
      />
      <div class="org-ws-menu">
        <el-menu-item v-for="(item,index) in organizationList" :key="index" @click="changeOrg(item)">
          {{ item.name }}
          <i
            v-if="item.id === currentUserInfo.lastOrganizationId"
            class="el-icon-check"
          />
        </el-menu-item>
      </div>
    </el-submenu>
    <el-submenu v-roles="['test_manager', 'test_user', 'test_viewer']" index="2" popper-class="submenu">
      <template v-slot:title>{{ $t('commons.workspace') }}: {{ currentWorkspaceName }}</template>
      <el-input
        v-model="searchWs"
        :placeholder="$t('project.search_by_name')"
        prefix-icon="el-icon-search"
        clearable
        class="search-input"
        size="small"
      />
      <div class="org-ws-menu">
        <el-menu-item v-for="(item,index) in workspaceList" :key="index" @click="changeWs(item)">
          {{ item.name }}
          <i v-if="item.id === currentUserInfo.lastWorkspaceId" class="el-icon-check" />
        </el-menu-item>
      </div>
    </el-submenu>
  </el-menu>
</template>

<script>
import {
  PROJECT_ID,
  ROLE_ORG_ADMIN,
  ROLE_TEST_MANAGER,
  ROLE_TEST_USER,
  ROLE_TEST_VIEWER,
  WORKSPACE_ID
} from '@/metersphere/common/js/constants'
import { getCurrentUser, hasRoles, saveLocalStorage } from '@/metersphere/common/js/utils'

export default {
  name: 'MsHeaderOrgWs',
  data() {
    return {
      organizationList: [
        { name: this.$t('organization.none') }
      ],
      workspaceList: [
        { name: this.$t('workspace.none') }
      ],
      currentUserInfo: {},
      currentUserId: getCurrentUser().id,
      workspaceIds: [],
      currentOrganizationName: '',
      currentWorkspaceName: '',
      searchOrg: '',
      searchWs: '',
      orgListCopy: [{ name: this.$t('organization.none') }],
      wsListCopy: [{ name: this.$t('workspace.none') }]
    }
  },
  computed: {
    currentUser: () => {
      return getCurrentUser()
    }
  },
  watch: {
    searchOrg(val) {
      this.query('org', val)
    },
    searchWs(val) {
      this.query('ws', val)
    }
  },
  created() {
    this.initMenuData()
    this.getCurrentUserInfo()
  },
  methods: {
    initMenuData() {
      if (hasRoles(ROLE_ORG_ADMIN, ROLE_TEST_VIEWER, ROLE_TEST_USER, ROLE_TEST_MANAGER)) {
        this.$get('/organization/list/userorg/' + encodeURIComponent(this.currentUserId), response => {
          const data = response.data
          this.organizationList = data
          this.orgListCopy = data
          const org = data.filter(r => r.id === this.currentUser.lastOrganizationId)
          if (org.length > 0) {
            this.currentOrganizationName = org[0].name
          }
        })
      }
      if (hasRoles(ROLE_TEST_VIEWER, ROLE_TEST_USER, ROLE_TEST_MANAGER)) {
        if (!this.currentUser.lastOrganizationId) {
          return false
        }
        this.$get('/workspace/list/orgworkspace/', response => {
          const data = response.data
          if (data.length === 0) {
            this.workspaceList = [{ name: this.$t('workspace.none') }]
          } else {
            this.workspaceList = data
            this.wsListCopy = data
            const workspace = data.filter(r => r.id === this.currentUser.lastWorkspaceId)
            if (workspace.length > 0) {
              this.currentWorkspaceName = workspace[0].name
              localStorage.setItem(WORKSPACE_ID, workspace[0].id)
            }
          }
        })
      }
    },
    getCurrentUserInfo() {
      this.$get('/user/info/' + encodeURIComponent(this.currentUserId), response => {
        this.currentUserInfo = response.data
      })
    },
    changeOrg(data) {
      const orgId = data.id
      if (!orgId) {
        return false
      }
      this.$post('/user/switch/source/org/' + orgId, {}, response => {
        saveLocalStorage(response)
        if (response.data.workspaceId) {
          localStorage.setItem('workspace_id', response.data.workspaceId)
        }
        localStorage.removeItem(PROJECT_ID)
        this.$router.push('/').then(() => {
          window.location.reload()
        }).catch(err => err)
      })
    },
    changeWs(data) {
      const workspaceId = data.id
      if (!workspaceId) {
        return false
      }
      this.$post('/user/switch/source/ws/' + workspaceId, {}, response => {
        saveLocalStorage(response)
        localStorage.setItem('workspace_id', workspaceId)
        localStorage.removeItem(PROJECT_ID)
        this.$router.push('/').then(() => {
          window.location.reload()
        }).catch(err => err)
      })
    },
    query(sign, queryString) {
      if (sign === 'org') {
        this.organizationList = queryString ? this.orgListCopy.filter(this.createFilter(queryString)) : this.orgListCopy
      }
      if (sign === 'ws') {
        this.workspaceList = queryString ? this.wsListCopy.filter(this.createFilter(queryString)) : this.wsListCopy
      }
    },
    createFilter(queryString) {
      return item => {
        return (item.name.toLowerCase().indexOf(queryString.toLowerCase()) !== -1)
      }
    }
  }
}
</script>

<style scoped>
.el-icon-check {
  color: #44b349;
  margin-left: 10px;
}

::-webkit-scrollbar {
  width: 10px;
  height: 10px;
  position: fixed;
}

::-webkit-scrollbar-thumb {
  border-radius: 1em;
  background-color: #595591;
  position: fixed;
}

::-webkit-scrollbar-track {
  border-radius: 1em;
  background-color: transparent;
  position: fixed;
}

.org-ws-menu {
  height: 180px;
  overflow: auto;
}

.search-input {
  padding: 0;
  margin-top: -4px;
  background-color: #595591;
}

.search-input >>> .el-input__inner {
  border-radius: 0;
  background-color: #2d2a49;
  color: #d2ced8;
  border-color: #b4aebe;
}
</style>
