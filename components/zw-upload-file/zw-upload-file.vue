<template>
	<view>
		<view class="feedback-title" style="background-color: #efeff4;">
			<text>{{ title }}</text>
		</view>
		<!-- #ifdef APP-PLUS -->
		<view class="uni-list">
			<view class="uni-list-cell pro-input" v-for="(path, index) in dataPaths" :key="index">
				<view class="uni-list-cell-left asterisk" style="width:100rpx;">附件({{ index + 1 }})</view>
				<view class="uni-list-cell-db input" @tap="_openBrowser(path)"><input class="uni-input list-input" name="input" :value="path" disabled="disabled" /></view>
				<view class="deleteBtn" v-if="step == 1" @click="_removePro(index)"><icon type="clear" size="20" /></view>
				<view class="deleteBtn" v-if="step == 2" @click="_downLoad(path)"><icon type="download" size="20" /></view>
			</view>
			<view><button class="mini-btn up-btn" type="default" size="mini" @click="_bingUpFile()">上传附件</button></view>
		</view>
		
		<!-- #endif -->
	</view>
</template>

<script>
export default {
	name: 'ZwUploadFile',
	props: {
		title: {
			type: String,
			default: '文档(选填,提供文档,总大小10M以下)'
		},
		path: {
			type: String,
			default: ''
		},
		ip: {
			type: String,
			default: ''
		},
		step: {
			type: Number,
			default: 1
		},
		dataPaths: {
			type: Array,
			default() {
				return [];
			}
		}
	},
	data() {
		return {
			docType: ['doc', 'xls', 'ppt', 'pdf', 'docx', 'xlsx', 'pptx']
		};
	},
	methods: {
		openFile(type) {
			this.$refs.data._openFile();
		},
		_openBrowser(path) {
			if (this.isFile(path)) {
				console.log(path);
				var filePath = path;
				//获取最后一个.的位置
				var index = filePath.lastIndexOf('.');
				//获取后缀
				var ext = filePath.substr(index + 1);
				//输出结果
				console.log(this.docType.indexOf(ext));
				if (this.docType.indexOf(ext) != -1) {
					uni.showModal({
						title: '提示',
						content: '查看文档',
						success: function(res) {
							if (res.confirm) {
								uni.downloadFile({
									url: this.ip + path,
									success: res => {
										console.log(res);
										if (res.statusCode === 200) {
											uni.openDocument({
												filePath: res.tempFilePath,
												success: () => {
													console.log('文档打开成功');
												},
												fail: error => {
													console.log(error);
													uni.showToast({
														icon: 'none',
														title: error.errMsg,
														position: 'bottom'
													});
												}
											});
										} else {
											uni.showToast({
												icon: 'none',
												title: '文件下载失败',
												position: 'bottom'
											});
										}
									}
								});
							}
						}
					});
				} else {
					plus.runtime.openURL(encodeURI(url), function(err) {
						plus.nativeUI.alert('暂不支持打开该文件');
					});
				}
			} else {
				plus.runtime.openURL(encodeURI(url), function(err) {
					plus.nativeUI.alert('暂不支持打开该文件');
				});
			}
		},
		isFile(path) {
			let bool = true;
			let File = plus.android.importClass('java.io.File');
			let fd = new File(path);
			if (fd.isDirectory()) {
				bool = false;
			}
			if (fd.isFile()) {
				bool = true;
			}
			return bool;
		},
		_removePro(index) {
			//删除文件
			uni.showModal({
				title: '提示',
				content: '确认删除此文件?',
				success: res => {
					if (res.confirm) {
						this.dataPaths.splice(index, 1);
					}
				}
			});
			this.$emit('removePro', this.dataPaths);
		},
		_downLoad(path) {
			plus.runtime.openURL(this.ip + path);
		},
		_bingUpFile() {
			this.$emit('bingUpFile');
		}
		
	}
};
</script>

<style>
.mini-btn {
	margin-left: 20rpx;
}

.uni-list-cell {
	justify-content: flex-start;
}

.deleteBtn {
	width: 100rpx;
	text-align: center;
	line-height: 0;
	border-left: 1px solid #ebdfdf;
}
</style>
