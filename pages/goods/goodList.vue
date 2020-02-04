 <template>
	<view>
		<view class="header">
			<view class="target" 
				v-for="(item,index) in filterByList" 
				:key="index" 
				:class="{'on':item.selected}"
				@tap="handleFilter(item)">
				{{item.text}}
			</view>
		</view>
		<!-- 占位tabbar 有定位 -->
		<view class="place"></view>
		<!-- 商品列表 -->
		<view class="goods-list">
			<view class="product-list">
				<view class="product" @tap="handleGoods(good)" v-for="(good,index) in goodList" :key="index">
					<image :src="good.img" mode="widthFix"></image>
					<view class="name">
						{{good.name}}
					</view>
					<view class="info">
						<view class="price">
							<text>¥</text>
							{{good.price}}
						</view>
						<view class="slogan">
							{{good.slogan}}
							<text>人付款</text>
						</view>
					</view>
				</view>
			</view>
			<view class="loading-text">
				{{loadingText}}
			</view>
		</view>
	</view>
		
</template>

<script>
	import interfaces from '../../utils/interfaces.js'
	export default{
		data(){
			return{
				 filterByList:[{
					 text:"全部",
					 selected:true,
					 filterby:"all"
				 },{
					 text:"口碑",
					 selected:false,
					 filterby:"public"
				 },{
					 text:"热门",
					 selected:false,
					 filterby:"hot"
				 }],
				 goodList:[],
				 filterby:'all',
				 page:1,
				 size:6,
				 loadingText:'正在加载中...'
			}
		},
		onLoad(option){
			// console.log(option)
			// 动态修改nav title
			uni.setNavigationBarTitle({
				title:option.name
			})
			//加载数据
			this.loadData();
		},
		methods:{
			handleFilter(item){
				this.filterByList.forEach((v)=>{
					v.selected=false
				})
				item.selected=true;
				this.filterby=item.filterby;
				this.page=1;
				this.loadingText="加载中";
				this.goodList=[];
				this.loadData();
			},
			loadData(){
				this.request({
					url:`${interfaces.getGoodsList}/${this.filterby}/${this.page}/${this.size}`,
					success:res=>{
						if(res.data.length>0){
							res.data.forEach((item)=>{
								this.goodList.push(item)
							})
						}else{
							this.loadingText="数据加载完毕"
						}
						// this.goodList=res.data
						// console.log(this.goodList)
					}
				})
			},
			handleGoods(good){
				//页面跳转 商品详情
				uni.navigateTo({
					url:"./goods?goodsInfo="+JSON.stringify(good)
				})
			}
		},
		onPullDownRefresh() {//下拉刷新
			setTimeout(()=>{
				this.page=1;
				this.loadingText='加载中...';
				this.goodList=[];
				this.loadData();
				uni.stopPullDownRefresh();
			},1000)
		},
		onReachBottom() {//上拉加载
			this.page++;
			this.loadData();
		}
	}
</script>

<style lang="scss">
	.header {
		width: 92%;
		padding: 0 4%;
		height: 79upx;
		display: flex;
		justify-content: space-around;
		align-items: flex-end;
		position: fixed;
		z-index: 10;
		background-color: #fff;
		border-bottom: solid 1upx #eee;

		.target {
			width: 20%;
			height: 60upx;
			display: flex;
			justify-content: center;
			align-items: center;
			font-size: 28upx;
			margin-bottom: -2upx;
			color: #aaa;

			&.on {
				color: #555;
				border-bottom: 4upx solid #f06c7a;
				font-weight: 600;
				font-size: 30upx;
			}

			.icon {
				font-size: 26upx;
			}
		}
	}

	.place {
		background-color: #ffffff;
		height: 100upx;
	}

	.goods-list {
		.loading-text {
			width: 100%;
			display: flex;
			justify-content: center;
			align-items: center;
			height: 60upx;
			color: #979797;
			font-size: 24upx;
		}

		.product-list {
			width: 92%;
			padding: 0 4% 3vw 4%;
			column-count: 2;
			column-gap: 1em;

			.product {
				break-inside: avoid;
				border-radius: 20upx;
				background-color: #fff;
				margin: 0 0 15upx 0;
				box-shadow: 0upx 5upx 25upx rgba(0, 0, 0, 0.1);

				image {
					width: 100%;
					border-radius: 20upx 20upx 0 0;
				}

				.name {
					width: 92%;
					padding: 6upx 4%;
					display: -webkit-box;
					-webkit-box-orient: vertical;
					-webkit-line-clamp: 2;
					text-align: justify;
					overflow: hidden;
					font-size: 30upx;
				}

				.info {
					display: flex;
					justify-content: space-between;
					align-items: flex-end;
					width: 92%;
					padding: 10upx 4% 10upx 4%;

					.price {
						color: #e65339;
						font-size: 30upx;
						font-weight: 600;
					}

					.slogan {
						color: #807c87;
						font-size: 24upx;
					}
				}
			}

		}
	}
</style>
