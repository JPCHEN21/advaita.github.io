---
title: Advaita's Blog
date: 2020=10-05 08:20:06
---

地址：https://www.wangeditor.com/v5/for-frame.html#%E5%AE%89%E8%A3%85
一、安装
npm install @wangeditor/editor --save
npm install @wangeditor/editor-for-vue --save

二、使用
1. 新建一个Editor.vue组件，然后把Editor代码写在里面
2. 代码
<template>
    <div style="border: 1px solid #ccc;">
            <Toolbar
                style="border-bottom: 1px solid #ccc"
                :editor="editor"
                :defaultConfig="toolbarConfig"
                :mode="mode"
            />
            <Editor
                style="height: 500px; overflow-y: hidden;"
                v-model="html"
                //这里绑定了编辑器的配置了！！
                :defaultConfig="editorConfig" 
                :mode="mode"
                @onCreated="onCreated"
                @onChange="onChange"
            />
        </div>
</template>

<script>
import Vue from 'vue'
import { Editor, Toolbar } from '@wangeditor/editor-for-vue'

export default Vue.extend({
    components: { Editor, Toolbar },
    data() {
        return {
            editor: null, //实例
            html: '<p>hello</p>',
            toolbarConfig: { }, //工具栏配置
            editorConfig: { placeholder: '请输入内容...' },//编辑器配置
            mode: 'default', // or 'simple'
        }
    },
    methods: {
        onCreated(editor) {
            this.editor = Object.seal(editor) // 一定要用 Object.seal() ，否则会报错
        },
      onChange(editor) { console.log('onChange', editor.children) },
    },
    mounted() {
        // 模拟 ajax 请求，异步渲染编辑器
        setTimeout(() => {
            this.html = '<p>模拟 Ajax 异步设置内容 HTML</p>'
        }, 1500)
    },
    beforeDestroy() {
        const editor = this.editor
        if (editor == null) return
        editor.destroy() // 组件销毁时，及时销毁编辑器，不然会出现内存泄露问题
    }
})
</script>

<style src="@wangeditor/editor/dist/css/style.css"></style>
3. 在需要的地方进行引入即可。
4. 编辑器配置，通过绑定vue事件来进行操作，包括像上面那个创建编辑器实例的时候onCreated用的Object.seal()，还有onChange，onBlur等等事件
  a. onChange(editor) { console.log('onChange', editor.children) }，编辑器的内容每发生一次变化（包括鼠标点击，或者滑选），都会打印出编辑器里面的内容，或者直接打印onChange(editor) { console.log(editor.children[0].children[0].text) }，就可以打印出编辑器的内容
  b. 或者直接调用实例的api就可以获取内容了，editor.getText()/editor.getHtml();
  c. 打印editor实例，都可以看到它满满的属性和方法。那么也可以调用它的API。
    ⅰ. https://www.wangeditor.com/v5/API.html
  d. 
  e. 找到一些比较可能比较有用的API
    ⅰ. editor.showProgressBar(progress) ：显示进度条，一般用于上传功能，progress 为 0-100 的数字
    ⅱ. editor.getConfig()  ：查看编辑器默认配置

    ⅲ. editor.getConfig().hoverbarKeys可以查看当前的hoverbarKeys
    ⅳ. editor.children里面通过type可以看到当前编辑器区元素的类型

三、问题
菜单配置：图片、视频等menuKey
1. 图片上传提示：Cannot get upload server address 没有配置上传地址
2. 步骤：
  a. editor.getAllMenuKeys() 获取菜单所有的key
  b. editor.getMenuConfig('uploadImage') 获取某个key，例如uploadImage 的当前配置


3. 官方文档是拿到menukeys，针对这个key的配置文件去改，但是不清楚怎么去找这个配置文件，就直接改它的server属性。文档是让写"/api/upload"，但是这样写不行，因为前端项目没有去配baseURL为http://127.0.0.1:3000，所以这里server服务器地址要写全为"http://127.0.0.1:3000/api/upload-img"，这样在框框那里点上传图片，图片才会搞到编辑器内容区里。那么到时候要确定后端存图片设置的静态目录，这样才能正常地对接后台服务器。


