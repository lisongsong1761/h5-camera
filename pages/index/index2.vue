<template>
	<view class="content">
		<view class="header">
			<!-- <image src="@/static/open_light.png" v-if="openlight" @click="changelight('off')"></image>
			<image src="@/static/close_light.png" v-else @click="changelight('on')"></image> -->
			<view class="title">拍照上传{{type===1 ? '007':'008'}}</view>
		</view>
		<view id="myVideo" style="position: relative;" v-show="videoPlaying"></view>
		<!-- <video id="v" :src="videoSrc"></video> -->
		<view class="btns">
			<view class="goBack" @click="goBack">取消</view>
			<image src="@/static/take_photo.png" @click="takePhoto"></image>
			<view class="switchType" @click="switchType">切换</view>
		</view>
		<view class="all_canvas" v-show="!photoSrc">
			<canvas class="canvas_1" style="width: 100vw; height: 100vh;" canvas-id="firstCanvas" id="firstCanvas"></canvas>
		</view>
		<view class="my_photo" v-if="photoSrc">
			<image mode="widthFix" class="camera_photo" :src="photoSrc"></image>
			<image class="cancel_btn" src="@/static/cancel.png" @click="cancel"></image>
			<image class="confirm_btn" src="@/static/confirm.png" @click="confirm"></image>
		</view>
	</view>
</template>

