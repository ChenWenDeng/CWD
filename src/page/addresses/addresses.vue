<template>
	<div class="order-container">
		<headerTop />
		<div class="receiving-container">
			<header>
				收货信息
			</header>
			<div class="information-container">
				<div class="add-container" v-if="addressLists.length>0" v-for="(item ,index) in addressListFilter" :key="index"
				 :class="checkIndex?{check:checkIndex == item.isDefault}:{check:checkIndexs == index}" @click="setaddress(index,item.addressId)">
					<div v-show="!item.isDefault" :class="{modifyBoxTop: addressLists.length>=1}">
						<!-- <p :class="{defaColor:item.isDefault}" v-show="item.isDefault">默认地址</p> -->
						<p v-show="!item.isDefault" @click="setDefault(item.addressId)">设置为默认</p>
					</div>
					<p :class="{defaColor:item.isDefault}" v-show="item.isDefault">默认地址</p>
					<p>
						收货地址 : {{item.cityName}}
					</p>
					<p>
						具体地址 : {{item.streeName}}
					</p>
					<p>
						收货名称 : {{item.userName}}
					</p>
					<p>
						收货电话 : {{item.tel}}
					</p>
					<p>
						邮政编码 : {{item.postCode}}
					</p>
					<div :class="{modifyBox: addressLists.length>=1}">
						<span @click="open($event,item.addressId)">修改</span>
						<span @click="delAddress(item.addressId)">删除</span>
					</div>
				</div>
				<div class="add-box">
					<span class="iconfont icon-jia"></span>
					<p @click="open($event)">增加新地址</p>
				</div>
				<div class="look" @click="expand">
					{{this.lookTitle}}
					<i class="iconfont" :class="this.lookTitle == '查看更多' ? 'icon-zhankai' : 'icon-shouqi'"></i>
				</div>
			</div>
		</div>

		<el-dialog title="收货地址" :visible.sync="dialogFormVisible" class="modal-container">
			<div class="information-box">
				<div v-if="this.errMsg!=''" class="regular-container">
					<i class="iconfont icon-cuowu"></i>
					<p>{{this.errMsg}}</p>
				</div>
				<div class="address-container">
					收货地址：<area-cascader type="text" :level='1' v-model="selected" :data="pcaa"></area-cascader>
				</div>
				<div class="specific-containe">
					具体地址：<textarea v-model="streeName" class="specific-input" type="text" placeholder="楼层或门牌号等"></textarea>
				</div>
				<div>
					收件名字：<input v-model="userName" class="name-container" type="text" placeholder="请输入收件名字">
				</div>
				<div>
					收件电话：<input v-model="tel" class="phone-container" type="text" placeholder="请输入收件电话">
				</div>
				<div>
					邮政编码：<input v-model="postCode" class="post-code-container" type="text" placeholder="请输入邮政编码">
				</div>
			</div>
			<div slot="footer" class="dialog-footer">
				<el-button @click="close">取 消</el-button>
				<el-button type="primary" v-if="this.modalBtn=='保存地址'" @click=" address">保存地址</el-button>
				<el-button type="primary" v-if="this.modalBtn=='确定修改'" @click=" modify()">确定修改</el-button>
			</div>
		</el-dialog>


		<div class="detailed-container">
			<header>
				全部清单
			</header>
			<div class="commodity-container">
				<ul class="header-top">
					<li>
						<span>商品信息</span>
						<span>颜色</span>
						<span>尺寸</span>
						<span>单价</span>
						<span>数量</span>
						<span>小计</span>
					</li>
				</ul>
				<ul class="commodity-row">
					<li ref="commoditys" v-for="(item, index) in cartList" :key="index" v-if="modes!='purchase' && item.details[0].checked=='1'">
						<span class="left-box">
							<img ref="imgs" :src="item.details[0].smImg[0]">
							<span class="img-title">{{item.details[0].productName}}</span>
						</span>
						<span>{{item.details[0].colours}}</span>
						<span>{{item.details[0].sizes}}</span>
						<span>￥{{item.details[0].salePrice}}</span>
						<span>x{{item.details[0].num}}</span>
						<span>￥{{item.details[0].salePrice * item.details[0].num}}</span>
					</li>

					<!-- ================   立即购买列表   ============== -->

					<li v-for="(item, index) in purchaseList" :key="index" v-if="modes=='purchase'">
						<span class="left-box">
							<img ref="imgs" :src="item.details[0].smImg[0]">
							<span class="img-title">{{item.details[0].productName}}</span>
						</span>
						<span>{{item.details[0].colours}}</span>
						<span>{{item.details[0].sizes}}</span>
						<span>￥{{item.details[0].salePrice}}</span>
						<span>x{{item.details[0].num}}</span>
						<span>￥{{item.details[0].salePrice * item.details[0].num}}</span>
					</li>
				</ul>
				<div class="price-container">
					<span v-if="modes=='cart'">应付总额：￥{{totalPrice}}</span>
					<span v-if="modes=='purchase'">应付总额：￥{{purchaseTotalPrice}}</span>
					<button class="btn btn-primary" @click="subOder">
						提交订单
					</button>
				</div>
			</div>
		</div>

		<Footer />
	</div>
