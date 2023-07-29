<template>

	<div class="login-container">
		<div class="login-area">
			<div class="login-header">
				<h2>CJLU Auth</h2>
				<p> 使用计量统一认证账号登录 </p>
			</div>
			<div class="login-body">
				<div class="login-form">
					<n-form ref="formRef" :model="model" :rules="rules" @submit="login">
						<n-form-item path="username" label="用户名">
							<n-input placeholder="请输入用户名" v-model:value="model.username" @keydown.enter.prevent
								@blur.native.capture="checkUser(model.username)" />
						</n-form-item>
						<n-form-item path="password" label="密码">
							<n-input placeholder="请输入密码" v-model:value="model.password" type="password" />
						</n-form-item>
						<n-form-item path="captcha" label="验证码" v-show="need_captcha">
							<n-grid x-gap="12" :cols="4">
								<n-gi span="3">
									<n-input placeholder="请输入验证码" v-model:value="model.captcha" type="text" />
								</n-gi>
								<n-gi>
									<div>
										<n-tooltip trigger="hover">
											<template #trigger>
												<img :src="captcha_src" @click="reloadCaptcha">
											</template>
											点击刷新验证码
										</n-tooltip>

									</div>
								</n-gi>
							</n-grid>
						</n-form-item>

						<n-button type="primary" attr-type="submit" block="true" :loading="login_loading"
							:disabled="login_disabled">
							登录
						</n-button>

					</n-form>
				</div>
			</div>
		</div>
	</div>
	<n-watermark content="CJLU Auth" cross fullscreen :font-size="16" :line-height="16" :width="384" :height="384"
		:x-offset="12" :y-offset="60" :rotate="-15" />
</template>

<script>
	import axios from 'axios'
	import {
		useMessage
	} from 'naive-ui'
	export default {
		name: 'Login',
		methods: {
			reloadCaptcha() {
				this.captcha_src = this.base_uri + '/auth/ajax/captcha?r=' + Math.random()
			},
			checkUser(user) {
				if (user == "") return false;
				axios({
					method: 'post',
					url: this.base_uri + '/auth/ajax/check_user',
					data: {
						username: user
					}
				}).then((res) => {
					this.need_captcha = res.data.need_captcha
					if (res.data.need_captcha) {
						this.rules.captcha = [{
							required: true
						}]
						this.captcha_src = this.base_uri + '/auth/ajax/captcha?r=' + Math.random()
					} else {
						this.rules.captcha = [{
							required: false
						}]
						this.captcha_src = ''
					}
				}).catch((err) => {
					console.log(err)
				})
			},
			login(e) {
				e.preventDefault()
				if (this.model.user == '') {
					window.$message.warning('请输入用户名')
					return
				}
				if (this.model.password == '') {
					window.$message.warning('请输入密码')
					return
				}
				if (this.need_captcha && this.model.captcha == '') {
					window.$message.warning('请输入验证码')
					return
				}
				this.login_loading = true
				axios({
					method: 'post',
					url: this.base_uri + '/auth/ajax/login',
					data: this.model
				}).then((res) => {
					this.login_loading = false
					if (res.data.error == 1) {
						window.$message.warning(res.data.msg)
						checkUser(this.model.username)
					} else {
						window.$message.success('登录成功')
						this.login_disabled = true
						setTimeout(() => {
							let redirect = location.hash.slice(1)
							if (redirect == '') redirect = '/'
							window.location.href = redirect
						}, 1500)
					}
				}).catch((err) => {
					console.log(err)
					window.$message.warning('登录出错，请重试')
					this.login_loading = false
				})
			}
		},
		data() {
			return {
				//base_uri: 'http://cjludev.top',
				base_uri: '',
				need_captcha: false,
				login_loading: false,
				login_disabled: false,
				captcha_src: '',
				model: {
					username: '',
					password: '',
					captcha: ''
				},
				rules: {
					username: [{
						required: true
					}],
					password: [{
						required: true
					}],
					captcha: [{
						required: false
					}]
				}
			}
		},
		setup() {
			window.$message = useMessage()
		}
	}
</script>

<style>
	body {
		background: #d2d2d2;
	}

	h2 {
		font-size: 1.5em;
		margin-block-start: 0em;
		margin-block-end: 0em;
		margin-inline-start: 0px;
		margin-inline-end: 0px;
		font-weight: bold;
	}

	p {
		display: block;
		margin-block-start: 0em;
		margin-block-end: 0em;
		margin-inline-start: 0px;
		margin-inline-end: 0px;
	}

	.login-container {
		display: block !important;
		position: relative;
		left: 0;
		top: 0;
		padding: 110px 0;
		min-height: 100%;
		box-sizing: border-box;
	}

	.login-area {
		position: relative;
		width: 376px;
		margin: 0 auto;
		box-sizing: border-box;
	}

	.login-area::before {
		backdrop-filter: blur(20px);
		content: '';
		position: absolute;
		width: 100%;
		height: 100%;
		left: 0;
		top: 0;
		box-shadow: 0 25px 50px #00000030;
		background: hsla(0, 0%, 100%, .3);
	}

	.login-header {
		position: relative;
		text-align: center;
		backdrop-filter: blur(0);
		box-shadow: 0 5px 10px -5px #d2d2d2;
		padding: 16px;
		background-color: #f3f3f3;
	}

	.login-header h2 {
		margin-bottom: 8px;
		font-weight: 600;
		font-size: 24px;
		color: #000;
	}

	.login-header p {
		font-weight: 400;
		font-size: 16px;
		color: #333;
	}

	.login-body {
		padding: 20px 30px;
		position: relative;
		background-color: #ffffff;
	}

	@media screen and (max-width:768px) {
		.login-container {
			padding-top: 60px
		}

		.login-area {
			width: 300px
		}

		.login-box {
			padding: 10px
		}

		body {
			margin: 0
		}
	}

	@media screen and (max-width:600px) {
		.login-container {
			padding-top: 0
		}

		body {
			background: #f3f3f3 !important;
		}

		.login-area {
			width: 100%;
		}

		.login-area::before {
			box-shadow: none !important;
		}

		.login-area::before {
			backdrop-filter: none;
		}

		.login-header {
			background-color: #fff;
		}

		.login-header {
			box-shadow: none;
		}
	}
</style>
