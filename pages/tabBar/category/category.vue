<template>
	<view>
		<page-status></page-status>
		<pageHeader/>
		<view class="category-list">
		<!-- 左侧分类导航 -->
			<scroll-view scroll-y="true" class="left">
				<view class="row" 
				v-for="(item,index) in categoryList" 
				:key="index"
				:class="{'on':showCategoryIndex==index}"
				@tap="handleCategory(index)">
					<view class="text">
						<view class="block"></view>
						{{item.title}}
					</view>
				</view>
			</scroll-view>
			<!-- 右侧 -->
			<scroll-view scroll-y="true" class="right">
				<view class="category" 
				v-for="(item,index) in categoryList"
				:key="index"
				v-if="index==showCategoryIndex">
					<view class="banner">
						<image :src="item.banner" mode=""></image>
					</view>
					<view class="list">
						<view class="box" v-for="(list,i) in item.list" :key="i" @tap="handleCategoryList(list)">
							<image :src="`/static/img/category/list/`+list.img" mode=""></image>
							<view class="text">
								{{list.name}}
							</view>
						</view>
					</view>
				</view>
			</scroll-view>
		</view>
	</view>
</template>

<script>
	import pageHeader from '../home/pageHeader'
	import interfaces from '../../../utils/interfaces.js'
	export default{
		components:{
			pageHeader
		},
		data(){
			return{
				categoryList:[],
				showCategoryIndex:0
			}
		},
		onLoad() {
			this.initData()
		},
		methods:{
			initData(){
				this.request({
					url:interfaces.getCategory,
					success:res=>{
						this.categoryList=res.data
					}
				})
			},
			handleCategory(index){
				this.showCategoryIndex=index
			},
			handleCategoryList(list){
				// console.log(list.name)
				uni.navigateTo({
					url:"../../goods/goodList?name="+list.name
				})
			}
		}
	}
</script>

<style lang="scss">
	.category-list {
		width: 100%;
		background-color: #fff;
		display: flex;

		.left {
			width: 24%;
			background-color: #f2f2f2;

			.row {
				width: 100%;
				height: 90upx;
				display: flex;
				align-items: center;

				.text {
					width: 100%;
					position: relative;
					font-size: 28upx;
					display: flex;
					justify-content: center;
					color: #3c3c3c;

					.block {
						position: absolute;
						width: 0upx;
						left: 0;
					}
				}

				&.on {
					height: 100upx;
					background-color: #fff;

					.text {
						font-size: 30upx;
						font-weight: 600;
						color: #2d2d2d;

						.block {
							width: 10upx;
							height: 80%;
							top: 10%;
							background-color: #f06c7a;
						}
					}
				}
			}
		}

		.right {
			position: absolute;
			width: 76%;
			left: 24%;

			.category {
				width: 94%;
				padding: 20upx 3%;

				.banner {
					width: 100%;
					height: 28vw;
					border-radius: 10upx;
					overflow: hidden;
					box-shadow: 0upx 5upx 20upx rgba(0, 0, 0, 0.3);

					image {
						width: 100%;
						height: 100%;
					}
				}

				.list {
					margin-top: 40upx;
					width: 100%;
					display: flex;
					flex-wrap: wrap;

					.box {
						width: calc(71.44vw / 3);
						margin-bottom: 30upx;
						display: flex;
						justify-content: center;
						align-items: center;
						flex-wrap: wrap;

						image {
							width: 60%;
							height: calc(71.44vw / 3 * 0.6);
						}

						.text {
							margin-top: 5upx;
							width: 100%;
							display: flex;
							justify-content: center;
							font-size: 26upx;
						}
					}
				}
			}
		}
	}
</style>