</template>

<script>
	import axios from 'axios';
	import {AreaCascader} from "vue-area-linkage";
	import {pca,pcaa} from 'area-data';
	import headerTop from '../../components/headerTop/headerTop';
	import Footer from '../../components/footer/footer';
	export default {
		components: {
			headerTop,
			Footer
		},
		data() {
			return {
				//selected[0]省。selected[1]市。selected[2]区。
				selected : [], //数组对应的就是选中的那个省市区。根据type来确定是省市区汉字的数组还是编码组成的数组。
				pca : pca,
				pcaa : pcaa,
				dialogFormVisible : false,
				dialogFormVisible_modify : false, //地址修改框
				formLabelWidth : '120px',
				errMsg : '', //验证输入的信息，
				streeName : '', //收货具体地址信息,
				userName : '', //收货人姓名
				tel : '', //收货人电话,
				addressLists : [], //所有的收货地址,
				cityName : '', //保存选择后的城市地址字符串
				postCode : '', //邮政编码,
				modalBtn : '', //切换是添加按钮还是修改按钮
				addressId : '', //保存要修改的地址id
				cartList : [], //购物车列表
				purchaseList : [], //立即购买列表
				modes : '', //接收是购物车方式还是立即购买方式
				limit : 3, //限制地址列表显示的数量
				lookTitle : '查看更多', //展开收缩标题
				checkIndex : true, //默认地址选中样式
				checkIndexs: 0, //地址选中样式
				subAddressId: 0,//选中的本次购物的地址id

			}
		},
		methods: {
			//选中的本次购物的地址id
			setaddress(index,addressId){
				this.checkIndex = false;
				this.checkIndexs = index;
				this.subAddressId = addressId
				console.log(this.subAddressId)
			},
			
			
			//打开模态框
			open(e, addressId) {
				this.dialogFormVisible = true;
				console.log(e.target.innerHTML);
				if (e.target.innerHTML == '增加新地址') {
					//切换为保存按钮
					this.modalBtn = '保存地址'
				} else {
					//切换为修改按钮
					this.modalBtn = '确定修改'
					//保存要修改的地址id 在 addressId 中
					this.addressId = addressId
				}
			},
			//关闭模态框
			close() {
				this.dialogFormVisible = false;
				this.errMsg = '';
			},

			//添加收货地址
			address() {
				var reg = /^1[3456789]\d{9}$/;
				var postCodeReg = /^[0-9]{6}$/;
				if (this.selected.length == 0) {
					this.errMsg = '请选择收件地址';
					this.dialogFormVisible = true;
				} else if (this.streeName == '') {
					this.errMsg = '请填写收货具体地址信息';
					this.dialogFormVisible = true;
				} else if (this.userName == '') {
					this.errMsg = '请填写收货人姓名';
					this.dialogFormVisible = true;
				} else if (this.tel == '') {
					this.errMsg = '请填写收货人电话';
					this.dialogFormVisible = true;
				} else if (!reg.test(this.tel)) {
					this.errMsg = '收货人电话不正确';
					this.dialogFormVisible = true;
				} else if (this.postCode == '') {
					this.errMsg = '请填写收件城市邮政编码';
					this.dialogFormVisible = true;
				} else if (!postCodeReg.test(this.postCode)) {
					this.errMsg = '输入的城市邮政编码格式不正确';
					this.dialogFormVisible = true;
				} else {
					this.errMsg = '';
					this.dialogFormVisible = false;

					this.selected.forEach((item) => {
						this.cityName += item
					})
					console.log(this.cityName)
					// console.log(this.selected)
					console.log(this.streeName)
					console.log(this.userName)
					console.log(this.tel)
					console.log(this.postCode)

					axios.post('/users/addAddress', {
						cityName: this.cityName,
						streeName: this.streeName,
						userName: this.userName,
						tel: this.tel,
						postCode: this.postCode
					}).then((response) => {
						let res = response.data;
						if (res.status == '0') {
							console.log('成功')
							this.addressInit()
						} else {
							console.log('失败' + res.msg)
						}
					})

				}
			},
			//修改收货地址信息
			modify() {
				this.selected.forEach((item) => {
					this.cityName += item
				})
				axios.post('/users/modifyAddress', {
					addressId: this.addressId,
					cityName: this.cityName,
					streeName: this.streeName,
					userName: this.userName,
					tel: this.tel,
					postCode: this.postCode,
				}).then((response) => {
					let res = response.data;
					if (res.status == '0') {
						console.log('成功')
						this.addressInit()
					} else {
						console.log('失败' + res.msg)
					}
				})
				this.dialogFormVisible = false;
				this.addressInit()
			},

			//删除收货地址信息
			delAddress(addressId) {
				axios.post('/users/delAddress', {
					addressId: addressId,
				}).then((response) => {
					let res = response.data;
					if (res.status == '0') {
						console.log('成功')
						this.addressInit()
					} else {
						console.log('失败' + res.msg)
					}
				})
			},

			//设置默认地址
			setDefault(addressId) {
				axios.post('/users/setDefault', {
					addressId: addressId,
				}).then((response) => {
					let res = response.data;
					if (res.status == '0') {
						console.log('成功')
						this.addressInit()
					} else {
						console.log('失败' + res.msg)
					}
				})
			},


			//张开地址列表
			expand() {
				if (this.limit == 3) {
					this.limit = this.addressLists.length;
					this.lookTitle = '收起所有'
				} else {
					this.limit = 3;
					this.lookTitle = '查看更多'
				}
			},

			//地址列表
			addressInit() {
				axios.get('/users/addressList').then((response) => {
					let res = response.data;
					this.addressLists = res.result
					console.log(this.addressLists[0])
				})
			},
			//购物车列表
			cartInit() {
				axios.get('/users/cartList').then((response) => {
					let res = response.data;
					if (res.status == '0') {
						this.cartList = res.result;
						console.log(this.cartList[0].details[0].smImg[0])
						// console.log(this.cartList[0]._id)
					} else {
						console.log('失败' + res.msg)
					}
				})
			},

			//立即购买列表
			purchaseInit() {
				axios.get('/users/purchase').then((response) => {
					let res = response.data;
					if (res.status == '0') {
						this.purchaseList = res.result;
						console.log(this.purchaseList[0].details[0].smImg[0])
					} else {
						console.log('失败' + res.msg)
					}
				})
			},
			
			//提交订单
			subOder(){
				if(this.addressLists.length != 0){
					this.$router.push({path: '/payment', query: {modes: this.modes,subAddressId:this.subAddressId}})
				}else{
					this.$message.error('请选择收货地址！');
				}
				// this.$router.push({path: '/payment', query: {modes: this.modes,subAddressId:this.subAddressId}})
			}
		},
		computed: {
			totalPrice() {
				var total = 0
				this.cartList.forEach((item, index) => {
					if (item.details[0].checked == '1') {
						total += item.details[0].salePrice * item.details[0].num
					}
				})
				return total
			},
			purchaseTotalPrice() {
				var total = 0
				this.purchaseList.forEach((item, index) => {
					total += item.details[0].salePrice * item.details[0].num
				})
				return total
			},
			//根据传入的参数计算要渲染多少条数据
			addressListFilter() {
				return this.addressLists.slice(0, this.limit)
			}
		},
		mounted() {
			//接收details页的参数判断显示购物车列表还是立即购买列表
			this.modes = this.$route.query.modes
			console.log('modes===' + this.modes)
			this.addressInit(),
			this.cartInit(),
			this.purchaseInit()
		}
	}
