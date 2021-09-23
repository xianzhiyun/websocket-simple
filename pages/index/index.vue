<template>
	<view class="content">
		<view style="margin-top: 20px;">
			<button class="mini-btn" type="primary" size="mini" @click="executeSQL">添加数据</button>
			<button class="mini-btn margin-left" type="default" size="mini" @click="selectSQL">查询数据</button>
			<button class="mini-btn margin-left" type="warn" size="mini" @click="deleteSQL">清除</button>
		</view>
		<view>
			<button class="mini-btn margin-left" type="warn" size="mini" @click="testSocket">原生方法</button>
		</view>
		<view class="text-area">
			<!-- <text class="title">{{title}}</text> -->
			<text class="title">{{textData}}</text>
		</view>
		<view class="uni-btn-v">
				<view class="websocket-msg">{{showMsg}}</view>
				<button type="primary" @click="connect">连接websocket服务</button>
				<button v-show="connected" type="primary" @click="send">发送一条消息</button>
				<button type="primary" @click="close">断开websocket服务</button>
				<view class="websocket-tips">发送消息后会收到一条服务器返回的消息（与发送的消息内容一致）</view>
		</view>
	</view>
</template>

<script>
	let platform = uni.getSystemInfoSync().platform
	export default {
		data() {
			return {
				title: 'Hello',
				textData: '',
				count: 0,
				//websocket
				connected:false,
				connecting: false,
				msg: false,
				roomId: ''
			}
		},
		onLoad() {
			this.openDB()
		},
		onUnload() {
			uni.closeSocket()
			uni.hideLoading()
		},
		computed: {
			showMsg() {
				if (this.connected) {
					if (this.msg) {
						return '收到消息：' + this.msg
					} else {
						return '等待接收消息'
					}
				} else {
					return '尚未连接'
				}
			}
		},
		methods: {
			testSocket(){
				try{
					if (plus.os.name == "Android") {
            console.log('android开始连接')
						let Socket = plus.android.importClass("java.net.Socket");
						let PrintWriter = plus.android.importClass("java.io.PrintWriter");
						let BufferedWriter = plus.android.importClass("java.io.BufferedWriter");
						let OutputStreamWriter = plus.android.importClass("java.io.OutputStreamWriter");
						let BufferedReader = plus.android.importClass("java.io.BufferedReader");
						let InputStreamReader = plus.android.importClass("java.io.InputStreamReader");

						//新建一个socket链接
						let socket = new Socket('10.100.0.102', 7777);
						let date1 = new Date(); //开始时间
						let outputStreamWriter = new OutputStreamWriter(socket.getOutputStream());
						let bufferWriter = new BufferedWriter(outputStreamWriter);
						let out = new PrintWriter(bufferWriter, true);
						console.log("请求的数据为-----"+'123');
						out.println('123'); //发送请求数据
						out.flush();
						socket.shutdownOutput();
						let inputStreamReader = new InputStreamReader(socket.getInputStream()); //接收请求数据
						let br = new BufferedReader(inputStreamReader);
						let data = br.readLine();
						let date2 = new Date(); //结束时间
						{
							if(data != null) {
								var date3 = date2.getTime() - date1.getTime() //时间差的毫秒数
								console.log("请求耗时为:(毫秒)-----" + date3);
								console.log("返回的数据为-----"+data);
								let datajson = JSON.stringify(data); //请求数据字符串转化为json对象
								return datajson; //返回json对象
							} else {
								console.log("socket链接错误");
								return false; //返回失败
							}
						}
					}
				}catch (e) {
					console.log('出现错误-------------------------')
					console.log(e)
				}
			},
			// 打开数据库
			openDB(){
				console.log('启动了')
				let option =
				plus.sqlite.openDatabase({
					name: 'first',
					path: '_doc/test.db',
					success: function(e){
						console.log('openDatabase success----------------------------!');
					},
					fail: function(e){
						console.log('openDatabase failed----------------------: '+JSON.stringify(e));
					}
				});
			},
			executeSQL(){
				this.textData = ''
				this.count+=1
				let _this = this
				plus.sqlite.executeSql({
					name: 'first',
					sql: 'create table if not exists database("where" CHAR(110),"location" CHAR(100),"age" INT(11))',
					success: function(e){
						console.log('executeSql success!');
						plus.sqlite.executeSql({
							name: 'first',
							sql: `insert into database values('北京','安乐林',${_this.count})`,
							success: function(e){
								console.log('executeSql success!');
							},
							fail: function(e){
								console.log('executeSql failed: '+JSON.stringify(e));
							}
						});
						uni.showToast({
						    title: '插入成功',
						    duration: 1000
						});
					},
					fail: function(e){
						console.log('executeSql failed: '+JSON.stringify(e));
					}
				});
			},
			selectSQL(){
				let _this = this
				plus.sqlite.selectSql({
					name: 'first',
					sql: 'select * from database',
					success: function(data){
						console.log('selectSql success: ');
						let textString = ''
						for(var i in data){
							textString+=JSON.stringify(data[i])
							console.log(data[i]);
						}
						console.log(textString)
						_this.textData = textString
						uni.showToast({
						    title: '查询成功',
						    duration: 1000
						});

					},
					fail: function(e){
						console.log('selectSql failed: '+JSON.stringify(e));
					}
				});
			},
			deleteSQL(){
				let _this = this
				plus.sqlite.selectSql({
					name: 'first',
					sql: 'delete from database',
					success: function(data){
						uni.showToast({
						    title: '删除成功',
						    duration: 1000
						});
						_this.count = 0
						_this.textData = ''
						console.log('删除成功 success: ');
					},
					fail: function(e){
						console.log('selectSql failed: '+JSON.stringify(e));
					}
				});
			},
			connect() {
				if (this.connected || this.connecting) {
					uni.showModal({
						content: '正在连接或者已经连接，请勿重复连接',
						showCancel: false
					})
					return false
				}
				this.connecting = true
				uni.showLoading({
					title: '连接中...'
				})
				uni.connectSocket({
					// url: 'ws://localhost:8888',
					url: 'ws://10.100.0.102:7777',
					// url: 'ws://127.0.0.1:7777',
					// url: 'ws:/127.0.0.1:8088',
					data() {
						return {
							msg: 'Hello'
						}
					},
					// #ifdef MP
					header: {
						'content-type': 'application/json'
					},
					// #endif
					// #ifdef MP-WEIXIN
					method: 'GET',
					// #endif
					success(res) {
						// 这里是接口调用成功的回调，不是连接成功的回调，请注意
            console.log('res')
						console.log('连接成功')
					},
					fail(err) {
						console.log('连接异常')
						// 这里是接口调用失败的回调，不是连接失败的回调，请注意
					}
				})
				uni.onSocketOpen((res) => {
					this.connecting = false
					this.connected = true
					uni.hideLoading()
					uni.showToast({
						icon: 'none',
						title: '连接成功'
					})
					console.log('onOpen', res);
				})
				uni.onSocketError((err) => {
					this.connecting = false
					this.connected = false
					uni.hideLoading()
					uni.showModal({
						content: '连接失败，可能是websocket服务不可用，请稍后再试',
						showCancel: false
					})
					console.log('onError', err);
				})
				uni.onSocketMessage((res) => {
					// this.msg = res.data
					this.msg= res.data
					console.log('onMessage', res)
				})
				uni.onSocketClose((res) => {
					this.connected = false
					this.startRecive = false
					this.msg = false
					console.log('onClose', res)
				})
			},
			send() {
				let _this =this
				uni.sendSocketMessage({
					data: JSON.stringify(`${new Date().getHours()}:${new Date().getMinutes()}:${new Date().getSeconds()}`),
					// data: _this.textData,
					success(res) {
						console.log(res);
						uni.showToast({
							icon: 'none',
							title: '发送成功一条内容'
						})
					},
					fail(err) {
						console.log(err);
					}
				})
			},
			close() {
				uni.closeSocket()
			}
		}
	}
</script>

<style>
	.margin-left{
		margin-left: 15px;
	}
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.logo {
		height: 50rpx;
		width: 50rpx;
		margin-top: 10rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 5rpx;
	}

	.text-area {
		display: flex;
		justify-content: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}
	.uni-padding-wrap {
		width: 690rpx;
		padding: 0 30rpx;
	}

	.uni-btn-v {
		padding: 10rpx 0;
	}

	.uni-btn-v button {
		margin: 20rpx 0;
	}

	.websocket-room {
		height: 40px;
		line-height: 40px;
		text-align: center;
		border-bottom: solid 1px #DDDDDD;
		margin-bottom: 20px;
	}

	.websocket-msg {
		padding: 40px 0px;
		text-align: center;
		font-size: 14px;
		line-height: 40px;
		color: #666666;
	}

	.websocket-tips{
		padding: 40px 0px;
		text-align: center;
		font-size: 14px;
		line-height: 24px;
		color: #666666;
	}
</style>
