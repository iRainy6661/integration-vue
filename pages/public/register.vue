<template>
	<view class="container">
		<!-- <view class="left-bottom-sign"></view>
		<view class="right-top-sign"></view> -->
		<!-- 设置白色背景防止软键盘把下部绝对定位元素顶上来盖住输入框等 -->
		<view class="wrapper">
			<view class="left-top-sign"></view>
			<view class="welcome">
				欢迎加入！
			</view>
			<view class="input-content">
				<view class="input-item">
					<input type="number" v-model="phone" placeholder-class="placeholder" placeholder="请输入手机号码" maxlength="11" />
				</view>
				<view class="input-item">
					<view class="verify-code u-f u-f-jsb">
						<input type="text" v-model="captcha" placeholder-class="placeholder" placeholder="图形验证码" />
						<image :src="`${baseUrl}/app/captcha.jpg?uuid=${uuid}`" @click="getCaptcha" mode="scaleToFill"></image>
					</view>
				</view>
				<view class="input-item">
					<view class="verify-code u-f u-f-jsb">
						<input type="text" v-model="phoneCaptcha" placeholder-class="placeholder" placeholder="短信验证码" maxlength="6" @confirm="doRegister" />
						<button :disabled="sendDisabled" @click="doGetVerify">{{sendText}}</button>
					</view>
				</view>
				<view class="input-item">
					<input type="text" v-model="password" placeholder-class="placeholder" placeholder="6-18位不含特殊字符的数字、字母组合" maxlength="18" password />
				</view>
			</view>
			<button class="confirm-btn" @click="doRegister" :disabled="registering">注册</button>
		</view>
		<view class="register-section">
			已经有账号？
			<text @click="toLogin">马上登录</text>
		</view>
		<view class="user-protocol">
			登录注册代表同意
			<text @click="navTo('/pages/user/agreement')">《用户协议》</text>
			和
			<text @click="navTo('/pages/user/policy')">《隐私政策》</text>
		</view>
	</view>
</template>

<script>
	import { mapMutations } from 'vuex'
	import { getUUID } from '@/common/public.js'
	import * as config from '@/config'
	export default {
		data() {
			return {
				baseUrl: '',
				uuid: '',
				phone: '',
				captcha: '',
				password: '',
				phoneCaptcha: '',
				registering: false,
				sendText: '获取短信验证码',
				sendDisabled: false
			}
		},
		onShow() {
			this.getCaptcha()
		},
		onLoad() {
			this.baseUrl = config.def().baseUrl
		},
		methods: {
			...mapMutations(['login']),
			// 获取验证码
			getCaptcha () {
				const that = this
				const uuid = getUUID().substr(0, 8)
				that.uuid = uuid
				// that.$api.request(`/app/captcha.jpg?uuid=${uuid}`, 'GET')
			},
			toLogin() {
				uni.redirectTo({
					url: "/pages/public/login"
				})
			},
			doGetVerify() {
				const that = this
				if (!that.phone || that.phone.length != 11) {
					uni.showToast({
						title:'请输入正确手机号！',
						icon:'none'
					})
					return
				}
				that.$api.request('/app/send/register', 'POST', {
					phone: that.phone,
					captcha: that.captcha,
					uuid: that.uuid
				}, failres => {
					that.$api.msg(failres.msg)
				}).then(res => {
					that.sendDisabled = true
					let sec = 120
					let interval = setInterval(() => {
						sec--;
						that.sendText = sec + 's后重发'
						if (sec <= 0) {
							that.sendDisabled = false
							that.sendText = "获取验证码"
							clearInterval(interval)
						}
					}, 1000)
				})
			},
			async doRegister() {
				const that = this
				that.registering = true;
				that.$api.request('/app/register', 'POST', {
					mobile: that.phone,
					phoneCaptcha: that.phoneCaptcha,
					password: that.password
				}, failres => {
					uni.showToast({
						title:failres.msg
					})
					that.registering = false
				}).then(res => {
					that.registering = false
					//注册成功
					uni.showToast({
						title: '注册成功!',
						icon: 'success',
						duration: 2000,
						success: () => {
							that.$store.commit('login', res)
							uni.setStorageSync('userInfo', res)
							uni.reLaunch({
								url: '/pages/public/perfect'
							})
						}
					})
				})
			},
			navTo(url) {
				uni.navigateTo({
					url: url
				})
			}
		}
	}
