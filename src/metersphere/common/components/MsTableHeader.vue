<template>

  <div>
    <el-row v-if="title" class="table-title" type="flex" justify="space-between" align="middle">
      <slot name="title">
        {{ title }}
      </slot>
    </el-row>
    <el-row type="flex" justify="space-between" align="middle">
      <span class="operate-button">
        <ms-table-button
          v-if="showCreate"
          :is-tester-permission="isTesterPermission"
          icon="el-icon-circle-plus-outline"
          :content="createTip"
          @click="create"
        />
        <ms-table-button
          v-if="showRun"
          :is-tester-permission="isTesterPermission"
          icon="el-icon-video-play"
          type="primary"
          :content="runTip"
          @click="runTest"
        />
        <ms-table-button
          v-if="showRun"
          :is-tester-permission="isTesterPermission"
          icon="el-icon-circle-plus-outline"
          content="转场景测试"
          @click="historicalDataUpgrade"
        />

        <slot name="button" />
      </span>
      <span>
        <slot name="searchBarBefore" />
        <ms-table-search-bar :condition.sync="condition" class="search-bar" :tip="tip" @change="search" />
        <ms-table-adv-search-bar v-if="isCombine" :condition.sync="condition" @search="search" />
      </span>
    </el-row>
  </div>

</template>

<script>
import MsTableSearchBar from './MsTableSearchBar'
import MsTableButton from './MsTableButton'
import MsTableAdvSearchBar from './search/MsTableAdvSearchBar'

export default {
  name: 'MsTableHeader',
  components: { MsTableAdvSearchBar, MsTableSearchBar, MsTableButton },
  props: {
    title: {
      type: String,
      default() {
        return this.$t('commons.name')
      }
    },
    showCreate: {
      type: Boolean,
      default: true
    },
    showRun: {
      type: Boolean,
      default: false
    },
    condition: {
      type: Object
    },
    createTip: {
      type: String,
      default() {
        return this.$t('commons.create')
      }
    },
    runTip: {
      type: String

    },

    isTesterPermission: {
      type: Boolean,
      default: false
    },
    tip: {
      String,
      default() {
        return this.$t('commons.search_by_name')
      }
    }
  },
  computed: {
    isCombine() {
      return this.condition.components !== undefined && this.condition.components.length > 0
    }
  },
  methods: {
    search(value) {
      this.$emit('update:condition', this.condition)
      this.$emit('search', value)
    },
    create() {
      this.$emit('create')
    },
    runTest() {
      this.$emit('runTest')
    },
    historicalDataUpgrade() {
      this.$emit('historicalDataUpgrade')
    }
  }
}
</script>

<style>

  .table-title {
    height: 40px;
    font-weight: bold;
    font-size: 18px;
  }

</style>

<style scoped>

  .operate-button {
    margin-bottom: -5px;
  }

  .search-bar {
    width: 240px
  }

</style>