</script>

<style lang="scss" scoped>
	.order-container {
		width: 100%;
		background: #f3f3f3;

		.receiving-container {
			width: 80%;
			margin: 20px auto;

			header {
				width: 100%;
				padding: 20px;
				font-size: 18px;
				background: #000;
				color: #fff;
				border: 1px solid #ccc;
			}

			.information-container {
				width: 100%;
				padding: 10px 0;
				border: 1px solid #ccc;
				display: flex;
				flex-wrap: wrap;
				justify-content: flex-start;
				background: #fff;

				.add-container {
					width: 23%;
					min-height: 200px;
					padding: 30px 20px;
					margin: 20px 10px 0 10px;
					border: 2px solid #ccc;
					position: relative;
					cursor: pointer;

					.defaColor {
						position: absolute;
						top: 0;
						left: 0;
						right: 0;
						text-align: center;
						margin-top: 3px;
						color: coral;
					}

					&.check {
						border: 2px solid darkorange;
					}

					p {
						margin-bottom: 5px;
					}

					.modifyBoxTop {
						top: 2px;

						p {
							width: 100%;
							display: flex;
							justify-content: center;
							align-items: center;
							font-size: 13px;

							&.defaColor {
								color: coral;
							}
						}
					}

					.modifyBox,
					.modifyBoxTop {
						height: 30px;
						position: absolute;
						left: 0;
						right: 0;
						bottom: 0;
						background: #f3f3f3;
						display: flex;
						opacity: 0;
						visibility: hidden;
						-webkit-transition: all 0.3s;
						-moz-transition: all 0.3s;
						-o-transition: all 0.3s;
						transition: all 0.6s;

						span {
							display: flex;
							justify-content: center;
							flex: 1;
							padding: 5px 0;
							background: #fff;
							border-top: 1px solid #ccc;
							font-size: 12px;
						}

						span:hover {
							background: #f3f3f3;
						}

						span:nth-child(1) {
							border-right: 1px solid #ccc;
						}
					}

					&.add-container:hover .modifyBox,
					&.add-container:hover .modifyBoxTop {
						visibility: visible;
						opacity: 1;
						transform: translateY(-5%);
					}
				}

				.add-box {
					width: 23%;
					min-height: 200px;
					display: flex;
					flex-direction: column;
					justify-content: center;
					align-items: center;
					margin: 20px 10px;
					margin-bottom: 0;
					border: 2px solid #ccc;

					.icon-jia {
						font-size: 25px;
					}

					p {
						cursor: pointer;
					}
				}

				.add-box:hover {
					background: #f3f3f3;
				}

				.look {
					cursor: pointer;
					width: 100%;
					height: 40px;
					margin: 0 auto;
					display: flex;
					align-items: center;
					justify-content: center;

					.iconfont {
						margin: 3px 0 0 3px;
						font-size: 12px;
					}
				}
			}
		}

		.modal-container {
			.information-box {
				display: flex;
				flex-direction: column;
				padding-left: 100px;

				.regular-container {
					width: 400px;
					height: 40px;
					border: 1px solid red;
					margin: 20px 0;
					background: rgba(200, 54, 54, 0.2);
					display: flex;
					justify-content: center;
					align-items: center;
					i{
						font-size: 25px;
						color: red;
					}
					p {
						margin-bottom: 0;
						color: red;
					}
				}

				.address-container {
					display: flex;
					align-items: center;
				}

				.specific-containe {
					display: flex;
					align-items: center;

					.specific-input {
						min-width: 300px;
						min-height: 80px;
						margin-top: 20px;
						padding: 0 20px;
						outline: none
					}
				}

				.name-container {
					width: 300px;
					height: 40px;
					margin: 20px 0;
					padding: 0 20px;
					outline: none
				}

				.phone-container {
					width: 300px;
					height: 40px;
					padding: 0 20px;
					margin-bottom: 20px;
					outline: none
				}

				.post-code-container {
					width: 300px;
					height: 40px;
					padding: 0 20px;
					outline: none
				}
			}
		}

		.detailed-container {
			width: 80%;
			margin: 20px auto;
			background: #fff;

			// color: #fff;
			header {
				width: 100%;
				padding: 20px;
				font-size: 18px;
				background: #000;
				color: #fff;
				border: 1px solid #fff;
			}

			.commodity-container {
				width: 100%;
				padding: 30px;

				.header-top,
				.commodity-row {
					width: 100%;
					padding-left: 0;
					margin: 0;

					li {
						width: 100%;
						list-style: none;
						padding: 20px;
						display: flex;
						align-items: center;
						border-bottom: 1px solid #000;

						span {
							text-align: center;
							display: inline-block;
							width: 15%;
						}

						span:nth-child(1) {
							flex: 1;
							text-align: left;
						}
					}
				}

				.commodity-row {
					.left-box {
						display: flex;
						align-items: center;

						img {
							width: 80px;
							height: 80px;
						}

						.img-title {
							width: 300px;
							margin-left: 10px;
							text-align: left;
							overflow: hidden;
							text-overflow: ellipsis;
							white-space: nowrap;
						}
					}

					li:nth-child(1) {
						padding: 30px 20px;
					}
				}

				.price-container {
					width: 100%;
					text-align: right;
					padding: 20px 50px;

					.btn {
						margin-left: 40px
					}
				}
			}
		}
	}
</style>