</script>

<style lang='scss'>
	page {
		background: #fff;
	}

	.container {
		padding-top: 120rpx;
		position: relative;
		width: 100vw;
		height: 100vh;
		overflow: hidden;
		background: #fff;
	}

	.wrapper {
		position: relative;
		z-index: 90;
		background: #fff;
		padding-bottom: 40upx;
	}

	.left-top-sign {
		height: 140rpx;
		font-size: 120upx;
		color: $page-color-base;
		position: relative;
		left: -16upx;
	}

	.right-top-sign {
		position: absolute;
		top: 80upx;
		right: -30upx;
		z-index: 95;

		&:before,
		&:after {
			display: block;
			content: "";
			width: 400upx;
			height: 80upx;
			background: #b4f3e2;
		}

		&:before {
			transform: rotate(50deg);
			border-radius: 0 50px 0 0;
		}

		&:after {
			position: absolute;
			right: -198upx;
			top: 0;
			transform: rotate(-50deg);
			border-radius: 50px 0 0 0;
			/* background: pink; */
		}
	}

	.left-bottom-sign {
		position: absolute;
		left: -270upx;
		bottom: -320upx;
		border: 100upx solid #d0d1fd;
		border-radius: 50%;
		padding: 180upx;
	}

	.welcome {
		position: relative;
		left: 50upx;
		top: -90upx;
		font-size: 48upx;
		color: #333333;
		font-weight:bold;
	}

	.input-content {
		padding: 0 40upx;
	}

	.input-item {
		display: flex;
		justify-content: center;
		height: 80upx;
		margin-bottom: 96upx;
		&:last-child {
			margin-bottom: 0;
		}

		input {
			width: 100%;
			height: 100%;
			padding:  0 10rpx;
			font-size: 30upx;
			color: #333333;
			background:rgba(237,243,247,1);
			border-radius:20rpx;
			border:3rpx solid rgba(51,51,51,1);
		}
		
		.verify-code {
			width: 100%;
			height: 100%;
			input{
				width: 50%;
			}
			image{
				width: 40%;
				height: 100%;
				border-radius:20rpx;
				border:3rpx solid rgba(51,51,51,1);
			}
			button{
				width: 40%;
				height: 100%;
				margin: 0;
				border-radius:20rpx;
				color: #80C9DA;
				font-size:22rpx;
				font-weight:400;
				border:3rpx solid rgba(51,51,51,1);
				background:rgba(237,243,247,1);
			}
		}
	}
	.placeholder{
		font-size: 30rpx;
		color: #333333;
		font-weight:400;
	}
	.confirm-btn {
		width: 670upx;
		height: 72upx;
		line-height: 72upx;
		border-radius: 20rpx;
		margin-top: 110upx;
		background: #81C9DB;
		color: #fff;
		font-size: 34rpx;
		font-weight:400;
		border:3rpx solid rgba(51,51,51,1);
	}

	.forget-section {
		font-size: 22upx;
		color: #81C9DB;
		text-align: center;
		margin-top: 40upx;
		font-weight:400;
	}

	.register-section {
		position: absolute;
		left: 0;
		bottom: 50upx;
		width: 100%;
		font-size: 22upx;
		font-weight:400;
		color: #666666;
		text-align: center;
		text {
			color: #81C9DB;
			margin-left: 10upx;
		}
	}
	.user-protocol {
		position: absolute;
		left: 0;
		bottom: 6upx;
		width: 100%;
		font-size: 22upx;
		color: #666666;
		text-align: center;
		text {
			color: #81C9DB;
		}
	}
</style>
