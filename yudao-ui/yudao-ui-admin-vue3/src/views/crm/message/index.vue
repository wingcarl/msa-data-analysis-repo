<template>
  <el-row :gutter="20">
    <el-col :span="4" class="min-w-[200px]">
      <div class="side-item-list">
        <div
          v-for="(item, index) in leftSides"
          :key="index"
          :class="leftType == item.infoType ? 'side-item-select' : 'side-item-default'"
          class="side-item"
          @click="sideClick(item)"
        >
          {{ item.name }}
          <el-badge v-if="item.msgCount > 0" :max="99" :value="item.msgCount" />
        </div>
      </div>
    </el-col>
    <el-col :span="20" :xs="24">
      <TodayCustomer v-if="leftType === 'todayCustomer'" />
      <FollowLeads v-if="leftType === 'followLeads'" />
    </el-col>
  </el-row>
</template>
<!-- @dbh52：模块改成 backlog 会更合适，待办事项 -->
<script lang="ts" setup>
import TodayCustomer from './tables/TodayCustomer.vue'
import FollowLeads from './tables/FollowLeads.vue'

const leftType = ref('todayCustomer')
const leftSides = ref([
  {
    name: '今日需联系客户',
    infoType: 'todayCustomer',
    msgCount: 1,
    tips: '下次跟进时间为今日的客户'
  },
  {
    name: '分配给我的线索',
    infoType: 'followLeads',
    msgCount: 0,
    tips: '转移之后未跟进的线索'
  },
  {
    name: '分配给我的客户',
    infoType: 'followCustomer',
    msgCount: 0,
    tips: '转移、领取、分配之后未跟进的客户，默认显示自己负责的客户'
  },
  {
    name: '待进入公海的客户',
    infoType: 'putInPoolRemind',
    msgCount: 0,
    tips: ''
  },
  {
    name: '待审核合同',
    infoType: 'checkContract',
    msgCount: 0,
    tips: ''
  },
  {
    name: '待审核回款',
    crmType: 'receivables',
    infoType: 'checkReceivables',
    msgCount: 0,
    tips: ''
  },
  {
    name: '待回款提醒',
    infoType: 'remindReceivablesPlan',
    msgCount: 4,
    tips: ''
  },
  {
    name: '即将到期的合同',
    infoType: 'endContract',
    msgCount: 20,
    tips: '根据“合同到期时间”及设置的“提前提醒天数”提醒'
  }
])

/**
 * 侧边点击
 */
const sideClick = (item) => {
  leftType.value = item.infoType
}
</script>

<style lang="scss" scoped>
.side-item-list {
  top: 0;
  bottom: 0;
  left: 0;
  z-index: 1;
  font-size: 14px;
  background-color: white;
  border: 1px solid #e6e6e6;
  border-radius: 5px;

  .side-item {
    position: relative;
    height: 50px;
    padding: 0 20px;
    line-height: 50px;
    cursor: pointer;

    i {
      color: #999;
    }
  }
}

.side-item-default {
  color: #333;
  border-right: 2px solid transparent;
}

.side-item-select {
  color: #409eff;
  background-color: #ecf5ff;
  border-right: 2px solid var(--el-color-primary);
}

.el-badge :deep(.el-badge__content) {
  top: 0;
  border: none;
}

.el-badge {
  position: absolute;
  top: 0;
  right: 15px;
}
</style>
