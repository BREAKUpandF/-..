注释：token为用户ID
https://element-plus.org/zh-CN/component/layout.html  //组件库地址

小提示： 
但是在写 ElMessage.error('Oops, this is a error message.')vscode自动导包
import { ElMessage } from 'element-plus'，变成了手动导入，
所以
代码失效的原因 可能是两种导入方式冲突


api接口中{
  提示：get和delete  请求要在对象中写参数，put，post可以直接写data
  index： 统一导出使用store数据
  user：{接收登录信息，用户的token 
         userRegister //用户注册接口
        userLogin//用户登录接口
        userGetInfoService //用户信息获取接口
   } 
   article:{
    artGetChannel //分类列表的获取
    artAddChannel //文章分类的添加
    artEditChannel //文章分类的修改
    artDeteleChannel //删除文章分类
    artGetList    //获取文章列表
    artAddArtcle  //添加文章
    artGetDetail  //获取文章内容详情，实现回显
    artEdit  //提交编辑后的内容
   }
}



utils：{
  request：{
    封装请求函数，再每一次请求时，自动添加请求头，配置接口的基地址，设置超时响应时间为10s；

  当返回值code为0则返回值正确，否则抛出异常 return Promise.reject(res.data),并且提示错误    ElMessage.error(res.data.message || '服务异常') 前者是我们提供的数据存在问题，后者为服务器存在问题；

  当错误返回值为401 时，代表权限不足，或者token过期
   if (err.response?.status === 401) {
      router.push('/login')
    }
    则应该跳转到登录页进行重新登录，token的重新获取
  }

}

router:{
路由结构：{
    path	          文件	                            功能	    组件名	          路由级别
├─ /login	          views/login/LoginPage.vue	        登录&注册	LoginPage	       一级路由
├─ /                views/layout/LayoutContainer.vue	布局架子	LayoutContainer	 一级路由
├─ /article/manage	views/article/ArticleManage.vue	  文章管理	ArticleManage	   二级路由
├─ /article/channel	views/article/ArticleChannel.vue	频道管理	ArticleChannel	 二级路由
├─ /user/profile	  views/user/UserProfile.vue	      个人详情	UserProfile	     二级路由
├─ /user/avatar	    views/user/UserAvatar.vue	        更换头像	UserAvatar	     二级路由
├─ /user/password	  views/user/UserPassword.vue	      重置密码	UserPassword	   二级路由
—————————————————————————————————————————————————————————————————————————————————————————
}

  index:{
      history: createWebHistory(import.meta.env.BASE_URL) 代表这模式类型当前为history模式
      如果是hash模式则为 createHashWebHistory  
      import.meta.env.BASE_URL则是为配路劲的前缀路径  如 jd/pay 这个jd就是前缀，需要配合vite.config.js 中 base 来自定义 
  }
}



views:{
    login:{
      LoginPage:{
        <el-row class="login-page">将页面分为24份
        <el-col :span="12" class="bg"></el-col> //分为12分
        <el-col :span="6" :offset="3" class="form"> //分为12份，占6份，offset左边占3份
      <el-form ref="form" size="large" autocomplete="off" v-if="isRegister"> //建立表单
       <el-form-item>//代表表单的一行

       拓展在密码输入框增加密码可视化的小眼睛
         #导包和设置input文本是text或者password
          import { View, Hide} from '@element-plus/icons-vue';
          const passFlag=ref(false)//输入框类型标识
          const addPassFlag=ref(false)//图标显示标识
          const password=ref()//密码

           <el-input v-model="password" :type="addPassFlag ? 'text' : 'password'" placeholder="请输入密码"
           :prefix-icon='Lock'>
               <template #suffix>
            <span @click="addPassFlag = !addPassFlag">
            <el-icon v-if="addPassFlag"><View /></el-icon>
            <el-icon v-else><Hide /></el-icon>
          </span>
        </template>
      </el-input>
       Vue.js 和使用 Element UI 框架的上下文中，<template #suffix> 是一种特殊的语法，用于定义一个名为 suffix 的插槽（slot）。插槽是 Vue.js 的一个特性，它允许你定义可在组件外部填充的内容模板。在这个例子中，suffix 插槽用于自定义 <el-input> 组件的后缀内容。


       :modle="formModel"
        :rules="rules"
        绑定表单的输入内容到formModle的中，校验规则配置在rules中

      }
    }
    article:{
      component：{
        channelEdit：{
          对话框组件
          const emit = defineEmits(['success'])
            const onSubmit = async () => {
              await formRef.value.validate()
              const isEdit = formModel.value.id
              if (isEdit) {
                await artEditChannel(formModel.value)
                ElMessage.success('编辑成功')
              } else {
                await artAddChannel(formModel.value)
                ElMessage.success('添加成功')
              }
              dialogVisible.value = false
              emit('success')
            }
            在提交表单后要向父组件发送成功的信息，进行数据的重新获取和渲染
        }
        articleChannel:{
          //添加文章和编辑文章框

          const onPublish = async (state) => {
            formModel.value.state = state
            const fd = new FormData()
            for (let key in formModel.value) {
              fd.append(key, formModel.value[key])
            }
            if (formModel.value.id) {
              await artEdit(fd)
              visibleDrawer.value = false
              ElNotification({
                message: h('i', { style: 'color: green' }, '编辑成功'),
                type: 'success',
                duration: 1000
              })
              emit('success', 'edit')
            } else {
              await artAddArtcle(fd)
              visibleDrawer.value = false
              ElNotification({
                message: h('i', { style: 'color: green' }, '添加成功'),
                type: 'success',
                duration: 1000
              })
              emit('success', 'add')
            }
          }
          提交修改或编辑函数，注意点，在提交后要向父组件提交成功提示edit或add
           const fd = new FormData()
           提交到后端的内容与需要为FormData，需要转换成FormData
          imageUrlToFile将imgURL转换成File格式
        }
        channelSelect:{
          //渲染分类页
        }
      }
        ArticleMange:{
         分类页 进行了表格的构建
         const onSuccess = (state) => {
              if (state === 'edit') {
                getArticleList()
              } else {
                const lastPage = Math.ceil((total.value + 1) / params.value.pagesize)
                params.value.pagenum = lastPage
                getArticleList()
              }
            }
            接收子组件传递的信号，edit则直接请求数据，add则请求最后一样的数据
        }

        ArticleChenel:{
           进行了表格的构建

            <channelEdit @success="onSuccess" ref="dialog"></channelEdit>
            子组件的调用和，父子通讯和页面的重新渲染

            const addChanel = async () => {
              dialog.value.open({})
              // await artAddChannel(cate_name, cate_alias)
            }
            const onEditChannel = (row) => {
              dialog.value.open(row)
            }
            利用组件导出的函数
        }
    }

    layout:{
      LayoutContainer:{
        //存放架子
      }
    }
}

结合AI：{
  利用chatgpt和copilot优化代码和框架的建设，
  再，配置接口
}