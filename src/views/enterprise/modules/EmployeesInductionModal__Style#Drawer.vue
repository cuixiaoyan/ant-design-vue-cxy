<template>
  <a-drawer
    :title="title"
    :width="width"
    placement="right"
    :closable="false"
    @close="close"
    :visible="visible">
  
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item label="姓名" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'employeesName', validatorRules.employeesName]" placeholder="请输入姓名"></a-input>
        </a-form-item>
        <a-form-item label="年龄" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="[ 'employeesAge', validatorRules.employeesAge]" placeholder="请输入年龄"></a-input>
        </a-form-item>
        <a-form-item label="性别" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-dict-select-tag type="radio" v-decorator="['employeesSex', validatorRules.employeesSex]" :trigger-change="true" dictCode="sex" placeholder="请选择性别"/>
        </a-form-item>
        <a-form-item label="部门" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-select-depart v-decorator="['employeesDepartment', validatorRules.employeesDepartment]" :trigger-change="true"/>
        </a-form-item>
        <a-form-item label="入职日期" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择入职日期" v-decorator="[ 'employeesDateentry', validatorRules.employeesDateentry]" :trigger-change="true" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="生日" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择生日" v-decorator="[ 'employeesBirthday', validatorRules.employeesBirthday]" :trigger-change="true" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="头像" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-upload v-decorator="['employeesHeadportrait', validatorRules.employeesHeadportrait]" :trigger-change="true"></j-upload>
        </a-form-item>
        <a-form-item label="简历" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-upload v-decorator="['employeesResume', validatorRules.employeesResume]" :trigger-change="true"></j-upload>
        </a-form-item>
        <a-form-item label="简介" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-textarea v-decorator="['employeesIntroductiontothe', validatorRules.employeesIntroductiontothe]" rows="4" placeholder="请输入简介"/>
        </a-form-item>
        
      </a-form>
    </a-spin>
    <a-button type="primary" @click="handleOk">确定</a-button>
    <a-button type="primary" @click="handleCancel">取消</a-button>
  </a-drawer>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'
  import JDate from '@/components/jeecg/JDate'  
  import JUpload from '@/components/jeecg/JUpload'
  import JSelectDepart from '@/components/jeecgbiz/JSelectDepart'
  import JDictSelectTag from "@/components/dict/JDictSelectTag"
  
  export default {
    name: "EmployeesInductionModal",
    components: { 
      JDate,
      JUpload,
      JSelectDepart,
      JDictSelectTag,
    },
    data () {
      return {
        form: this.$form.createForm(this),
        title:"操作",
        width:800,
        visible: false,
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        confirmLoading: false,
        validatorRules: {
          employeesName: {rules: [
            {required: true, message: '请输入姓名!'},
            { validator: (rule, value, callback) => validateDuplicateValue('employees_induction', 'employees_name', value, this.model.id, callback)},
          ]},
          employeesAge: {rules: [
            {required: true, message: '请输入年龄!'},
            {pattern:/\d{6,18}/, message: '请输入6到16位数字!'},
          ]},
          employeesSex: {rules: [
            {required: true, message: '请输入性别!'},
          ]},
          employeesDepartment: {rules: [
          ]},
          employeesDateentry: {rules: [
          ]},
          employeesBirthday: {rules: [
          ]},
          employeesHeadportrait: {rules: [
          ]},
          employeesResume: {rules: [
          ]},
          employeesIntroductiontothe: {rules: [
          ]},
        },
        url: {
          add: "/enterprise/employeesInduction/add",
          edit: "/enterprise/employeesInduction/edit",
        }
      }
    },
    created () {
    },
    methods: {
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'employeesName','employeesAge','employeesSex','employeesDepartment','employeesDateentry','employeesBirthday','employeesHeadportrait','employeesResume','employeesIntroductiontothe'))
        })
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            let formData = Object.assign(this.model, values);
            console.log("表单提交数据",formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
              that.close();
            })
          }
         
        })
      },
      handleCancel () {
        this.close()
      },
      popupCallback(row){
        this.form.setFieldsValue(pick(row,'employeesName','employeesAge','employeesSex','employeesDepartment','employeesDateentry','employeesBirthday','employeesHeadportrait','employeesResume','employeesIntroductiontothe'))
      }
      
    }
  }
</script>

<style lang="less" scoped>
/** Button按钮间距 */
  .ant-btn {
    margin-left: 30px;
    margin-bottom: 30px;
    float: right;
  }
</style>