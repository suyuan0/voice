<template>
	<view class="content">
		<!-- {{text}} -->
		<button @click="startLuyin" class="recordingStyle">点击开始说话</button>
		<view>识别的结果：</view>
		<textarea v-model="searchText"></textarea>
		<view class="">
			{{msg}}
		</view>
		<button @tap="startRecord">开始录音</button>
		<button @tap="endRecord">停止录音</button>
		<button @tap="playVoice">播放录音</button>
		<button @click="upload">上传</button>
		<button @click="to">跳转</button>
	</view>
</template>

<script>
	// 录音
	const recorderManager = uni.getRecorderManager()
	// 播放录音
	const innerAudioContext = uni.createInnerAudioContext()
	innerAudioContext.autoplay = true
	export default {
		data() {
			return {
				speechEngine: 'baidu',
				searchText: '',
				msg: '',
				text: 'uni-app',
				voicePath: ''
			}
		},
		onLoad() {
			let self = this;
			recorderManager.onStop(function(res) {
				self.text = 'recorder stop' + JSON.stringify(res)
				console.log(res.tempFilePath);
				plus.io.resolveLocalFileSystemURL(res.tempFilePath, entry => {
					entry.file(file => {
						const fileReader = new plus.io.FileReader()
						// fileReader.readAsDataURL(file)
						fileReader.onloadend = e => {
							self.voicePath = e.target.result
						}
						// fileReader.close()
					})
				})
				// self.voicePath = res.tempFilePath;
			});
		},
		methods: {
			startLuyin() {
				this.msg = '语音输入'
				let _this = this
				let options = {}
				options.engine = _this.speechEngine
				options.punctuation = false // 是否需要标点符号
				options.timeout = 10 * 1000 // 超时时间
				plus.speech.startRecognize(options, e => {
					this.msg = '以上是识别结果'
					_this.searchText = e
					plus.speech.stopRecognize() // 关闭
				})
			},
			startRecord() {
				console.log('开始录音');
				recorderManager.start({
					sampleRate: '44100'
				});
			},
			endRecord() {
				console.log('录音结束');
				recorderManager.stop();
			},
			playVoice() {
				console.log('播放录音');
				if (this.voicePath) {
					innerAudioContext.src = this.voicePath;
					innerAudioContext.play();
				}
			},
			upload() {
				this.searchText = this.voicePath
				console.log(this.searchText);
				// uni.request({
				// 	url: "http://192.168.1.49:8090/paddlespeech/asr",
				// 	method: 'POST',
				// 	data: {
				// 		audio: this.voicePath,
				// 		'audio_format': 'wav',
				// 		'sample_rate': 16000,
				// 		lang: 'zh_cn',
				// 		'punc': 0
				// 	},
				// 	success: res => {
				// 		console.log(res);
				// 		// this.searchText = JSON.stringify(res)
				// 	}
				// })
			},
			to() {
				uni.navigateTo({
					url: '/pages/yuyin/yuyin'
				})
			}
		}
	}
</script>

<style>
	.content {
		padding: 40rpx;
	}

	.recordingStyle {
		border-radius: 40rpx;
		text-align: center;
		color: #fff;
		font-size: 30rpx;
		background-color: #409eff;
		margin-bottom: 15px;
	}
</style>
