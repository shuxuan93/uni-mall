<template>
	<view class="container">
		<view class="tops">
			<!-- <image src="../../static/images/caitiao.jpg" mode="widthFix"></image> -->
		</view>
		<view class="address flex a-center j-center" @click="addressEdit" v-if="addressList.length===0">
			<image src="../../static/images/add.png" mode="widthFix"></image>
			点击新增地址
		</view>
		<view class="addresss flex a-center j-between" v-else @click="goAddress">
			<view class="addresss-left">
				<view class="user flex a-center">
					<view class="name">
						{{address.name}}
					</view>
					<view class="phone">
						{{address.mobile}}
					</view>
				</view>
				<view class="address-details">
					{{address.address}}{{address.address_detail}}
				</view>
			</view>
			<view class="addresss-right">
				<image src="../../static/images/right.png" mode="widthFix"></image>
			</view>
		</view>
		<view class="content">
			<view class="content-item flex a-center j-between">
				<view class="item-left">
					商品合计
				</view>
				<view class="item-right">
					￥{{total}}
				</view>
			</view>
			<view class="content-item flex a-center j-between">
				<view class="item-left">
					运费
				</view>
				<view class="item-right">
					免运费
				</view>
			</view>
			<view class="content-item flex a-center j-between">
				<view class="item-left">
					优惠券
				</view>
				<view class="item-right">
					暂无
				</view>
			</view>
		</view>
		<view class="order">
			<view class="order-item content-item flex a-center j-between" v-for="(item,index) in orderList" :key="index">
				<view class="item-left flex a-center">
					<image lazy-load="true" style="height: auto;" :src="item.list_pic_url" mode="widthFix"></image>
					<view class="details">
						<view class="name">
							{{item.goods_name}}
						</view>
						<view class="price">
							￥{{item.retail_price}}
						</view>
					</view>
				</view>
				<view class="item-right">
					×{{number}}
				</view>
			</view>
		</view>
		<view class="content-bottom ">
			<view class="contents flex a-center j-between">
				<view class="bottom-left flex a-center">
					<view class="shop-number">
						实付：{{total}}
					</view>
				</view>
				<view class="bottom-right flex a-center">
					<!-- 结算 -->
					<view class="balance">
						<button type="default" @click="goOrder">支付</button>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		name: "",
		components: {

		},
		props: {},
		data() {
			return {
				address: {},
				addressList: [],
				number: null,
				total:null

			}
		},
		methods: {
			// 跳转到地址编辑页面
			addressEdit() {
				uni.navigateTo({
					url: "/pages/addressEdit/addressEdit"
				})
			},
			// 跳转到地址列表
			goAddress() {
				let id = this.address.id
				uni.navigateTo({
					url: `/pages/addressList/addressList?id=${id}`
				})
			},
			// 获取全部地址
			getAddressList(id) {
				this.$api.getAddressList(id).then(res => {
					if (res.status === 200) {
						this.$store.state.addressList = res.data.data
						this.addressList = res.data.data
						this.address = res.data.data[0]
						if (this.address) {
							this.address.address = this.address.address.replace(/\s*/g, "")
						}
					}
				}).catch(err => {
					console.log(err)
				})
			},
			goOrder() {
				let oderList = this.$store.state.orderList
				oderList.map(item => {
					let allPrise = item.number * item.retail_price
					let a = {
						allPrise,
						openId: item.user_id,
						goodsId: item.goods_id
					}
					console.log(item)
					this.$api.submitOrder(a).then(res => {
						console.log(res)
						if (res.status === 200 && res.data.data) {
							if (item.id) {
								this.$api.deleteAction(item.id).then(res => {
									if (res.status === 200 && res.data.data) {
										console.log(res)
									}
								}).catch(err => {
									console.log(err)
								})
							}

							uni.showToast({
								title: `购买成功,共计${this.total}元`,
								icon: "none"
							})
							setTimeout(() => {
								uni.navigateBack({
									delta: 1
								})
							}, 1000)
						}
					}).catch(err => {
						console.log(err)
					})
				})
			}
		},
		mounted() {

		},
		onLoad() {
			this.number = this.$store.state.orderList.reduce((pre, item) => {
				return pre + item.number
			}, 0)
			this.total = this.$store.state.orderList.reduce((pre, item) => {
				return pre + item.number * item.retail_price
			}, 0)
		},
		onShow() {
			console.log(this.$store.state.orderList)
			if (this.$store.state.address.name) {
				this.address = this.$store.state.address
				this.address.address = this.address.address.replace(/\s*/g, "")
			} else {
				this.getAddressList(this.$store.state.openId)
			}

		},
		filters: {

		},
		computed: {
			orderList() {
				return this.$store.state.orderList
			},
		},
		watch: {

		},
		directives: {

		}
	}
</script>

<style scoped lang="scss">
	.container {
		min-height: 100vh;
		background-color: #F4F4F4;
		/* #ifdef H5 */
		min-height: calc(100vh - 44px);

		/* #endif */
		.tops {
			width: 100%;
			height: 30rpx;
			background: url(../../static/images/caitiao.jpg) center no-repeat;
			background-size: 1800rpx;
		}

		.address {
			height: 200rpx;
			font-size: 30rpx;

			image {
				width: 40rpx;
				margin-right: 10rpx;
				height: 40rpx;
			}
		}

		.addresss {
			font-size: 30rpx;
			padding: 20rpx 30rpx;
			background: white;

			.addresss-left {
				flex: 1;

				.user {
					margin-bottom: 10rpx;
					font-size: 32rpx;

					.name {
						font-size: 34rpx;
						margin-right: 10rpx;
					}
				}

				.address-detail {}
			}

			.addresss-right {
				margin-left: 30rpx;

				image {
					width: 50rpx;
					height: 50rpx;
				}
			}
		}

		.content {
			margin: 20rpx auto;
			background: white;
		}

		.content-item {
			font-size: 30rpx;
			width: 92%;
			margin: 0 auto;
			padding: 30rpx 0;
			border-top: 1rpx solid #e4e4e4;
			// border: 1px solid #ccc;
			// border-right: 0 none;
			// margin-left: -2px;
			margin-top: -1rpx;
			// border-top: 0;
		}

		.order {
			background: white;

			.order-item {
				.item-left {
					image {
						width: 160rpx;
					}

					.details {
						.name {}

						.price {
							margin-top: 20rpx;
							color: #BE351C;
							font-size: 30rpx;
						}
					}
				}

			}
		}

		.content-bottom {
			position: fixed;
			bottom: 0;
			left: 0;
			background: white;
			width: 100%;
			/* #ifdef H5 */
			// bottom: 50px;

			/* #endif */
			.contents {
				width: 95%;
				margin: 20rpx auto;

				.shop-number {
					margin-left: 10rpx;
					font-size: 30rpx;
				}

				.bottom-right {
					.price {
						color: #BE351C;
						font-size: 30rpx;
					}

					.balance {
						// visibility: hidden;
						margin-left: 15rpx;

						button {
							width: 160rpx;
							height: 60rpx;
							font-weight: 500;
							font-size: 28rpx;
							line-height: 60rpx;
							color: white;
							border-radius: 0;
							background-color: #BE351C;
							border-radius: 30rpx;
						}

						button:active {
							background-color: #b02117;
						}
					}
				}
			}
		}
	}
</style>
