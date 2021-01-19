<template>
	<view class="content">
		<button type="primary" @click="btnTakePhoto">从相册或者拍照获得照片</button>
		<image :src="imgSrc" mode="widthFix"></image>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				imgSrc: ''
			}
		},
		onLoad() {

		},
		methods: {
			btnTakePhoto () {
				wx.chooseImage({
					count: 1,
					success: res => {
						console.log('res', res)
						this.imgSrc = res.tempFilePaths[0]
						this.imageToBase64(this.imgSrc)
					}
				})
			},
			// 图片转base64
			imageToBase64 (filepath) {
				wx.getFileSystemManager().readFile({
					filePath: filepath,
					encoding: 'base64',
					success: res => {
						console.log('res', res.data)
						this.classify(res.data)
					}
				})
			},
			// 调用百度ai接口
			// key SojyYNQ9Z7CGCCRmH1c9bpPQ appi: 23564009
			async classify (base64) {
				wx.showLoading()
				let [err, res] = await uni.request({
					url: `https://aip.baidubce.com/oauth/2.0/token?grant_type=client_credentials&client_id=SojyYNQ9Z7CGCCRmH1c9bpPQ&client_secret=TrwRu1fxXatBV06AzUM6ck2HXGbrbuPP`
				})
				// console.log('res', res)
				let access_token = res.data.access_token
				let [error, result] = await uni.request({
					url: 'https://aip.baidubce.com/rest/2.0/image-classify/v2/advanced_general',
					method: 'POST',
					header: {
						'Content-type': 'application/x-www-form-urlencoded'
					},
					data: {
						access_token: access_token,
						image: base64
					}
				})
				console.log('result', result.data.result)
				wx.hideLoading()
				this.parseResult(result.data.result)
			},
			// 展示结果
			parseResult (resultList) {
				let itemList = []
				for (let i = 0; i < resultList.length; i++) {
					itemList.push(resultList[i].keyword)
				}
				uni.showActionSheet({
					itemList: itemList,
					success: res => {
						console.log('res', res)
					}
				})
			}
		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}
</style>
