<script setup>
import { defineOptions, ref, provide, watch } from 'vue'
import { fetchSubscriberPage } from '@/api/activemq/subscriber.js'
import ActiveMqSubscribersTable from '@/views/activemq/dialog/subscriberTable.vue'
import { createDurableSubscriber } from '@/api/activemq/broker.js'
import { ElMessage } from 'element-plus'
import { commonQuery } from '@/utils/commonQuery.js'

defineOptions({
  name: 'ActiveMqSubscribers'
})
const activeName = ref('offlineDurable ')
const createForm = ref({
  clientId: '',
  subscriptionName: '',
  topicName: '',
  selector: null
})
const searchForm = ref({
  subscriptionName: '',
  active: null,
  durable: null
})
const page = ref({
  current: 1,
  size: 10,
  total: 0
})
const tableData = ref([])
const fetchList = () => {
  const data = commonQuery(searchForm.value, page.value)
  fetchSubscriberPage(data).then(resp => {
    tableData.value = resp.data.records
    page.value.total = resp.data.total
    page.value.current = resp.data.current
    page.value.size = resp.data.size
  })
}
const doCreate = () => {
  const data = commonQuery(createForm.value, null)
  createDurableSubscriber(data).then(resp => {
    ElMessage.success(`Create result: ${ resp.msg }`)
  }).finally(() => {
    fetchList()
    createForm.value = {
      clientId: '',
      subscriptionName: '',
      topicName: ''
    }
  })
}
provide('fetchFn', fetchList)
watch(activeName, (val) => {
  if (val === 'activeDurable') {
    searchForm.value.active = true
    searchForm.value.durable = true
  }
  if (val === 'offlineDurable') {
    searchForm.value.active = false
    searchForm.value.durable = false
  }
  if (val === 'activeNonDurable') {
    searchForm.value.active = true
    searchForm.value.durable = false
  }
  fetchList()
})
</script>

<template>
  <el-divider content-position="left"> Create Durable Topic Subscribers</el-divider>
  <el-form :model="createForm" :inline="true">
    <el-form-item label="Client ID">
      <el-input v-model="createForm.clientId" placeholder="Client ID" />
    </el-form-item>
    <el-form-item label="Subscriber Name">
      <el-input v-model="createForm.subscriptionName" placeholder="Subscriber Name" />
    </el-form-item>
    <el-form-item label="JMS Destination">
      <el-input v-model="createForm.topicName" placeholder="JMS Destination" />
    </el-form-item>
    <el-form-item label="Selector">
      <el-input v-model="createForm.selector" placeholder="Selector" />
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="doCreate">Create</el-button>
    </el-form-item>
  </el-form>
  <el-divider />
  <el-tabs v-model="activeName" type="border-card">
    <el-tab-pane label="Active Durable Topic Subscribers" name="activeDurable">
      <active-mq-subscribers-table
        v-model:page="page"
        v-model:tableData="tableData"
        v-model:searchForm="searchForm"
        v-model:fetchFn="fetchList"
        v-model:activeName="activeName"
      />
    </el-tab-pane>
    <el-tab-pane label="Offline Durable Topic Subscribers" name="offlineDurable" :lazy="true">
      <active-mq-subscribers-table
        v-model:page="page"
        v-model:tableData="tableData"
        v-model:searchForm="searchForm"
        v-model:fetchFn="fetchList"
        v-model:activeName="activeName"
      />
    </el-tab-pane>
    <el-tab-pane label="Active Non-Durable Topic Subscribers" name="activeNonDurable" :lazy="true">
      <active-mq-subscribers-table
        v-model:page="page"
        v-model:tableData="tableData"
        v-model:searchForm="searchForm"
        v-model:fetchFn="fetchList"
        v-model:activeName="activeName"
      />
    </el-tab-pane>
  </el-tabs>
</template>
<style></style>
