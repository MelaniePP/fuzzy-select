<template>
  <div class="fuzzy-select">
    <el-select 
      v-model="value"
      @change="handelChange"
      v-bind="$attrs"
      v-on="$listeners"
      :loading="loading"
      :remote="remote"
      :remote-method="remoteMethod"
      :placeholder="placeholder|| '请选择'">
      <el-option v-if="showAll" :value="defaultAllValue||'全部'" label="全部"></el-option>
      <el-option
        v-for="item in list"
        :key="getValue(item)"
        :label="getlabel(item)"
        :value="getValue(item)">
      </el-option>
    </el-select>
    <el-pagination
    v-if="isPager"
    :page-size="pageSize"
    :current-page="currentPage"
    small
    layout="prev, pager, next"
    @current-change="handleCurrentChange"
    :total="total">
    </el-pagination>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  model: {
    prop: 'modelValue',
    event: 'change'
  },
  data(){
    return {
      total: 0,
      currentPage: 1,
      loading: false,
      value:'1',
      placeholder:'请选择网点',
      list: [],
      queryString: ''
    }
  },
  props: {
    modelValue: {
      require: true
    },
    params: {
      type: Object,
      default: ()=>({})
    },
    defaultAllValue: String,
    labelName: String,
    valueName: String,
    // 静态数组不需要请求接口
    data: Array,
    pageSize: {
      default: 50
    },
    showAll: {
      type: Boolean,
      default: false
    },
    isPager: {
      type: Boolean,
      default: false
    },
    remote: {
      type: Boolean,
      default: false
    },
    remoteMethod: Function
  },
  methods:{
    getlabel(option){
      return option[this.labelName || 'label']
    },
    getValue(option){
      return option[this.valueName || 'value']
    },
    async fetchData(queryString, page){
      this.loading = true
      this.queryString = queryString
      const params = this.params? {...this.params} : {}
      if(params.keyWord){
        params[params.keyWord] = queryString
        delete params.keyWord
      }
      if(this.isPager){
        if(!page) this.currentPage = 1
        params.currentPage = this.currentPage
        params.pageSize = this.pageSize
      }
      if(this.remote && !this.remoteMethod){
        console.error('remote-method is required')
        return
      }
      const data = await this.remoteMethod(queryString).finally(() => this.loading = false)
      if(Array.isArray(data)){
        this.list = data
      }else{
        this.list = data.rows || []
        this.total = data.total || 0
      }
    },
    handleCurrentChange(page){
      this.fetchData(this.queryString,page)
    },
    handelChange(val){
      const selectedItem = this.list.find(item => {
        if(typeof val === 'string'){
          return item === val
        }else{
          return this.getValue(val) === val
        }
      })
      this.$emit('selected', selectedItem)
    }
  },
  watch:{
    modelValue:{
      immediate: true,
      handler(val){
        this.value = val
      }
    },
    data: {
      immediate: true,
      handler(val) {
        if (val) {
          this.list = [...this.data]
        }
      }
    },
  },
  created(){
    this.value = this.modelValue
  }
  
}
</script>