<script>
	import uniData from "@/utils/uniWebview1-5-2.js"
	const screenWidth = uni.getSystemInfoSync().screenWidth
	const screenHeight = uni.getSystemInfoSync().screenHeight
	const statusBarHeight = uni.getSystemInfoSync().statusBarHeight
	const safeArea = uni.getSystemInfoSync().safeArea
	const safeAreaBottom = uni.getSystemInfoSync().safeArea.bottom - uni.getSystemInfoSync().safeArea.height
	// const pixelRatio = uni.getSystemInfoSync().pixelRatio
	const pixelRatio = 5 // 4
	const bottomHeight = uni.upx2px(200)
	// 可见高度
	const realHeight = safeArea.height - bottomHeight
	// 屏幕长宽比
	const aspectRatio = realHeight / screenWidth
	// A4纸长宽比
	const A4_aspectRatio = 1.414286
	export default {
		data() {
			return {
				videoSrc: null,
				myVideo: null,
				videoPlaying: false,
				openlight: false,
				photoSrc: null,
				type: 1 ,// 1是007，2是008
				token: null
			}
		},
		onLoad(option) {
			this.token = option.token
		},
		onShow() {
			this.$nextTick(() => {
				this.init() // 初始化相机
				this.drawFirstCanvas() // 绘制定位框
			})
		},
		methods: {
			drawFirstCanvas() {				
				const context = uni.createCanvasContext('firstCanvas')
				context.setLineWidth(2)
				
				if(this.type === 1) {
					// 007
					if(aspectRatio > A4_aspectRatio) { // 先取宽
						context.moveTo(0, 44+statusBarHeight);
						context.lineTo(screenWidth, 44+statusBarHeight);
						context.lineTo(screenWidth, screenHeight-bottomHeight-safeAreaBottom);
						context.lineTo(0, screenHeight-bottomHeight-safeAreaBottom);
						context.closePath();
						
						context.moveTo(32, (realHeight-(screenWidth-64)*A4_aspectRatio)/2 + 44);
						context.lineTo(32, (realHeight-(screenWidth-64)*A4_aspectRatio)/2 + 44 + (screenWidth-64)*A4_aspectRatio);
						context.lineTo(screenWidth-32, (realHeight-(screenWidth-64)*A4_aspectRatio)/2 + 44 + (screenWidth-64)*A4_aspectRatio);
						context.lineTo(screenWidth-32, (realHeight-(screenWidth-64)*A4_aspectRatio)/2 + 44);
						context.closePath();
						context.fillStyle = 'rgba(0, 0, 0, 0.8)';
						context.fill()
						
						// 外实线
						context.setLineDash([1,0])
						context.setStrokeStyle('#00ff00')
						context.strokeRect(32, (realHeight-(screenWidth-64)*A4_aspectRatio)/2 + 44, screenWidth-64, (screenWidth-64)*A4_aspectRatio)
						// 内虚线
						context.setLineDash([10, 20], 5);
						// context.strokeRect(32, (realHeight-(screenWidth-64)*A4_aspectRatio*0.8)/2 + 44, screenWidth-140, (screenWidth-64)*A4_aspectRatio*0.8)
						context.strokeRect(32+(screenWidth-64)*0.07, (realHeight-(screenWidth-64)*A4_aspectRatio)/2 + 44 + (screenWidth-64)*A4_aspectRatio*0.057, (screenWidth-64)*0.8365, (screenWidth-64)*A4_aspectRatio*0.91)
					} else {
						// 先取长
					}
				} else {
					// 008
					if(aspectRatio > A4_aspectRatio) { // 先取宽
						context.moveTo(0, 44+statusBarHeight);
						context.lineTo(screenWidth, 44+statusBarHeight);
						context.lineTo(screenWidth, screenHeight-bottomHeight-safeAreaBottom);
						context.lineTo(0, screenHeight-bottomHeight-safeAreaBottom);
						context.closePath();
						
						context.moveTo(32, (realHeight-(screenWidth-64)*A4_aspectRatio)/2 + 44);
						context.lineTo(32, (realHeight-(screenWidth-64)*A4_aspectRatio)/2 + 44 + (screenWidth-64)*A4_aspectRatio);
						context.lineTo(screenWidth-32, (realHeight-(screenWidth-64)*A4_aspectRatio)/2 + 44 + (screenWidth-64)*A4_aspectRatio);
						context.lineTo(screenWidth-32, (realHeight-(screenWidth-64)*A4_aspectRatio)/2 + 44);
						context.closePath();
						context.fillStyle = 'rgba(0, 0, 0, 0.8)';
						context.fill()
						
						// 外实线
						context.setLineDash([1,0])
						context.setStrokeStyle('#00ff00')
						context.strokeRect(32, (realHeight-(screenWidth-64)*A4_aspectRatio)/2 + 44, screenWidth-64, (screenWidth-64)*A4_aspectRatio)
						// 内虚线
						context.setLineDash([10, 20], 5);
						// context.strokeRect(32, (realHeight-(screenWidth-64)*A4_aspectRatio*0.8)/2 + 44, screenWidth-140, (screenWidth-64)*A4_aspectRatio*0.8)
						context.strokeRect(32+(screenWidth-64)*0.053, (realHeight-(screenWidth-64)*A4_aspectRatio)/2 + 44 + (screenWidth-64)*A4_aspectRatio*0.2779, (screenWidth-64)*0.89, (screenWidth-64)*A4_aspectRatio*0.624)
					} else {
						// 先取长
					}
				}
				context.draw()
			},
			init() {
				// 老的浏览器可能根本没有实现 mediaDevices，所以我们可以先设置一个空的对象
				if (navigator.mediaDevices === undefined) {
					navigator.mediaDevices = {};
				}
				if (navigator.mediaDevices.getUserMedia === undefined) {
					navigator.mediaDevices.getUserMedia = function(constraints) {
						// 首先，如果有getUserMedia的话，就获得它
						const getUserMedia = navigator.webkitGetUserMedia || navigator.mozGetUserMedia || navigator.msGetUserMedia;
						// 一些浏览器根本没实现它 - 那么就返回一个error到promise的reject来保持一个统一的接口
						if (!getUserMedia) {
							return Promise.reject(new Error('getUserMedia is not implemented in this browser'));
						}
						// 否则，为老的navigator.getUserMedia方法包裹一个Promise
						return new Promise(function(resolve, reject) {
							getUserMedia.call(navigator, constraints, resolve, reject);
						});
					}
				}
				
				const constraints = {
					video: {
						width: window.innerHeight*pixelRatio,
						height: window.innerWidth*pixelRatio,
						facingMode: "environment",
						// facingMode: "user",
						frameRate: {
							ideal: 50,
							min: 30
						}
					},
					audio: false
				};
				if (!this.myVideo) {
					let videoView = document.getElementById('myVideo');
					let myVideo = document.createElement('video');
					
					myVideo.style.width = uni.getSystemInfoSync().screenWidth + 'px'
					// myVideo.style.minWidth = uni.getSystemInfoSync().screenWidth * 4 / 3 + 'px'
					myVideo.style.heigth = uni.getSystemInfoSync().screenHeight + 'px'
					videoView.appendChild(myVideo)
					this.myVideo = myVideo
					this.getUserMedia(constraints)
				}
			},
			getUserMedia(constraints) {
				let promise = navigator.mediaDevices.getUserMedia(constraints);
				promise.then(stream => {
					// 旧的浏览器可能没有srcObject
					if ("srcObject" in this.myVideo) {
						this.myVideo.srcObject = stream;
						console.log(11111111111111111);
						console.log(JSON.stringify(stream));
						console.log(22222222222222222);
					} else {
						// 防止在新的浏览器里使用它，应为它已经不再支持了
						this.myVideo.src = window.URL.createObjectURL(stream);
					}
					this.myVideo.onloadedmetadata = (e) => {
						this.myVideo.play();
						this.videoPlaying = true;
					};
				}).catch(err => {
					console.error(err.name + ": " + err.message);
				})
			},
			// 返回
			goBack() {
				uni.webView.postMessage({
					data: {
						action: 'goBack'  
					}
				});
			},
			// 切换类型
			switchType() {
				this.type = this.type === 1 ? 2 : 1
				this.drawFirstCanvas()
			},
			// 打开手电筒
			changelight(type) {
				this.openlight = !this.openlight
				if(type === 'on') {
					uni.webView.postMessage({
						data: {
							action: 'openLight'  
						}
					});
				} else if(type === 'off') {
					uni.webView.postMessage({
						data: {
							action: 'closeLight'  
						}
					});
				}
			},
			takePhoto() { // 添加防抖
				// uni.chooseImage({
				// 	count: 1, //默认9
				// 	sizeType: ['original'], //可以指定是原图还是压缩图，默认二者都有
				// 	sourceType: ['album'], //从相册选择
				// 	success: function(res) {
				// 		console.log(JSON.stringify(res.tempFilePaths));
				// 	}
				// });
				if (this.videoPlaying) {
					let myCanvas = document.createElement('canvas');
					let videoView = document.getElementById('myVideo');
					videoView.appendChild(myCanvas)
					myCanvas.style.display = 'none'
					// myCanvas.width = this.myVideo.videoWidth;
					// myCanvas.height = this.myVideo.videoHeight;
					// myCanvas.getContext('2d').drawImage(this.myVideo, 0, 0);
					/* myCanvas.width = (screenWidth-64)*pixelRatio;
					myCanvas.height = (screenWidth-64)*A4_aspectRatio*pixelRatio; */
					let canPixelRatio = 8
					myCanvas.width = (screenWidth-64)*canPixelRatio;
					myCanvas.height = (screenWidth-64)*A4_aspectRatio*canPixelRatio;
					// myCanvas.getContext('2d').drawImage(this.myVideo, -32*pixelRatio, -((realHeight-(screenWidth-64)*A4_aspectRatio)/2 + 44)*pixelRatio);
					myCanvas.getContext('2d').drawImage(this.myVideo, -32*canPixelRatio, -((realHeight-(screenWidth-64)*A4_aspectRatio)/2 + 44)*canPixelRatio, screenWidth*canPixelRatio, screenHeight*canPixelRatio);
					
					// let pixels = myCanvas.getContext('2d').getImageData(0,0,screenWidth*canPixelRatio, screenHeight*canPixelRatio);
					// let pixeldata = pixels.data;
					// for(let i=0,len = pixeldata.length ;i<len;i+=4){
					// 		let black =( pixels.data[i] + pixels.data[i+1] + pixels.data[i+2])/3;
					// 		if(black>=100){
					// 				black = 255
					// 		}else{
					// 				black = 0;
					// 		}
					// 		pixels.data[i] = black;
					// 		pixels.data[i+1] = black;
					// 		pixels.data[i+2] = black;
					// }
					
					let imageData = myCanvas.getContext('2d').getImageData(0,0,screenWidth*canPixelRatio, screenHeight*canPixelRatio)
					let pixels = imageData.data
					
					// 处理像素点
					for (let i=0; i<pixels.length; i+=4){
					    let r = pixels[i]
					    let g = pixels[i+1]
					    let b = pixels[i+2]
					
					    // 灰色
					    let gray = parseInt((r+g+b)/3)
					    imageData.data[i] = gray
					    imageData.data[i+1] = gray
					    imageData.data[i+2] = gray
					}
					// 重新画图
					myCanvas.getContext('2d').putImageData(imageData, 0,0)
					
					// let data = myCanvas.toDataURL('image/webp');
					let data = myCanvas.toDataURL("image/jpeg");
					this.photoSrc = data
					
					
					/* var urlObject = window.URL || window.webkitURL || window;
					// var export_blob = new Blob([data]);
					var export_blob = this.dataURLtoFile(data);
					var formData = new FormData()
					
					formData.set("file", export_blob);
					// formData.append('file', export_blob)
					// formData.append('fileType', 'png')
					formData.append('x', 0.06)
					formData.append('y1', 0.2)
					formData.append('y2', 0.95)
					formData.append('type', '007')
					
					var xhr = new XMLHttpRequest(); //创建对象
					
					// https://logistics.adicon.com.cn:4433/api-file/file/files/upload
					// xhr.open("POST", "http://10.0.28.18:8989/request/barcode/list/007?x=100&y1=100&y2=1000");
					// xhr.open("POST", "https://devlogistics.adicon.com.cn:6677/neo-logisticsApp/request/barcode/list");
					xhr.open("POST", "https://logistics.adicon.com.cn:4433/api-file/file/files/upload");
					
					xhr.setRequestHeader("Authorization", this.token);
					// xhr.setRequestHeader("Authorization", "bearer c5192f9f-edfe-4075-8da8-22efafec4a85");
					
					xhr.send(formData); //发送时  Content-Type默认就是: multipart/form-data; 
					xhr.onreadystatechange = function() {
						console.log('state change', xhr.readyState);
						if (this.readyState == 4 && this.status == 200) {
							var obj = JSON.parse(xhr.responseText); //返回值
							console.log(JSON.stringify(obj));
							if (obj.data) {
								console.log('上传成功');
							}
						} else {
							console.log('上传失败');
						}
					} */
				}
			},
			//将base64转换为文件
			dataURLtoFile(dataurl, filename) {
				let arr = dataurl.split(',')
				let mime = arr[0].match(/:(.*?);/)[1];
				if(!filename) {//若无文件名则取当前时间戳
					filename = Date.parse(new Date()) + '.jpg';
				}
				let bstr = atob(arr[1])
				let n = bstr.length
				let u8arr = new Uint8Array(n);
				while(n--){
						u8arr[n] = bstr.charCodeAt(n);
				}
				return new File([u8arr], filename, {type:mime});
			},
			//base64转化为blob
			dataURItoBlob(base64Data) {
				var byteString;
				if(base64Data.split(',')[0].indexOf('base64') >= 0)
						byteString = atob(base64Data.split(',')[1]);
				else{
						byteString = unescape(base64Data.split(',')[1]);
				}
				var mimeString = base64Data.split(',')[0].split(':')[1].split(';')[0];
				var ia = new Uint8Array(byteString.length);
				for(var i = 0; i < byteString.length; i++) {
						ia[i] = byteString.charCodeAt(i);
				}
				var blob = new Blob([ia], {
						type: mimeString
				});
				return blob;
			},
			cancel() {
				this.photoSrc = null
			},
			confirm() {
				uni.webView.postMessage({
					data: {
						action: 'photoSrc'  
					}
				});
			},
		}
	}
