<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">
          <a-col :md="6" :sm="8">
            <a-form-item label="主人名字">
              <a-input placeholder="请输入主人名字" v-model="queryParam.zhurenmingzi"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="电话">
              <a-input placeholder="请输入电话" v-model="queryParam.iphone"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
<!--              <a @click="handleToggleSearch" style="margin-left: 8px">-->
<!--                {{ toggleSearchStatus ? '收起' : '展开' }}-->
<!--                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>-->
<!--              </a>-->
            </span>
          </a-col>

        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('宠物主人')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
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
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange">

        <!--等级变色-->
        <template slot="member" slot-scope="text">
          <a-tag v-if="text === '普通用户'" color="pink">{{ text }} </a-tag>
          <a-tag v-else-if="text === 'vip'" color="green">{{ text }} </a-tag>
          <a-tag v-else-if="text === 'svip'" color="orange">{{ text }} </a-tag>
<!--          <a-tag v-else-if="text === 'vip'" color="red">{{ text }} </a-tag>-->
          <span v-else>{{ text }}</span>
        </template>


        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无此图片</span>
          <img v-else :src="getImgView(text)" height="25px" alt="图片不存在" style="max-width:80px;font-size: 12px;font-style: italic;"/>
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

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
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

    <petOwners-modal ref="modalForm" @ok="modalFormOk"></petOwners-modal>
  </a-card>
</template>

<script>

  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import PetOwnersModal from './modules/PetOwnersModal'
  import {initDictOptions, filterMultiDictText} from '@/components/dict/JDictSelectUtil'


  export default {
    name: "PetOwnersList",
    mixins:[JeecgListMixin],
    components: {
      PetOwnersModal
    },
    data () {
      return {
        description: '宠物主人管理页面',
        // 表头
        columns: [
          {
            title: '序号',
            dataIndex: '',
            key:'rowIndex',
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
          },
          {
            title:'主人名字',
            align:"center",
            dataIndex: 'zhurenmingzi'
          },
          {
            title:'年龄',
            align:"center",
            dataIndex: 'age'
          },
          {
            title:'性别',
            align:"center",
            dataIndex: 'sex_dictText'
          },
          {
            title:'电话',
            align:"center",
            dataIndex: 'iphone'
          },
          {
            title:'等级',
            align:"center",
            dataIndex: 'huiyuan_dictText',
            scopedSlots: { customRender: 'member' },
          },
          {
            title:'种类',
            align:"center",
            dataIndex: 'weiyangzhonglei_dictText',
            // customRender:(text)=>{
            //   if(!text){
            //     return ''
            //   }else{
            //     return filterMultiDictText(this.dictOptions['weiyangzhonglei'], text+"")
            //   }
            // }
          },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' },
          }
        ],
        url: {
          list: "/pet/petOwners/list",
          delete: "/pet/petOwners/delete",
          deleteBatch: "/pet/petOwners/deleteBatch",
          exportXlsUrl: "/pet/petOwners/exportXls",
          importExcelUrl: "pet/petOwners/importExcel",
        },
        dictOptions:{
         // weiyangzhonglei:[],
        }
      }
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      }
    },
    methods: {
      initDictConfig(){
        // initDictOptions('zhonglei').then((res) => {
        //   if (res.success) {
        //     this.$set(this.dictOptions, 'weiyangzhonglei', res.result)
        //   }
        // })

      }

    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>