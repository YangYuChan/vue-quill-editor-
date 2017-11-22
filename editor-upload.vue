<template>
	<section class="main-content-container clearfix">
		<div class="main-wrapper">
			<div class="person-box">
				<div class="editor-container">
					<quill-editor ref="myTextEditor"
                              v-model="content"
                              :options="editorOption"
                              @focus="onEditorFocus($event)"
                              @ready="onEditorReady($event)"                 
                	>
                  <div id="toolbar" slot="toolbar" class="clearfix">
                    <button class="ql-bold fl-left"></button>
                    <button class="ql-underline fl-left"></button>
                    <button class="ql-strike fl-left"></button>
                    <button class="ql-blockquote fl-left"></button>
                    <button class="ql-list fl-left" value="ordered"></button>
                    <button class="ql-list fl-left" value="bullet"></button>
                    <button class="ql-indent fl-left" value="-1"></button>
                    <button class="ql-indent fl-left" value="+1"></button>
                    <button class="ql-direction fl-left" value="rtl"></button>
                    <select class="ql-align">
                    	<option selected="selected"></option>
                    	<option value="center"></option>
                    	<option value="right"></option>
                    	<option value="justify"></option>
                    </select>
                    <button class="ql-clean fl-left"></button>
                     <button class="ql-link fl-left"></button>
                      <button class="ql-video fl-left"></button>
                    <select class="ql-size fl-left">
                      <option value="small"></option>
                      <option selected></option>
                      <option value="large"></option>
                      <option value="huge"></option>
                    </select>
                    <!--自定义一个上传按钮-->
                    <div class="image fl-left">
                    	<button type="button" class="ql-images"><i class="icon iconfont icon-icon-test"></i></button>
	                    <input type="file" class="custom-input" @change="uploadEditorImage" @click="customButtonClick">
                    </div>
                  </div>
               </quill-editor>
				</div>

			</div>
		</div>
	</section>
</template>

<script>
	import AXIOS from '../../axios/axios';
	import global_ from '../../common/js/common';
	import valid from '../../common/js/validate';
	import $ from 'jquery';
	import { quillEditor } from 'vue-quill-editor'
	const Axios = new AXIOS();
	export default {
		data() {
			return {
				path: global_.path,
				action: global_.action,
				token: '',
				sysUserId: '',
				isAdmin: null,
				content: '开始编辑您的文章吧',
				uploadVisible: false,
				qeditor: '',
				length: '',
				uploadImage:'',
				editorOption: {
					modules: {
						toolbar: '#toolbar',
					}
				}
			}
		},
		methods: {
			uploadUrl() {
				return this.action
			},
			uploadHeaders() {
				var headers = {
					'Authorization': this.token
				}
				return headers
			},
			onEditorFocus(editor) {
				this.qeditor = editor
			},
			onEditorReady(editor) {
				this.qeditor = editor;
			},
			customButtonClick() {
				var range
				if(this.qeditor.getSelection() != null) {
					range = this.qeditor.getSelection()
					this.length = range.index //content获取到焦点，计算光标所在位置，目的为了在该位置插入img
				} else {
					this.length = this.content.length //content没有获取到焦点时候 目的是为了在content末尾插入img
				}
			},
			uploadEditorImage(){
		        var img = event.target.files[0];
		        if(!img){  
		            return ;  
		        }
		        var that = this;
		       	var formData = new FormData(); //上传图片转换格式
 				formData.append("file", img);  
		        this.$axios({
						url: global_.action, //上传到阿里云、七牛云的路径，我们这边是阿里云，由后台配置
						method: 'put',
						data: formData,
						headers: {
						'Content-Type': 'application/x-www-form-urlencoded',
						'Authorization': this.token,
						}
						})
						.then((res) => {
							let self = this;
							self.qeditor.insertEmbed(self.length, 'image', res.data); //把返回的地址插入到编辑器中
						})
						.catch((err) => {
							console.log(err);
						})


			},
		},
		computed: {
			editor() {
				return this.$refs.myTextEditor.quill;
			}
		},
}
</script>

<style lang="scss">
	.editor-example {
		border: 1px solid #ccc;
	}
	
	.ql-container .ql-editor {
		min-height: 30em;
		padding-bottom: 1em;
		max-height: 35em;
	}
	
	.html {
		height: 9em;
		overflow-y: auto;
		border: 1px solid #ccc;
		border-top: none;
		resize: vertical;
	}
	#toolbar .image{
		position: relative;
		
	}
	
	#toolbar .icon-icon-test {
		font-size: 22px;
		cursor: pointer;
		display: block;
		
	}
	#toolbar .icon-icon-test:hover{
		color:#06c;
	}
	#toolbar .ql-images{
		cursor: pointer;
		display: block;
		width: 28px;
		height: 24px;
		padding: 0;
	}
	
	#toolbar .custom-input{
		position: absolute;
		top: 0;
		left: 0;
		width: 28px;
		height: 24px;
		opacity: 0;
		z-index: 2;
		font-size: 0;
	}
</style>