</script>

<style lang="scss" scoped>
	.content {
		background-color: #000000;
		height: 100vh;
		width: 100vw;
		overflow: hidden;
		padding: 0;
		margin: 0;

		.header {
			position: fixed;
			z-index: 99;
			left: 0;
			top: 0;
			width: 100vw;
			background-color: rgba(0, 0, 0, 1);
			
			padding-top: var(--status-bar-height);
			height: 44px;
			color: #FFFFFF;
			display: flex;
			align-items: center;
			justify-content: center;
			
			image {
				position: absolute;
				right: 64rpx;
				top: 50%;
				margin-top: -32rpx;
				z-index: 100;
				width: 64rpx;
				height: 64rpx;
			}

			.title {
				
			}
		}

		.btns {
			position: fixed;
			left: 0;
			bottom: 0;
			z-index: 99;
			padding-bottom: env(safe-area-inset-bottom);
			width: 100vw;
			height: 200rpx;
			background-color: rgba(0, 0, 0, 1);
			color: #FFFFFF;
			display: flex;
			align-items: center;
			justify-content: center;
			
			.goBack {
				position: absolute;
				left: 64rpx;
				top: 50%;
				margin-top: -32rpx;
				color: #FFFFFF;
				font-size: 32rpx;
			}
			
			.switchType {
				position: absolute;
				right: 64rpx;
				top: 50%;
				margin-top: -32rpx;
				color: #FFFFFF;
				font-size: 32rpx;
			}

			button {
				display: inline-block;
			}
			
			image {
				width: 120rpx;
				height: 120rpx;
				&:active {
					transform: scale(1.1);
				}
			}
		}
		.all_canvas {
			pointer-events: none;
			position: fixed;
			z-index: 9999;
			top: 0;
			left: 0;
		}
		.my_photo {
			position: fixed;
			z-index: 999;
			top: 0;
			left: 0;
			width: 100vw;
			height: 100vh;
			background-color: #000000;
			display: flex;
			align-items: center;
			justify-content: center;
			.camera_photo {
				width: 80vw;
				height: auto;
			}
			.cancel_btn {
				position: absolute;
				bottom: 100rpx;
				left: 64rpx;
				width: 96rpx;
				height: 96rpx;
			}
			.confirm_btn {
				position: absolute;
				bottom: 100rpx;
				right: 64rpx;
				width: 96rpx;
				height: 96rpx;
			}
		}
	}
</style>
