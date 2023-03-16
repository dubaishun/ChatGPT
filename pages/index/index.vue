<template>
	<view>
		<view style="color: #fff;font-size: 12px;opacity:0.8;padding-left: 25rpx;padding-top: 25rpx;">下方输入(客服)即可联系人工...
		</view>
		<scroll-view scroll-with-animation scroll-y="true" style="width: 100%;">
			<!-- 用来获取消息体高度 -->
			<view id="okk" scroll-with-animation>
				<!-- 消息 -->
				<view class="flex-column-start" v-for="(x,i) in msgList" :key="i">
					<!-- 用户消息 头像可选加入-->
					<view v-if="x.my" class="userinfo">
						<view class="flex justify-end" style="margin-right: 5%;">
							<view class="usermsg">
								<text style="word-break: break-all;color: #fff;">{{x.msg}}</text>
							</view>
						</view>
					</view>
					<!-- 机器人消息 -->
					<view v-if="!x.my" class="aiinfo">
						<view class="chat-img ">
							<zero-loading v-if="msgLoad" type="sword" position="absolute"></zero-loading>
							<zero-loading v-if="!msgLoad" type="circle" position="absolute"></zero-loading>
						</view>
						<view class="flex" style="width: 500rpx;position: relative;">
							<view class="aimsg" style="border-radius: 35rpx;background-color: #1F2937;">
								<zero-loading v-if="x.msg==true" type="pulse" position="absolute"></zero-loading>
								<text style="word-break: break-all;color: #fff;" v-if="x.msg != true">{{x.msg}}</text>
								<view v-if="x.msg=='当前咨询人数过多，请重试...'">
									<zero-loading type="circle" position="absolute"></zero-loading>
								</view>
							</view>
						</view>
					</view>
				</view>
				<!-- 防止消息底部被遮 -->
				<view style="height: 130rpx;">
				</view>
			</view>

		</scroll-view>
		<!-- 底部导航栏 -->
		<view class="flex-column-center">
			<view class="inpubut" v-if="msgLoad == false">
				<input v-model="msg" class="dh-input" type="text" @confirm="sendMsg" confirm-type="search"
					placeholder-class="my-neirong-sm" placeholder="描述您的问题" />
				<view style="position: relative;width: 200rpx;">
					<button class="setBtn" @click="sendMsg" :open-type="msg == '客服' ? 'contact':''">
						<zero-loading type="atom" position="absolute"></zero-loading>
					</button>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				apiurl: 'https://flask-web-fcapp.run',//地址插件小程序回复客服获取
				apisucc: false,
				api: '在此输入你的APIKEY',
				msgLoad: true,
				anData: {},
				animationData: {},
				showTow: false,
				msgList: [{
					my: false,
					msg: "你好呀，想问什么就问吧"
				}],
				msgContent: "",
				msg: "",
			}
		},

		onLoad() {
			this.apiset()
		},

		methods: {
			setsklocal(apikey) {
				uni.setStorage({
					key: 'sk',
					data: apikey,
					success: function(res) {
						console.log('success', res);
					}
				});
			},
			clopop() {
				this.$refs.popup.close('center')
			},
			openpop() {
				this.$refs.popup.open('center')
			},
			apiset() {
				this.apisucc = true
				this.msgLoad = false
			},
			sendMsg() {
				// 消息为空不做任何操作
				if (this.msg == "") {
					return 0;
				}
				if (this.msg == "客服") {
					return 0;
				}
				this.msgList.push({
					"msg": this.msg,
					"my": true,
				})
				if (this.msgList[this.msgList.length - 1].msg != true) {
					this.msgList.push({
						"msg": true,
						"my": false,
					})
				}
				this.msgContent += ('YOU:' + this.msg + "\n")
				console.log(this.msgContent);
				this.msgLoad = true
				// 清除消息
				let data = JSON.stringify({
					msg: this.msgContent,
					openaikey: this.api,
					maxtoken: 2048
				})
				uni.request({
					url: this.apiurl + '/message',
					data: data,
					method: 'POST',
					success: (res) => {
						console.log(res);
						if (res.data.code == 200) {
							let text = res.data.resmsg.choices[0].text.replace("openai:", "").replace(
									"openai：", "")
								.replace(/^\n|\n$/g, "")
							this.msgList[this.msgList.length - 1].msg = text
							this.msgContent += (text + "\n")
							this.msgLoad = false
							this.msg = ""
						} else {
							this.msgList[this.msgList.length - 1].msg = '当前咨询人数过多，请重试...'
							this.apisucc = false
							this.sendMsg()
						}
					},
				})
			},
		}
	}
</script>

<style>
	page {
		background-color: #111827;
	}

	.userinfo {
		animation: oneshow 0.8s ease 1;
		display: flex;
		flex-direction: column;
		align-items: flex-end;
		padding-right: 20rpx;
	}

	.usermsg {
		margin-left: 20rpx;
		padding: 17rpx 20rpx;
		border-radius: 35rpx;
	}

	.aiinfo {
		display: flex;
		flex-direction: row;
		align-items: center;
		margin-left: 20rpx;
		margin-top: 20rpx;
		animation: oneshow 0.8s ease 1;
	}

	.chat-img {
		border-radius: 50%;
		width: 100rpx;
		height: 100rpx;
		background-color: #374151;
		display: flex;
		flex-direction: row;
		justify-content: center;
		align-items: center;
		position: relative;
	}

	.aimsg {
		display: flex;
		flex-direction: column;
		justify-content: center;
		margin-left: 20rpx;
		padding: 17rpx 20rpx;
	}

	.flex-column-center {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		position: fixed;
		bottom: 0px;
		width: 100%;
	}

	.inpubut {
		display: flex;
		flex-direction: row;
		justify-content: space-around;
		align-items: center;
		background-color: #111827;
		width: 100%;
		height: 110rpx;
		font-size: 40rpx;
	}

	.dh-input {
		width: 90%;
		height: 80rpx;
		border-radius: 100rpx;
		padding-left: 40rpx;
		margin-left: 20rpx;
		color: #fff;
		background-color: #374151;
	}

	.my-neirong-sm {
		font-size: 23rpx;
		color: #616161;
	}

	.btn {
		height: 90rpx;
		width: 35%;
		background-color: cornflowerblue;
		color: #ffffff;
		border-radius: 2500px;

	}

	.popcls {
		width: 80vw;
		height: 40vh;
		background: white;
		border-radius: 20rpx;
		display: flex;
		flex-direction: column;
		justify-items: center;
		align-items: center;
	}

	.setBtn {
		height: 120rpx;
		background-color: #111827;
		position: relative;
	}
</style>
