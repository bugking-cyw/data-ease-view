<template xmlns:el-col="http://www.w3.org/1999/html">
  <el-col>
    <!-- group -->
    <el-col v-if="!sceneMode">
      <el-row class="title-css">
        <span class="title-text">
          {{ $t('dataset.datalist') }}
        </span>
      </el-row>
      <el-divider />

      <el-row>
        <el-button icon="el-icon-circle-plus" type="primary" size="mini" @click="add('group')">
          {{ $t('dataset.add_group') }}
        </el-button>
        <el-button icon="el-icon-folder-add" type="primary" size="mini" @click="add('scene')">
          {{ $t('dataset.add_scene') }}
        </el-button>
      </el-row>

      <el-row>
        <el-form>
          <el-form-item class="form-item">
            <el-input
              v-model="search"
              size="mini"
              :placeholder="$t('dataset.search')"
              prefix-icon="el-icon-search"
              clearable
            />
          </el-form-item>
        </el-form>
      </el-row>

      <el-col class="custom-tree-container">
        <div class="block">
          <el-tree
            :default-expanded-keys="expandedArray"
            :data="data"
            node-key="id"
            :expand-on-click-node="true"
            @node-click="nodeClick"
          >
            <span slot-scope="{ node, data }" class="custom-tree-node">
              <span>
                <span v-if="data.type === 'scene'">
                  <el-button
                    icon="el-icon-folder"
                    type="text"
                    size="mini"
                  />
                </span>
                <span style="margin-left: 6px">{{ data.name }}</span>
              </span>
              <span>
                <span v-if="data.type ==='group'" @click.stop>
                  <el-dropdown trigger="click" size="small" @command="clickAdd">
                    <span class="el-dropdown-link">
                      <el-button
                        icon="el-icon-plus"
                        type="text"
                        size="small"
                      />
                    </span>
                    <el-dropdown-menu slot="dropdown">
                      <el-dropdown-item icon="el-icon-circle-plus" :command="beforeClickAdd('group',data,node)">
                        {{ $t('dataset.group') }}
                      </el-dropdown-item>
                      <el-dropdown-item icon="el-icon-folder-add" :command="beforeClickAdd('scene',data,node)">
                        {{ $t('dataset.scene') }}
                      </el-dropdown-item>
                    </el-dropdown-menu>
                  </el-dropdown>
                </span>
                <span style="margin-left: 12px;" @click.stop>
                  <el-dropdown trigger="click" size="small" @command="clickMore">
                    <span class="el-dropdown-link">
                      <el-button
                        icon="el-icon-more"
                        type="text"
                        size="small"
                      />
                    </span>
                    <el-dropdown-menu slot="dropdown">
                      <el-dropdown-item icon="el-icon-edit-outline" :command="beforeClickMore('rename',data,node)">
                        {{ $t('dataset.rename') }}
                      </el-dropdown-item>
                      <!--                  <el-dropdown-item icon="el-icon-right" :command="beforeClickMore('move',data,node)">-->
                      <!--                    {{$t('dataset.move_to')}}-->
                      <!--                  </el-dropdown-item>-->
                      <el-dropdown-item icon="el-icon-delete" :command="beforeClickMore('delete',data,node)">
                        {{ $t('dataset.delete') }}
                      </el-dropdown-item>
                    </el-dropdown-menu>
                  </el-dropdown>
                </span>
              </span>
            </span>
          </el-tree>
        </div>
      </el-col>

      <el-dialog :title="dialogTitle" :visible="editGroup" :show-close="false" width="30%">
        <el-form ref="groupForm" :model="groupForm" :rules="groupFormRules">
          <el-form-item :label="$t('commons.name')" prop="name">
            <el-input v-model="groupForm.name" />
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button size="mini" @click="close()">{{ $t('dataset.cancel') }}</el-button>
          <el-button type="primary" size="mini" @click="saveGroup(groupForm)">{{ $t('dataset.confirm') }}</el-button>
        </div>
      </el-dialog>
    </el-col>

    <!--scene-->
    <el-col v-if="sceneMode">
      <el-row class="title-css">
        <span class="title-text">
          {{ currGroup.name }}
        </span>
        <el-button icon="el-icon-back" size="mini" style="float: right" @click="back">
          {{ $t('dataset.back') }}
        </el-button>
      </el-row>
      <el-divider />
      <el-row>
        <el-dropdown style="margin-right: 10px;" size="small" trigger="click" @command="clickAddData">
          <el-button type="primary" size="mini" plain>
            {{ $t('dataset.add_table') }}
          </el-button>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item :command="beforeClickAddData('db')">
              {{ $t('dataset.db_data') }}
            </el-dropdown-item>
            <el-dropdown-item :command="beforeClickAddData('sql')">
              {{ $t('dataset.sql_data') }}
            </el-dropdown-item>
            <el-dropdown-item :command="beforeClickAddData('excel')">
              {{ $t('dataset.excel_data') }}
            </el-dropdown-item>
            <el-dropdown-item :command="beforeClickAddData('custom')">
              {{ $t('dataset.custom_data') }}
            </el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
        <!-- <el-button type="primary" size="mini" plain>
          {{ $t('dataset.update') }}
        </el-button>
        <el-button type="primary" size="mini" plain>
          {{ $t('dataset.process') }}
        </el-button> -->
      </el-row>
      <el-row>
        <el-form>
          <el-form-item class="form-item">
            <el-input
              v-model="search"
              size="mini"
              :placeholder="$t('dataset.search')"
              prefix-icon="el-icon-search"
              clearable
            />
          </el-form-item>
        </el-form>
      </el-row>
      <span v-show="false">{{ sceneData }}</span>
      <el-tree
        :data="tableData"
        node-key="id"
        :expand-on-click-node="true"
        @node-click="sceneClick"
      >
        <span slot-scope="{ node, data }" class="custom-tree-node">
          <span>
            <span>
              ({{ data.type }})
            </span>
            <span>
              <span v-if="data.mode === 0" style="margin-left: 6px"><i class="el-icon-s-operation" /></span>
              <span v-if="data.mode === 1" style="margin-left: 6px"><i class="el-icon-time" /></span>
            </span>
            <span style="margin-left: 6px">{{ data.name }}</span>
          </span>
          <span>
            <span style="margin-left: 12px;" @click.stop>
              <el-dropdown trigger="click" size="small" @command="clickMore">
                <span class="el-dropdown-link">
                  <el-button
                    icon="el-icon-more"
                    type="text"
                    size="small"
                  />
                </span>
                <el-dropdown-menu slot="dropdown">
                  <el-dropdown-item icon="el-icon-edit-outline" :command="beforeClickMore('editTable',data,node)">
                    {{ $t('dataset.edit') }}
                  </el-dropdown-item>
                  <!--                  <el-dropdown-item icon="el-icon-right" :command="beforeClickMore('move',data,node)">-->
                  <!--                    {{$t('dataset.move_to')}}-->
                  <!--                  </el-dropdown-item>-->
                  <el-dropdown-item icon="el-icon-delete" :command="beforeClickMore('deleteTable',data,node)">
                    {{ $t('dataset.delete') }}
                  </el-dropdown-item>
                </el-dropdown-menu>
              </el-dropdown>
            </span>
          </span>
        </span>
      </el-tree>

      <el-dialog :title="$t('dataset.table')" :visible="editTable" :show-close="false" width="30%">
        <el-form ref="tableForm" :model="tableForm" :rules="tableFormRules">
          <el-form-item :label="$t('commons.name')" prop="name">
            <el-input v-model="tableForm.name" />
          </el-form-item>
          <el-form-item :label="$t('dataset.mode')" prop="mode">
            <el-radio v-model="tableForm.mode" label="0">{{ $t('dataset.direct_connect') }}</el-radio>
            <el-radio v-model="tableForm.mode" label="1">{{ $t('dataset.sync_data') }}</el-radio>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button size="mini" @click="closeTable()">{{ $t('dataset.cancel') }}</el-button>
          <el-button type="primary" size="mini" @click="saveTable(tableForm)">{{ $t('dataset.confirm') }}</el-button>
        </div>
      </el-dialog>

    </el-col>
  </el-col>
