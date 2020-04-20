<template>
  <a-modal
    :title="title"
    :width="width"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-row class="form-row" :gutter="16">
          <a-col :lg="12">
            <a-form-item label="姓名" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="[ 'employeesName', validatorRules.employeesName]" placeholder="请输入姓名"></a-input>
            </a-form-item>
          </a-col>
          <a-col :lg="12">
            <a-form-item label="年龄" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="[ 'employeesAge', validatorRules.employeesAge]" placeholder="请输入年龄"></a-input>
            </a-form-item>
          </a-col>
        </a-row>

        <a-row class="form-row" :gutter="16">
          <a-col :lg="12">
            <a-form-item label="性别" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="radio" v-decorator="['employeesSex', validatorRules.employeesSex]"
                                 :trigger-change="true" dictCode="sex" placeholder="请选择性别"/>
            </a-form-item>
          </a-col>
          <a-col :lg="12">
            <a-form-item label="部门" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-select-depart v-decorator="['employeesDepartment', validatorRules.employeesDepartment]"
                               :trigger-change="true"/>
            </a-form-item>
          </a-col>
        </a-row>

        <a-row class="form-row" :gutter="16">
          <a-col :lg="12">
            <a-form-item label="入职日期" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-date placeholder="请选择入职日期" v-decorator="[ 'employeesDateentry', validatorRules.employeesDateentry]"
                      :trigger-change="true" style="width: 100%"/>
            </a-form-item>
          </a-col>
          <a-col :lg="12">
            <a-form-item label="生日" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-date placeholder="请选择生日" v-decorator="[ 'employeesBirthday', validatorRules.employeesBirthday]"
                      :trigger-change="true" style="width: 100%"/>
            </a-form-item>
          </a-col>
        </a-row>

        <a-row class="form-row" :gutter="16">
          <a-col :lg="12">
            <a-form-item label="头像" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-image-upload class="avatar-uploader" text="上传" v-model="fileList"></j-image-upload>
            </a-form-item>
          </a-col>
          <a-col :lg="12">
            <a-form-item label="简历" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-upload v-decorator="['employeesResume', validatorRules.employeesResume]"
                        :trigger-change="true"></j-upload>
            </a-form-item>
          </a-col>
        </a-row>

<!--        <a-row class="form-row" :gutter="16">-->
<!--          <a-form-item label="简介" :labelCol="labelCol" :wrapperCol="wrapperCol">-->
<!--            <a-textarea v-decorator="['employeesIntroductiontothe', validatorRules.employeesIntroductiontothe]" rows="2"-->
<!--                        placeholder="请输入简介"/>-->
<!--          </a-form-item>-->
<!--        </a-row>-->

        <!-- JEditor -->
        <a-row class="form-row" :gutter="16">
            <a-form-item label="简介" :labelCol="labelCol" :wrapperCol="wrapperCol" >
              <j-editor  v-decorator="[ 'employeesIntroductiontothe', {} ]" triggerChange/>
            </a-form-item>
        </a-row>


      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'
  import JDate from '@/components/jeecg/JDate'
  import JUpload from '@/components/jeecg/JUpload'
  import JSelectDepart from '@/components/jeecgbiz/JSelectDepart'
  import JDictSelectTag from '@/components/dict/JDictSelectTag'
  //增加图片上传包
  import JImageUpload from '../../../components/jeecg/JImageUpload'
  //富文本编辑器
  import JEditor from '@/components/jeecg/JEditor'

  export default {
    name: 'EmployeesInductionModal',
    components: {
      JDate,
      JUpload,
      JSelectDepart,
      JDictSelectTag,
      JEditor,

      JImageUpload
    },
    data() {
      return {
        form: this.$form.createForm(this),
        title: '操作',
        width: 800,
        visible: false,
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 }
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 }
        },
        confirmLoading: false,
        validatorRules: {
          employeesName: {
            rules: [
              { required: true, message: '请输入姓名!' },
              { validator: (rule, value, callback) => validateDuplicateValue('employees_induction', 'employees_name', value, this.model.id, callback) }
            ]
          },
          employeesAge: {
            rules: [
              { required: true, message: '请输入年龄!' }
            ]
          },
          employeesSex: {
            rules: [
              { required: true, message: '请输入性别!' }
            ]
          },
          employeesDepartment: {
            rules: []
          },
          employeesDateentry: {
            rules: []
          },
          employeesBirthday: {
            rules: []
          },
          employeesHeadportrait: {
            rules: []
          },
          employeesResume: {
            rules: []
          },
          employeesIntroductiontothe: {
            rules: []
          }
        },
        url: {
          //图片地址
          fileUpload: window._CONFIG['domianURL'] + '/sys/common/upload',
          imgerver: window._CONFIG['staticDomainURL'],
          add: '/enterprise/employeesInduction/add',
          edit: '/enterprise/employeesInduction/edit'
        },
        //存放图片变量
        picUrl: '',
        fileList: []
      }
    },
    created() {
    },
    computed: {
      //调用上传方法
      uploadAction: function() {
        return this.url.fileUpload
      }
    },
    methods: {
      add() {
        this.picUrl = ''
        this.edit({})
      },
      edit(record) {
        this.form.resetFields()
        this.model = Object.assign({}, record)
        this.visible = true
        //图片变量赋值
        setTimeout(() => {
          this.fileList = record.employeesHeadportrait
        }, 5)
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model, 'employeesName', 'employeesAge', 'employeesSex', 'employeesDepartment', 'employeesDateentry', 'employeesBirthday', 'employeesHeadportrait', 'employeesResume', 'employeesIntroductiontothe'))
        })
      },
      close() {
        this.$emit('close')
        this.visible = false
      },
      handleOk() {
        const that = this
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true
            let httpurl = ''
            let method = ''
            if (!this.model.id) {
              httpurl += this.url.add
              method = 'post'
            } else {
              httpurl += this.url.edit
              method = 'put'
            }
            let formData = Object.assign(this.model, values)
            //验证赋值
            formData.employeesHeadportrait = that.fileList
            console.log('表单提交数据', formData)
            httpAction(httpurl, formData, method).then((res) => {
              if (res.success) {
                that.$message.success(res.message)
                that.$emit('ok')
              } else {
                that.$message.warning(res.message)
              }
            }).finally(() => {
              that.confirmLoading = false
              that.close()
            })
          }

        })
      },
      handleCancel() {
        this.close()
      },
      popupCallback(row) {
        this.form.setFieldsValue(pick(row, 'employeesName', 'employeesAge', 'employeesSex', 'employeesDepartment', 'employeesDateentry', 'employeesBirthday', 'employeesHeadportrait', 'employeesResume', 'employeesIntroductiontothe'))
      },

      //图片回显
      normFile(e) {
        console.log('Upload event:', e)
        if (Array.isArray(e)) {
          return e
        }
        return e && e.fileList
      },
      beforeUpload: function(file) {
        var fileType = file.type
        if (fileType.indexOf('image') < 0) {
          this.$message.warning('请上传图片')
          return false
        }
        //TODO 验证文件大小
      },
      handleChange(info) {
        this.picUrl = ''
        if (info.file.status === 'uploading') {
          this.uploadLoading = true
          return
        }
        if (info.file.status === 'done') {
          var response = info.file.response
          this.uploadLoading = false
          console.log(response)
          if (response.success) {
            this.model.avatar = response.message
            this.picUrl = 'Has no pic url yet'
          } else {
            this.$message.warning(response.message)
          }
        }
      }


    }
  }
</script>