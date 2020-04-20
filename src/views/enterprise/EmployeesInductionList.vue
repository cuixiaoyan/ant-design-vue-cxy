<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="姓名">
              <a-input placeholder="请输入姓名" v-model="queryParam.employeesName"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="性别">
              <j-dict-select-tag placeholder="请选择性别" v-model="queryParam.employeesSex" dictCode="sex"/>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
            </span>
          </a-col>

          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <!-- 高级查询区域 -->
            <j-super-query :fieldList="fieldList" ref="superQueryModal"
                           @handleSuperQuery="handleSuperQuery"></j-super-query>

          </a-col>
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('员工入职信息')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl"
                @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel">
            <a-icon type="delete"/>
            删除
          </a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作
          <a-icon type="down"/>
        </a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{
        selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>

        <span style="float:right;">
          <a @click="loadData()"><a-icon type="sync"/>刷新</a>
          <a-divider type="vertical"/>
          <a-popover title="自定义列" trigger="click" placement="leftBottom">
            <template slot="content">
              <a-checkbox-group @change="onColSettingsChange" v-model="settingColumns" :defaultValue="settingColumns">
                <a-row>
                  <template v-for="(item,index) in defColumns">
                    <template v-if="item.key!='rowIndex'&& item.dataIndex!='action'">
                        <a-col :span="12"><a-checkbox :value="item.dataIndex">{{ item.title }}</a-checkbox></a-col>
                    </template>
                  </template>
                </a-row>
              </a-checkbox-group>
            </template>
            <a><a-icon type="setting"/>自定义列</a>
          </a-popover>
        </span>

      </div>

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{fixed:true,selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"

        @change="handleTableChange">

        <!--图片显示-->
        <template slot="avatarslot" slot-scope="text, record, index">
          <div class="anty-img-wrap">
            <a-avatar shape="square" :src="getAvatarView(record.employeesHeadportrait)" icon="user"/>
          </div>
        </template>


        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无此文件</span>
          <a-button
            v-else
            :ghost="true"
            type="primary"
            icon="download"
            size="small"
            @click="uploadFile(text)">
            下载
          </a-button>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical"/>
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>

    <employeesInduction-modal ref="modalForm" @ok="modalFormOk"></employeesInduction-modal>
  </a-card>
</template>

<script>

  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import EmployeesInductionModal from './modules/EmployeesInductionModal'
  import JDictSelectTag from '@/components/dict/JDictSelectTag.vue'
  import { filterMultiDictText } from '@/components/dict/JDictSelectUtil'

  //高级查询
  import JSuperQuery from '@/components/jeecg/JSuperQuery.vue'
  import JInput from '@/components/jeecg/JInput.vue'
  import { initDictOptions, filterDictText } from '@/components/dict/JDictSelectUtil'
  import Vue from 'vue'
  import { filterObj } from '@/utils/util'


  //增加图片显示接口
  import { putAction, getFileAccessHttpUrl } from '@/api/manage'

  //高级查询modal需要参数
  const superQueryFieldList = [{
    type: 'int',
    value: 'employeesAge',
    text: '年龄'
  }, {
    type: 'date',
    value: 'employeesDateentry',
    text: '入职时间'
  }, {
    type: 'date',
    value: 'employeesBirthday',
    text: '生日'
  }]


  export default {
    name: 'EmployeesInductionList',
    mixins: [JeecgListMixin],
    components: {
      JSuperQuery,
      JInput,
      JDictSelectTag,
      EmployeesInductionModal
    },

    data() {
      return {
        description: '员工入职信息管理页面',
        //表头
        columns: [],
        //列设置
        settingColumns: [],
        // 列定义
        defColumns: [
          {
            title: '序号',
            dataIndex: '',
            key: 'rowIndex',
            width: 60,
            align: 'center',
            customRender: function(t, r, index) {
              return parseInt(index) + 1
            }
          },
          {
            title: '姓名',
            align: 'center',
            dataIndex: 'employeesName'
          },
          {
            title: '年龄',
            align: 'center',
            dataIndex: 'employeesAge'
          },
          {
            title: '性别',
            align: 'center',
            dataIndex: 'employeesSex_dictText'
          },
          {
            title: '部门',
            align: 'center',
            dataIndex: 'employeesDepartment_dictText'
          },
          {
            title: '入职日期',
            align: 'center',
            dataIndex: 'employeesDateentry',
            customRender: function(text) {
              return !text ? '' : (text.length > 10 ? text.substr(0, 10) : text)
            }
          },
          {
            title: '生日',
            align: 'center',
            dataIndex: 'employeesBirthday',
            customRender: function(text) {
              return !text ? '' : (text.length > 10 ? text.substr(0, 10) : text)
            }
          },
          {
            title: '头像',
            align: 'center',
            dataIndex: 'employeesHeadportrait',
            scopedSlots: { customRender: 'avatarslot' }
          },
          {
            title: '简历',
            align: 'center',
            dataIndex: 'employeesResume',
            scopedSlots: { customRender: 'fileSlot' }
          },
          // {
          //   title:'简介',
          //   align:"center",
          //   dataIndex: 'employeesIntroductiontothe'
          // },
          {
            title: '操作',
            dataIndex: 'action',
            align: 'center',
            scopedSlots: { customRender: 'action' }
          }
        ],
        url: {
          imgerver: window._CONFIG['staticDomainURL'],
          list: '/enterprise/employeesInduction/list',
          delete: '/enterprise/employeesInduction/delete',
          deleteBatch: '/enterprise/employeesInduction/deleteBatch',
          exportXlsUrl: '/enterprise/employeesInduction/exportXls',
          importExcelUrl: 'enterprise/employeesInduction/importExcel'
        },
        fieldList: superQueryFieldList,
        dictOptions: {}
      }
    },
    computed: {
      importExcelUrl: function() {
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
      }
    },
    methods: {
      //获取图片方法
      getAvatarView: function(employeesHeadportrait) {
        return getFileAccessHttpUrl(employeesHeadportrait, this.url.imgerver, 'http')
      },
      //列设置更改事件
      onColSettingsChange(checkedValues) {
        var key = this.$route.name + ':colsettings'
        Vue.ls.set(key, checkedValues, 7 * 24 * 60 * 60 * 1000)
        this.settingColumns = checkedValues
        const cols = this.defColumns.filter(item => {
          if (item.key == 'rowIndex' || item.dataIndex == 'action') {
            return true
          }
          if (this.settingColumns.includes(item.dataIndex)) {
            return true
          }
          return false
        })
        this.columns = cols
      },
      initColumns() {
        //权限过滤（列权限控制时打开，修改第二个参数为授权码前缀）
        //this.defColumns = colAuthFilter(this.defColumns,'testdemo:');
        var key = this.$route.name + ':colsettings'
        let colSettings = Vue.ls.get(key)
        if (colSettings == null || colSettings == undefined) {
          let allSettingColumns = []
          this.defColumns.forEach(function(item, i, array) {
            allSettingColumns.push(item.dataIndex)
          })
          this.settingColumns = allSettingColumns
          this.columns = this.defColumns
        } else {
          this.settingColumns = colSettings
          const cols = this.defColumns.filter(item => {
            if (item.key == 'rowIndex' || item.dataIndex == 'action') {
              return true
            }
            if (colSettings.includes(item.dataIndex)) {
              return true
            }
            return false
          })
          this.columns = cols
        }
      },


      initDictConfig() {
      }
    },
    //初始化列
    created() {
      this.initColumns()
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>