</template>

<script>
import { loadTable, getScene, addGroup, delGroup, addTable, delTable, groupTree } from '@/api/dataset/dataset'
export default {
  name: 'Group',
  data() {
    return {
      sceneMode: false,
      dialogTitle: '',
      search: '',
      editGroup: false,
      editTable: false,
      data: [],
      tableData: [],
      currGroup: {},
      expandedArray: [],
      groupForm: {
        name: '',
        pid: null,
        level: 0,
        type: '',
        children: [],
        sort: 'type desc,name asc'
      },
      tableForm: {
        name: '',
        mode: '',
        sort: 'type asc,create_time desc,name asc'
      },
      groupFormRules: {
        name: [
          { required: true, message: this.$t('commons.input_content'), trigger: 'blur' }
        ]
      },
      tableFormRules: {
        name: [
          { required: true, message: this.$t('commons.input_content'), trigger: 'blur' }
        ],
        mode: [
          { required: true, message: this.$t('commons.input_content'), trigger: 'blur' }
        ]
      }
    }
  },
  computed: {
    sceneData: function() {
      console.log(this.$store.state.dataset.sceneData + ' do post')
      this.tableTree()
      return this.$store.state.dataset.sceneData
    }
  },
  watch: {
    // search(val){
    //   this.groupForm.name = val;
    //   this.tree(this.groupForm);
    // }
  },
  mounted() {
    this.tree(this.groupForm)
    this.refresh()
    this.tableTree()
    // this.$router.push('/dataset');
  },
  methods: {
    clickAdd(param) {
      // console.log(param);
      this.add(param.type)
      this.groupForm.pid = param.data.id
      this.groupForm.level = param.data.level + 1
    },

    beforeClickAdd(type, data, node) {
      return {
        'type': type,
        'data': data,
        'node': node
      }
    },

    clickMore(param) {
      console.log(param)
      switch (param.type) {
        case 'rename':
          this.add(param.data.type)
          this.groupForm = JSON.parse(JSON.stringify(param.data))
          break
        case 'move':

          break
        case 'delete':
          this.delete(param.data)
          break
        case 'editTable':
          this.editTable = true
          this.tableForm = JSON.parse(JSON.stringify(param.data))
          this.tableForm.mode = this.tableForm.mode + ''
          break
        case 'deleteTable':
          this.deleteTable(param.data)
          break
      }
    },

    beforeClickMore(type, data, node) {
      return {
        'type': type,
        'data': data,
        'node': node
      }
    },

    add(type) {
      switch (type) {
        case 'group':
          this.dialogTitle = this.$t('dataset.group')
          break
        case 'scene':
          this.dialogTitle = this.$t('dataset.scene')
          break
      }
      this.groupForm.type = type
      this.editGroup = true
    },

    saveGroup(group) {
      // console.log(group);
      this.$refs['groupForm'].validate((valid) => {
        if (valid) {
          addGroup(group).then(res => {
            this.close()
            this.$message({
              message: this.$t('commons.save_success'),
              type: 'success',
              showClose: true
            })
            this.tree(this.groupForm)
          })
        } else {
          this.$message({
            message: this.$t('commons.input_content'),
            type: 'error',
            showClose: true
          })
          return false
        }
      })
    },

    saveTable(table) {
    //   console.log(table)
      table.mode = parseInt(table.mode)
      this.$refs['tableForm'].validate((valid) => {
        if (valid) {
          addTable(table).then(response => {
            this.closeTable()
            this.$message({
              message: this.$t('commons.save_success'),
              type: 'success',
              showClose: true
            })
            this.tableTree()
            this.$router.push('/dataset/home')
            this.$store.dispatch('dataset/setTable', null)
          })
        } else {
          this.$message({
            message: this.$t('commons.input_content'),
            type: 'error',
            showClose: true
          })
          return false
        }
      })
    },

    delete(data) {
      this.$confirm(this.$t('dataset.confirm_delete'), this.$t('dataset.tips'), {
        confirmButtonText: this.$t('dataset.confirm'),
        cancelButtonText: this.$t('dataset.cancel'),
        type: 'warning'
      }).then(() => {
        delGroup(data.id).then(response => {
          this.$message({
            type: 'success',
            message: this.$t('dataset.delete_success'),
            showClose: true
          })
          this.tree(this.groupForm)
        })
      }).catch(() => {
      })
    },

    deleteTable(data) {
      this.$confirm(this.$t('dataset.confirm_delete'), this.$t('dataset.tips'), {
        confirmButtonText: this.$t('dataset.confirm'),
        cancelButtonText: this.$t('dataset.cancel'),
        type: 'warning'
      }).then(() => {
        delTable(data.id).then(response => {
          this.$message({
            type: 'success',
            message: this.$t('dataset.delete_success'),
            showClose: true
          })
          this.tableTree()
          this.$router.push('/dataset/home')
          this.$store.dispatch('dataset/setTable', null)
        })
      }).catch(() => {
      })
    },

    close() {
      this.editGroup = false
      this.groupForm = {
        name: '',
        pid: null,
        level: 0,
        type: '',
        children: [],
        sort: 'type desc,name asc'
      }
    },

    closeTable() {
      this.editTable = false
      this.tableForm = {
        name: ''
      }
    },

    tree(group) {
      groupTree(group).then(res => {
        this.data = res.data.data
      })
    },

    tableTree() {
      this.tableData = []
      if (this.currGroup.id) {
        loadTable({
          sort: 'type asc,create_time desc,name asc',
          sceneId: this.currGroup.id
        }).then(res => {
          this.tableData = res.data.data
        })
      }
    },

    nodeClick(data, node) {
      // console.log(data);
      // console.log(node);
      if (data.type === 'scene') {
        this.sceneMode = true
        this.currGroup = data
        this.$store.dispatch('dataset/setSceneData', this.currGroup.id)
      }
      if (node.expanded) {
        this.expandedArray.push(data.id)
      } else {
        const index = this.expandedArray.indexOf(data.id)
        if (index > -1) {
          this.expandedArray.splice(index, 1)
        }
      }
      // console.log(this.expandedArray);
    },

    back() {
      this.sceneMode = false
      //   const route = this.$store.state.permission.currentRoutes
      //   console.log(route)
      this.$router.push('/dataset/index')
    },

    clickAddData(param) {
      // console.log(param);
      switch (param.type) {
        case 'db':
          this.addDB()
          break
        case 'sql':
          this.$message(param.type)
          break
        case 'excel':
          this.$message(param.type)
          break
        case 'custom':
          this.$message(param.type)
          break
      }
    },

    beforeClickAddData(type) {
      return {
        'type': type
      }
    },

    addDB() {
      this.$router.push({
        name: 'add_db',
        params: {
          scene: this.currGroup
        }
      })
    },

    sceneClick(data, node) {
      // console.log(data);
      this.$store.dispatch('dataset/setTable', null)
      this.$store.dispatch('dataset/setTable', data.id)
      this.$router.push({
        name: 'table',
        params: {
          table: data
        }
      })
    },

    refresh() {
      const path = this.$route.path
      if (path === '/dataset/table') {
        this.sceneMode = true
        const sceneId = this.$store.state.dataset.sceneData
        getScene(sceneId).then(res => {
          this.currGroup = res.data.data
        })
      } else {
        this.$router.push('/dataset')
      }
    }
  }
}
</script>

<style scoped>
  .el-divider--horizontal {
    margin: 12px 0
  }

  .search-input {
    padding: 12px 0;
  }

  .custom-tree-node {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 14px;
    padding-right: 8px;
  }

  .custom-position {
    flex: 1;
    display: flex;
    align-items: center;
    font-size: 14px;
    flex-flow: row nowrap;
  }

  .form-item {
    margin-bottom: 0;
  }

  .title-css {
    height: 26px;
  }

  .title-text {
    line-height: 26px;
  }
</style>