4. 官方是koa做的后台，通过save-files方法来处理上传的图片，并且配置图片存储的文件位置，还有处理路径/链接/名称等等。

5. 必须的步骤
  a. post测请求（上传图片）是成功的
  b. 成功和失败必须按官方的响应格式去做（如果不按规定格式响应的话，那就自己去按官网的说法去配置上传）
  c. 返回的url是一个在线的图片，可以直接访问



6. 官网那个创建editorConfig来统一配置属性，方法的，用cs语法不行，用到ts应该才可以，所以，然后看网上的教程，直接在data里面去配置也行，因为需要配置的key不算多，而且只需要改需要配置的属性，其它属性还是按默认配置走。

编辑器配置：
1. 直接在editorConfig的下一层去加就行了。有生命周期函数，也有基本的属性配置。
  a. placeholder、readOnly、autoFocus、scroll、maxLength onMaxLength、hoverbarKeys
  b. onCreated、onChange、onDestroyed、onFocus、onBlur
  c. customPaste、customAlert
2. 编辑器API：
  a. 分为config相关（getConfig/getMenuConfig/getAllMenuKeys/alert等等）、
  b. 内容处理相关（getHtml、getText等等）、
  c. 节点操作相关（insertNodes、removeNodes等等）、
  d. DOM相关（focus、blur、disable等等）、
  e. section相关（select、selectAll、move等等）、
  f. 自定义事件（on、off、once、emit等等）

工具栏配置

1. 这里工具栏配置是通过toolbarConfig来进行配置
2. getConfig、toolbarKeys、insertKeys、excludeKeys、modalAppendToBody
四、总结
<template>
    <div style="border: 1px solid #ccc;">
            <Toolbar
                style="border-bottom: 1px solid #ccc"
                :editor="editor"
                :defaultConfig="toolbarConfig"  //工具栏配置
                :mode="mode"
            />
            <Editor
                style="height: 500px; overflow-y: hidden;"
                v-model="html"
                :defaultConfig="editorConfig"  //编辑器配置
                :mode="mode"
                @onCreated="onCreated"
            />
            <el-progress type="circle" :percentage="0"></el-progress>
    </div>
</template>

<script>
import Vue from 'vue'
import { Editor, Toolbar } from '@wangeditor/editor-for-vue'
import "@/ui/elementui"

export default Vue.extend({
    components: { Editor, Toolbar},
    data() {
        return {
            editor: null,
            html: '<p>hello</p>',
            toolbarConfig: {},
            editorConfig: { 
                placeholder: '请输入内容...' ,
                MENU_CONF:{// 配置都写在这里边
                    uploadImage : { 
                      // 每一个menuKey的配置，写的就是更新的，没写的就按原来默认的
                        server:"http://127.0.0.1:3000/api/upload-img",
                        maxFileSize: 1 * 1024 * 1024, // 1M 
                        //响应如果不能按格式的话，自定义插图
                        // customInsert(res, insertFn) {
                            //res 即服务端的返回结果
                            //从 res 中找到 url alt href ，然后插入图片
                            // console.log(res); 可以拿到alt，href，url
                            //insertFn(url, alt, href);
                        // },
                        // 小于该值就插入 base64 格式（而不上传），默认为 0
                        base64LimitSize: 5 * 1024 // 5kb
                    },
                    uploadVideo : {
                        server:"http://127.0.0.1:3000/api/upload-video",
                        maxFileSize: 20 * 1024 * 1024, 
                    }
                }
                
            },
            mode: 'default', // or 'simple',
            imgConf:{},
            percentage:''
        }
    },
    methods: {
        onCreated(editor) {
            this.editor = Object.seal(editor); // 一定要用 Object.seal() ，否则会报错
            this.imgConf = editor.getMenuConfig('uploadImage');
 
        }

    },
    mounted() {
        // 模拟 ajax 请求，异步渲染编辑器
        setTimeout(() => {
            this.html = '<p>模拟 Ajax 异步设置内容 HTML</p>'
        }, 1500)
    },
    beforeDestroy() {
        const editor = this.editor
        if (editor == null) return
        editor.destroy() // 组件销毁时，及时销毁编辑器
    }
})
</script>

<style src="@wangeditor/editor/dist/css/style.css"></style>